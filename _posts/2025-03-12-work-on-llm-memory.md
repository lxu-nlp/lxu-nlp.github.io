---
layout: post
title: "Works on LLM Memory"
date: 2025-03-12
categories: [NLP]
tags: [nlp, LLM, memory]
math: true
---

## Attention Design

**Leave No Context Behind: Efficient Infinite Context Transformers with Infini-attention**. Munkhdalai et al. 2024\
<https://arxiv.org/abs/2404.07143>


## Training-time Context Compression

**Learning to Compress Prompts with Gist Tokens**. Mu et al. NIPS'23\
Compress **instructions** within LLM (**objective: specific tasks**): instruction + GIST (soft tokens) + input_context, such that:\
(1) GIST is learned to be generalized to compress arbitrary instructions.\
(1) GIST can be cached and reused.\
Training: simply mask out instruction after GIST.\
<https://arxiv.org/abs/2304.08467>

**Adapting Language Models to Compress Contexts**. Chevalier et al. EMNLP'23\
Compress **input context** (**objective: general LM**).\
Next prediction: current text segment + past vectors.\
<https://aclanthology.org/2023.emnlp-main.232/>

**In-context Autoencoder for Context Compression in a Large Language Model**. Ge et al. ICLR'24\
Compress **input context** (**objectives: (reconstruction + general LM as pretraining) + specific tasks**).\
<https://arxiv.org/abs/2307.06945>


## Test-time Context Pruning: Hard Drop Non-Salient Tokens

**Compressing Context to Enhance Inference Efficiency of Large Language Models**. Li et al. EMNLP'23\
Finding: dropping tokens with high likelihood (less informative) could obtain similar generation quality.\
(no training; need full forward once)\
<https://aclanthology.org/2023.emnlp-main.391>

**Not All Tokens Are What You Need for Pretraining**. Lin et al. NIPS'24\
Use a pretrained model to get token likelihood to: 1) filter out noisy corpus; 2) enable loss only on hard tokens for more pretraining.\
<https://openreview.net/forum?id=0NMzBwqaAJ>

**LLMLingua-2: Data Distillation for Efficient and Faithful Task-Agnostic Prompt Compression**. Pan et al. ACL Findings'24\
Use GPT annotation to train a small token classifier on whether to retain.\
<https://aclanthology.org/2024.findings-acl.57>

**Fewer is More: Boosting LLM Reasoning with Reinforced Context Pruning**. Huang et al. 2024\
RL to select and prune few-shot prompts.\
<https://arxiv.org/abs/2312.08901>


## Test-time Cache Pruning: Drop Cache per Head/Layer

**Dynamic Context Pruning for Efficient and Interpretable Autoregressive Transformers**. Anagnostidis et al. NIPS'23\
Each layer drops kv cache independently.\
(not dropping entire tokens/positions, as each layer can have different patterns.)\
Train with sparse sigmoid on attention.\
<https://arxiv.org/abs/2305.15805>

**Analyzing Multi-Head Self-Attention: Specialized Heads Do the Heavy Lifting, the Rest Can Be Pruned**. Voita et al. ACL'19\
Hard-gate on attention heads, regularizing to closing gate.\
<https://aclanthology.org/P19-1580/>

**Are Sixteen Heads Really Better than One?**. Michel et al. NIPS'19\
Define sensitivity by loss on heads and prune greedily and sequentially (after sort).\
<https://openreview.net/forum?id=ByxXhSBgIS>

**Differentiable Subset Pruning of Transformer Heads**. Li et al. TACL'21\
User-specified number of pruned heads.\
<https://arxiv.org/abs/2108.04657>

**Scissorhands: Exploiting the Persistence of Importance Hypothesis for LLM KV Cache Compression at Test Time**. Liu et al. NIPS'23\
Critical tokens always receive high attention; prune non-critical tokens.\
<https://openreview.net/forum?id=JZfg6wGi6g>

**H2O: Heavy-Hitter Oracle for Efficient Generative Inference of Large Language Models**. Zhang et al. NIPS'23\
Prune based on accumulated attention scores.\
<https://openreview.net/forum?id=RkRrPp7GKO>

**D2O : Dynamic Discriminative Operations for Efficient Generative Inference of Large Language Models**. Wan et al. 2024\
<https://arxiv.org/abs/2406.13035>

**Efficient Streaming Language Models with Attention Sinks**. Xiao et al. ICLR'24\
<https://openreview.net/forum?id=NG7sS51zVF>

**Model Tells You What to Discard: Adaptive KV Cache Compression for LLMs**. Ge et al. ICLR'24\
Attention head possesses different distribution patterns, which is also consistent across positions.\
Thus, able to prune attention heads adaptively (per head per layer).\
<https://openreview.net/forum?id=uNrFpDPMyo>

**SnapKV: LLM Knows What You are Looking for Before Generation**. Li et al. NIPS'24\
<https://openreview.net/forum?id=poE54GOq2l>


## Cache Merging

**MiniCache: KV Cache Compression in Depth Dimension for Large Language Models**. Liu et al. 2024\
<https://arxiv.org/abs/2405.14366>


## Memory Modeling

**Augmenting Language Models with Long-Term Memory**. Wang et al. 2023\
<https://arxiv.org/pdf/2306.07174>

**RetrievalAttention: Accelerating Long-Context LLM Inference via Vector Retrieval**. Liu et al. 2025\
KV cache as memory: retrieve most similar K upon Q (topk), but not full search; using approximate search e.g. cluster-based for sublinear retrieval.\
Cache Retrieval vs. Pruning: retrieval is dynamic, as each latest Q obtains different set of K, due to observation that different Q has different set of similar K, as generation goes\
Novel: Q does not directly retrieve well with cluster-based K index, as K index does not see Q distribution\
Method: regard as few-shot problem, use existing Q distribution as anchor to find the closest K, instead of finding K directly\
<https://arxiv.org/abs/2409.10516>

**Titans: Learning to Memorize at Test Time**. Behrouz et al. 2025\
Memory vector design: meta learning that learns to read/update memory.\
<https://arxiv.org/pdf/2501.00663>
