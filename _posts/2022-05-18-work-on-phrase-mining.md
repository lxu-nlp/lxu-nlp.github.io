---
layout: post
title: "Works on Phrase & Attribute Mining"
date: 2022-05-18
categories: [NLP]
tags: [nlp, phrase mining]
math: true
---

## Phrase/Attribute Mining

Loosely similar to open-domain NER.\
Usually in unsupervised or weak/distant supervised setting.

**Automated Phrase Mining from Massive Text Corpora**. Shang et al. TKDE'18\
Distant-supervised using KB + POS tagging as second source.\
(1) utilize KB as positive pool, filtered ngram as large but noisy negative pool.\
(2) use random forest to conduct positive-only distant training to reduce noise.\
(3) pos segmentation to assist.\
<https://arxiv.org/abs/1702.04457>

**Are Pre-trained Language Models Aware of Phrases? Simple but Strong Baselines for Grammar Induction**. Kim et al. ICLR'20\
Unsupervised w/o training: compute adjacent token distance from embedding & attention, then induce tree.\
<https://arxiv.org/abs/2002.00737>

**UCPhrase: Unsupervised Context-aware Quality Phrase Tagging**. Gu et al. KDD'21\
Unsupervised.\
(1) Obtain silver labels unsupervisedly by heuristics w/o KB for better domain generalization\
(2) Use attention-map as pure features w/o embedding to avoid surface form memorizing\
Ablation shows attention-map-based classification is more improvement.\
<http://arxiv.org/abs/2105.14078>

**AutoName: A Corpus-Based Set Naming Framework**. Huang et al. SIGIR'21\
Task: name a set of given entities unsupervisedly.\
(1) use BERT to probe hypernyms of an entity set based on masked templates.\
(2) use hypernyms as anchors to extract name candidates using POS patterns.\
(3) filter and group candidates by clustering; then perform ranking within a cluster.\
<http://doi.org/10.1145/3404835.3463100>

**Unsupervised Concept Representation Learning for Length-Varying Text Similarity**. Zhang et al. NAACL'21\
Use phrases to form concepts, leveraging \
<https://aclanthology.org/2021.naacl-main.445>

**Unsupervised Deep Keyphrase Generation.** Shen et al. AAAI'22\
Ideas not interesting and techniques are old.\
<https://arxiv.org/abs/2104.08729>

**Knowledge-guided Open Attribute Value Extraction with Reinforcement Learning**. Liu et al. EMNLP'20\
<https://aclanthology.org/2020.emnlp-main.693>

---

[Key Phrase Extraction](https://mp.weixin.qq.com/s/j4aQ5_iIWxW4GgJ5wZ3_tA)

