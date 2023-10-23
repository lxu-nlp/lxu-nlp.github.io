---
layout: post
title: "Work on GNN"
date: 2022-10-24
categories: [NLP]
tags: [nlp, gnn]
math: true
---

## Basic GNN

[Laplacian matrix normalization](https://en.wikipedia.org/wiki/Laplacian_matrix#Laplacian_matrix_normalization)
* Each edge is normalized by the degrees of both two nodes.

Messaging passing: $H^{t+1} = AH^tW$
* $A$ is the normalized adjacency matrix (with self-connection)
* $H^t W$ is the node representation

Adding node self-attention: GAT

* Graph inductive learning: train and test are on DIFFERENT graphs; model should generalize to unseen graphs.
* Graph transductive learning: train and test are on the SAME graphs; model only need to handle node labels.

## Papers

**SEMI-SUPERVISED CLASSIFICATION WITH GRAPH CONVOLUTIONAL NETWORKS**. Kipf and Welling. ICLR'17\
GNN.\
<https://arxiv.org/abs/1609.02907>

**COMPOSITION-BASED MULTI-RELATIONAL GRAPH CONVOLUTIONAL NETWORKS**. Vashishth et al. ICLR'20\
Incorporate edge types: use composition (start node, edge) to represent end node in the messaging passing.\
Benefits: (1) no need to edge-type specific messaging passing, naturally integrate edge types in one pass (2) can use existing edge representation\
<https://arxiv.org/abs/1911.03082v2>
