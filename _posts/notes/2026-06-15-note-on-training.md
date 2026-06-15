---
layout: post
title: "Notes on Training"
date: 2026-06-15
categories: [Notes]
tags: [optimization, ML]
math: true
---

## Training memory and parallelism

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

---

## Padding

For SFT training of autoregressive LLM: **right for training, left for generation**.

Why the split — and why the train/inference mismatch is harmless for **RoPE** models (Llama / Qwen / Mistral / etc.):

1. **Training right-pad** — industry convention used by TRL `SFTTrainer` / Axolotl / LLaMA-Factory; the HF + FA2 unpad/repad path is more battle-tested with right-padding than with left.
2. **No real mismatch under RoPE.** The *attention mask* (zeroed out before softmax) makes pad positions invisible to real-token attention regardless of side. RoPE encodes only the *relative distance* between real tokens, and that distance is identical under either padding side. So the tensor math seen by real tokens is the same in both phases.
3. **`position_ids` are NOT what aligns the two phases.** HF `generate()` does rebuild `position_ids` from the attention mask at inference, but during training the model receives a naive `arange(0, seq_len)` (HF Trainer never passes `position_ids`). **The two phases would disagree on absolute positions if the model relied on absolute positional embeddings — but RoPE doesn't, which is the whole reason this works.** The alignment is **on the RoPE side, not the dynamic-position-id side**. (For absolute-position models like GPT-2 / BERT, left-padding during training really would be broken.)

---

## Training Efficiency: Padding Removal

### 1. The Problem

Sequences in a training batch have different lengths. A standard rectangular
batch pads every sequence to the longest one, so a batch of `[128, 256, 512,
2048]`-token sequences becomes a `4 × 2048` rectangle. The PAD positions are
not free — the GPU still computes attention and FFN over them, then masks
the result. On length-skewed data (instruction tuning, RL completions),
**half to three-quarters of the GPU's work can be on PAD tokens**.

Padding removal eliminates that waste. Compute scales with **real tokens**
instead of padded tokens. Typical wins: 1.5–2× throughput, ~20% less peak
memory.

### 2. The Idea Behind the Fix

There exists a special attention kernel — Flash Attention 2's *variable-
length* kernel — that takes a **flat stream of tokens** plus a list of
**segment boundaries**, and computes attention block-diagonally: each
segment attends only to itself. PAD tokens never enter the picture.

```
flat tokens:   sample1 │ sample2 │ sample3
attention:        ◯         ◯         ◯
                  └ each circle = self-attention within that segment
```

So the trick reduces to: **produce that flat stream, hand it to the model,
get the result back in whatever shape your training loop wants.**

#### How the model knows to use the special kernel

You don't flip a switch. HuggingFace transformers detect packed input
automatically when **two conditions** hold in the model call:

1. `attention_mask` is `None`.
2. `position_ids` restarts at 0 at every segment boundary.

When both are true, HF's attention layer derives the segment boundaries
from the position-id resets and dispatches to the variable-length kernel.
Otherwise it falls back to the standard rectangular kernel — silently. So
"forgetting" either condition costs you all the speedup with no error.

### 3. Two Philosophies

The mechanism is fixed. What differs is **where in the pipeline you produce
the flat stream** — and that single decision propagates into very different
shapes of code, different composability properties, and different mental
models for downstream code.

```
                  ┌──────────────────────────────────────────┐
   Dataset        │                                          │
       │          │   Route A flattens HERE                  │
       ▼          │   (data-side fix, HuggingFace way)       │
   Collator    ───┘                                          │
       │                                                     │
       ▼  batch handed to trainer                            │
   training_step(batch)                                      │
       │          ┌──────────────────────────────────────────┘
       │          │
       ▼          │   Route B flattens HERE
   forward(batch) │   (forward-side fix, verl way)
       │          │
       ▼          │
   model(...) ◄───┘
       │
       ▼
   loss → backward → optimizer step
```

#### Route A — Data-side fix (the HuggingFace way)

A specialised data collator does the flattening **at data-loading time,
before the batch ever enters the trainer**. It takes the list of variable-
length samples that the dataset yields and concatenates them into one long
flat row. Critically, it also constructs the `position_ids` correctly — they
restart at 0 at the boundary of each original sample — and inserts an
ignore-marker between samples so that the next-token-prediction label of
the last token of sample `i` doesn't "leak" into the first token of sample
`i+1`.

By the time the batch reaches your forward, **padding does not exist
anywhere in the pipeline**. The model receives a single segmented flat
stream, dispatches to the variable-length kernel via the two trigger
conditions, and produces a flat output that your loss code consumes
directly with the standard cross-entropy `ignore_index=-100` convention —
no special handling required.

**Conceptual properties:**
- The batch dimension `B` ceases to exist downstream of the collator. The
  forward sees one row of shape `(1, total_tokens)`. Sample identity is
  preserved only through the position-id resets and (optionally) a separate
  segment-id tensor the collator can emit on request.
- Because flattening happens before the trainer, **everything downstream
  of the collator can stay oblivious to the optimisation**. The forward,
  loss, metrics, gradient handling, and optimiser are exactly the code you
  would write for a non-padded model — minus the `attention_mask` argument.
- Memory profile is the best possible: padded activations are never
  materialised at any point in the loop, including in optimiser state and
  gradient buffers.

**Conceptual cost:**
- It assumes the rest of the stack is willing to consume a `(1, total)`
  batch. Code that wants to operate **per sample** (e.g. "give me the logits for
  sample 3 in this batch") cannot do so by indexing — there is no sample-3
  axis. You have to re-derive it from the segment boundaries. For ordinary
  SFT, where the loss is just a token-level cross-entropy reduced to a
  scalar, this never comes up. For RL or multi-task training, where you
  often need per-sample reductions, it bites.

**This is a one-line change at the trainer call site** in modern HF / TRL.
The whole point of the design is that the rest of your code is untouched.

#### Route B — Forward-side fix (the verl way)

The collator still hands you a normal padded `(B, T)` batch with an
`attention_mask`. The flattening happens **inside the forward function**,
in three explicit steps:

1. **Unpad.** A helper takes the padded `input_ids` and `attention_mask`,
   produces a flat `(1, total_real_tokens)` stream, and returns an
   `indices` tensor that records each surviving token's original `(row,
   column)` position in the rectangle.
2. **Forward** the model on the flat stream, with `attention_mask=None`,
   exactly as Route A does. Same kernel, same trigger conditions.
3. **Repad.** A helper scatters the per-token output (logits, log-probs,
   per-token loss — whatever you need) back into the original `(B, T)`
   rectangle using `indices`. Positions that were padding are filled with
   zeros.

The result: the model internally ran the padding-free fast path, but the
**rest of your training code sees rectangular `(B, T)` tensors**, just like
the padded baseline. Per-sample indexing works. Loss masks work. Metrics
work. Anything that expected a batch dimension still has one.

**Conceptual properties:**
- Padding removal is a **local optimisation inside the forward**, not a
  pipeline-wide contract. The collator, the dataset, the optimiser, the
  gradient handler, and any downstream consumer all see the same shapes
  they always saw. Drop-in.
- It can be applied **selectively** — per micro-batch, per training stage,
  or only on the actor and not the reference model in RL. The decision is
  made inside the forward, where all relevant state is in scope.
- It composes with **sequence parallelism**. The flat stream produced by
  `unpad_input` can be sliced across SP ranks; gathered after the model;
  then repadded. Each step is a well-defined operation on a flat tensor.
  This is exactly why verl chose this design — RL post-training with long
  sequences essentially requires SP, and SP requires forward-side control.
- It composes with **token-budget micro-batching** (see
  `SIMPLE_DYNAMIC_BATCH.md`). The dynamic-batching layer decides which
  sequences land in this micro-batch based on their padded lengths; the
  unpad step inside the forward then strips the pads. The two layers don't
  need to know about each other.

**This is the right choice when your training loop has structure that the
HF collator can't represent** — multi-stage RL pipelines, sequence
parallelism, or any forward whose output is consumed by code that expects
rectangular tensors.

### 4. How They Compare

| Question | HF route | verl route |
|---|---|---|
| Where does flattening happen? | in the collator | inside the forward |
| What shape does the forward see? | one long flat row | normal padded `(B, T)` then briefly flat |
| What shape does loss code see? | flat | back to padded `(B, T)` |
| Code change | ~1 line | ~30 lines |
| Composes with sequence parallelism? | no | yes |
| Composes with custom RL micro-batching? | no | yes |
| Right for standard SFT? | **yes** | overkill |
| Right for RL post-training? | no | **yes** |

---

## Training Efficiency: Dynamic Batching

### 1. The Problem

Sequence lengths vary widely in real training data — sometimes by 40× or
more between the shortest and longest sample. If you batch a fixed
**number** of sequences per step, two bad things happen:

- A batch of all-short sequences uses a tiny fraction of GPU memory →
  wasted capacity.
- A batch of all-long sequences blows past memory → out-of-memory crash.
- For distributed training, imbalanced batches introduce rank waiting.

To stay safe, you must size the batch for the worst case, which means most
of the time the GPU is far below its memory budget. RL post-training is
even worse: each prompt produces several rollouts, and one prompt's
rollouts might be 40× longer than another's.

The fix: **stop counting sequences. Start counting tokens.**

---

### 2. The Two Core Ideas

#### Idea 1 — Token budget instead of sequence count

Set `max_tokens_per_batch = M` and pack as many sequences into a batch as
fit under `M`. Memory usage is now a function of `M`, not of which
sequences happened to land together. Steps become predictable.

#### Idea 2 — Balanced partitioning

Given a pile of sequences of varying lengths and a target of `K` groups
(= `K` micro-batches, or `K` GPUs), assign sequences to groups so the
**total tokens are as equal as possible across groups**. The slowest group
sets the wall-clock; balancing makes the slowest group as fast as possible.

These ideas compose: tokens-not-sequences fixes memory variance, balanced
partitioning fixes time variance.

### 3. Where Length Skew Bites — Two Scopes

In a multi-GPU training step, length variance hurts in two distinct
places:

```
   full batch (all data for this step)
        │
        │ split across GPUs               ← scope A: cross-GPU
        ▼
   GPU 0     GPU 1     GPU 2     GPU 3
        │
        │ split into micro-batches        ← scope B: within-GPU
        ▼
     μ1 μ2 μ3   μ1 μ2     μ1 μ2 μ3 μ4   ...
```

- **Scope A — cross-GPU:** if one GPU gets the long sequences, it's slow
  while the others sit waiting in synchronisation barriers. Step time =
  slowest GPU's time.
- **Scope B — within-GPU:** if one micro-batch packs the long sequences,
  it sets per-step memory. The whole job has to fit it.

Both scopes need balancing. They are independent decisions made at
different points in the loop.

### 4. Common Implementations (Tiered by Effort)

#### Tier 1 — Length-grouped sampling

Sort the dataset by length, then chunk into batches. Sequences in the same
batch are similar lengths, so padding waste shrinks even without padding
removal. Built into HuggingFace as a one-flag option.

**Trade-off:** batches are no longer truly random; they form a soft
length-curriculum. Quality impact is usually small but non-zero.

**Use when:** simple SFT, single GPU, low-effort win.

#### Tier 2 — Packing-based collators

Concatenate multiple short samples into one row at the collator stage,
emitting segment boundaries so attention doesn't leak across samples. This
**both** removes padding and packs short samples into the same forward.
For most regular SFT projects this single change solves the whole problem.

**Use when:** standard SFT with HuggingFace or TRL.

#### Tier 3 — Dynamic token-budget micro-batching (verl-style)

Decide micro-batch composition **on the fly per step**, after data has
already been routed to each GPU. The unit is a token budget, the
partitioning is recomputed from the actual lengths in this step's data.
Handles both scope A (cross-GPU reordering) and scope B (within-GPU
splitting).

This is what RL post-training pipelines use because RL data composition
changes step-to-step in ways the dataloader can't predict.

**Use when:** RL training, sequence parallelism, multi-stage pipelines, or
any case where step-time data distribution is unpredictable.

#### Tier 4 — Pre-bucketed dataloaders

Bucket the dataset by length at preprocessing time, sample one bucket per
step. Maximum kernel efficiency, more preprocessing complexity.

**Use when:** very large pretraining runs where preprocessing is amortised
over millions of steps.

### 5. The Partitioning Algorithm

The core subroutine: given lengths and a partition count `K`, minimise the
gap between the heaviest and lightest partition.

- **Greedy:** sort by length descending; assign each item to the currently
  lightest partition. Easy, fast, usually within a few percent of optimal.
- **Karmarkar-Karp:** an exact-ish method that minimises the gap by
  iteratively pairing the two heaviest items and merging. Materially
  better when length variance is high and partitions are small.

verl uses Karmarkar-Karp. HF's length-grouped sampler uses a simpler
bucket-based scheme. For most practical cases — handful of partitions,
hundreds of items — the algorithmic upgrade is a ~5–15% gain on
worst-case partition spread.

### 6. Implementation Hygiene Worth Knowing

Two non-obvious rules show up in every real implementation:

#### All ranks must run the same number of micro-batches

Under data parallelism (DDP / FSDP), every rank must reach gradient
synchronisation at the same time. If rank 0 decides 3 micro-batches and
rank 1 decides 2, rank 1 hangs forever waiting on rank 0's third
all-reduce. Fix: take the maximum across ranks and use that count
everywhere.

#### Restore order after partitioning

Balanced partitioning permutes your data. If your downstream code matches
results to the original input order (labels, rewards, prompt IDs), build
an inverse permutation and apply it before returning. Easy to forget,
hard to debug — the loss looks plausible but data-row mappings are
scrambled.
