---
layout: post
title: "Works on Retrieval"
date: 2026-04-01
categories: [nlp]
tags: [nlp, LLM, retrieval]
math: true
---


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
