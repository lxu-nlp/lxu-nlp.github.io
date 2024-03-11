---
layout: post
title: "Works on Heterogeneous Graph"
date: 2022-02-21
categories: [NLP]
tags: [nlp, graph, heterogeneous]
math: true
---

## GNN Basics

Ways of node updating: (1) simply use aggregated (with self connection) (2) simply add residual connection (3) use
gating (4) use FFNN on [old; new].

Multiple iterations/propagation: higher-order/multi-hops. However, if complete-graph, then no need to do higher-order.

## Add More Nodes (of Different Types) as Heterogeneous
w
**Connecting the Dots: Document-level Neural Relation Extraction with Edge-oriented Graphs**. Christopoulou et al. EMNLP'19\
Add and connect **latent** nodes of different types: entities, sentences; then perform graph propagation. Added latent nodes only serve for information flow (better original node updating), and won't be used for final classification directly.\
<https://aclanthology.org/D19-1498/>

**Incorporating Syntax and Semantics in Coreference Resolution with Heterogeneous Graph Attention Network**. Jiang and Cohn.
NAACL'21\
Add and connect latent nodes of different types: SRL arguments & predicates; then perform graph propagation at a certain order.\
<https://aclanthology.org/2021.naacl-main.125>

**Heterogeneous Graph Neural Networks for Extractive Document Summarization**. Wang et al. ACL'20\
Add and connect latent nodes of different types: sentences, documents; then perform graph propagation at a certain order. Edges to latent nodes: assign weight by TF-IDF.\
<https://aclanthology.org/2020.acl-main.553>

**Extractive Summarization Considering Discourse and Coreference Relations based on Heterogeneous Graph**. Huang and
Kurohashi. EACL'21\
Add and connect latent nodes of different types: discourse units, coreferent entities, sentences; then perform graph propagation.\
<https://aclanthology.org/2021.eacl-main.265>

## Same Nodes with Multiple Subgraphs as Heterogeneous

**Heterogeneous Graph Transformer for Graph-to-Sequence Learning**. Yao et al. ACL'20\
Decompose original graphs into different subgraphs and perform graph propagation for each subgraph. Then, concat each node repr of different subgraphs and then transform to a single node repr.\
<https://aclanthology.org/2020.acl-main.640>

**Heterogeneity-aware Twitter Bot Detection with Relational Graph Transformers**. Feng et al. AAAI'22\
GAT per heterogeneous graph, then aggregate each node across graphs.\
<https://arxiv.org/pdf/2109.02927>
