---
layout: post
title: "Works on Few Shot and Meta Learning"
date: 2022-02-19
categories: [NLP]
tags: [nlp, few-shot, meta learning, ML]
math: true
---

Some online resources:

* [Meta-Learning: Learning to Learn Fast](https://lilianweng.github.io/lil-log/2018/11/30/meta-learning.html)
* [few-shot learning and meta-learning](https://www.borealisai.com/en/blog/tutorial-2-few-shot-learning-and-meta-learning-i/)


# Metric/Similarity Learning

Dataset are divided into:

* Support Set $S$: N classes with K examples of each, noted as N-way K-shot classification task.
* Query Set or Prediction Set $Q$: used for training and testing

Core: learn similarity/distance and use the support set as the proxy.


**Siamese Neural Networks for One-shot Image Recognition**. Koch et al. ICML Workshop'15\
Siamese network:\
Training: encode $x_i \in S$ and $x_j \in Q$ respectively; obtain similarity $a(x_i, x_j)$ by different distance metrics (L1, L2, cosine, etc.); compute loss by cross-entropy.\
Inference: $\text{argmax}_{x_i \in S} a(x_i, x_j)$ for input $x_j$ (nearest neighbors on learned embedding space).\
<http://www.cs.toronto.edu/~rsalakhu/papers/oneshot1.pdf>

**Deep metric learning using Triplet network**. Hoffer and Ailon. SIMBAD'15\
Triplet network: similar to Siamese network, but with $x_+, x_-, x_j$ as input and use contractive or pairwise comparison loss.\
<https://arxiv.org/abs/1412.6622>

**Matching Networks for One Shot Learning**. Vinyals et al. NIPS'16\
Matching network: similar to Siamese network; however, instead of using argmax at inference, we optimize the weighted sum of labels over the entire support set $y' = \sum_{i} a(x_i, x_j)y_i, x_i \in S$ towards ground truth, closing the gap between training and inference.\
Use cosine similarity and softmax in $a(x_i, x_j)$ (attended nearest neighbors on learned embedding space).\
<https://arxiv.org/abs/1606.04080>

**Prototypical Networks for Few-shot Learning**. Snell et al. NIPS'17\
Prototypical network: similar to Matching network, however: (1) obtain a centroid representation of each class by mean; (2) obtain probability distribution $P(y = c | x_j) = \text{softmax}_{x_i \in S} a(x_i, x_j)$ (3) optimize by cross-entropy (finding nearest class prototype/centroid by classification on learned embedding space).\
<https://arxiv.org/abs/1703.05175>

**Learning to Compare: Relation Network for Few-Shot Learning**. Sung et al. CVPR'18\
Relation network: combine Siamese network and Prototypical network: (1) learn the metric $a(x_i, x_j)$ instead of using pre-defined metrics (2) use one centroid representation for each class (by sum or mean) (3) optimize the metric score $a(x_i, x_j)$ by MSE (can still be cross-entropy though).\
<https://arxiv.org/abs/1711.06025>


# Meta Learning

**Model-Agnostic Meta-Learning for Fast Adaptation of Deep Networks**. Finn et al. ICML'17\
<https://arxiv.org/abs/1703.03400>

**On First-Order Meta-Learning Algorithms**. Nichol et al. 2018\
<https://arxiv.org/abs/1803.02999>

### in NLP Venue

**Discriminative Nearest Neighbor Few-Shot Intent Detection by Transferring Natural Language Inference**. Zhang et al. EMNLP'20\
Similar to Siamese network. Combine coarse retrieval for faster NN.\
Task: intent classification.\
<https://www.aclweb.org/anthology/2020.emnlp-main.411>

**Learn to Cross-lingual Transfer with Meta Graph Learning Across Heterogeneous Languages**. Li et al. EMNLP'20\
General cross-lingual transfer: apply metric-based meta-learning in downstream tasks to learn similarity of language pairs (language as task). Then use this similarity as a graph and perform GCN to refine node.\
<https://www.aclweb.org/anthology/2020.emnlp-main.179>

**Zero-Shot Cross-Lingual Transfer with Meta Learning**. Nooralahzadeh et al. EMNLP'20\
General cross-lingual transfer: apply optimization-based meta-learning to learn the parameters that can be finetuned rapidly on different languages.\
<https://www.aclweb.org/anthology/2020.emnlp-main.368>

**ContrastNet: A Contrastive Learning Framework for Few-shot Text Classification**. Chen et al. AAAI'22\ 
Main (supervised): pull same class instances together among $Q$ and $S$ (cluster by labels).\
Reg (unsupervised)?: (1) push difference tasks/classes separated from each other (2) push repr distant from each other.\
Inference: nearest-neighbor on entire $S$ (similar to Siamese).\
<https://aaai-2022.virtualchair.net/poster_aaai10254>

---

# Self-Learning (Pseudo Label)

**Uncertainty-aware Self-training for Text Classification with Few Labels**. Mukherjee and Awadallah. NIPS'20\
Normal SL process but (1) select by BALD (dropout entropy) (2) select with exploration (3) weight loss by uncertainty from teacher.\
<https://arxiv.org/abs/2006.15315>

**Contrast-enhanced Semi-supervised Text Classification with Few Labels**. Tsai et al. AAAI'22\
Based on Mukherjee'20 but (1) add loss to push predicted same class towards similar repr for smooth boundary (2) in that process, only use on high confident predicted pairs (3) apply
huberised loss for noise-agnostic training.\
Improvement is not that much though.\
<https://aaai-2022.virtualchair.net/poster_aaai4001>

## Generation

**Variational Inference for Training Graph Neural Networks in Low-Data Regime through Joint Structure-Label Estimation**. Lao et al. KDD'22\
For graph with few observed node labels and edges.\
(1) graph structure generator (2) node GNN. Co-learn on weak-supervised graph to handle both edges and node labels together.\
<https://dl.acm.org/doi/10.1145/3534678.3539283>
