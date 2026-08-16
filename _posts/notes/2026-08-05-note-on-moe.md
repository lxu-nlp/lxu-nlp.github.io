---
layout: post
title: "Notes on LLM MOE"
date: 2026-08-05
categories: [Notes]
tags: [llm]
math: true
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

### Qwen3（技术报告，235B-A22B）
**Motivation**：反例路线——**不加共享专家**，塞入 global-batch 均衡 + 线性注意力混合，证明 DeepSeekMoE 模板非唯一解。

- 每层 128 routed、top-8、**无 shared**（注：Qwen3-Next 又加回共享专家 + Gated DeltaNet，家族不统一）。
- Sigmoid + **global-batch loss-free balancing**：跨**整个 global batch** 聚合统计（非 per-microbatch），大尺度下信号更稳。
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

---

## 三、专题：Routing Collapse 解法演进

**导语**：现代路由仍是 per-token，下面方法多在 per-token 路由外加一层均衡校正。

### 1. 早期：梯度内辅助损失 + 容量截断
- **Auxiliary load-balancing loss**（GShard / Switch）：加鼓励均匀分配的损失项（专家 importance × 均衡项），强迫 router 摊开 token。
- **Expert capacity + dropping**：每专家上限 $capacity = (tokens/batch / N)\times\alpha$，超额丢弃走残差。限损不防坍缩、还丢信息。
- **为何被弃**：aux loss 进**主任务梯度**、伤害最终质量（正是 V3 转向 loss-free 的原因）。

### 2. 现代主流：无梯度均衡（Loss-free）
核心 insight（DeepSeek, 2408.15664）：把均衡信号**移出梯度**——用梯度外的控制器调 router logit，分布仍均衡却不污染表征。
- **Bias-term（V3，事实标准）**：梯度外 per-expert bias $b_i$，按负载 $\pm\gamma$，**永不回传到模型权重**。
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

## 一句话默认配方（今天要设计 MoE）

Fine-grained experts + 1 shared + sigmoid selected-inner-norm + loss-free balancing + 3–5% activation + 首 1–3 层 dense + 1 MTP + hybrid attention + FP8→MXFP4 + Muon.
