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

## Entailment Graph

**Global Learning of Focused Entailment Graphs**. Berant et al. ACL'10\
Entailment graph construction.\
<https://aclanthology.org/P10-1124>

**Efficient Tree-based Approximation for Entailment Graph Learning**. Berant et al. ACL'12\
<https://aclanthology.org/P12-1013/>

**Entailment Graph Learning with Textual Entailment and Soft Transitivity**. Chen et al. ACL'22\
<https://aclanthology.org/2022.acl-long.406>

**From Key Points to Key Point Hierarchy: Structured and Expressive Opinion Summarization**. Cattan et al. ACL'23\
Directional relation graph construction: 1) pairwise scores: NLI; distributional context similarity; 2) edge building by score threshold.\
<https://aclanthology.org/2023.acl-long.52>


