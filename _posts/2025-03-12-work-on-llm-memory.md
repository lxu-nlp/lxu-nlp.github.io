---
layout: post
title: "Works on LLM Memory"
date: 2025-03-12
categories: [NLP]
tags: [nlp, LLM, memory]
math: true
---

## Hidden Memory Design

**Augmenting Language Models with Long-Term Memory**. Wang et al. NIPS'23\
<https://arxiv.org/pdf/2306.07174>

**Leave No Context Behind: Efficient Infinite Context Transformers with Infini-attention**. Munkhdalai et al. 2024\
Linear memory.\
<https://arxiv.org/abs/2404.07143>

**TransformerFAM: Feedback attention is working memory**. Hwang et al. 2024\
<https://arxiv.org/abs/2404.09173>

**Titans: Learning to Memorize at Test Time**. Behrouz et al. 2025\
Memory vector design: meta learning that learns to read/update memory.\
<https://arxiv.org/pdf/2501.00663>


## Explicit Memory Design

**Infinite Retrieval: Attention Enhanced LLMs in Long-Context Processing**. Ye et al. 2025\
Observation: attention pattern aligns with retrieval-augmented in latter layers.\
Memory: simply add sentences of top salient tokens as context.\
<https://arxiv.org/abs/2502.12962>


## Attention w/ Retrieval

**RetrievalAttention: Accelerating Long-Context LLM Inference via Vector Retrieval**. Liu et al. 2025\
KV cache as memory: retrieve most similar K upon Q (topk), but not full search; using approximate search e.g. cluster-based for sublinear retrieval.\
Cache Retrieval vs. Pruning: retrieval is dynamic, as each latest Q obtains different set of K, due to observation that different Q has different set of similar K, as generation goes\
Novel: Q does not directly retrieve well with cluster-based K index, as K index does not see Q distribution\
Method: regard as few-shot problem, use existing Q distribution as anchor to find the closest K, instead of finding K directly\
<https://arxiv.org/abs/2409.10516>
