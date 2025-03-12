---
layout: post
title: "Works on Embedding Representation"
date: 2025-03-12
categories: [NLP]
tags: [nlp, embedding]
math: true
---

## Embedding by Non-LLM

TODO

## Embedding by LLM

**LLM2Vec: Large Language Models Are Secretly Powerful Text Encoders**. BehnamGhader et al. COLM'24\
<https://arxiv.org/pdf/2404.05961>

## Other Representation by LLM

**Scaling Sentence Embeddings with Large Language Models**. Jiang et al. 2023\
Use template for CLM (using the last hidden state):
This sentence: \[text\] means in one word:\
<http://arxiv.org/abs/2307.16645>

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
