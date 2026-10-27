---
layout: post
title: "Notes on LLM MOE"
date: 2026-08-05
categories: [Notes]
tags: [llm]
math: true
---

## 术语

读 MoE 论文绕不开下面四个词。并行体系的完整版（TP / CP / PP / SP / DP、通信原语、Parallel Folding、框架对比）见 [Notes on Training Parallelism](https://lxu-nlp.github.io/posts/note-on-parallelism)。

- **EP（Expert Parallelism）**：把专家按**整数个**切到不同 GPU，token 通过 **all-to-all** 发给持有目标专家的卡（dispatch）、算完再发回（combine）。与 TP 的区别：TP 切的是**矩阵维度**、靠 all-reduce 把结果补回完整张量；EP 不动矩阵，但 token 要搬来搬去。
- **ETP（Expert Tensor Parallelism）**：在 EP 之上再对专家内部做张量并行。NVIDIA 调优指南建议**默认设 1** —— 专家本来就窄（Qwen3.5 才 512 维），再切一次张量并行的通信开销不划算。
- **Grouped GEMM（分组矩阵乘）**：MoE 层里每个专家只处理分给自己的那批 token，于是本来"一个大的矩阵乘"变成"每个专家一个小矩阵乘"，而且各专家的 token 数不一样多。Grouped GEMM 把这些小矩阵乘**塞进一次 kernel launch 里一起算完**，省掉逐个启动 kernel 的开销与 GPU 空转。HF Transformers 的参考实现没用它，是 `for expert_idx in expert_hit` 的 Python 循环。（矩阵乘的维度记法与完整算子清单见第八节。）
- **变长 all-to-all**：MoE 的 dispatch / combine 是**变长**的（每专家 token 数由路由决定、运行时才知道，各卡还不均），比定长 all-to-all 难优化得多，这是 MoE 通信瓶颈的根因。配套手段：permute / un-permute（让同专家的 token 内存连续）、node-limited routing（V3 每 token 最多发 4 个节点）、DualPipe（把 all-to-all 藏进计算里）。

---

## 前置知识：从 dense Transformer 到 MoE

### 1. 为什么在 Transformer 里塞 MoE
标准 Transformer 中，self-attention 之后的 **FFN（Feed-Forward Network）** 通常是两层 MLP，**参数量约占整个模型的 2/3**。MoE 的核心想法：把"一份共用的大 FFN"换成"**一组并行 Expert + 一个 Router（门控）**"，每个 token 只由 router 挑出的少数专家处理。

> 直觉：总参数可以很大（很多专家），但单 token 实际计算量很小——这就是 **sparse activation（稀疏激活）**。它在"总参数（容量）"与"单 token 算力（成本）"之间解耦。

### 2. 核心术语
- **Expert（专家）**：一份独立 FFN 权重，MoE 层并行存在 N 个。
- **Router / Gating（门控）**：小**线性层**，输入 token 隐藏向量，输出对各专家的打分并据此选 top-k。
- **Top-k routing**：每个 token 只激活 k 个专家（k ≪ N，如 8/256）。在 decoder-only LLM 中路由是 **per-token** 的（global-batch / sequence-level 只是均衡校正的统计粒度）。唯一反例是 Expert Choice（专家选 token）。
- **Total params vs Active params**：总参数 = 所有专家之和（巨大）；激活参数 = 单 token 实际用到的 k 个专家（很小），模型卡常写 $671B / 37B$。
- **Capacity factor / dropping**：每专家单步最多接收 token 数，超出则被丢弃或走残差。

### 3. 三大核心难题（贯穿全文）
1. **Routing collapse（路由坍缩）**：router 偷懒把 token 全送进同一两个专家 → 其余饿死。头号病，解法叫 **load balancing**。
2. **均衡的代价**：早期用 auxiliary loss（往梯度加一项鼓励均匀），但**污染主任务梯度、拖累质量**；现代主流改 **aux-loss-free**（见下）。
3. **通信瓶颈**：专家分布在不同 GPU，token 需 all-to-all 收发 → 衍生 expert parallelism、node-limited routing、DualPipe。
- （附）**训练稳定性**：router logit 可能爆炸 → **router z-loss**（见 ST-MoE）。

---

## 一、基础工作：DeepSeekMoE 与 DeepSeek-V3（合并讲解）

这一节是今天所有开源 MoE 的源头：DeepSeekMoE 给出**现代 MoE 层的结构模板**（专家怎么切、怎么分），DeepSeek-V3 把它落成**可复现的完整训练配方**（怎么训出不崩、还快、还均衡）。

### DeepSeekMoE — arXiv:2401.06066（结构模板）
**Motivation**：此前专家太"粗"（整块大 FFN），top-k 组合有限、知识难解耦；且所有专家都参与路由，通用知识被反复算。两招解决：

- **Fine-grained expert segmentation**：把原 FFN 切成 *m* 个小专家（总参数不变，更多更窄）。top-k 组合空间**指数增长** → token 可表达为更丰富的 mixture，知识分配更灵活、解耦。
- **Shared expert isolation**：留一组 *shared experts* 对每个 token 常开（承载语法/格式等通用知识），其余 *routed experts* 才按 token 选（承载专业知识）。把"人人都需要"与"这个 token 才需要"分开。
- **Gating**：routed 专家用 sigmoid 亲和度 $s_{i,t}$ 打分，选 top-$(mK-K_s)$ 个，全部 $K_s$ 个 shared 常开；层输出 $h_t = \sum_{\text{shared}} \text{FFN}_i(u_t) + \sum_{\text{routed-topk}} s_{i,t}\cdot \text{FFN}_i(u_t) + u_t$。
- **Load balancing（原始 aux-loss 方案）**：加 Expert-Level + Device-Level Balance Loss 阻止坍缩（往梯度注入鼓励均匀使用的项）。V3 后改为 aux-loss-free。
- **Canonical config**：16.4B 变体 = 64 routed + 2 shared、top-6；236B = 160 routed + 2 shared。"细粒度 + 共享"是 V3 / Kimi K2 / GLM-4.5 / V4 的直系祖先。

### DeepSeek-V3 — arXiv:2412.19437（完整工程配方）
**Motivation**：好结构还要解决三件实事——(a) 均衡不伤质量；(b) 万卡上藏住 all-to-all 通信；(c) 低精度压成本。后续开源模型基本照抄其骨架。

- **Scale**：671B total / 37B active；每层 256 routed + 1 shared，每 token top-8。
- **Gating**：**sigmoid + selected-expert normalization**——算 sigmoid 后取 top-k，再**只在选中的 k 个内 renormalize**（替代 softmax-over-all），分数更稳定、跨 token 可比。
- **Load balancing（aux-loss-free, 2408.15664）**：维护梯度外的 per-expert **bias 项 $b_i$**，每隔几步按负载 $\pm\gamma$（低于 batch 均值就加、否则减），加到 router logit 上引导 token 去欠用专家，**不污染主任务梯度**；保留微小 sequence-level aux loss 兜底。
- **Infra**：1 个 **MTP** 层（speculative decoding）+ **FP8** 训练 + **DualPipe**（重叠 forward/backward 块以隐藏 all-to-all）+ **node-limited routing**（每 token 最多发 M=4 节点）+ **无 token dropping**。
- **长上下文（YaRN）**：预训练后两段扩展 4K → 32K → 128K。
- **Value**：最完整的"怎么把万亿 MoE 训出来"手册。

---

## 二、新主流 setup（近两年真正落地的开源 MoE）

**总览**：到 2024 年底，MoE **层结构已收敛**成 DeepSeekMoE 样子（细粒度 + 共享 + sigmoid + 无损失均衡）。近两年差异集中在三条线：① 均衡策略变体；② 稀疏 scaling（专家越堆越多、激活率越压越低）；③ 注意力侧混合改造（MoE 同质化后，军备竞赛转移到 attention）。

### Qwen3（技术报告，235B-A22B / 30B-A3B）
**Motivation**：反例路线——**不加共享专家**，塞入 global-batch 均衡 + 线性注意力混合，证明 DeepSeekMoE 模板非唯一解。

- 每层 128 routed、top-8、**无 shared**（注：Qwen3-Next 又加回共享专家 + Gated DeltaNet，家族不统一）。config 实测（30B-A3B）：`num_experts=128`、`moe_intermediate_size=768`、`shared_expert_intermediate_size` 字段**不存在**、`mlp_only_layers=[]`（即**没有 dense 首层**，48 层全是 MoE）；235B-A22B 同构，94 层、`moe_intermediate_size=1536`。
- **Softmax + global-batch loss-free balancing**：注意路由不是 DeepSeek 的 sigmoid——HF 实现里是 `softmax(router_logits.float(), -1)` 取 top-k 后**再在 k 个内 renormalize**（`norm_topk_prob=True`）。均衡跨**整个 global batch** 聚合统计（非 per-microbatch），大尺度下信号更稳。
- **完全无 aux loss**，均衡纯靠 global-batch 统计；差异化放在 attention：混 gated delta rule（线性注意力）分支。

### Kimi K2 / K3（Moonshot，2025 / 2026）
**Motivation**：把"更多专家 + 更高激活"的稀疏 scaling 推到极致（K2 384 → K3 896 专家），配套专用优化器（MuonClip）与均衡算法（Quantile Balancing）——专家越多，固定步长 bias 更新越易震荡。

- **K2**：1T / 32B，384 routed + 1 shared、top-8，MuonClip 优化器（防 attention logit 爆炸）。
- **K3 (2026)**：≈2.8T，**Stable LatentMoE**，896 routed + 2 shared、top-16（~56× 路由稀疏）。关键：隐藏态先**压缩 7168→3584** 再算路由专家、再投影回，让专家数/激活数增长而不让通信随全隐藏维膨胀。
  - **SiTU-GLU**：有界软截断激活，主要为 **MXFP4 低精度训练** 下保稳定（配 post-expert RMSNorm）。
  - **Quantile Balancing**：大专家数下固定步长 bias 会震荡，故把均衡建模为**对偶线性规划并求解析解**，把每专家负载拉向目标分位数。
  - 注意力侧 KDA 3:1 混合线性注意力 + AttnRes 跨层残差；权重存/训于 MXFP4。

### LongCat-Flash — arXiv:2509.01322（美团，零计算专家）
**Motivation**：让激活量**随 token 难度自适应**——简单 token 直接短路跳过专家，难 token 才用满。往专家池塞"什么都不算"的恒等专家实现。

- 560B，动态激活 **18.6B–31.3B**（均值 ~27B），非固定值而是**按 token 难度的范围**。机制：路由池加入 **shortcut / identity 专家**（不计算、原样返回），top-k 从"真专家 + shortcut"中抽；简单 token 多走 shortcut → 近零 FLOPs，难 token 走满真专家。
- 同一 shortcut 路径兼作**残差**，重叠 all-to-all 通信与专家计算，掩盖 MoE 层通信停顿。

### DeepSeek-V4 (Pro)（技术报告）
**Motivation**：MoE 层已完全收敛的最强信号——主体仍是 DeepSeekMoE（384 routed + 1 shared、top-6），仅最浅几层试确定性 hash 路由；创新全在 **attention（CSA/HCA）**、**超连接稳定性（mHC）** 与 **FP4**。

- 修正谣言：MoE 主体**未**整体换成 hash 路由，只最浅几层 dense FFN 换为确定性 content-hash 路由（零路由通信）。
- **mHC**：把残差 B 矩阵约束在 Birkhoff 双随机流形上，防超深/宽模型的谱范数爆炸。
- 专家权重 FP4、训练用 Muon；两个尺寸：V4-Pro (1.6T / 49B)、V4-Flash (284B / 13B)，32T tokens、1M 上下文。

### Qwen3.5（35B-A3B，2026）
**Motivation**：把"细粒度 + 共享"推到更极端——MoE 侧的关注点从"结构创新"转成"**在极窄 hidden 上堆极多小专家**"。

- 35B total / 3B active，40 层、`hidden_size=2048`（很窄），**256 routed + 1 shared**、top-8，专家 `moe_intermediate_size=512`（比 Qwen3 的 768 更细）。
- **共享专家回来了，而且带 gate**：`shared_expert_gate = Linear(h, 1)`，输出 `sigmoid(·)` 逐 token 标量乘在共享专家输出上。这条线是 **Qwen2-MoE → Qwen3-Next → Qwen3.5**，DeepSeek 系没有（V3 是 `hidden_states + shared_experts(residuals)`，恒权 1.0）。
- **每层都是 MoE**（`mlp_only_layers=[]`），无 capacity、无 token dropping。
- 参数分布极端：**专家权重 32.2B ≈ 总参 92%**；其余全部（注意力 + 嵌入 + 共享专家 + ViT）加起来才 ~3B。

### Gating 实现对照（读代码时最直接的分野）

| | DeepSeek-V3 | Qwen3 / Qwen3.5 |
|---|---|---|
| 打分 | **sigmoid**，跨专家**独立** | **softmax over 全部 E 个**，归一化后竞争 |
| 选人 | group-limited：分 `n_group=8` 组、先选 `topk_group=4` 组再组内 top-k | 直接全局 top-k |
| 权重 | 选中后 renormalize，再乘 `routed_scaling_factor` | 选中后 renormalize（`norm_topk_prob=True`） |
| 均衡载体 | `e_score_correction_bias`（**Buffer**）加到 logit 上 | global-batch 统计，无可见 bias 参数 |
| shared expert | **无 gate**，恒权相加 | **有 sigmoid gate**，逐 token 可调 |

一句话：DeepSeek 系"**独立打分 + 结构限选**"，Qwen 系"**全局归一化 + 直接 top-k**"。softmax 让专家分数互相竞争、天然更均匀一些（代价是分数不可跨层/跨 token 直接比较），sigmoid + scaling factor 则更稳、更好配 loss-free bias。

---

## 三、专题：Routing Collapse 解法演进

**导语**：现代路由仍是 per-token，下面方法多在 per-token 路由外加一层均衡校正。

### 1. 早期：梯度内辅助损失 + 容量截断
- **Auxiliary load-balancing loss**（GShard / Switch）：加鼓励均匀分配的损失项（专家 importance × 均衡项），强迫 router 摊开 token。
- **Expert capacity + dropping**：每专家上限 $capacity = (tokens/batch / N)\times\alpha$，超额丢弃走残差。限损不防坍缩、还丢信息。
- **为何被弃**：aux loss 进**主任务梯度**、伤害最终质量（正是 V3 转向 loss-free 的原因）。

### 2. 现代主流：无梯度均衡（Loss-free）
核心 insight（DeepSeek, 2408.15664）：把均衡信号**移出梯度**——用梯度外的控制器调 router logit，分布仍均衡却不污染表征。
- **Bias-term（V3，事实标准）**：梯度外 per-expert bias $b_i$，按负载 $\pm\gamma$，**永不回传到模型权重**。代码层面的直接证据：HF `DeepseekV3TopkRouter` 里它是 `self.e_score_correction_bias = nn.Buffer(...)`（**Buffer 不是 Parameter**，`_init_weights` 里 `init.zeros_` 初始化），加到 sigmoid 后的分数上只用于选人、不参与反传；同类还有 `routed_scaling_factor`、`n_group` / `topk_group`。
- **Global-batch（Qwen3）**：跨整个 global batch 聚合统计，大尺度更稳。
- **EP-group（Step 3.5 Flash）**：在 expert-parallel 组内均衡，换一点全局均衡降通信。
- **Quantile Balancing（K3, 896 专家）**：大专家数下固定步长 bias 震荡，改用对偶线性规划解析解拉向目标分位数。

### 3. 结构性去坍缩（免训练 / 确定性）
- **Expert Choice**（2202.09368）：专家选 token，每专家恰得 k 个 → **天然均衡、无需 aux loss**。败因：decoder-only 推理中专家需看到未来 token 才能选 → **causal leakage**，无法自回归。
- **Hash routing**（确定性，V4 浅层）：content-hash 固定映射 → 无 router、故**不可能坍缩**、零通信。局限：不灵活，V4 仅敢用在最浅层。

### 4. 稳定性补充：Router z-loss（ST-MoE）
router logit 无界增长会饱和门控、间接加重坍缩。$z_{\text{loss}} = c\cdot\log^2(\text{mean}(\exp(\text{logits})))$ 惩罚 logit 幅度，保数值稳定，现代配方几乎都带。

**小结**：aux loss（伤梯度）→ capacity/dropping（限损）→ **loss-free bias（主流）** → global-batch / EP-group / quantile（scale-aware）；外加结构性 **Expert Choice / Hash** 从设计上避免坍缩；**z-loss** 保稳定。

---

## 四、专题：Top-k Selection 不可导时，梯度怎么回传

**问题**：`top-k`（在 N 个 logit 中取最大的 k 个）本质是**离散选择**，选出的 index 本身不可微——你无法对"选了哪个专家"这个整数下标求梯度。那 router 怎么训？

**核心思路（Shazeer 2017 的标准做法）**：我们其实**不需要对 index 求梯度**。门控权重 $g_i$（专家的组合系数）是 router 输出的**可微函数**（softmax / sigmoid），梯度正是通过 $g_i$ 流回 router 的。

- 具体：层输出 $h = \sum_{i\in\text{top-k}} g_i \cdot \text{FFN}_i(x)$。其中 $\text{FFN}_i(x)$ 不依赖 router 参数，而 $g_i$ 是 router logit 的连续函数 → $\partial L / \partial(\text{router})$ 完全来自可微的 $g_i$。top-k 只是个**不可微的 mask**，但被保留下来的 $g_i$ 的"大小"仍可反传。
- 效果：router 通过加权组合获得**"重要性梯度"**，学会把能降低 loss 的 token 分给合适的专家；被选中的专家也拿到标准梯度（只来自路由到它的 token）。

**局限与注意**：
- 梯度**只流向被选中专家** → router 信号稀疏、有偏（探索不足），这也是为什么仍要靠 load balancing 辅助训练 router。
- index 无梯度，router 只能"学怎么打分"、不能直接"学怎么选"。

**其他松弛 / 方案**：
1. **Soft MoE**（2308.00951）：把 token 融合成连续 **slots** 再喂专家，路由**完全可微**（已在背景补充提及）。
2. **Gumbel-Softmax / relaxed top-k**：用温度退火把离散选择松弛成连续采样，训练时可导、推理时取硬 top-k。
3. **Straight-Through Estimator (STE)**：前向用硬 top-k，反向把 selection mask 的梯度**直通（copy-through）**，近似给 router "选谁"的梯度。
4. **REINFORCE / policy gradient**：把 router 当 policy、用专家贡献做 reward 做策略梯度（早期方法，方差大、现已少用；softmax gating 本质上就是它的可微替代）。

---

## 五、背景补充（没进主流，但值得读「为什么好想法没赢」）

- **Expert Choice**（2202.09368）：专家选 token，天然均衡高质量，但败于 decoder-only 的 causal leakage。
- **Hash Layer**（2106.04426）：确定性、零通信、公平但僵；V4 只敢用在最浅层。
- **Soft MoE**（2308.00951）：token 融成可微 slot，质量好但内存高、改动侵入，未主流。
- **Scaling-law tension**：Ling 2.0（2507.17702）称激活率越低越高效（U-optimal 粒度≈12）；但 2509.23678 算出理论最优激活率 22%–40%。业界 ≤4% 是成本驱动的 over-provisioning，非 loss-optimal——两面都要讲。

### 扩展阅读（基础补课，非重点）
Sparsely-Gated MoE (1701.06538, 奠基/routing collapse) · GShard (2006.16668, 专家并行/capacity) · Switch (2101.03961, top-1) · ST-MoE (2202.08906, z-loss) · Aux-Loss-Free (2408.15664, bias 均衡).

---

## 六、参数量拆解：从 config 反推 total / active

记号：$h$ 残差宽度、$E$ 专家数、$k$ 是 top-k、$m$ 单个专家 FFN 中间维、$L$ MoE 层数、$V$ 词表。

单个专家就是一个标准 SwiGLU FFN：`gate_up_proj` 是 $2m\times h$（gate / up 合并成一个矩阵），`down_proj` 是 $h\times m$。所以每个专家 $2mh + mh = 3mh$ 个参数（记住这个 $3$，就是 SwiGLU 的三个矩阵）。

$$
\text{每层专家参数} = E\cdot 3mh
$$

整层 MoE 加上共享专家和 router：

$$
\text{total} \approx \underbrace{L\cdot 3Emh}_{\text{routed}} + \underbrace{L\,(3mh + h)}_{\text{shared expert + 它的 gate}} + \underbrace{L\,E h}_{\text{router}} + \text{注意力} + \underbrace{2Vh}_{\text{embed + lm\_head（untied 时）}}
$$

$$
\text{active} \approx L\cdot 3kmh + L(3mh+h) + \text{注意力} + Vh
$$

关键观察：total 与 active **只差一个 $E \to k$ 的替换**。稀疏的全部算术就是这一行。

### 实例：Qwen3.5-35B-A3B（$h=2048$，$L=40$，$E=256$，$k=8$，$m=512$，$V=248320$）

| 模块 | 公式 | 数值 |
|---|---|---|
| 路由专家 | $L\cdot 3Emh$ | **32.21 B** |
| 嵌入 + lm_head（untied） | $2Vh$ | 1.02 B |
| 注意力（30 层线性 + 10 层全） | — | 1.28 B |
| shared expert | $L(3mh+h)$ | 0.13 B |
| router | $LEh$ | 0.02 B |
| **文本合计** | | **34.66 B** |
| \+ ViT（depth 27） | | 0.56 B |
| **合计** | | **≈ 35.2 B** |

官方 safetensors 标 35.95 B，差的 ~0.7 B 是 MTP 层（transformers 加载时被 `_keys_to_ignore_on_load_unexpected = ["^mtp.*"]` 忽略）。

激活侧：$L\cdot 3kmh = 1.01$ B + 注意力 1.28 B + shared 0.13 B + lm_head 0.51 B ≈ **2.93 B**，正好对上模型卡的 "A3B"。

### 两个能直接用的结论

1. **专家权重占总参 ≈ 92%**。稀疏比 $E/k = 32$ 意味着"每 1 份算力对应 32 份参数"。模型卡上 "35B-A3B" 的 35B 里九成是**每 token 只读 8/256 的死重**，A3B 才是真实算力。所以跨模型比"总参"几乎没有意义，要比就算 active。
2. **细粒度细分是"免费"的**：在**保持总参与激活都不变**的前提下做细分——$E \to 2E$、$m \to m/2$、$k \to 2k$——则 $3Emh$ 不变（总参不动）、$3kmh$ 不变（算力不动），但可选组合数从 $C(128,8)\approx1.4\times10^{12}$ 涨到 $C(256,16)\approx1.0\times10^{25}$（约 $7\times10^{12}$ 倍）。这就是大家敢不停加专家、把 $m$ 越切越小的定量理由：算力不动，表达空间指数爆炸。（Qwen3 的 128×768 → Qwen3.5 的 256×512 正是朝这个方向走。）

---

## 七、工程落地：HF Transformers 侧的坑

看论文时不会遇到、但一上手微调就会踩的几个点（均对着 transformers 源码核对，4.40 / 4.46 / 4.49 / 4.57.6 / 5.x）。

- **aux loss 默认根本没开**。`output_router_logits` 在 config 里默认 `False`，而负载均衡损失只在它为 `True` 时才算并加进 loss（模型 forward 内部）。
- **`average_tokens_across_devices` 在 4.x 默认是 `False`**（5.x 才改成 `True`）。MoE 各卡真实 token 数差异比 dense 更大，建议显式设 `True`，走 `num_items_in_batch` 做 token 级平均，否则梯度累积下 loss 是错的。
- **专家层的实现随版本变，直接决定 LoRA 能不能打到专家**：4.57.x 是 `nn.ModuleList([Qwen3MoeMLP(...)])`（每层是标准 `nn.Linear`，可以 LoRA）；5.x 改成 3D `nn.Parameter` 的 `gate_up_proj: (E, 2m, h)`，**不是 `nn.Linear`，标准 LoRA 包不住** → 只能 LoRA 到注意力 / `shared_expert` / `gate`。
- **HF 参考实现没有 grouped GEMM、没有 EP**。MoE block 是 Python `for expert_idx in expert_hit:` 循环遍历命中的专家再 `index_add_`。256 专家下训练吞吐会很差；生产要用 Liger（`apply_liger_kernel_to_qwen3_moe` 等）或 Megatron / DeepSpeed-MoE 的专家并行。
- **导出推理前把 `output_router_logits` 改回 `False`**；另外开了之后 eval loss 里含 aux 项，与 dense 模型的 eval loss 不可直接比较。

---

## 八、GEMM 与 MoE 的关键算子

这一节补两件事：**矩阵乘的维度记法**（读 kernel / 算子文档时到处都是 M、N、K），以及 **MoE 层里到底在跑哪些算子**。看得懂这一节，就能看懂为什么 MoE 的工程优化几乎全在做"怎么把一堆小矩阵乘算快"。

### 1. GEMM 与 M / N / K

**GEMM** = GEneral Matrix Multiply，就是通用矩阵乘 $C = A \times B$。约定写法：

$$
\underbrace{C}_{M\times N} \;=\; \underbrace{A}_{M\times K}\;\times\;\underbrace{B}_{K\times N}
$$

三个字母就是三个维度的名字，记住它就不怕了：

- **M**：输出有几行 —— 在 LLM 里就是**这一批有多少个 token**（行数 = token 数）。
- **N**：输出有几列 —— 权重矩阵"把特征变宽/变窄到多少"，也就是**输出维度**。
- **K**：中间那个被消掉的维度 —— **输入维度 / 累加长度**，两个矩阵必须在这里对齐。

> 助记：**M 是"样本数"，N 是"输出宽"，K 是"输入宽"**。GPU 上 GEMM 的效率基本正比于这三个数够不够大（太小就喂不饱 tensor core）。

**为什么它在 LLM 里这么重要**：Transformer 里九成以上的算力都是 GEMM —— QKV 投影、attention 的 $QK^\top$ 与 $\times V$、FFN 的两次升降维，全是它。所以"模型快不快"在底层就是"GEMM 跑得够不够满"。

拿 Qwen3.5 的一个专家举个具体例子（$h=2048$，专家中间维 $m=512$）。假设有 $T$ 个 token 被分给这个专家：

```python
# 第一步 gate_up_proj：2048 -> 1024（gate 和 up 拼在一个矩阵里，之后 chunk 成两半）
#   A: T x 2048   W: 2048 x 1024   输出: T x 1024
#   M = T（token 数）    K = 2048（输入宽）    N = 1024（输出宽）
gate, up = (x @ W_gate_up.T).chunk(2, dim=-1)

# 第二步 down_proj：512 -> 2048
#   M = T    K = 512    N = 2048
y = (act(gate) * up) @ W_down.T
```

### 2. 为什么 MoE 的 GEMM 天生更难

- **dense**：一个 batch 的所有 token 走**同一份** FFN 权重 → 一个规整的大 GEMM，$M$ 就是全部 token 数，喂得饱。
- **MoE**：token 先被路由分给不同专家，每个专家只算属于自己的那部分 → 变成**每个专家一个小 GEMM**。
  - 各专家的 $M$ **各不相同**（取决于这个专家分到几个 token，运行时才知道）；
  - 但 $N$ 和 $K$ **完全相同**，因为所有专家的权重形状一模一样。

每个小 GEMM 的 $M$ 可能只有几十（一个专家分到几十个 token），这种形状单个跑 GPU 利用率极低；更糟的是如果专家数很多（256 个），就是几百次 kernel launch。这就是 **grouped GEMM** 存在的理由：把这一批"M 不同、N/K 相同"的小 GEMM 合并进**一次** kernel launch。

```python
# 朴素：每个专家一次 GEMM（多次 launch，形状小 -> GPU 空转）
for i in hit_experts:
    y[tokens_of_i] = x[tokens_of_i] @ W_i.T

# grouped GEMM：一次 launch，内部按 token 数（M 轴）分段，每段一个专家
grouped_mm(x_sorted_by_expert, offsets_per_expert, W_stacked)
```

顺着这条线还能理解几个工程选择：

- **DeepGEMM 只在 M 轴分组**，就是吃准了"专家权重形状一致"这一点（CUTLASS 的通用 grouped GEMM 允许 M/N/K 都变，但配置开销更大）。
- **contiguous 布局**（各专家 token 沿 M 轴拼接，段内对齐 block size）用于训练前向 / prefill；**masked 布局**（用 mask 标有效区域）用于 decode，可以塞进 CUDA graph。
- **专家不能太大也不能太小**：$m$ 越大单个 GEMM 越饱满，但组合灵活性下降；$m$ 越小越灵活但 GEMM 越碎（这也解释了为什么大家一边加专家数一边把 $m$ 越切越小，然后必须靠 grouped GEMM 兜底）。

### 3. MoE 层的关键算子清单

一个 MoE 层从输入到输出，依次是这些算子：

| 算子 | 干什么 | 为什么关键 |
|---|---|---|
| **top-k gating / routing** | 算 router logits → 取 top-k → 归一化出权重 | 决定 token 去向；**离散选择不可导**（见第四节） |
| **permute / scatter** | 按专家编号重排 token，让同一专家的 token 在内存里连续 | GEMM 要求连续输入；也是后续 all-to-all 的前提 |
| **grouped GEMM** | 一次 kernel 算完所有专家的 FFN（含 gate_up / SwiGLU / down 三段） | MoE 的**计算主体**，效率全看它 |
| **SwiGLU 激活** | 专家内部的 $\mathrm{act}(gate)\odot up$ | 与 GEMM 强耦合，常被融合进 kernel |
| **un-permute / gather** | 按权重加权、累加回原 token 位置（`index_add_` / combine） | 还原序列顺序 |
| **all-to-all** | 跨卡把 token 发到持有目标专家的卡，再发回来 | EP 的通信本体，且是**变长**的（各专家 token 数不同） |
| **load-balancing loss** | 统计专家负载，加一项辅助损失 | 只在训练时用，用来防路由坍缩 |

工程上的终极形态是**把这些融合掉**：例如 DeepGEMM 的 Mega MoE 把 dispatch、两段低精度 GEMM、SwiGLU 和 combine 融成一条流水，并让 NVLink 上的通信与 tensor core 的计算重叠。融合越多，越能掩盖 MoE 天然的小 GEMM 与通信开销——这也是"MoE 训练框架"最核心的护城河（Megatron / DeepSpeed-MoE 的价值主要在这里，而不在模型定义本身）。

---

## 一句话默认配方（今天要设计 MoE）

Fine-grained experts + 1 shared（可带 sigmoid gate） + **sigmoid selected-inner-norm（DeepSeek 系）/ softmax-over-all + top-k renorm（Qwen 系）** + loss-free balancing + 3–5% activation + 1 MTP + hybrid attention + FP8→MXFP4 + Muon.

（两条修正：① 首 1–3 层 dense 只是 DeepSeek 系的稳定性惯例，Qwen3 / Qwen3.5 的 `mlp_only_layers=[]`，**每层都是 MoE**；② gating 不是唯一解，sigmoid 与 softmax 是并列的两条路线。）
