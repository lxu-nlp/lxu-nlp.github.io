---
layout: post
title: "Notes on Training"
date: 2026-06-15
categories: [Notes]
tags: [optimization, ML]
math: true
---

## 基础知识：训练显存拆解与 ZeRO / FSDP 分片

先建立直觉，再谈并行与长上下文。

### 1. 训练一个 8B 模型，显存花在哪

以 bf16 混合精度 + AdamW 训练 ~8B 参数模型（Llama-3-8B 量级）为例，显存分四大块：

| 组成 | 精度 / 每参数字节 | 8B 模型估算 |
|---|---|---|
| Parameters（模型权重） | bf16，2 B | ~16 GB |
| Gradients | bf16，2 B | ~16 GB |
| Optimizer states（AdamW: fp32 master weight + m + v） | fp32，12 B | ~96 GB |
| **Model state 小计** | **16 B/param** | **~128 GB** |
| Activations | 取决于 batch×seq，见下 | ~2–18 GB（视 checkpointing） |

前三项是 ZeRO 论文里的 **model state**，合计 ≈ **16 bytes/param**（经典结论）：每个参数在 Adam 混合精度下要 2(B 权重) + 2(B 梯度) + 12(B 优化器状态) = 16 B。8B × 16 ≈ 128 GB —— 单卡（80 GB A100/H100）放不下，必须分片或 offload。

**Activations** 不随参数固定，而随 micro-batch × 序列长度增长。粗略量级：8B（H≈4096, L≈32）在 S=4096, B=1 时约 2–18 GB，开 activation checkpointing 后可压到 ~2 GB 量级。它正是后文 long-context 显存的主要变量。

### 2. ZeRO / FSDP 各 stage 切什么

DeepSpeed 的 ZeRO（Zero Redundancy Optimizer）把 "model state" 三件套（params / grads / optimizer states）按 stage 逐步分片；**FSDP 等价于 ZeRO-3**，并允许控制分片粒度。

| Stage | 分片对象 | 每 rank 常驻 model state | 通信 |
|---|---|---|---|
| Baseline（DDP） | 无，全复制 | 100%（16 B/param） | 1× all-reduce/step |
| **ZeRO-1** | 仅 optimizer states | 12/N + 4 | 更新时 all-gather opt states |
| **ZeRO-2** | optimizer states + gradients | 12/N + 2(params) | all-gather opt + reduce-scatter grads |
| **ZeRO-3 / FSDP** | params + grads + optimizer states（全部） | 16/N | 每层 all-gather params + reduce-scatter grads |

- **ZeRO-1**：只分片 Adam 的 m/v/fp32 master，params 与 grads 仍全量复制 → 省下最大头（优化器 12 B），权重仍在每卡。
- **ZeRO-2**：再分片 gradients，反向结束后 grads 也只留本 rank 那一 shard。params 仍复制。
- **ZeRO-3（= FSDP）**：params 也分片，三件套全 1/N。显存最低，但 forward/backward 每层都要临时 all-gather 回完整权重。

FSDP2（torch.distributed 现代默认）与 FSDP1 区别在底层实现：FSDP2 按 **per-parameter** 分片、不做 flatten 打包，粒度更自然；FSDP1 用 flat param 分组。两者分片语义一致。

### 3. forward 时如何从分片块得到正确 output

关键：**分片只在「休息态」存在，计算前必现完整权重**。FSDP 对每个 FSDP unit（通常一个 transformer block）这样工作：

1. **Forward pre-hook**：对该 unit 的所有分片参数做 **all-gather** —— 各 rank 把持有的 1/N shard 拼回完整张量，每卡都拿到全量权重。
2. 用完整权重正常计算该层 forward，得到正确 output（数学上与单卡完全一致）。
3. **Forward hook**：计算完立即释放 gathered 的完整张量，每卡只保留自己那 1/N shard。
4. **Backward**：再次 all-gather 该 unit 参数算反向，再用 **reduce-scatter** 把梯度 averaging 并只保留本 rank 对应 shard。

于是：**任意时刻每 rank 常驻 1/N 的 params/grads/opt states；完整权重仅在每层 forward+backward 的瞬间存在**。output 正确性的来源就是「计算前 all-gather 出完整权重」—— 分片不影响单层数学，只是把峰值显存从「全模型」限制到「单个 unit」。

（DeepSpeed ZeRO-3 同理：forward 前 `param.all_gather()` 拼回，计算后 `partition()`；并支持 ZeRO-Offload 把分片状态 offload 到 CPU 进一步省 GPU 显存。）

---

## 训练显存与并行（Training memory and parallelism）

选 DDP / FSDP 时的参考要点，以及如何推算 long-context 显存。

### 显存随什么增长（What scales with what）

对一层 transformer（序列长度 S，hidden size H），activation 拆成两项：

| Term | Source | Scaling |
|---|---|---|
| Linear-in-S activations | residual stream, MLP, layer norms | `O(S · H)` per layer |
| Quadratic-in-S attention scores | `Q @ K^T` 与 softmax 概率（materialized 时才有） | `O(S²)` per head |

无优化时总计 ≈ `L · (S·H + S²)`。

### activation checkpointing 换来什么

只在 **block 边界** 存 activation，backward 时重算中间量。于是：
- 层数因子 `L` 塌缩为 `N_checkpoints`（通常每层一个边界）；
- 但 **单层内的项不变** —— 重算时该 block 的 `O(S²)` attention 仍会短暂 materialize；
- 净效果：peak memory ≈ `N_checkpoints · S · H + max_block_activations`，**对 S 线性，S² 仍潜伏在被重算的 block 内**。

一句话：**checkpointing fixes layer-count scaling, not sequence-length scaling.**

### 真正打破 S² 缩放的方法

| Technique | Kills which term |
|---|---|
| **FlashAttention** | `O(S²) → O(S)`，从不 materialize score 矩阵 |
| **Selective activation checkpointing** | 重算 attention、保留廉价 activation，压低常数 |
| **Sequence parallelism (SP)** | 沿 S 轴切分 `S·H`：`O(S·H / world_size)` |
| **Context / Ring parallelism (CP)** | 沿 S 轴切分 attention 本身：`O(S·H / world_size)` |

long-context 实用配方：**FlashAttention + activation checkpointing** → `O(N_layers · S · H)`（对 S 仍线性，但常数小）。要真正打破线性缩放需上 SP 或 CP。

### 注意：FSDP 切参数，不切 activation

常见误解。FSDP1 / FSDP2 都只 shard **parameters, gradients, optimizer states**，永不碰 activations。切 DDP ↔ FSDP 不影响 activation 显存。管 activation 的正确工具是 checkpointing、FlashAttention，以及（超长 context 时）SP/CP/TP。

### DDP vs FSDP — 何时用哪个

| | DDP | FSDP |
|---|---|---|
| Forward | 无通信 | 每个 FSDP unit all-gather 参数 |
| Backward | all-reduce grads（与计算 overlap） | 再次 all-gather + reduce-scatter grads |
| 每 rank 显存 | 完整 params + grads + opt states | sharded |

DDP 每步只通信一次（gradient all-reduce），几乎与 backward 完全 overlap；FSDP 在 forward/backward 各 all-gather 一次再加 reduce-scatter，同等 eff. batch 下通常慢 5–20%。**If memory isn't binding, DDP wins**（更快、更简单、更可调试：每 rank 都有完整模型）。

### FSDP Quick Grasp

「哪些参数共享一次 all-gather」：太粗（整模型一次）→ 峰值等于 DDP；太细（逐参数）→ collective 太小低效。**折中是 per transformer block**：块内参数共享一次 gather，块后释放，峰值被限制在单块全精度参数内，且 gather 够大以摊薄通信开销。

**Lifecycle: Leaf vs. Root.** FSDP 用 `forward_pre_hook` 触发 all-gather（分配全精度权重）、`forward_hook` 释放。位置决定一切：
- **Leaf（如 transformer block）**：块运行前分配、运行后立即释放 → **Ephemeral**，显存只在该块计算的瞬间尖峰。
- **Root（顶层模型，含 Embeddings & LM Head）**：整个 forward 开始分配、结束才释放 → **Persistent**，全程常驻显存。

**Memory Trap：** Root 被迫常驻，导致 **LM Head / Embeddings 的非分片权重全程不被释放**，叠加在顺序执行的 Leaf 临时尖峰之下。vocab 极大时，Root 管理的 LM Head 会成为 peak VRAM 的沉默瓶颈，抵消 block 级分片收益。

**`_no_split_modules`：** accelerate 的 `TRANSFORMER_BASED_WRAP` 据此知道哪些 leaf class 该包成 FSDP unit。每个 HF 模型暴露 `_no_split_modules: List[str]`（如 `['Qwen3DecoderLayer']`），策略遍历模块树，对类名在表中的实例调用 `fully_shard(module)` 并停止下钻，使其成为独立 FSDP unit。

---

## Padding

自回归 LLM 的 SFT：**right for training, left for generation**。

RoPE 模型（Llama / Qwen / Mistral…）下 train/inference 不一致为何无害：
1. **训练右 padding** 是 TRL `SFTTrainer` / Axolotl / LLaMA-Factory 的业界约定；HF + FA2 的 unpad/repad 路径对右 padding 更成熟。
2. **attention mask**（softmax 前清零）让 pad 位置对任意侧真实 token 都不可见。RoPE 只编码真实 token 间 **相对距离**，两种 padding 下距离一致 → 真实 token 看到的张量运算相同。
3. **`position_ids` 不是对齐两阶段的依据。** `generate()` 在推理时依 attention mask 重建 `position_ids`，训练时 HF Trainer 只给朴素 `arange(0, seq_len)`。若模型依赖绝对位置编码，两边绝对位置会冲突 —— 但 RoPE 不依赖，这正是它能工作的原因。对齐发生在 **RoPE 侧，而非 dynamic position-id 侧**（绝对位置模型如 GPT-2 / BERT 用左 padding 训练才会坏）。

---

## 训练效率：Padding Removal

### 1. 问题
同 batch 内序列长度不一，标准矩形 batch 把每条 pad 到最长，例如 `[128,256,512,2048]` 变 `4×2048`。PAD 不免费 —— GPU 仍对它们算 attention/FFN 再 mask。长度倾斜数据（指令微调、RL 生成）下，**half to three-quarters 的算力耗在 PAD 上**。移除 padding 后算力随 **real tokens** 而非 padded tokens 缩放，典型收益：吞吐 1.5–2×、峰值显存 ~20%↓。

### 2. 思路
FlashAttention 2 的 *variable-length* kernel 接收 **flat token 流 + segment 边界**，按 block-diagonal 算 attention（每段只 attend 自身），PAD 从不存在。

HF transformers 在模型调用时自动启用该 kernel，需同时满足：**`attention_mask is None` 且 `position_ids` 在每个 segment 边界归 0**。任一不满足就静默回退到矩形 kernel —— 忘掉任一条件，加速全没且不报错。

### 3. 两条路线（核心分歧：flat 流在 pipeline 何处生成）

- **Route A — Data-side fix（HuggingFace 方式）**：专用 collator 在 **数据加载时** 把变长样本拼成一条 flat 行，正确构造 `position_ids`（每段边界归 0），并在样本间插入 ignore-marker 防止 NTP label 泄漏到下一样本。到 forward 时 padding 已不存在，下游 loss 直接用标准 `ignore_index=-100` 的 cross-entropy，无需特殊处理。
  - 性质：collator 之后 `B` 维度消失，forward 看到 `(1, total_tokens)`；下游代码（forward/loss/optim）完全 oblivious，只少一个 `attention_mask`。显存最优（pad activation 从不 materialize）。
  - 代价：per-sample 索引失效（无 sample 轴），普通 SFT 用不到，但 RL/多任务需 per-sample reduction 时会踩坑。
  - **现代 HF / TRL 中仅一行改动**，其余代码不动。

- **Route B — Forward-side fix（verl 方式）**：collator 仍给普通 padded `(B,T)`；flatten 发生在 **forward 内部**，三步：① Unpad → flat `(1, total_real)` + `indices`（记录原 `(row,col)`）；② 同 Route A 跑模型（`attention_mask=None`）；③ Repad → 用 `indices` 把输出 scatter 回 `(B,T)`，pad 位填 0。
  - 性质：padding removal 是 forward 内的局部优化，collator/dataset/optimizer 看到的形状不变（drop-in）；可按 micro-batch / 训练阶段 / 仅 actor 选择性启用；**能组合 SP**（flat 流可切片跨 SP rank，再 gather+repad）—— 这正是 verl 选它的原因；也能组合 token-budget micro-batching。
  - 当训练循环有 HF collator 表达不了的结构（多阶段 RL、SP、输出需矩形张量的 forward）时选它。

### 4. 对比

| Question | HF route | verl route |
|---|---|---|
| flatten 在哪 | collator | forward 内 |
| forward 看到形状 | 一条 flat 行 | 短暂 flat 的 `(B,T)` |
| loss 看到形状 | flat | 回到 `(B,T)` |
| 改动量 | ~1 行 | ~30 行 |
| 组合 SP？ | 否 | 是 |
| 组合自定义 RL micro-batching？ | 否 | 是 |
| 标准 SFT？ | **是** | overkill |
| RL post-training？ | 否 | **是** |

---

## 训练效率：Dynamic Batching

### 1. 问题
真实数据长度差异可达 40×+。若按固定 **序列数** 组 batch：全短 batch 浪费显存；全长 batch OOM；分布式下还引入 rank 等待。为安全只能按最坏情况定 batch size → 多数时候 GPU 远低于显存预算。RL post-training 更甚（每 prompt 多条 rollout，长度差异大）。

**修复：stop counting sequences, start counting tokens.**

### 2. 两个核心思想
- **Token budget 而非序列数：** 设 `max_tokens_per_batch = M`，能塞多少序列塞多少。显存只由 `M` 决定，step 可预测。
- **Balanced partitioning：** 把变长序列分到 `K` 组（= `K` 个 micro-batch / GPU），使各组总 token 尽量相等。最慢组决定 wall-clock，均衡即让最慢组尽可能快。
- 两者组合：tokens-not-sequences 修显存方差，balanced partitioning 修时间方差。

### 3. 常见实现（按代价分层）
- **Tier 1 — Length-grouped sampling：** 按长度排序切块，同 batch 长度相近，无 padding removal 也能减浪费。HF 一行 flag。代价：batch 不再是真随机（软长度课程）。用于简单单卡 SFT。
- **Tier 2 — Packing-based collators：** collator 阶段把多条短样本拼成一行并标 segment 边界，同时去 padding + 打包短样本。多数常规 SFT 这一改即解。
- **Tier 3 — Dynamic token-budget micro-batching（verl 风格）：** step 时、数据已路由到各卡后，**动态**决定 micro-batch 组成，单位 token budget，partition 依本步实际长度重算。同时处理 Scope A（跨卡重排）与 Scope B（卡内切分）。RL post-training 用此，因数据组成 step-to-step 不可预测。
- **Tier 4 — Pre-bucketed dataloaders：** 预处理时按长度分桶，每步取一桶。kernel 效率最高，预处理更复杂。用于超大规模 pretraining。

### 4. 划分算法
给定长度与分区数 `K`，最小化最重/最轻分区差：
- **Greedy：** 长度降序，每项分给当前最轻分区。快、通常离最优几个百分点。
- **Karmarkar-Karp：** 迭代配对最重两项合并，方差大、分区少时明显更优。verl 用它；HF length-grouped 用更简单 bucket 方案。实际（少量分区、数百项）算法升级对最坏 spread 约 5–15% 收益。
