---
layout: post
title: "Works on LLM Editing"
date: 2024-03-11
categories: [NLP]
tags: [nlp, model editing, LLM]
math: true
---

# Model Editing

Goal: 1) efficacy of editing; 2) efficacy of generalization; 3) no forgetting others & no neighbor perturbation

### Gradient-based

**Editing Factual Knowledge in Language Models**. De Cao et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.522/>

**Fast Model Editing at Scale**. Mitchell et al. ICLR'22\
<https://arxiv.org/abs/2110.11309>

Locate-then-edit:

**Transformer feed-forward layers are key-value memories**. Geva et al. EMNLP'21\
Argue that linear operations, e.g. MLP, can operate as a kv storage.\
<https://aclanthology.org/2021.emnlp-main.446/>

**Transformer feed-forward layers build predictions by promoting concepts in the vocabulary space**. Geva et al. EMNLP'22\
<https://aclanthology.org/2022.emnlp-main.3/>

**Locating and Editing Factual Associations in GPT**. Meng et al. NIPS'22\
A new kv pair can be inserted in MLP by solving a constrained least-squares problem.\
<https://arxiv.org/abs/2202.05262>

**Mass-Editing Memory in a Transformer**. Meng et al. ICLR'23\
<https://arxiv.org/abs/2210.07229>

**Editing models with task arithmetic**. Ilharco et al. ICLR'23\
<https://openreview.net/forum?id=6t0Kwf8-jrj>

### Cache-based

**Memory-Based Model Editing at Scale**. Mitchell et al. ICML'22\
Pseudo-editing: not touching parameters; simply store update examples, and train a binary classifier to check if input is related to any examples.\
<http://arxiv.org/abs/2206.06520>

### Evaluation

**Neighboring Perturbations of Knowledge Editing on Large Language Models**. Ma et al. ICML'24\
<https://arxiv.org/abs/2401.17623>

**Long-form evaluation of model editing**. Rosati et al. NAACL'24\
<https://aclanthology.org/2024.naacl-long.208/>
