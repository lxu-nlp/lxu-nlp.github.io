---
layout: post
title: "Works on GNN"
date: 2022-10-24
categories: [NLP]
tags: [nlp, GNN, graph]
math: true
---

# Basic GNN

[Laplacian matrix normalization](https://en.wikipedia.org/wiki/Laplacian_matrix#Laplacian_matrix_normalization)
* Each edge is normalized by the degrees of both two nodes.

Messaging passing: $H^{t+1} = AH^tW$
* $A$ is the normalized adjacency matrix (with self-connection)
* $H^t W$ is the node representation

Adding node self-attention: GAT

* Graph inductive learning: train and test are on DIFFERENT graphs; model should generalize to unseen graphs.
* Graph transductive learning: train and test are on the SAME graphs; model only need to handle node labels.

Ways of node updating: (1) simply use aggregated (with self connection) (2) simply add residual connection (3) use
gating (4) use FFNN on [old; new].

Multiple iterations/propagation: higher-order/multi-hops. However, if complete-graph, then no need to do higher-order.

# General Methods

**SEMI-SUPERVISED CLASSIFICATION WITH GRAPH CONVOLUTIONAL NETWORKS**. Kipf and Welling. ICLR'17\
GNN.\
<https://arxiv.org/abs/1609.02907>

**COMPOSITION-BASED MULTI-RELATIONAL GRAPH CONVOLUTIONAL NETWORKS**. Vashishth et al. ICLR'20\
Incorporate edge types: use composition (start node, edge) to represent end node in the messaging passing.\
Benefits: (1) no need to edge-type specific messaging passing, naturally integrate edge types in one pass (2) can use existing edge representation\
<https://arxiv.org/abs/1911.03082v2>

# Heterogeneous Graph

### Add More Nodes (of Different Types) as Heterogeneous
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

### Same Nodes with Multiple Subgraphs as Heterogeneous

**Heterogeneous Graph Transformer for Graph-to-Sequence Learning**. Yao et al. ACL'20\
Decompose original graphs into different subgraphs and perform graph propagation for each subgraph. Then, concat each node repr of different subgraphs and then transform to a single node repr.\
<https://aclanthology.org/2020.acl-main.640>

**Heterogeneity-aware Twitter Bot Detection with Relational Graph Transformers**. Feng et al. AAAI'22\
GAT per heterogeneous graph, then aggregate each node across graphs.\
<https://arxiv.org/pdf/2109.02927>

