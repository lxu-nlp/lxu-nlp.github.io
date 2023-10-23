---
layout: post
title: "Work on Contrastive Learning"
date: 2022-03-18
categories: [NLP]
tags: [nlp, contrastive]
math: true
---

Primarily focusing on NLP domains.

[对比学习中的4种经典训练模式](https://mp.weixin.qq.com/s/rq-1m483bbG1xW88mw7CUA)

## Self-supervised Contrastive on Augmentation

Unsupervised augmentation as positive pairs; for representation learning or smoother encoding.

**Fast, Effective, and Self-Supervised: Transforming Masked Language Models into Universal Lexical and Sentence Encoders**. Liu et al. EMNLP'21\
Push sentence-similarity on augmented/modified sentences with BERT.\
<https://aclanthology.org/2021.emnlp-main.109>

**SimCSE: Simple Contrastive Learning of Sentence Embeddings**. Gao et al. EMNLP'21\
(1) push sentence-similarity on dropout sentences.\
(2) supervised contrastive label-space learning on NLI; utilizing NLI dataset is found effective.\
<https://aclanthology.org/2021.emnlp-main.552>

**Few-Shot Intent Detection via Contrastive Pre-Training and Fine-Tuning**. Zhang et al. EMNLP'21\
(1) self-supervised contrastive on target dataset for few-shot: Push sentence-similarity on augmented/modified sentences.\
(2) supervised contrastive on label space.\
<https://aclanthology.org/2021.emnlp-main.144>

## Supervised Contrastive on Label Space

Same supervised labels as positive pairs; for smoother representation towards decision boundary.

**Supervised Contrastive Learning**. Khosla et al. NIPS'20\
Foundation work that formulates multiple positive pairs (either through augmentation or supervised labels).\
<https://arxiv.org/abs/2004.11362>

**Supervised Contrastive Learning for Pre-trained Language Model Fine-tuning**. Gunel et al. ICLR'21\
Push representation similarity for same-class samples.\
Findings: (1) improve few-shot (2) more robust to noise, better generalization (3) improvement on general is dataset-specific.\
<https://arxiv.org/pdf/2011.01403>

**Not All Negatives are Equal: Label-Aware Contrastive Loss for Fine-grained Text Classification**. Suresh and Ong. EMNLP'21\
Add a weighted term in contrastive loss, with the weight being the confidence/probability predicted by another network,
incorporating label-awareness (confusable classes have higher weight, similar to foco loss).\
Note that no improvement for already easy classes; more effective for large number of similar classes.\
<https://aclanthology.org/2021.emnlp-main.359>

**Improved Text Classification via Contrastive Adversarial Training**. Pan et al. AAAI'22\
Adversarial training + contrastive training.\
Perturbation on word embedding matrix (determined by gradient); used as positive pair for contrastive as well.\ 
<https://arxiv.org/abs/2107.10137>
