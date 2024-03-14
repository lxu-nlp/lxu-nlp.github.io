---
layout: post
title: "Works on Structure Mining"
date: 2024-03-10
categories: [NLP]
tags: [nlp]
math: true
---

## Use RL

My BioNLP work.

**Struct-XLM: A Structure Discovery Multilingual Language Model for Enhancing Cross-lingual Transfer through Reinforcement Learning**. Wu and Lu. EMNLP'23\
Binary action for constitute segmentation.\
REINFORCE.\
<https://aclanthology.org/2023.emnlp-main.207>

## Unsupervised

**Sentence Centrality Revisited for Unsupervised Summarization**. Zheng and Lapata. ACL'19\
Sentence-pair score by defined similarity.\
<https://aclanthology.org/P19-1628/>

**Discourse-Aware Unsupervised Summarization of Long Scientific Documents**. Dong et al. EACL'21\
Interpolate sentence-pair score with within-section and inter-section similarity.\
<https://aclanthology.org/2021.eacl-main.93>

**Unsupervised Extractive Summarization using Pointwise Mutual Information**. Padmakumar et al. EACL'21\
Sentence-pair score: pointwise mutual information.\
Sequentially select sentences of more PMI with doc (more relevance), and less PMI with existing selection (less redundancy).\
<https://aclanthology.org/2021.eacl-main.213>
