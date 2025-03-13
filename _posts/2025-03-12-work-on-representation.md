---
layout: post
title: "Works on Embedding Representation"
date: 2025-03-12
categories: [NLP]
tags: [nlp, embedding]
math: true
---

## Benchmark

**MMTEB: Massive Multilingual Text Embedding Benchmark**. Enevoldsen et al. ICLR'25\
<https://openreview.net/forum?id=zl3pfz4VCV>

---

## Embedding by Non-LLM

**Text and Code Embeddings by Contrastive Pre-Training**. Neelakantan et al. 2022\
OpenAI embedding: w/ unsupervised pretraining.\
<https://arxiv.org/abs/2201.10005>

**Text Embeddings by Weakly-Supervised Contrastive Pre-training**. Wang et al. 2022\
E5 model.\
<https://arxiv.org/abs/2212.03533>

**Multilingual E5 Text Embeddings: A Technical Report**. Wang et al. 2023\
mE5.\
<https://arxiv.org/pdf/2402.05672>

**Towards General Text Embeddings with Multi-stage Contrastive Learning**. Li et al. 2023\
GTE model.\
<https://arxiv.org/abs/2308.03281>

**mGTE: Generalized Long-Context Text Representation and Reranking Models for Multilingual Text Retrieval**. Zhang et al. EMNLP Industry'24\
GTE with enhanced Transformers: match BGE-M3.\
<https://aclanthology.org/2024.emnlp-industry.103>

## Embedding by LLM

**LLM2Vec: Large Language Models Are Secretly Powerful Text Encoders**. BehnamGhader et al. COLM'24\
<https://arxiv.org/pdf/2404.05961>

## Other Representation by LLM

**Scaling Sentence Embeddings with Large Language Models**. Jiang et al. 2023\
Use template for CLM (using the last hidden state):
This sentence: \[text\] means in one word:\
<https://arxiv.org/abs/2307.16645>

**Meaning Representations from Trajectories in Autoregressive Models**. Liu et al. ICLR'24\
Core idea: Wittgenstein’s use theory of meaning (meaning is use).\
Meaning representation: the likelihood of the continuation in language space.\
Similarity: sampling continuation equal times from both two inputs as the approximated continuation space.\
<https://arxiv.org/abs/2310.18348>

**BeLLM: Backward Dependency Enhanced Large Language Model for Sentence Embeddings**. Li and Li. NAACL'24\
Bidirectional on last layer + contrastive.\
<https://arxiv.org/pdf/2311.05296>

## Embedding Privacy

**Sentence Embedding Leaks More Information than You Expect: Generative Embedding Inversion Attack to Recover the Whole Sentence**. Li et al. ACL Findings'23\
<https://aclanthology.org/2023.findings-acl.881/>

**Text Embeddings Reveal (Almost) As Much As Text**. Morris et al. EMNLP'23\
Train a generator to recover input of its embedding from a black-box encoder.\
(which can be a strong argument that auto-encoder != semantic comprehension, as embedding performance is not perfect on downstream tasks)\
<https://aclanthology.org/2023.emnlp-main.765>
