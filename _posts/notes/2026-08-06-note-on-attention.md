---
layout: post
title: "Notes on LLM Attentions"
date: 2026-08-06
categories: [Notes]
tags: [llm]
math: true
---

## 前置知识：从标准 Multi-Head Attention 出发

**为什么 attention 成了瓶颈。** 一个 decoder-only LLM 里，每个 token 生成时都要把它的 Query 和**所有历史 token 的 K、V**做点积。这就带来两个随序列长度 $L$ 增长的成本：

- **计算量 $O(n^2)$**：每个新 token 都要和历史全部 token 算一遍注意力，长上下文时 prefill 阶段尤其贵。
- **KV cache 显存 $O(n)$**：必须把每个历史位置的 K、V 缓存下来供后续 token 复用，长上下文时 KV cache 常常超过模型权重本身。

标准 MHA（Vaswani et al., 2017）里，**每个注意力头都有自己独立的 K、V**，因此每层每 token 要缓存 $2 \cdot n_h \cdot d_h$ 个数。当 $n_h=128$、$d_h=128$ 时，单 token 就要 ~32K 个数的 KV cache——这是后续所有优化的出发点。

> 注意：下面三类方法分别从不同角度切这个瓶颈。它们**不是互相替代**，而是被各大模型**混合堆叠**使用（最后会讲）。

---

## 〇、底层算法：Flash Attention（IO-aware 精确注意力）

所有前面基于 softmax 的注意力（MHA / GQA / MLA / SWA / NSA），最终都要落到「怎么在 GPU 上真正算这个 softmax」上。Flash Attention 不是新的注意力架构、也不改数学结果——它是让**精确的全注意力**在硬件上跑得飞快的底层算法，是今天几乎所有高效 LLM 训练与推理内核的基石。如果只记住一点：它和本文其他三路线**正交**——稀疏/线性/压缩是「改算什么、算多少」，Flash Attention 是「怎么把该算的全注意力算得又快又省显存」。

标准实现（naive attention）的坑：先把 $N\times N$ 的注意力分数矩阵整个算出来、写进 GPU 显存（HBM），再做 softmax。但 HBM 很慢、片上 SRAM 快却极小。序列一长，这个 $N\times N$ 矩阵巨大，瓶颈根本不是算力，而是**在 HBM 和算力之间反复搬运数据的带宽**。Flash Attention 的核心洞察是：别把整个注意力矩阵存下来，把它**分块（tiling）**在 SRAM 里算，softmax 增量更新，需要的中间量**重算**而非存储。

> **关键澄清**：Flash Attention 算出来的注意力和朴素实现**数学上完全相等（exact）**——它不近似、不稀疏、不线性化，只是少搬数据、少占显存。因此它和压缩/稀疏/线性三条线是**正交**的。事实上，§6 NSA 里提到的「blockwise selection 保持访存连续、FlashAttention-compatible」正是因为它刻意对齐了 Flash Attention 的分块访存模式；§3 MLA、§5 的 SWA 在真实训练/推理里也全都跑在 FlashAttention 风格的内核上。

- Tri Dao et al. (Stanford), *"FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness"*, NeurIPS 2022, arXiv:2205.14135.
- **IO-aware exact attention**: for long sequences the bottleneck is **memory bandwidth (HBM ↔ SRAM)**, not FLOPs. Naive attention materializes the full $N\times N$ softmax matrix in HBM → $O(N^2)$ memory and enormous data movement.
- **Tiling — partition the matrices into SRAM-sized blocks.** Let $N$ = sequence length, $d$ = head dim, and $Q, K, V \in \mathbb{R}^{N\times d}$. Split $Q$ into **T-row blocks** $\{Q_i\}$ and $K, V$ into **B-column blocks** $\{K_j, V_j\}$ ($B, T$ chosen so a block fits in on-chip SRAM). The local score block is $S_{ij} = Q_i K_j^T \in \mathbb{R}^{T\times B}$. We never materialize the full $N\times N$ matrix; instead we loop over KV blocks and stream them through SRAM, holding only a tiny per-query state.
- **Online ("flash") softmax — incremental exact rescaling.** For each query row $q$ keep running statistics across KV blocks: $m_q$ (running max of scores), $l_q$ (running denominator), $O_q$ (running output). When block $j$ arrives with local scores $s = q K_j^T \in \mathbb{R}^{B}$ and values $V_j \in \mathbb{R}^{B\times d}$:
  - $m' = \max(m, \max(s))$  (new running max)
  - $p = \exp(s - m')$  (unnormalized local probabilities, $\mathbb{R}^{B}$)
  - $l' = e^{m - m'} \cdot l + \sum_b p_b$  (rescale old denominator + add new)
  - $O' = e^{m - m'} \cdot O + p V_j$  (rescale old output + add new block's contribution)
  - $m \leftarrow m',\; l \leftarrow l',\; O \leftarrow O'$
  After all $J = N/B$ blocks, the final attention row is $o_q = O / l$. This is **mathematically equal** to standard softmax attention: subtracting the running max keeps it numerically safe, and the $e^{m-m'}$ factor correctly rescales *all previously-seen keys* by the same constant — and softmax is invariant to a constant shift of the exponent applied consistently across the row.
- **Why no $N\times N$ matrix is needed (memory).** HBM stores only $Q, K, V$ ($\approx 3\cdot N\cdot d$, unavoidable), the output $O$ ($N\cdot d$), and the per-query $m_q, l_q$ scalars ($\approx 2\cdot N$, negligible). The per-query running $O_q$ lives in SRAM and is written back once at the end. The score/probability matrices never exist in HBM → total **$O(N\cdot d)$** instead of $O(N^2)$.
- **Why it is fast (bandwidth).** Each $Q$ block and each $KV$ block is loaded from HBM to SRAM **once**; all matmuls and the softmax rescale happen in SRAM; only the small running state is updated in place. HBM traffic is dominated by reading $Q,K,V$ once and writing $O$ once → **$O(N\cdot d)$**, versus $O(N^2)$ for naive attention that reads/writes the score matrix. For long $N$ with $d \ll N$ this is the dominant speedup.
- **Backward pass — recompute, don't store.** To save memory, the $N\times N$ matrix is not kept. During backward, $S$ and $P$ are **recomputed** in SRAM from $Q, K$ using the saved $m_q, l_q$ logsumexp stats. This trades a little extra FLOP for the huge memory saving — the essence of the "flash" idea.
- **Speed/memory payoff (v1):** ~7–20× faster and ~10–20× less memory than naive attention; directly enables far longer contexts on the same GPU.
- **Why it matters in this survey**: it is the kernel that *all* softmax-based methods here run on (PyTorch `scaled_dot_product_attention`, vLLM, xFormers, TensorRT-LLM, etc.). It is a major reason exact attention stays viable at long context — and exactly the baseline that sparse/linear attention must beat on **both quality and real wall-clock speed** to earn their extra complexity.

---

## 一、压缩 KV Cache 路线：保留 softmax 全注意力，只缩小每 token 存的东西

这一支的核心思想最简单：**注意力计算不变，但别把完整 K、V 存下来**。

### 1. MQA — Multi-Query Attention（MQA）

最直接的一刀：让所有 Query 头**共享同一套 K、V**。既然要共享，那干脆只缓存一份 K、V，KV cache 直接除以头数 $n_h$。代价是不同头无法再关注不同的"视角"，表达力下降；但它最早证明"共享 KV 几乎不损质量却大幅省显存"。

- Shazeer (2019), *"Fast Transformer Decoding: One Write-Head is All You Need"*; later popularized by **PaLM** (Chowdhery et al., 2022).
- All $n_h$ query heads attend to a **single shared K/V head** ($h_q = n_h,\; h_{kv} = 1$).
- KV cache per token: $2 \cdot d_h \cdot L$ (independent of $n_h$) → roughly **$n_h\times$ smaller** than MHA.
- Open-model adoption: **Falcon** (2023) shipped MQA; earlier GPT-J/PaLM lineage. Now largely superseded by GQA in newer open models because GQA recovers most of MHA quality at similar cache savings.

### 2. GQA — Grouped-Query Attention（GQA）

MQA 太激进、MHA 太贵，GQA 是折中：把 Query 头**分组**，每组共享一套 K、V。比如 32 个 Q 头、8 个 KV 头 → 每组 4 个 Q 头共用一个 KV 头。它正好卡在 MHA（每组 1 头）和 MQA（全模型 1 组）之间，且论文证明可以从已训好的 MHA checkpoint **升采样**得到，迁移几乎免费。**这是过去两年开源模型的绝对主流默认配置。**

- Ainslie et al. (2023), *"GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints"*, EMNLP.
- $h_q = n_h,\; h_{kv} = n_h / g$ ($g$ = group size, integer). $g$ consecutive query heads share one K/V head.
- KV cache per token: $2 \cdot h_{kv} \cdot d_h \cdot L$ = $(h_{kv} / n_h)$ fraction of MHA (e.g. 1/4–1/8).
- Recovers MHA quality at near-MQA cache cost; enabled by **uptraining from an MHA checkpoint** (interpolate toward MQA).
- **Representative open models (near-universal):** LLaMA 2/3, Mistral, Qwen2/3 (dense + MoE variants), Gemma 2/3, Phi-3/4, OLMo, DeepSeek-V3 (uses MLA instead, see §3), etc. Effectively the *de facto* standard attention head format since 2023.

### 3. MLA — Multi-Head Latent Attention（MLA）

DeepSeek 的招牌创新，比 GQA 更激进：**干脆不缓存完整 K、V，而是把它们压成一个低秩 latent 向量缓存下来，用时再"解压"**。GQA 是"少存几份完整 KV"，MLA 是"每份 KV 都压成极小的一团"。配合一个巧妙的"解耦 RoPE"技巧，MLA 把 KV cache 压到 MHA 的 **~1.8%（降 98%+）**，这是 DeepSeek 能用很低成本服务大模型的关键。

> **核心概念（你的理解完全正确）**：MLA 的本质不是"少存几份 KV"（那是 GQA），而是 **先把 hidden state 压进一个很小的低秩 latent 空间，再让 Q/K/V 都在这个压缩空间里算注意力**。标准 MHA 的 attention 在 $n_h\cdot d_h = 16384$ 维的完整 KV 空间里算；MLA 把 KV 压到 $d_c = 512$ 维的 latent，缓存的也就只剩下这个 512 维向量，所以 KV cache 才降了 98%+。
> 补一个容易漏的微妙点：**Q 和 KV 各自有独立的下投影**——KV 压到 $d_c=512$（这个才被缓存），Q 压到 $d_c'=1536$，两者不是同一个 latent。推理时靠 **matrix absorption trick**（$W^{UK}$ 吸收进 $W^Q$、$W^{UV}$ 吸收进 $W^O$）把上投影矩阵"折叠"掉，于是注意力分数 $q\cdot k$ 实际上就直接在 512 维的内容 latent 上计算，与原版注意力数学等价、但完整 16384 维 KV 从头到尾都不出现。再叠加解耦 RoPE（$k_t^R$，64 维）补位置信息。

- DeepSeek-V2 (2024), *"DeepSeek-V2: A Strong, Economical, and Efficient MoE Language Model"*.
- **Low-rank joint KV compression — and attention is computed *in* this latent, not merely cached smaller** (the core idea):
  - **KV latent**: $c_t^{KV} = W^{DKV} h_t \in \mathbb{R}^{d_c}$ ($d_c=512 \ll 16384$); **only $c_t^{KV}$ is cached** (plus decoupled RoPE key $k_t^R \in \mathbb{R}^{d_h^R}$, $d_h^R=64$).
  - **Query latent (also compressed!)**: $c_t^Q = W^{DQ} h_t \in \mathbb{R}^{d_c'}$ ($d_c'=1536$) — Q is *separately* down-projected; it is **not** the same latent as KV.
  - The score $q\cdot k$ is taken between the query reconstructed from $c_t^Q$ and KV reconstructed from $c_t^{KV}$. Through **matrix absorption** ($W^{UK}\to W^Q$, $W^{UV}\to W^O$), the up-projections fold away, so the score equals standard attention evaluated in the **$d_c=512$ content latent** — the full 16384-dim KV never materializes at inference.
  - Up-projection (what absorption removes): $k_t^C = W^{UK} c_t^{KV}$, $v_t^C = W^{UV} c_t^{KV}$.
- **Decoupled RoPE**: standard RoPE is incompatible with low-rank KV (position-dependent rotation blocks the absorption). MLA splits Q/K into a *content* part (no RoPE) and a *position* part ($q_t^R = \text{RoPE}(W^{QR} c_t^Q)$, $k_t^R = \text{RoPE}(W^{KR} h_t)$) and concatenates; only the tiny $k_t^R$ is cached for position.
- **Cache math (DeepSeek-V2):** per-token cache = $d_c + d_h^R$ = 512 + 64 = **576** numbers vs MHA's $2\cdot128\cdot128 = 32768$ → **98.2% reduction**.
- Hyperparams: $n_h=128,\; d_h=128,\; d_c=512,\; d_c'=1536$ (query latent), $d_h^R=64$.
- **Representative open models:** DeepSeek-V2, DeepSeek-V3, DeepSeek-R1, and (as the latent backbone of) DeepSeek-V4's dual-pool attention. Also adopted by GIM/Grok-style and several Chinese open models. MLA is the single most impactful attention innovation for *inference economics* in the 2024–2026 window.

---

## 二、稀疏 / 局部化路线：KV 照常存，但每个 token 只看一部分位置

这条线和压缩 KV cache 是正交的——它不改"存什么"，而是改"算什么"：让注意力**结构性地跳过**大部分历史位置。

### 4. SWA — Sliding Window Attention（滑动窗口注意力）

最简单也最早被工业界采用的稀疏模式：每个 token 只和**最近 $w$ 个 token**做注意力（比如 4096），超出窗口的直接不看。窗口内是完整注意力，窗口外完全丢弃。它把attention的显存从"随 $L$ 线性增长"变成"恒定窗口大小"，是 Mistral 把 7B 模型跑得又快又长的关键。代价是**纯窗口注意力没有全局感受野**，所以工业界几乎总是把它和全局层混用（见 §5、§6）。

- Rooted in Longformer / Blockwise Sparse Attention (Beltagy et al., 2020); brought into mainstream open LLMs by **Mistral 7B** (2023, Jiang et al.) with a **4096-token sliding window**.
- Each token attends only to positions $[t-w, t]$ (causal). KV cache grows with $w$, **not $L$** → constant memory per layer.
- Pure SWA loses global receptive field; therefore almost always **hybridized** with periodic global/full-attention layers (the pattern that §5/§6 generalize).
- **Representative open models:** Mistral 7B/8x7B, Mixtral, and as the *local* component inside Gemma 2 (§5), Llama 4 (§5), NSA (§6), DeepSeek-V4 CSA/HCA (§6).

### 5. 交替局部-全局注意力（Interleaved Local–Global）

SWA 的升级版：与其全模型都是窗口，不如**隔层切换**——一层只看局部窗口（省），下一层看全局（保全局信息）。这样每两层就有一次全局聚合，既压住了显存又没丢长程能力。Gemma 2 和 Llama 4 是两种不同做法的代表。

**Gemma 2 (Google, 2024, arXiv:2408.00118).**
- **Alternating local/global every other layer**: SWA layers use window = **4096**, global layers use span = **8192**.
- Full global receptive field is restored **every two layers**, so quadratic cost is halved while fidelity is preserved.
- Paired with **GQA (num_groups=2)**, **logit soft-capping** ($\text{soft\_cap} \cdot \tanh(\text{logits}/\text{soft\_cap})$, 50 for attn / 30 for final), and **post-norm + pre-norm RMSNorm** for stability.
- **Representative open models:** Gemma 2 (2B/9B/27B), Gemma 3.

**Llama 4 iRoPE (Meta, 2025).**
- **iRoPE = "interleaved" RoPE**: ~3 of every 4 layers use **chunked RoPE attention** (fixed chunk = 8192 tokens, attend within chunk + global tokens); **every 4th layer is a NoPE layer** (No Positional Encoding) running **full causal attention over the entire sequence**.
- **NoPE layers** have no position bias → they generalize to lengths far beyond training (act as global context aggregators). This is what lets **Scout reach 10M context from a 256K training length**.
- **Inference-time temperature scaling** on the NoPE-layer softmax prevents attention-entropy collapse at very long context (applied post-training, no retrain).
- Scout also adds **QK RMSNorm** on RoPE layers for stability.
- **Representative open models:** Llama 4 Scout (109B/17B active, 16 experts, top-1), Llama 4 Maverick (400B/17B active, 128 experts; MoE and dense layers interleaved so experts apply in ~half the layers).

### 6. NSA / DSA / CSA+HCA — 原生可训练的分层稀疏注意力（DeepSeek 线）

这是稀疏路线里最"讲究"的一支，也是 DeepSeek 这两年的主线。核心洞察：**事后把全注意力稀疏化会掉点，但如果从预训练起就让模型自己学"看哪里"，稀疏注意力能和全注意力一样好甚至更好**。它把注意力拆成三条并行分支——粗粒度压缩看全局、细粒度选择挑重点块、滑动窗口看局部——用可学习的门控融合。这个设计是硬件对齐的（按块连续访存），所以能真正加速。NSA 拿下了 ACL 2025 最佳论文，并一路演化：V3.2 叫 DSA，V4 叫 CSA+HCA。

**NSA: Native Sparse Attention (DeepSeek, arXiv:2502.11089, ACL 2025 Best Paper).**
- Replaces full attention with **three parallel, natively-trainable branches** fused by a learned gate (MLP + sigmoid):
  1. **Compression**: coarse KV over block summaries (learnable MLP pooling of length-$l$ blocks, stride $d<l$ to avoid fragmentation) → global coarse context.
  2. **Selection**: reuses compression-attention scores as **block importance** (zero-cost), aggregates scores across a **GQA group** (shared selection), picks **top-n fine-grained blocks** → local precision. Blockwise (not token-level) selection keeps memory contiguous → FlashAttention-compatible.
  3. **Sliding window**: isolated recent $w$ tokens, kept separate so local patterns don't shortcut the other branches.
- **Key property**: end-to-end trainable (gradients flow through selection) and **hardware-aligned** (contiguous block access, GQA-shared KV) → real wall-clock speedup, not just FLOP reduction. At 64K, beats full attention on loss + reasoning while ~10× faster decode.
- **Evolution**: NSA → **DSA (DeepSeek Sparse Attention, V3.2)** → **CSA + HCA (V4)**:
  - **CSA (Compressed Sparse Attention)**: compress KV 4× via softmax-gated pooling with learned positional bias; a **lightning indexer** (FP4 ReLU-scored dot product) selects top-k compressed blocks per query; a **128-token SWA branch** covers recent raw tokens.
  - **HCA (Heavily Compressed Attention)**: compress 128× (disjoint 128-token windows), then **dense** attention over all compressed entries (no sparse selection). Cheaper per-entry constant.
  - V4 interleaves CSA/HCA across 61 layers (layers 0–1 HCA, then alternate), served by a **unified dual-pool MLA operator**. Net: ~27% of V3.2 FLOPs at 1M context, KV cache ~2% of equivalent bfloat16 GQA.
- **Representative open models:** DeepSeek-V3.2 (DSA), DeepSeek-V4 (CSA+HCA); the architectural pattern also influenced GLM-5 (DSA-style) and was validated across model families.

---

## 三、线性 / 循环注意力路线：把 $O(n^2)$ 彻底改成 $O(n)$

前面两条线都还要存每 token 的 KV。这一支更激进：**干脆不要 softmax 全注意力，改成线性递归**——用固定大小的"状态矩阵"记住过去，每个新 token 只读写这个状态，复杂度变成 $O(n)$、显存恒定。难点在于固定状态会"记忆冲突"，所以近年突破都在"怎么更聪明地写/忘这个状态"。

### 7. Gated DeltaNet — Gated Delta Rule（门控 Delta 规则）

线性注意力的老问题是"只会往里塞、不会擦"：旧信息越积越乱，长程检索弱。Gated DeltaNet（NVIDIA，ICLR 2025）把两个互补机制合在一起：**delta rule**（写入前先把"这个 key 旧的值"减掉再写新值，精准纠错）+ **数据依赖的遗忘门**（一个标量门控，让模型在上下文切换时快速清空记忆）。它成了 2025 年混合架构里最常被采用的线性注意力组件。

- Yang, Kautz, Hatamizadeh (NVIDIA), *"Gated Delta Networks: Improving Mamba2 with Delta Rule"*, ICLR 2025, arXiv:2412.06464.
- **Linear attention recurrence** with matrix-valued state $S_t \in \mathbb{R}^{d_v\times d_k}$:
  - Pure linear attention: $S_t = S_{t-1} + v_t k_t^T$, $o_t = S_t q_t$ (no softmax → associative → $O(n)$ trainable via chunkwise parallel form).
  - **Delta rule** (error-correction write): $S_t = S_{t-1} + \beta_t (v_t - S_{t-1} k_t) k_t^T$ = erases the old value stored at key $k_t$ before writing the new one (fixes memory collisions of naive accumulation).
  - **Gated (forget) decay** (from Mamba2/GLA): $S_t = \alpha_t \cdot [S_{t-1}(I - \beta_t k_t k_t^T) + \beta_t v_t k_t^T]$, $\alpha_t \in (0,1)$ data-dependent → rapid forgetting on context switch.
- **Hybrid**: combine Gated DeltaNet layers with SWA or Mamba2 layers; GLA uses a diagonal (fine-grained) gate, Mamba2 a single scalar gate, Gated DeltaNet adds the delta-rule targeted write on top.
- **Representative open models:** **Qwen3-Next** uses **Gated DeltaNet** at a **3:1 ratio** (3 Gated DeltaNet layers + 1 Gated Attention/GQA layer per block); the Qwen3.5/3.6 generation makes this the flagship standard (see also §8 Kimi, which refines the gate).

### 8. KDA — Kimi Delta Attention（Kimi Delta 注意力）

Kimi（月之暗面）在 Gated DeltaNet 基础上的精细化版本，用在 K2/K3 上。关键改进有两点：① 把 Gated DeltaNet 的**单一标量遗忘门**换成**逐通道（向量）遗忘门**——每个记忆通道按自己的节奏忘，表达力更强；② 给衰减率加了一个**下界**，这样所有计算都能走统一快速路径（否则某些值会无限增长、被迫走慢路径）。KDA 完全**去掉了位置编码**，靠门控/衰减隐式编码位置，从而无需 RoPE 重缩放就能外推到 1M 上下文。它和 Gated MLA 按 **3:1** 混合——作者自己调侃"如果线性注意力真打赢了全注意力，比例就该是 4:0"，保留 1/4 的 MLA 正是为了补回线性注意力的全局检索短板。

- Introduced in the **Kimi Linear** paper (Moonshot AI, Oct 2025); scaled in **Kimi K3** (2026, 2.78T total / 104.2B active, 93 layers).
- **Delta rule + per-channel (vector) forget gate**: $\alpha_t$ becomes a vector so each channel in the recurrent state $S_t$ (shape $d_k \times d_v$) forgets at its own learned rate — finer memory control than Mamba2's scalar or GLA's diagonal gate.
- **Decay floor** (lower bound on forget rate): prevents intermediate values from growing unbounded → all compute stays on the fast Tensor-Core path (one small hyperparameter unblocks the whole compute chain).
- **No explicit positional encoding (NoPE)**: position is encoded implicitly through KDA gating/decay → extrapolates to 1M tokens without RoPE rescaling.
- **Hybrid 3:1**: 3 KDA layers + 1 **Gated MLA** layer (compressed latent KV, restores exact global retrieval), repeated; 93 layers → **69 KDA + 24 MLA**, plus a final MLA layer. Periodic MLA = "confession and achievement" — recovers global lookup that a fixed-size state misses.
- **Serving win**: 3 of 4 attention layers carry a **constant-size recurrent state** instead of a growing KV cache → up to **6.3× faster decode and 75% less KV-cache memory at 1M context**; enables state-aware prefix caching.
- **Representative open models:** Kimi K2 (pure MLA + FP8), Kimi K3 (hybrid KDA+MLA + MXFP4), Kimi Linear (48B proof-of-concept).

### 9. Lightning Attention（MiniMax-01）

MiniMax 在 2025 年初把线性注意力**第一次推到商用级规模**的尝试。它本质是"分块并行的线性注意力"：块内用类 softmax 的精确计算、块间用线性递归累加状态，从而兼顾训练并行和推理线性复杂度。和 Kimi 一样，它也不迷信纯线性——每 8 层里保留 1 层传统 softmax 注意力（带 GQA）来补回检索能力。最终 456B 总参数、45.9B 激活、训练 1M / 推理外推 4M 上下文。

- MiniMax-01 (Jan 2025), *"Scaling Foundation Models with Lightning Attention"*; first **production-scale** linear-attention LLM.
- **Chunkwise linear attention**: rewrite $O = \text{Norm}((QK^T)V)$ as $O = \text{Norm}(Q (K^T V))$; for causal attention use recurrence $kv_t = kv_{t-1} + k_t v_t^T$, $o_t = Q_t kv_t$. Split into **intra-chunk** (causal, precise) and **inter-chunk** (linear recurrence over accumulated $kv$) for hardware-efficient parallel training.
- **Hybrid 7:1**: 7 Lightning-Attention layers + 1 **softmax-attention** layer (with **GQA, group size 8**) every 8th layer, across **80 layers** → recovers retrieval / in-context learning that pure linear attention lacks.
- Spec: 456B total / 45.9B active (32 experts, top-2), hidden 6144, 64 heads, head dim 128, RoPE on half head dim. Context: **1M train / 4M inference**; LASP+ + varlen ring attention for long-sequence parallelism; MFU > 75% on H20.
- **Representative open models:** MiniMax-Text-01, MiniMax-VL-01 (vision), MiniMax-M1 (reasoning).

---

## 四、元趋势：混合堆叠（Hybrid Stacking）

把上面三条线放在一起看，会发现一个清楚的共识：**没有哪种单一注意力是万能的，真正的进步在于"按层混合"**。几乎每一个 2025–2026 的新开源模型都是某种 hybrid：

- 线性/循环层负责"便宜地扫过序列"（$O(n)$、恒定显存）；
- 周期性保留少量 softmax/全局层负责"精确的长程检索与全局聚合"；
- 稀疏/窗口层负责"局部精度且省显存"；
- MLA 这类压缩负责"把必须存的 KV 压到极小"。

**The interleaving recipes seen in production open models:**
- **Qwen3-Next / Qwen3.5 / 3.6**: 3 × Gated DeltaNet + 1 × Gated Attention (GQA), 3:1.
- **Kimi K3**: 3 × KDA + 1 × Gated MLA, 3:1 (+ final MLA).
- **MiniMax-01**: 7 × Lightning Attention + 1 × softmax (GQA), 7:1.
- **Gemma 2/3**: alternate SWA(4096) + global(8192), 1:1.
- **Llama 4**: 3 × chunked RoPE + 1 × NoPE global, 3:1 (+ temperature scaling).
- **DeepSeek-V4**: interleave CSA + HCA (dual-pool MLA operator), layered compression ratios.

> 一句话：**"attention 之战"在 2025–2026 已经从"换掉 softmax"变成"怎么把 softmax / 稀疏 / 线性三种机制按层编排"**。纯线性注意力单独用会掉检索能力（MiniMax 也承认），所以 3:1 / 7:1 的"线性为主、全局为辅"成了事实标准。

---

## 五、速查表 & 默认配方

| Mechanism | Paper / Year | Core idea | KV cost / complexity | Representative open models |
|---|---|---|---|---|
| **MHA** (baseline) | Vaswani 2017 | per-head Q/K/V, full softmax | $2\cdot n_h\cdot d_h\cdot L$, $O(n^2)$ | original LLaMA-1, Gemma-1 7B |
| **Flash Attention** (kernel) | Dao 2022 (NeurIPS) | IO-aware exact attn via tiling + online softmax; same math as MHA | exact, $O(n^2)$ compute / **$O(n)$ HBM** | (kernel — underlies all softmax-based rows above) |
| **MQA** | Shazeer 2019 / PaLM 2022 | all heads share 1 K/V | $2\cdot d_h\cdot L$, $O(n^2)$ | Falcon |
| **GQA** | Ainslie 2023 | groups of heads share K/V | $2\cdot h_{kv}\cdot d_h\cdot L$, $O(n^2)$ | **LLaMA2/3, Qwen2/3, Mistral, Gemma, Phi, OLMo** |
| **MLA** | DeepSeek-V2 2024 | low-rank KV latent + decoupled RoPE | ~576/token (**−98%**), $O(n^2)$ but tiny cache | DeepSeek-V2/V3/R1/V4 |
| **SWA** | Longformer / Mistral 2023 | attend only to recent $w$ tokens | $2\cdot h_{kv}\cdot d_h\cdot w$ (constant) | Mistral, Mixtral |
| **Interleaved Local-Global** | Gemma 2 2024 / Llama 4 2025 | alternate window / global (or NoPE) | windowed + periodic global | Gemma 2/3, Llama 4 |
| **NSA / DSA / CSA+HCA** | DeepSeek 2025 (ACL Best) | compressed + selected + window branches, trainable | block-sparse, hardware-aligned | DeepSeek-V3.2 (DSA), V4 (CSA+HCA) |
| **Gated DeltaNet** | NVIDIA ICLR 2025 | linear attn + delta rule + forget gate | **fixed-size state, $O(n)$** | Qwen3-Next (3:1) |
| **KDA** | Kimi 2025/2026 | delta rule + per-channel forget, NoPE | fixed-size state, $O(n)$ | Kimi K2/K3 |
| **Lightning Attention** | MiniMax 2025 | chunkwise parallel linear attn | fixed-size state, $O(n)$ | MiniMax-01 (7:1) |

**给"今天要设计一个开源模型"的默认注意力配方（2025–2026 共识）：**
1. **头部格式用 GQA**（或 MLA 若你追求极致 KV 压缩）；
2. **必上某种稀疏/窗口**做长上下文（SWA 或 NSA/DSA 式压缩+选择）；
3. **若上线性注意力，按 3:1~7:1 混回 softmax/MLA 全局层**，不要纯线性；
4. **RoPE 仍主流**，但 Llama 4 的 NoPE 全局层 + 温度缩放、Kimi 的 NoPE 线性是两条值得关注的去位置编码路线；
5. **训练原生稀疏/线性**（NSA、Gated DeltaNet 都证明：从预训练起就稀疏/线性，质量不输全注意力，且真能加速）。
