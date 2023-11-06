---
layout: post
title: "Works on Graph Encoding (NLP Venues)"
date: 2022-02-17
categories: [NLP]
tags: [nlp, graph]
math: true
---

Also see another post regarding heterogeneous encoding.

## Model Hierarchical Structure

**Document Modeling with Graph Attention Networks for Multi-grained Machine Reading Comprehension**. Zheng et al. ACL'20\
Self-attention on same level + GAT on adjacent level.\
<https://aclanthology.org/2020.acl-main.599/>

## Model Linguistic Structure

**Neural Machine Translation with Source-Side Latent Graph Parsing**. Hashimoto and Tsuruoka. EMNLP'17\
Learn soft graph (complete graph) for dependency.\
<https://www.aclweb.org/anthology/D17-1012/>

**Neural Machine Translation with Source Dependency Representation**. Chen et al. EMNLP'17\
Syntax graph: concat as features; CNN for feature extraction.\
<https://www.aclweb.org/anthology/D17-1304/>

**Neural AMR: Sequence-to-Sequence Models for Parsing and Generation**. Konstas et al. ACL'17\
AMR graph: linearized representation.\
<https://www.aclweb.org/anthology/P17-1014/>

**A Graph-to-Sequence Model for AMR-to-Text Generation**. Song et al. ACL'18\
AMR graph: GraphLSTM\
<https://www.aclweb.org/anthology/P18-1150/>

**Graph-to-Sequence Learning using Gated Graph Neural Networks**. Beck et al. ACL'18\
AMR graph: Levi graph + Gated GNN\
<https://www.aclweb.org/anthology/P18-1026>

**Densely Connected Graph Convolutional Networks for Graph-to-Sequence Learning**. Guo et al. ACL'19\
AMR graph: extended Levi graph + GAN with residual connection on all layers\
<https://www.aclweb.org/anthology/Q19-1019/>

**Self-Attention with Relative Position Representations**. Shaw et al. NAACL'18\
First work on encoding node relations in self-attention.\
<https://www.aclweb.org/anthology/N18-2074/>

**Transformer-XL: Attentive Language Models beyond a Fixed-Length Context**. Dai et al. ACL'19\
Enhanced work on encoding node relations in self-attention.\
<https://www.aclweb.org/anthology/P19-1285/>

**Structural Neural Encoders for AMR-to-text Generation. Damonte and Cohen**. NAACL'19\
AMR graph: stack TreeLSTM and GCN encoders\
<https://www.aclweb.org/anthology/N19-1366>

**Modeling Graph Structure in Transformer for Better AMR-to-Text Generation**. Zhu et al. EMNLP'19\
AMR graph: apply structure-aware self-attention (Shaw'18) on graph + different ways to encode paths between nodes.\
<https://www.aclweb.org/anthology/D19-1548>

**Graph Transformer for Graph-to-Sequence Learning**. Cai and Lam. AAAI'20\
AMR graph: apply structure-aware self-attention (Shaw'18) on graph + LSTM to encode paths between nodes.\
<https://arxiv.org/pdf/1911.07470>

**Heterogeneous Graph Transformer for Graph-to-Sequence Learning**. Yao et al. ACL'20\
AMR graph: build heterogeneous graphs and do attention masking in self-attention for each graph; concat each representation.\
vhttps://www.aclweb.org/anthology/2020.acl-main.640>

**SG-Net: Syntax-Guided Machine Reading Comprehension**. Zhang et al. AAAI'20\
Syntax graph: do attention masking in self-attention.\
<https://arxiv.org/pdf/1908.05147>

**Encoding Syntactic Knowledge in Transformer Encoder for Intent Detection and Slot Filling**. Wang et al. AAAI'21\
Syntax graph: add prediction of ancestors in self-attention (additional attention head), with KL loss between predicted logits and logits from another parser.\
<https://arxiv.org/abs/2012.11689>
