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

---

# RMSNorm 核心 Insight（少数学版）

## 1. RMSNorm 到底是什么
- RMSNorm = LayerNorm 砍掉「减均值（中心化）」和「偏置 β」，只留「按 RMS 缩放 + 可学习 γ」。
- 它与 LayerNorm 的唯一差别就是**不做中心化**。

## 2. 真正的稳定因子是「方差重置」，不是「减均值」
- **Insight**：归一化的全部稳定作用来自「把输入缩放成单位尺度」那一步（÷RMS）。减均值只挪动分布中心、不改尺度，对深度稳定性**毫无贡献**。
- 所以砍掉中心化的 RMSNorm 照样稳——这恰恰证明中心化是冗余的。

## 3. 方差和范数本是同一个东西
- **Insight**：谈「方差」还是谈「范数」是一回事，二者都由 $\sum x_i^2$ 决定。范数是 L2 大小，方差是它减均值后的散布。
- $\|\mathbf x\|_2=\sqrt H\cdot\text{RMS}$，$\operatorname{Var}=\text{RMS}^2-\mu^2$。除以 RMS 就是强制方差=1，几何视角与统计视角是同一操作。

## 4. 为什么 var=1 时不随深度漂移
- **Insight**：深度漂移来自「尺度逐层相乘、复利增长」。归一化的巧妙在于——它除以的是**当前输入自己的 RMS**，这个值刚好把前面所有层累积出来的尺度**一次性除掉**。
- 于是深度 $L$ 从方差表达式里被消掉，不再出现。剩下的尺度只有可学习 γ，它是每层独立参数、不成乘积链，自然也不漂移。

## 5. 为什么这能稳住梯度
- **Insight**：梯度大小和输入「有多大」成正比。固定方差 = 固定输入尺度 = 固定梯度量级，于是全网络能用一个全局学习率稳定训练，不会浅层暴走、深层不动。

## 6. Transformer 里两条具体失效链（方差失控时）
- **残差流发散**：$h=h+\text{Sublayer}(h)$ 的方差随深度单调增长，早期信号被淹没。
- **softmax 饱和**：logit$=QK^\top/\sqrt d$，范数失控→logit 极大→softmax 趋 one-hot，梯度 $\propto p(1-p)\to0$，$Q,K$ 学不动。

## 7. γ 不丢表达力
- **Insight**：归一化只剥掉「不受控、会随深度累积」的尺度；γ 再把「可学习、每层独立」的尺度加回来。需要什么尺度网络都能学，所以既稳又不掉能力。

## 8. 一句话总结
归一化 = 用输入自身 RMS 相除清零历史累积尺度，使方差恒为 1、深度被消去，梯度随之稳定；**稳定来自「方差重置」而非「减均值」**，因此去掉中心化的 RMSNorm 反而更干净。
