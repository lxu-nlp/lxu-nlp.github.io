---
layout: post
title: "Works on Retrieval Internalization"
date: 2026-04-01
categories: [NLP]
tags: [nlp, LLM, retrieval]
math: true
---


# Internalize to Adapter

**Doc-to-LoRA: Learning to Instantly Internalize Contexts**. Charakorn et al. 2026\
Learn: input context -> lora weight, without actual lora training.\
Training: for each context, use LLM-generated queries.\
(since the meta model is shared across contexts and their queries, it mitigates query coverage?)\
<https://arxiv.org/abs/2602.15902>


# Generative Retrieval

Keys: 1) how to choose meaningful doc identifier; 2) how to generalize to unseen doc; 3) generalize to unseen query

**Transformer Memory as a Differentiable Search Index**. Tay et al. NIPS'22\
Indexing: doc -> doc identifier (memorization).\
Retrieval: query -> doc identifier.\
Interplay training between Indexing and Retrieval.\
<https://openreview.net/abs?id=Vu-B0clPfq>

**Autoregressive Search Engines: Generating Substrings as Document Identifiers**. Bevilacqua et al. NIPS'22\
SEAL: query -> doc ngrams.\
Use FM-Index to achieve ngrams <=> docs.\
<https://openreview.net/abs?id=Z4kZxAjg8Y>

**Generative Retrieval as Multi-Vector Dense Retrieval**. Wu et al. SIGIR'24\
<https://arxiv.org/abs/2404.00684>
