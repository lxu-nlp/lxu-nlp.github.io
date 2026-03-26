---
layout: post
title: "Works on LLM Latent Training"
date: 2026-03-21
categories: [LLM]
tags: [nlp, LLM, latent, training]
math: true
---

Also see the [post](https://lxu-nlp.github.io/posts/basics-latent-modeling/) on latent variable modeling.

# Latent Training

**Latent Retrieval for Weakly Supervised Open Domain Question Answering**. Lee et al. ACL'19\
Retriever as latent variable; joint training for QA.\
<https://aclanthology.org/P19-1612/>

**REALM: retrieval-augmented language model pre-training**. Guu et al. ICML'20\
Retriever as latent variable in pretraining on generative masked language modeling.\
<https://dl.acm.org/doi/abs/10.5555/3524938.3525306>

**Retrieval-augmented generation for knowledge-intensive NLP tasks**. Lewis et al. NIPS'20\
RAG: joint training on retriever and generation.\
<https://dl.acm.org/doi/abs/10.5555/3495724.3496517>

**REPLUG: Retrieval-Augmented Black-Box Language Models**. Shi et al. NAACL'24\
Retriever + Frozen generator: adapt retrieval that improve LM perplexity.\
<https://aclanthology.org/2024.naacl-long.463>
