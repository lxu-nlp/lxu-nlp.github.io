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

