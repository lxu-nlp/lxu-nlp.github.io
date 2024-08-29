---
layout: post
title: "Works on Efficient Decoding"
date: 2024-08-05
categories: [NLP]
tags: [nlp]
math: true
---

Plugin for efficient LLM decoding without any training.

## Drop Entire Position

**Compressing Context to Enhance Inference Efficiency of Large Language Models**. Li et al. EMNLP'23\
Finding: dropping tokens with high likelihood (less informative) could obtain similar generation quality.\
(no training; not pruning on the fly; need full forward once)\
<https://aclanthology.org/2023.emnlp-main.391>

**Fewer is More: Boosting LLM Reasoning with Reinforced Context Pruning**. Huang et al. 2024\
RL to select and prune few-shot prompts.\
<http://arxiv.org/abs/2312.08901>

## Prune per Head/Layer

**Dynamic Context Pruning for Efficient and Interpretable Autoregressive Transformers**. Anagnostidis et al. NIPS'23\
Each layer drops kv cache independently.\
(not dropping entire tokens/positions, as each layer can have different patterns.)\
Train with sparse sigmoid on attention.\
<http://arxiv.org/abs/2305.15805>

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

## Parameter Merging or KV Cache Merging

**MiniCache: KV Cache Compression in Depth Dimension for Large Language Models**. Liu et al. 2024\
<https://arxiv.org/abs/2405.14366>

## Speculative Decoding

**Fast Inference from Transformers via Speculative Decoding**. Leviathan et al. ICML'23\
n-times autoregression by smaller draft model + LLM verification once that recovers original distribution through rejection sampling strategies.\
Easier positions are now handled by the smaller draft model.\
<https://openreview.net/forum?id=C9NEblP8vS>

**EAGLE: Speculative Sampling Requires Rethinking Feature Uncertainty**. Li et el. ICML'24\
(1) learn a small draft model that mimics the final hidden states (features) generated from the original LLM, to be consumed directly by the LM head of original LLM\
(2) draft model: (latest token, previous feature) -> latest feature, next token; same effect as original LLM: (latest token, past kv) -> latest kv, next token\
(3) draft tree rather than chain; LLM verification once on the tree.\
<https://openreview.net/forum?id=1NdN7eXyb4>
