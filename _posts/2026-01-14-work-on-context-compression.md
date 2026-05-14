---
layout: post
title: "Works on LLM Task and Context Compression"
date: 2026-01-14
categories: [LLM]
tags: [nlp, LLM, compression]
math: true
---

## Supervised Compression

#### Task/Prompt Compression: task-specific training

**Prefix-Tuning: Optimizing Continuous Prompts for Generation**. Li and Liang. ACL'21\
Prefix-tuning: optimize a continuous prompt prefix per task, instead of discrete task prompt.\
Close to finetuning, and better than finetuning when data is small.\
<https://aclanthology.org/2021.acl-long.353>

**The Power of Scale for Parameter-Efficient Prompt Tuning**. Lester et al. EMNLP'21\
Prompt-tuning: similar to prefix-tuning, only optimizing a continuous prompt prefix.\
(1) Prompt length can be critical.\
(2) Initializing prompt by class-label vocab performs the best.\
(3) Prompt tuning becomes more competitive with scale (same performance as finetuning).\
<https://aclanthology.org/2021.emnlp-main.243>

**Learning to Compress Prompts with Gist Tokens**. Mu et al. NIPS'23\
Compress **instructions** within LLM (**objective: specific tasks**): instruction + GIST (soft tokens) + input_context, such that:\
(1) GIST is learned to be generalized to compress arbitrary instructions.\
(1) GIST can be cached and reused.\
Training: simply mask out instruction after GIST.\
<https://arxiv.org/abs/2304.08467>

**TokMem: One-Token Procedural Memory for Large Language Models**. Wu et al. ICLR'26\
Similar to Prompt-tuning: learn tok emb as instruction; can be used interleaved in thinking.\
<https://openreview.net/forum?id=RWjEf9PdiJ>

#### Context Compression: preserving general LM context

Also see the [post](https://lxu-nlp.github.io/posts/work-on-agent-memory/#latent-organization-w-training) on latent agentic memory.

**Adapting Language Models to Compress Contexts**. Chevalier et al. EMNLP'23\
AutoCompressor: compress **input context** recursively (**objective: general LM**).\
Next prediction: current text segment + past vectors.\
<https://aclanthology.org/2023.emnlp-main.232/>

**In-context Autoencoder for Context Compression in a Large Language Model**. Ge et al. ICLR'24\
ICAE: learn to encode context to k memory tokens. More than 4x compression is challenging.\
Pretraining (reconstruction + LM) + finetuning.\
<https://openreview.net/forum?id=uREj4ZuGJE>

**Context Embeddings for Efficient Answer Generation in Retrieval-Augmented Generation**. Rau et al. WSDM'25\
COCOM: same as ICAE.\
<https://dl.acm.org/doi/10.1145/3701551.3703527>

**500xCompressor: Generalized Prompt Compression for Large Language Models**. Li et al. ACL'25\
Same as ICAE, but using KV cache instead of hidden states.\
<https://aclanthology.org/2025.acl-long.1219>

**xRAG: Extreme Context Compression for Retrieval-augmented Generation with One Token**. Cheng et al. NIPS'24\
Modality alignment: embedding -> LM. Learn to utilize given embedding for generation.\
Pretraining (reconstruction for alignment) + distillation + task finetuning.  But reconstruction < finetuning (including self-distillation).\
<https://openreview.net/forum?id=6pTlXqrO0p>

**PISCO: Pretty Simple Compression for Retrieval-Augmented Generation**. Louis et al. ACL Findings'25\
Similar to ICAE.\
Only do distillation training. Clear presentation.\
<https://aclanthology.org/2025.findings-acl.800>

**Pretraining Context Compressor for Large Language Models with Embedding-Based Memory**. Dai et al. ACL'25\
Similar to ICAE. Bad presentation.\
<https://aclanthology.org/2025.acl-long.1394>

**MemoRAG: Moving towards Next-Gen RAG Via Memory-Inspired Knowledge Discovery**. Qian et al. WWW'25\
Train model to learn gist token that memorizes the past context to provide approximate clues, serving as a model to bridge the query and full context.\
Pretraining (LM) + finetuning.\
<https://dl.acm.org/doi/10.1145/3696410.3714805>

**Enhancing RAG Efficiency with Adaptive Context Compression**. Guo et al. EMNLP Findings'25\
Similar to ICAE, trained with multiple granularity? (unclear written)\
<https://aclanthology.org/2025.findings-emnlp.1307>

**Autoencoding-Free Context Compression for LLMs via Contextual Semantic Anchors**. Liu et al. ICLR'26\
Similar to 500xCompressor, but use interleaved position for compression rather than new tokens.\
<https://openreview.net/forum?id=8Pi6Du0n7F>

**R3Mem: Bridging Memory Retention and Retrieval via Reversible Compression**. Wang et al. ACL Findings'25\
<https://aclanthology.org/2025.findings-acl.235>

**M+: Extending MemoryLLM with Scalable Long-Term Memory**. Wang et al. ICML'25\
ICAE setup + making memory hidden states retrievable by similarity.\
<https://openreview.net/forum?id=OcqbkROe8J>

**CLaRa: Bridging Retrieval and Generation with Continuous Latent Reasoning**. He et al. 2026\
Stage 1: similar to PISCO.\
Stage 2: frozen document compressor; learn a query compressor and generator, jointly train top-k retrieval and generation.\
<https://arxiv.org/abs/2511.18659>

#### Online Context Compression

**Long Context Compression with Activation Beacon**. Zhang et al. ICLR'25\
Online, interleaved gist token compression (new QKV).\
Pretraining (LM) + finetuning.\
<https://openreview.net/forum?id=1eQT9OzfNQ>

**OSCAR: Online Soft Compression for RAG**. Louis et al. ICLR'26\
Online query-dependent compression using small model. Train similar to PISCO, adding a reranking objective as well.\
<https://openreview.net/forum?id=ideKAUWvFE>

## Unsupervised KV-Based Compression

#### Test-time Context Pruning: Hard Drop Non-Salient Tokens

**Compressing Context to Enhance Inference Efficiency of Large Language Models**. Li et al. EMNLP'23\
Finding: dropping tokens with high likelihood (less informative) could obtain similar generation quality.\
(no training; need full forward once)\
<https://aclanthology.org/2023.emnlp-main.391>

**LLMLingua-2: Data Distillation for Efficient and Faithful Task-Agnostic Prompt Compression**. Pan et al. ACL Findings'24\
Use GPT annotation to train a small token classifier on whether to retain.\
<https://aclanthology.org/2024.findings-acl.57>

**Fewer is More: Boosting Math Reasoning with Reinforced Context Pruning**. Huang et al. EMNLP'24\
RL to select and prune few-shot prompts.\
<https://aclanthology.org/2024.emnlp-main.758/>

**Not All Tokens Are What You Need for Pretraining**. Lin et al. NIPS'24\
Use a pretrained model to get token likelihood to: 1) filter out noisy corpus; 2) enable loss only on hard tokens for more pretraining.\
<https://openreview.net/forum?id=0NMzBwqaAJ>

#### Test-time Cache Pruning: Drop Cache per Head/Layer

**Analyzing Multi-Head Self-Attention: Specialized Heads Do the Heavy Lifting, the Rest Can Be Pruned**. Voita et al. ACL'19\
Hard-gate on attention heads, regularizing to closing gate.\
<https://aclanthology.org/P19-1580/>

**Are Sixteen Heads Really Better than One?**. Michel et al. NIPS'19\
Define sensitivity by loss on heads and prune greedily and sequentially (after sort).\
<https://openreview.net/forum?id=ByxXhSBgIS>

**Differentiable Subset Pruning of Transformer Heads**. Li et al. TACL'21\
User-specified number of pruned heads.\
<https://arxiv.org/abs/2108.04657>

**Dynamic Context Pruning for Efficient and Interpretable Autoregressive Transformers**. Anagnostidis et al. NIPS'23\
Train with sparse sigmoid on attention.\
<https://arxiv.org/abs/2305.15805>

**Scissorhands: Exploiting the Persistence of Importance Hypothesis for LLM KV Cache Compression at Test Time**. Liu et al. NIPS'23\
High attention scores are persistent/consistent across steps; prior highly attended positions are likely important for future.\
Persist top cache entries by: 1) low-score (by a threshold) ratio across recent steps; 2) recent entries are always kept due to lack of information.\
<https://openreview.net/forum?id=JZfg6wGi6g>

**SnapKV: LLM Knows What You are Looking for Before Generation**. Li et al. NIPS'24\
Similar to Scissorhands: aggregate attention scores across recent steps, then select top.\
To prevent info loss: use pooling on raw positions to identify important areas, rather than individual positions.\
<https://openreview.net/forum?id=poE54GOq2l>

**H2O: Heavy-Hitter Oracle for Efficient Generative Inference of Large Language Models**. Zhang et al. NIPS'23\
Similar to Scissorhands: aggregate attention scores across all history steps, then evict.\
<https://openreview.net/forum?id=RkRrPp7GKO>

**Efficient Streaming Language Models with Attention Sinks**. Xiao et al. ICLR'24\
Initial positions are attention sink.\
Retaining KV: initial positions + sliding window. Deal with position encoding.\
<https://openreview.net/forum?id=NG7sS51zVF>

**D2O : Dynamic Discriminative Operations for Efficient Generative Inference of Large Language Models**. Wan et al. ICLR'25\
KV pruning: Scissorhands + reserve attention sink.\
KV merging: cosine similarity by key vectors.\
<https://openreview.net/forum?id=HzBfoUdjHt>

**Model Tells You What to Discard: Adaptive KV Cache Compression for LLMs**. Ge et al. ICLR'24\
Attention head possesses different distribution patterns, which is also consistent across positions.\
Thus, able to prune attention heads adaptively (per head per layer).\
<https://openreview.net/forum?id=uNrFpDPMyo>

**PyramidKV: Dynamic KV Cache Compression based on Pyramidal Information Funneling**. Cai et al. COLM'25\
<https://openreview.net/forum?id=ayi7qezU87>

**Twilight: Adaptive Attention Sparsity with Hierarchical Top-p Pruning**. Lin et al. NIPS'25\
<https://openreview.net/forum?id=Ve693NkzcU>

**Free(): Learning to Forget in Malloc-Only Reasoning Models**. Zheng et al. 2026\
Iteratively switching between reasoning and cleaning modes, Free()LM dynamically identifies and prunes useless context chunks.\
<https://arxiv.org/abs/2602.08030>

**Neural Garbage Collection: Learning to Forget while Learning to Reason**. Li et al. 2026\
Scoring: attention weight; selection by top.\
Make key cache selection learnable in grpo (same reward).\
<https://arxiv.org/abs/2604.18002>

**Attention Sinks and Compression Valleys in LLMs are Two Sides of the Same Coin**. Enrique  et al. ICLR'26\
<https://openreview.net/forum?id=c5TFhCJ6fs>

**The Spike, the Sparse and the Sink: Anatomy of Massive Activations and Attention Sinks**. Sun et al. 2026\
<https://arxiv.org/abs/2603.05498>

#### Cache Merging

**MiniCache: KV Cache Compression in Depth Dimension for Large Language Models**. Liu et al. 2024\
<https://arxiv.org/abs/2405.14366>

**Dynamic Memory Compression: Retrofitting LLMs for Accelerated Inference**. Nawrot et al. ICML'24\
Sublinear KV cache: whether to append the current key and value representations to the cache or to perform a weighted average.\
<https://openreview.net/forum?id=tDRYrAkOB7>

**COMI: Coarse-to-fine Context Compression via Marginal Information Gain**. Tang et al. ICLR'26\
<https://openreview.net/forum?id=OGDIXDfaN4>

## KV-Cache Reuse

Utilize independent KV cache blocks (e.g. each document) in one context, resolving dependency.

**TurboRAG: Accelerating Retrieval-Augmented Generation with Precomputed KV Caches for Chunked Text**. Lu et al. EMNLP'25\
(i) re-order position embedding in KV; (ii) finetune with the same format as test (independent attention blocks), without addressing cross-dependency.\
<https://aclanthology.org/2025.emnlp-main.334>

**KVLink: Accelerating Large Language Models via Efficient KV Cache Reuse**. Yang et al. NIPS'25\
Append special tokens for each doc, which encodes all preceding context (anchor to address cross-dependency). Also re-order position embedding.\
A small forward pass is needed for special tokens with custom attention.\
<https://openreview.net/forum?id=oDcAGSXZZP>

## Efficient Cache/Attention

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

**RetrievalAttention: Accelerating Long-Context LLM Inference via Vector Retrieval**. Liu et al. 2025\
KV cache as memory: retrieve most similar K upon Q (topk), but not full search; using approximate search e.g. cluster-based for sublinear retrieval.\
Cache Retrieval vs. Pruning: retrieval is dynamic, as each latest Q obtains different set of K, due to observation that different Q has different set of similar K, as generation goes\
Novel: Q does not directly retrieve well with cluster-based K index, as K index does not see Q distribution\
Method: regard as few-shot problem, use existing Q distribution as anchor to find the closest K, instead of finding K directly\
<https://arxiv.org/abs/2409.10516>

## Others

**Language Modeling Is Compression**. Deletang et al. ICLR'24\
Adapt LLM to compress general data.\
<https://openreview.net/forum?id=jznbgiynus>
