---
layout: post
title: "Works on Sentence-Level Parsing"
date: 2022-02-28
categories: [NLP]
tags: [nlp, parsing]
math: true
---

Sentence-level parsing or NER.

## General Structured Inference

Also see Information Extraction for generation-based approaches.

**Autoregressive Structured Prediction with Language Models**. Liu et al. EMNLP'22 Finding\
Sequence generation.\
<https://aclanthology.org/2022.findings-emnlp.70>

**Linear-Time Modeling of Linguistic Structure: An Order-Theoretic Perspective**. Liu et al. EMNLP'23\
Linear regression.\
<https://aclanthology.org/2023.emnlp-main.52>

## Span Selection

**A Structured Span Selector**. Liu et al. NAACL'22\
Model span selection as parsing selection: natural tree formulation as recursive partitioning.\
Selected spans: parsed nodes with NN mention score > 1 (and skip score < 1).\
Advantages: fully differentiable (by CYK, similar to CRF training and inference), rather than hard pruning.\
<https://aclanthology.org/2022.naacl-main.189>

## Task: Named Entity Recognition (NER)

**A Unified Generative Framework for Various NER Subtasks**. Yan et al. ACL'21\
S2S generation: pointer(index)-based.\
<https://aclanthology.org/2021.acl-long.451>

**Unified Named Entity Recognition as Word-Word Relation Classification**. Li et al. AAAI'22\
Novel formulation.\
<https://arxiv.org/pdf/2112.10070>


## Semantic Parsing (e.g. Semantic Role Labeling)

**Simpler but More Accurate Semantic Dependency Parsing**. Dozat and Manning. ACL'18\
(1) pairwise scoring; (2) edge by threshold (binary decision) for general graph dependency.\
<https://aclanthology.org/P18-2077>

**High-order Semantic Role Labeling**. Li et al. EMNLP Findings'20\
Designed path for second order (one-hop) flowing + iterative for higher order.\
Train with final higher order edge likelihood.\
<https://aclanthology.org/2020.findings-emnlp.102>

**Question-Answer Driven Semantic Role Labeling: Using Natural Language to Annotate Natural Language**. He et al. EMNLP'15\
SRL as QA: ask who/what/when/how questions regarding each verb/predicate to identify arguments.\
<https://aclanthology.org/D15-1076>

**Jointly Predicting Predicates and Arguments in Neural Semantic Role Labeling**. He et al. ACL'18\
Similar to Lee'17: enumerate and prune for both arguments and predicates; then perform (ARG x PRED) scoring on label/relation space.\
<https://aclanthology.org/P18-2058/>

**Syntax-aware Multilingual Semantic Role Labeling**. He et al. EMNLP'19\
<https://aclanthology.org/D19-1538/>

**Span-Based Semantic Role Labeling with Argument Pruning and Second-Order Inference**. Jia et al. AAAI'22\
Independently prune arguments first; then perform bilinear scoring between each words/predicates and arguments. Note that edges are predicted first, and labels are predicted separately (necessary?).\
Second-order helps little.\
<https://www.aaai.org/AAAI22Papers/AAAI-7985.JiaZ.pdf>

**Fast and Accurate End-to-End Span-based Semantic Role Labeling as Word-based Graph Parsing**. Zhou et al. COLING'22\
Good related work: span-based, word-based approaches.\
<https://aclanthology.org/2022.coling-1.365>

## Latent Discourse

**Pre-training Multi-party Dialogue Models with Latent Discourse Inference**. Li et al. ACL'23\
EM to adopt latent addressee (discourse relation) in multi-party dialogue without annotations.\
E: P(current response utterance | past history, current addressee); M: P(current addressee | past history, current response).\
<https://aclanthology.org/2023.acl-long.533>

## Others

**A Cross-Task Analysis of Text Span Representations**. Toshniwal et al. 2020\
Max Pooling ~= Endpoint ~= Coherent. Although Endpoint takes 2x size.\
<https://aclanthology.org/2020.repl4nlp-1.20>
