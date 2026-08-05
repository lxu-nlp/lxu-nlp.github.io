---
layout: post
title: "Notes on LLM MOE"
date: 2026-08-05
categories: [Notes]
tags: [llm]
math: true
hidden: true
---

# MoE 重点论文清单（精读版）

> **适用读者**：有 Transformer / LLM 基础（知道 attention、FFN、pretraining、scaling law），但没系统学过 Mixture-of-Experts。
> **读法**：先看「前置知识」建立术语和直觉；再读「基础工作」把 DeepSeekMoE 的结构模板与 DeepSeek-V3 的训练配方打通；接着看「新主流 setup」了解近两年开源模型如何演化；最后用「专题：Routing Collapse」补全负载均衡方法的全貌。
> **语言约定**：整体介绍、动机、直觉用**中文**；具体机制、公式、配置用**英文**展开。

---

## 前置知识：从 dense Transformer 到 MoE

### 1. 为什么要在 Transformer 里塞 MoE
在标准 Transformer 中，每个 token 经过 self-attention 后，会进入一个 **FFN（前馈网络，Feed-Forward Network）**。FFN 通常是两层 MLP（上投影 → 激活 → 下投影），**参数量约占整个模型的 2/3**。MoE 的核心想法是：把"一份大家共用的大 FFN"换成"**一组并行的专家 FFN + 一个 router（门控网络）**"。每个 token 不再过完整的 FFN，而是由 router 只挑出少数几个专家来处理它。

> 直觉：模型总参数可以做得非常大（很多专家），但**单个 token 实际参与计算的部分很小**——这就是"稀疏激活（sparse activation）"。于是你在"总参数量（决定模型容量）"和"单 token 计算量（决定推理/训练成本）"之间解耦了。

### 2. 核心术语
- **Expert（专家）**：一份独立的 FFN 权重。MoE 层里通常有 N 个专家并行存在。
- **Router / Gating（门控）**：一个小线性层，输入 token 的隐藏向量，输出对每个专家的打分，再据此选出 top-k 个专家。
- **Top-k routing**：每个 token 只激活 k 个专家（k 通常远小于 N，例如 8/256）。**粒度说明**：在 decoder-only LLM 中，路由决策是 **per-token** 的——每个 token 的隐藏向量独立过 router 选 top-k；所谓 global-batch / sequence-level 只是「均衡校正」的统计粒度，不改变「路由本身是逐 token」这一事实。唯一反例是 Expert Choice（专家选 token，见背景补充）。
- **Total params vs Active params**：总参数是所有专家之和（巨大）；激活参数是单 token 实际用到的那 k 个专家 + 其他固定层（很小）。模型卡上常写成 `671B / 37B`。
- **Capacity factor（容量因子）**：每个专家单步最多能接收多少 token；超出的 token 会被 **dropping（丢弃）**或走残差。

### 3. 三大核心难题（贯穿下面所有论文）
1. **Routing collapse（路由坍缩）**：router 可能"偷懒"，把所有 token 都送进同一两个专家 → 其余专家永远不被激活、参数浪费。这是 MoE 的头号病，解法叫 **load balancing（负载均衡）**。
2. ** Loading balancing (负载均衡)**：早期用 auxiliary loss（往梯度里加一项鼓励专家使用均匀），但它会**污染主任务的梯度、拖累质量**。早期代表即 DeepSeekMoE 的 **Expert-Level + Device-Level Balance Loss**（见基础工作）；现代主流改成 **aux-loss-free（无辅助损失）**方案（见 V3）。
3. **通信瓶颈**：专家通常分布在不同 GPU / 节点上，token 要先 all-to-all 发到对应专家、算完再收回 → 通信开销巨大。衍生出 expert parallelism、node-limited routing、DualPipe 等工程手段。
- （附带）**训练稳定性**：router 的 logit 可能数值爆炸 → **router z-loss**（见 ST-MoE）。

---

## 一、基础工作：DeepSeekMoE 与 DeepSeek-V3（合并讲解）

这一节是今天所有开源 MoE 的源头。DeepSeekMoE 提出了现代 MoE 层的**结构模板**——它回答了"专家该怎么切、怎么分"；DeepSeek-V3 把这套模板落成了**可复现的完整训练配方**——它回答了"怎么把它训出来还不崩、还快、还均衡"。两篇应放在一起读：前者讲"层怎么长"，后者讲"怎么把它养大"。

### DeepSeekMoE — arXiv:2401.06066（现代 MoE 层的结构模板）

**Motivation**：在 DeepSeekMoE 之前，MoE 的痛点是——专家太"粗"（每个专家是一整个大 FFN），导致 top-k 选出来的组合很有限，知识混在一起难以解耦；同时所有专家都参与路由，通用知识会被反复算。DeepSeekMoE 用两招解决：**把专家切细**，以及**专门留一组"人人都用"的共享专家**。

- **Fine-grained expert segmentation**: split each original FFN expert into *m* smaller experts (same total parameter budget, but more, narrower experts). With top-k routing over this finer granularity, the **combinatorial space of expert combinations grows exponentially**, so a token can be expressed as a much richer mixture → more flexible, disentangled knowledge allocation. Intuition: "many narrow specialists" beats "few broad generalists" under a fixed active-parameter budget.
- **Shared expert isolation**: reserve a small set of *shared experts* that are activated for **every** token (carrying common, token-agnostic knowledge such as syntax/formatting), while the remaining *routed experts* are selected per-token (carrying specialized knowledge). This separates "what everyone needs" from "what this specific token needs", preventing the routed experts from having to re-learn the common part.
- **Gating & MoE layer output**: routed experts are scored by a sigmoid affinity `s_{i,t}`; a token selects the top-`(mK − Ks)` routed experts (others get gate 0) while all `Ks` shared experts are always on. Layer output: `h_t = Σ_shared FFN_i(u_t) + Σ_routed-topk s_{i,t}·FFN_i(u_t) + u_t`.
- **Load balancing — Expert-Level & Device-Level Balance Loss** (the *original auxiliary-loss* scheme; V3 later replaces it with aux-loss-free): to stop routing collapse, DeepSeekMoE adds two gradient-affecting losses:
  - **Expert-Level Balance Loss**: `L_ExpBal = α₁ · (1/N′) · Σ_i f_i·P_i`, where N′ = #routed experts, `f_i` = normalized selection frequency of expert i, `P_i` = its average routing affinity. Minimizing `f_i·P_i` steers tokens toward under-used experts.
  - **Device-Level Balance Loss**: `L_DevBal = α₂ · (1/D) · Σ_d f′_d·P′_d`, where D = #devices, `f′_d` = mean `f_i` over experts on device d, `P′_d` = sum of `P_i` over those experts. Balances **compute per device** (what drives all-to-all comms), even if individual experts aren't perfectly balanced.
  - Practice: a **small** α₁ (expert-level, to avoid hurting quality) + a **larger** α₂ (device-level, to balance compute). (DeepSeek-V2 later adds a third *Communication Balance Loss* for balanced per-device traffic.)
- **Canonical config**: paper's 16.4B variant = **64 routed experts + 2 shared, top-6 routed**; 236B variant scales to **160 routed + 2 shared**。The "fine-grained + shared" coupling is the **direct ancestor** of DeepSeek-V3 / Kimi K2 / GLM-4.5 / DeepSeek-V4.

### DeepSeek-V3 — arXiv:2412.19437（完整工程配方，把模板训出来）

**Motivation**：有了好的层结构，还要解决三件工程实事——(a) 怎么均衡而不伤质量；(b) 怎么在万卡上把 all-to-all 通信藏起来；(c) 怎么用低精度把训练成本压下来。V3 给出了当时最完整的答案，后续开源模型基本照抄它的骨架。

- **Scale**: **671B total / 37B active**。Per MoE layer: **256 routed experts + 1 shared expert**, **top-8** of the routed ones activated per token.
- **Gating**: **sigmoid gating with selected-expert normalization** — compute a sigmoid score per expert, pick top-k, then **renormalize only among the chosen k** (replacing the classic softmax-over-all-experts). This keeps scores well-behaved and makes the active set comparable across tokens.
- **Load balancing — aux-loss-free** (arXiv:2408.15664): maintain a per-expert **bias term bᵢ outside the gradient path**. Every few steps: if expert *i*'s token count < batch mean → bᵢ += γ; else bᵢ -= γ. The bias is added to the router logits at routing time, steering tokens toward under-used experts **without polluting the main-task gradient**. A small complementary **sequence-level auxiliary loss** is kept as a safety net.
- **Decoding & infra**: **1 MTP (Multi-Token Prediction) layer** (depth D=1, predicts one extra token; loss weight λ=0.3 for first 10T tokens then 0.1) — enables **speculative decoding** at inference.
- **Bias-update schedule (aux-loss-free)**: the bias `bᵢ` updates with γ=0.001 for the first 14.3T tokens, then **γ=0** for the final 500B — balancing is frozen late once experts have specialized.
- **Training footprint**: **14.8T tokens**, 2048× H800, ~2.788M GPU-hours (~$5.6M).
- **Value**: the most complete "how to actually train a trillion-parameter MoE" manual; its skeleton (fine-grained + shared + sigmoid + loss-free + MTP + FP8/DualPipe) is what later open models copy.

---

## 二、新主流 setup（近两年真正落地的开源 MoE）

**中文总览**：读完基础你会发现，到 2024 年底 MoE **层结构本身已经收敛**——几乎都长成 DeepSeekMoE 的样子（细粒度 + 共享专家 + sigmoid + 无损失均衡）。所以近两年的差异主要集中在三条线上：
1. **均衡策略的变体**：global-batch、EP-group、Quantile Balancing；
2. **稀疏 scaling**：专家数越堆越多（256 → 384 → 896）、激活率越压越低；
3. **注意力侧的混合改造**：MoE 同质化后，真正的军备竞赛转移到了 attention（滑动窗口、线性注意力、压缩稀疏注意力）。

### Qwen3（技术报告，235B-A22B）

**Motivation**：Qwen3 走了一条"反例"路线——**不加共享专家**，并把自己特色的均衡（global-batch）和低精度/线性注意力混合塞进来，证明 DeepSeekMoE 模板并非唯一解。

- Per layer **128 routed experts, top-8, no shared expert** (caveat: **Qwen3-Next** re-adds a shared expert + 3:1 Gated DeltaNet, so the family is not uniform).
- Sigmoid gating; **global-batch load balancing** — a loss-free variant that aggregates balancing statistics across the **entire global batch** (not per-microbatch), giving a more stable load signal at large scale.
- **No auxiliary routing loss at all**: balancing is *purely* the global-batch statistics trick — there is **no gradient-affecting aux term**, and (unlike DeepSeekMoE) **no shared expert**. This is the clearest departure from the DeepSeekMoE template among major open models.
- Differentiation moved to attention: mixes a **gated delta rule (linear attention)** branch alongside standard softmax attention.

### Kimi K2 / K3（Moonshot，2025 / 2026）

**Motivation**：Kimi 这条线把"更多专家 + 更高激活"的稀疏 scaling 推到极致（K2 384 专家 → K3 896 专家），并为此配套了专门的优化器（MuonClip）和均衡算法（Quantile Balancing），因为专家越多，固定步长的偏置更新越容易震荡。

- **K2**: **1T total / 32B active**; **384 routed + 1 shared, top-8**; **MuonClip** optimizer (Muon with a Clip term to prevent attention-logit explosion during the aggressive update); explicitly favors a "more experts + higher activation" sparsity regime.
- **K3 (2026)**: **≈2.8T total**, built on **Stable LatentMoE** — **896 routed + 2 shared, top-16** per token → **~56× routed-expert sparsity**. The key trick: the hidden state is **compressed 7168 → 3584 dim** before the routed experts compute, then projected back, so you can grow expert count *and* activation count without communication scaling with the full hidden dim. Key new pieces:
  - **Quantile Balancing** — instead of the fixed-step bias update (which oscillates at 896-expert scale), it frames balancing as a **dual linear program** and derives an **analytical solution** that drives each expert's load toward a target quantile.
  - **KDA hybrid linear attention at 3:1** (KDA : Gated MLA) + **AttnRes** (block-level cross-layer residual that lets each layer retrieve past-layer representations) — the "attention is the new battleground" trend, paired with the **Moon Clip** second-order optimizer.
  - Weights stored / trained in **MXFP4** (from SFT onward via QAT).

### LongCat-Flash — arXiv:2509.01322（美团，零计算专家）

**Motivation**：前面所有模型都是"固定激活 k 个专家"。LongCat 想更进一步——让激活量**随 token 难度自适应**：简单 token 直接短路跳过专家计算，难 token 才用满。它往专家池里塞"什么都不算"的恒等专家来实现这一点。

- **560B total** (Shortcut-connected MoE, **ScMoE** framework), **dynamic per-token activation of 18.6B–31.3B** (mean ~27B) — *not* a fixed active count, but a **token-difficulty-adaptive range**. The mechanism is **adaptive expert count via zero-computation experts**:
  - The routed expert pool is augmented with **shortcut / identity experts** that perform **no FFN compute** — they return the input unchanged (a pass-through residual). The router's top-k selection is drawn from this **combined** pool (real experts + shortcut experts).
  - Because a token may be routed to *shortcut* experts instead of *real* ones, the number of **computing** experts varies per token even when the selected-set size is fixed: an **easy** token gets high weight on shortcut experts → few (or zero) real experts fire → near-zero added FLOPs; a **hard** token selects mostly real experts → full compute. The effective active-parameter count therefore **slides across the 18.6B–31.3B window** instead of being pinned at one value.
  - Intuition: the model learns a **difficulty-aware gate** — simple tokens self-route to the "do nothing" experts, so compute tracks token hardness rather than a constant k.
- **Shortcut-connected MoE**: the same shortcut path doubles as a **residual connection that flows while the real experts compute**, overlapping **all-to-all communication with expert computation** and masking the comms stall that normally bottlenecks MoE layers.

### DeepSeek-V4 (Pro)（技术报告，含 V4-Pro）

**Motivation**：V4 是"MoE 层已完全收敛"的最强信号——它**没有**重新发明 MoE 层，主体仍是 DeepSeekMoE 的老样子，只在最浅几层试了确定性 hash 路由；真正的创新全砸在**注意力（CSA/HCA）**和**超连接稳定性（mHC）**以及**FP4 低精度**上。

- **Correction to web rumors**: the MoE body is **still 384 routed + 1 shared, top-6** DeepSeekMoE — *not* a wholesale hash-routing replacement of learned routing. Only the **first few layers' dense FFN is swapped for hash-routed MoE** (deterministic, training-free, content-based hashing → zero routing comms at those layers).
- Real innovations are in attention: **CSA (Compressed Sparse Attention)** for compressing KV context + **HCA (Hierarchical Cache Attention)** for long-context retrieval efficiency.
- **mHC (modified Hyper-Connections)**: constrains the residual **B-matrix on the Birkhoff doubly-stochastic manifold** (sum-to-one rows & cols) to **prevent spectral-norm explosion** in the hyper-connection pathway — a stability trick for very deep/wide models.
- Expert weights in **FP4**; training uses **Muon**.
- **Value**: marks the field's center of gravity moving **entirely off the MoE layer** and onto attention architecture + low-precision. Two released sizes: **V4-Pro (1.6T / 49B active)** and **V4-Flash (284B / 13B active)**, trained on 32T tokens with 1M context.

---

## 三、专题：如何解决 Routing Collapse（负载均衡方法演进）

**中文导语**：Routing collapse 是 MoE 的头号病——router 偷懒，把所有 token 都塞进同一两个专家，其余专家饿死、参数浪费、训练低效。下面按"从污染梯度 → 完全去梯度 → 结构性去坍缩"的脉络，梳理主流解法。注意：现代路由决策仍是 **per-token**（见前置知识），下面这些方法大多是"在 per-token 路由之外，加一层均衡校正"。

### 1. 早期：梯度内辅助损失 + 容量截断（Auxiliary loss + Capacity）
- **Auxiliary load-balancing loss** (GShard / Switch): add a loss term that encourages **uniform expert assignment**. Typically combines (a) per-expert *importance* = mean router probability over tokens, and (b) a *balance* term (e.g. coefficient of variation / entropy) pushing all experts toward equal usage. Minimizing it forces the router to spread tokens.
- **Expert capacity + token dropping**: cap each expert at `capacity = (tokens_per_batch / N) × α`; tokens exceeding capacity are **dropped** (bypass the MoE via the residual). Caps the damage of collapse but doesn't prevent it; also loses information on dropped tokens.
- **Why it fell out of favor**: the auxiliary loss enters the **main-task gradient**, acting as a regularizer that **hurts final quality** (the very thing DeepSeekMoE/V3 moved away from). This motivates all "loss-free" methods below.

### 2. 现代主流：无梯度均衡（Loss-free balancing）
The key insight (DeepSeek, 2408.15664): keep the balancing signal **out of the gradient** — adjust the router *logits* with a separate, gradient-free controller, so the expert distribution stays balanced without polluting representations.
- **Bias-term adjustment (DeepSeek-V3, the de-facto standard)**: maintain a per-expert bias `bᵢ` added to router logits at routing time. Every few steps: if expert *i*'s token count < batch mean → `bᵢ += γ`; else `bᵢ -= γ`. The bias steers tokens toward under-used experts but **never backprops into the model weights**. V3 also keeps a tiny complementary sequence-level aux loss as a safety net.
- **Global-batch balancing (Qwen3)**: same loss-free spirit, but aggregate the load statistics across the **entire global batch** (not per-microbatch) for a more stable signal at large scale; no shared expert, 128 routed / top-8.
- **EP-group balancing (Step 3.5 Flash)**: balance within each **expert-parallel group** (a subset of experts co-located on a device/group), trading a little global balance for lower communication.
- **Quantile Balancing (Kimi K3, 896 experts)**: at very large expert counts, the fixed-step bias (`±γ`) **oscillates** (overshoots experts). K3 frames balancing as a **dual linear program** and solves it **analytically**, driving each expert's load toward a target quantile — stable even at 896 experts.

### 3. 结构性去坍缩（免训练 / 确定性，不走梯度）
- **Expert Choice routing** (arXiv:2202.09368): flip the direction — *each expert picks its top-k tokens* (instead of each token picking experts). Balanced **by construction** (every expert gets exactly k tokens), no aux loss needed. Why it lost: in decoder-only generation an expert would need to see *future* tokens to choose → **causal leakage**, so it can't be used for autoregressive inference. A clean idea defeated by a deployment constraint.
- **Hash routing** (deterministic, V4 shallow layers): map token (or its hash) to a fixed expert via a **content hash** → no learned router, therefore **no collapse possible** (assignment is predetermined). Zero routing comms. Why it's limited: inflexible / can't specialize; DeepSeek-V4 uses it only on the **first few layers** where routing mistakes are cheap.

### 4. 稳定性补充：Router z-loss（ST-MoE）
- Even with balancing, router logits can grow unbounded and **saturate** the gate, indirectly worsening collapse-like behavior. **Router z-loss** (arXiv:2202.08906) adds `z_loss = c · log²(mean(exp(logits)))` to penalize logit magnitude, keeping the gate numerically stable. Present in most modern MoE training recipes alongside the methods above.

**小结（演进主线）**：
auxiliary loss (gradient-hurting) → capacity/dropping (damage-cap) → **loss-free bias (DeepSeek, mainstream)** → global-batch / EP-group / quantile (scale-aware variants) ; plus structural routes (**Expert Choice, Hash**) that avoid collapse by design ; **z-loss** for stability.

---

## 四、背景补充（没进主流，但值得读「为什么好想法没赢」）

- **Expert Choice** (arXiv:2202.09368): flip the direction — *experts choose tokens* instead of tokens choosing experts → naturally balanced and higher quality, but excluded by **causal leakage** in decoder-only inference (an expert would need to see future tokens to pick). A clean idea that lost on a deployment constraint.
- **Hash Layer** (arXiv:2106.04426): deterministic, training-free, **zero-communication** routing; fair and cheap but inflexible (V4 only dares use it at the shallowest layers where mistakes are cheap).
- **Soft MoE** (arXiv:2308.00951): fuse incoming tokens into continuous differentiable *slots* before experts, making routing fully differentiable → good quality but high memory / intrusive changes, never mainstream.
- **Scaling-law tension**: Ling 2.0 (arXiv:2507.17702) claims **lower activation ratio → monotonically higher efficiency leverage** (U-optimal expert granularity ≈ 12); but arXiv:2509.23678 computes the **theoretical optimal activation ratio at 22%–40%**. Industry's ≤4% is **cost-driven over-provisioning**, not loss-optimal — always present both sides.

### 扩展阅读（基础补课，非重点，按需）
Sparsely-Gated MoE (1701.06538, 奠基/routing collapse) · GShard (2006.16668, 专家并行/capacity) · Switch Transformer (2101.03961, top-1 scaling) · ST-MoE (2202.08906, router z-loss 稳定性) · Aux-Loss-Free (2408.15664, 偏置项均衡，见 V3 内联).

---

## 一句话默认配方（今天要设计 MoE）

Fine-grained experts + 1 shared expert + sigmoid selected-inner-norm + loss-free balancing + 3–5% activation + first 1–3 dense layers + 1 MTP + hybrid attention + FP8→MXFP4 + Muon.
