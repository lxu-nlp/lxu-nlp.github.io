---
layout: post
title: "Notes on Optimization"
date: 2023-06-29
categories: [Notes]
tags: [optimization, ML]
math: true
---


[Optimizer](https://www.cnblogs.com/guoyaohua/p/8542554.html):
* Momentum: add previous direction to reduce oscillation
* Adagrad/RMSprop: normalize by previous norm per parameter, to achieve adaptive LR per parameter (less frequent has higher weight, good for sparse)
* Adam: combine Adagrad & Momentum

LayerNorm & BatchNorm
* BatchNorm: normalize on each feature across batch; normalization is sample-dependent, prone to outliners for small batch size; harder for distributed models
* LayerNorm: normalize on single-input features; each normalization is independent

[Weight decay](https://benihime91.github.io/blog/machinelearning/deeplearning/python3.x/tensorflow2.x/2020/10/08/adamW.html):
* Instead of modify loss (e.g. L2 reg), modify update step: w = w - learning_rate * gradients - learning_rate * lamdba * w
* Weight decay != L2 Reg; only equivalent under SGD, and become different when adding momentum etc.

Optimization with normal Adam:
* Model state (FIXED): 
  * Optimizer state: 8p bytes (momentum, norm/variance)
  * Gradients: 4p
  * Parameters: 4p
* Forward activations (DYNAMIC): input-dependent (seq length, batch size)
* Buffers, fragments

Mixed precision FP16/32 training with Adam:
* Model state: 
  * Optimizer state: still 8p bytes
  * Gradients: 2p scaled to 4p
  * Parameters: 6p (4+2, actually more fixed memory)
* Activations (forward): in FP16 (saving more memory and faster for large input)
* Buffers, fragments

Gradient checkpointing:
* Backprop starts with loss, so only the last layer activations are needed to start backprop
* For normal training, forward activations are kept to enable backprop at each layer directly, but using major GPU memory
* With checkpointing, only keeping activations at certain layers and RECOMPUTE the other in-between activations on-the-fly

Data parallelism (DP):
* Partition batch
* Replicate model with memory redundancy

Model parallelism (MP):
* Partition model
* Replicate activations with more communications: each model partition needs full layer activation for forward and backward

Deepspeed (ZeRO): combining DP and MP
* Observations to leverage: for large model, computation cost is so large that it can hide communication cost; so ZeRO partitions activations (or even to CPU) and all_gather on-the-fly
* Also see PyTorch [FSDP](https://pytorch.org/blog/introducing-pytorch-fully-sharded-data-parallel-api/)

memory_efficient_attention (xformers):
* "Self-Attention Does Not Need O(n^2) Attention": for a query, compute in-sequence for k&v in O(1) memory
* Implementation: compute by chunk to strike a balance between parallelism and memory: O(sqrt(n)) with slight time increase
* Implement in c so to reduce runtime as well

flash_attention:
* "FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness"
* Compute softmax by blocks in fast SRAM then combine results: 1) reduce main GPU memory access; 2) no nxn attention matrix at once; 2) easy recompute for backward pass, as gradient checkpointing

## Notes on training memory and parallelism

Reference notes for choosing between DDP / FSDP and reasoning about long-context memory.

### What scales with what

For a transformer block with sequence length S and hidden size H, the activations split into two distinct terms:

| Term | Source | Scaling |
|---|---|---|
| Linear-in-S activations | residual stream, MLP intermediates, layer norms | `O(S · H)` per layer |
| Quadratic-in-S attention scores | the `Q @ K^T` matrix and softmax probs (only if materialized) | `O(S²)` per attention head |

Without anything else: total ≈ `L · (S·H + S²)`.

### What activation checkpointing buys you

Gradient/activation checkpointing stores activations only at **block boundaries** and recomputes intermediates during backward. So:

- The `L · ...` factor collapses to `~ N_checkpoints · ...` — typically one boundary per transformer block.
- The **per-block term is unchanged** — one block's activations (including any `O(S²)` attention scores) are still transiently materialized during recompute.
- Net effect: peak memory ≈ `N_checkpoints · S · H + max_block_activations`. **Linear in S, with the `S²` term still lurking inside the block being recomputed.**

In short: checkpointing fixes layer-count scaling, not sequence-length scaling.

### What actually breaks the sequence-length scaling

| Technique | Kills which term |
|---|---|
| **FlashAttention** | `O(S²) → O(S)` for the attention block — never materializes the score matrix |
| **Selective activation checkpointing** | recomputes attention but keeps cheap activations stored — squeezes the constant |
| **Sequence parallelism (SP)** | shards `S · H` activations along the seq axis: `O(S · H / world_size)` |
| **Context / Ring parallelism (CP)** | shards attention itself along seq: `O(S · H / world_size)` end-to-end |

Practical recipe for long context: **FlashAttention + activation checkpointing** gives `O(N_layers · S · H)` — still linear in S but with a small constant. To break linear scaling you need SP or CP.

### Note: parameter sharding (FSDP) does NOT shard activations

A common point of confusion. Both **FSDP1** and **FSDP2** shard **parameters, gradients, and optimizer states** — never activations. Activation memory is unaffected by switching DDP ↔ FSDP. The right tools for activation memory are activation checkpointing, FlashAttention, and (for very long context) SP/CP/TP.

### DDP vs FSDP — when each makes sense

| | DDP | FSDP |
|---|---|---|
| Forward | no comm | all-gather params per FSDP unit |
| Backward | all-reduce grads (overlapped with compute) | all-gather params again + reduce-scatter grads |
| Memory per rank | full params + grads + opt states | sharded |

DDP communicates exactly once per step (gradient all-reduce), almost fully overlapped with the backward pass. FSDP all-gathers in both forward and backward plus reduce-scatter for grads — typically 5–20% slower per step at the same effective batch size. **If memory isn't binding, DDP wins** on speed, simplicity, and debuggability (full model on every rank).

Reach for FSDP when:
- Params + grads + Adam states + activations don't fit on one GPU.
- You want larger per-rank microbatch enabled by the memory savings.
- You're training models large enough (≥10B params) that sharded checkpoint I/O matters.
- You need to compose with TP/PP via `DeviceMesh` (FSDP2 only).

For a LoRA-only training run (most params frozen, small trainable subset), DDP is almost always sufficient.

### FSDP Quick Grasp

#### Quick mental model of FSDP sharding

DDP replicates every parameter on every GPU, so each GPU stores a full copy. That hits
a wall when the model doesn't fit on one GPU.

FSDP (Fully Sharded Data Parallel) breaks each Parameter into N shards across N GPUs
and stores only that 1/N slice on each rank. To run a layer's forward, FSDP issues an
**all-gather** that collects every rank's shards into a full tensor on every rank,
runs the layer, then frees the gather. Backward does the same gather, then a
**reduce-scatter** to average gradients back to sharded form. End result: each GPU
permanently holds 1/N of the model's parameters / gradients / optimizer state, but
sees full weights briefly during each layer's forward + backward.

The granularity question is "which subset of parameters share one all-gather?" Too
coarse (one gather for the whole model) → defeats the point, peak memory equals DDP.
Too fine (per-Parameter gather) → tiny inefficient collectives. The middle ground is
**per transformer block**: each block's params share a gather, the gather is freed
after the block runs, and the next block does its own gather. That keeps peak memory
bounded to one block's worth of full-precision params, and the gather is large enough
to amortize the collective overhead.

#### FSDP Lifecycle: Leaf vs. Root

FSDP manages VRAM by triggering an `all-gather` (allocating full-precision weights) via a `forward_pre_hook` and freeing them via a `forward_hook`. Their position in the model hierarchy changes everything:

* **Leaf (e.g., a Transformer Block)**
    * **Allocates:** Right before the specific block runs.
    * **Frees:** Immediately after the block finishes.
    * **Lifecycle:** **Ephemeral.** VRAM spikes only for the fraction of a second that block is computing, then clears instantly.

* **Root (Top-Level Model, absorbing Embeddings & LM Head)**
    * **Allocates:** At the very start of the entire model's forward pass.
    * **Frees:** At the very end of the entire model's forward pass.
    * **Lifecycle:** **Persistent.** These weights sit fully materialized in VRAM for the *entire duration* of the forward and backward passes.

#### The Consequence: The Memory Trap
Because the Root is forced to hold its parameters continuously, the un-sharded weights of the **LM Head** and **Embeddings** never get freed during execution. This persistent memory footprint stacks underneath the sequential, temporary memory spikes of every Leaf block. If your vocabulary size is massive, the Root-managed LM Head becomes the silent bottleneck for peak VRAM, negating the efficiency of your block-level sharding.

#### Why `_no_split_modules`

accelerate's `TRANSFORMER_BASED_WRAP` policy needs to know which leaf classes to wrap
as FSDP units. Each HF model exposes `_no_split_modules: List[str]` listing its block
class names (e.g. `['Qwen3DecoderLayer']`). The policy walks the module tree, calls
`fully_shard(module)` on every instance whose class name is in that list, and stops
descending — those become independent FSDP units.
