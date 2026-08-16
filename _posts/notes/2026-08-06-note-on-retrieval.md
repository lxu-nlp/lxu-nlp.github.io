---
layout: post
title: "Notes on Retrieval"
date: 2026-08-06
categories: [Notes]
tags: [retrieval]
math: true
hidden: true
---

## Dense Retrieval

Hard negative design:
- By badcases: **imperfect world knowledge**
  - Restrict topic generalization (for precision)
  - Add paraphrasing and semantic meaning (for recall)
  - Enable taxonomy: add hypernym
- By badcases: **imperfect phrasing concept**
  - For fine-grained char (big problem for zh): create nonsense queries to prevent char overlap
  - For fine-grained span: query corruption (concat, replacement, preceding)
- For **covering multiple keywords** (challenge: uncertain AND or OR intent)
  - Add positives by keyword combination (regard as AND)
  - Add negatives by overlapping keyword (regard as AND)

Loss:
- Contrastive
- Tried margin loss: a least-gap between POS and NEG
  - Problem: less phrasing; more char match
- Divergence loss (using a soft GT distribution from another model)
- Matryoshka Representation Learning (MRL): enable nested successive-refinement property
  - Sample multiple prefix representation and do loss on the same label

## Sparse Retrieval

E.g. SPLADE: 每个 token 位置都过一遍 MLM head 得到 vocab logits，then max-pool over seq
```python
z = log(1 + relu(mlm_logits))          # 形状 [L, V]        # L=序列长度, V=词表大小(~30k)
v = maxpool(z, over_seq_len)           # 只要模型在文本任何一处觉得词 t 相关，t 就拿到一个权重
```

Properties:
- Inference: 1) either cosine similarity; 2) or as a generalized BM25
- Sparsity: 使用类似 L2 norm of activated tokens in a batch
- vs. Dense: fine-grained friendly

## Late Interaction

Colbert: max-sim
- Can be borrowed when multi-feature (original is token-level max-sim)

## Ensemble/Fusion

Reciprocal Rank Fusion: 奖励跨系统的共识 / 鲁棒性，而不是某个系统的单点第一
- 只看排名位置，不碰分数（避免分数归一化操作）

$$RRF(d) = Σ_{r=1}^{R}  1 / (k + rank_r(d))$$

## Metrics

- MRR
- nDCG
- Recall@k
- MAP

## Faiss

Modes:
- Flat: exact
- IVF（检索子空间）: 使用 k-means cluster；检索时取最近的 n 个 cluster（query 与 centroid 距离），只在这 n 个 cluster 内做精确计算
- PQ (表征层 Product Quantization): 把向量切成 M 段，对每一段独立跑 k-means 学 k 个 cluster，每段用一个标量「最近的 cluster 编号 0-k」表示
- HNSW: 图直接走到近邻，coarse to fine
- LSH (Locality-Sensitive Hashing): less effective
