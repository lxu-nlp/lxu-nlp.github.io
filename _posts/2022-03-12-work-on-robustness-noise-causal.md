---
layout: post
title: "Works on Robustness & Noise Reduction & Causality"
date: 2022-03-12
categories: [NLP]
tags: [nlp, robustness]
math: true
---

Directions: 1) data: augmentation, adversarial; 2) training: ensemble, noise-resilient Loss; 3) model: causality/reasoning robustness

In training, model learns pattern in a greedy fashion: from easy to hard, based on the data distribution (hard is usually inconsistent/wrong/long-tail).\
At early stage, model learns general patterns, as they are easier to bias against initially, largely reducing loss based on data;\
At later stage, model will memorize (overfit to) the hard labels to further reduce loss; noisy labels can be regarded as part of the hard labels.

## Noise-Awareness: Label Correction from Co-Training

**Co-teaching: Robust Training of Deep Neural Networks with Extremely Noisy Labels**. Han et al. NIPS'18\
Each model has different bias (especially for noise).\
Co-teaching: select small-loss instances as clean instances and feed to the other model.\
<https://arxiv.org/pdf/1804.06872>

**CrossWeigh: Training Named Entity Tagger from Imperfect Annotations**. Wang et al. EMNLP'19\
Similar to k-fold CV: find disagreement and down-weight those samples.\
<https://aclanthology.org/D19-1519>

## Noise-Awareness: Regularization from Co-Training, Ensemble

**Learning from Noisy Labels for Entity-Centric Information Extraction**. Zhou and Chen. EMNLP'21\
Leverage observations that noise labels (1) need more steps to capture (2) easily forgotten in later epochs; therefore, models tend to disagree on noisy labels throughout training.\
Add regularization in co-training: obtain an averaged "soft" prediction as a true (smoother) label distribution, instead of a one-hot label (KL-divergence as loss).\
<https://aclanthology.org/2021.emnlp-main.437>

## Noise-Awareness: Noise-Resilient Loss

Incorporate data-uncertainty: see uncertainty post.

**Generalized Cross Entropy Loss for Training Deep Neural Networks with Noisy Labels**. Zhang et al. NIPS'18\
Generalization of CE and MAE loss to achieve balance between learning and noise-robustness.\
MAE: noise-robust but harder to learn; CE: effective learning but overfit to noisy samples.\
<http://arxiv.org/abs/1805.07836>

**Symmetric Cross Entropy for Robust Learning with Noisy Labels**. Wang et al. ICCV'19\
Adding a symmetric term to CE for better noise-resilience.\
<https://arxiv.org/abs/1908.06112>

## Robustness by Augmentation

**Distantly-Supervised Named Entity Recognition with Noise-Robust Learning and Language Model Augmented Self-Training**. Meng et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.810>

**AEDA: An Easier Data Augmentation Technique for Text Classification**. Karimi et al. EMNLP Findings'21\
<https://aclanthology.org/2021.findings-emnlp.234>

## Robustness by Adversarial Attacking

**Contextualized Perturbation for Textual Adversarial Attack**. Li et al. NAACL'21\
Contextualized word perturbations of replacement, insertion, merging, by MLM mask-and-fill; positions selected greedily that cause most degradation on gold likelihood.\
Half selected positions are noun phrases; train with augmentation can better defend attacking.\
<https://aclanthology.org/2021.naacl-main.400>

**Unifying Model Explainability and Robustness for Joint Text Classification and Rationale Extraction**. Li et al. AAAI'22\
Discrete word-replacement attack & embedding perturbation attack.\
<http://arxiv.org/abs/2112.10424>

## Robustness by Causality, Rationale, Logic

**Deep Probabilistic Logic: A Unifying Framework for Indirect Supervision**. Wang and Poon. EMNLP'18\
Utilizing external knowledge including (1) distant-supervision (2) constraints such as logic forms or rules.\
<https://aclanthology.org/D18-1215>

**Self-training with Few-shot Rationalization**. Bhat et al. EMNLP'21\
Integrate rationale span labels: 1) pos: consistent prediction on rationale span; 2) neg: max uncertain on non-rationale span.\
Use self-learning in semi-supervised settings, w/ a few rationale labels.\
<https://aclanthology.org/2021.emnlp-main.836>

**C2L: Causally Contrastive Learning for Robust Text Classification**. Choi et al. AAAI'22\
Improve robustness and generalization by identifying more causal features.\
(1) use gradient magnitude to identify candidate causal words\
(2) validate candidates by replacing with other plausible words (selected by BERT masking) and observing label change\
(3) push repr of non-causal replacement samples similar, so that encoding focuses more on the causal features\
<https://aaai-2022.virtualchair.net/poster_aaai11767>

**De-Bias for Generative Extraction in Unified NER Task**. Zhang et al. ACL'22\
Causal perspective for S2S NER: (1) pre-context confounder (2) entity order confounder.\
<https://aclanthology.org/2022.acl-long.59>

## Analysis: superficial clues

**Right for the Wrong Reasons: Diagnosing Syntactic Heuristics in Natural Language Inference**. McCoy et al. ACL'19\
NLI: hand-crafted syntactic patterns as superficial clues that previous models are shown leveraged.\
Data augmentation against those patterns can help performance.\
<https://aclanthology.org/P19-1334>

## Analysis: Noises

**Understanding and Mitigating the Label Noise in Pre-training on Downstream Tasks**. Chen et al. ICLR'24\
Noises help with in-domain (more robust for ID cases) but always hurt out-of-domain (less transferable).
<https://openreview.net/forum?id=TjhUtloBZU>
