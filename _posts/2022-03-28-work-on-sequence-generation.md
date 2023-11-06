---
layout: post
title: "Works on Sequence Generation"
date: 2022-03-28
categories: [NLP]
tags: [nlp, generation]
math: true
---

This post is for general generation techniques only; not application-specific.

## Pointer or Hybrid

**Get To The Point: Summarization with Pointer-Generator Networks**. See et al. ACL'17\
Probability: vocab + pointer (gated).\
<https://aclanthology.org/P17-1099>

## Non-Autoregressive

**NON-AUTOREGRESSIVE NEURAL MACHINE TRANSLATION**. Gu et al. ICLR'18\
Challenges: (1) proper nondeterministism (2) decoder initialization.\
<https://arxiv.org/pdf/1711.02281>

**ELMER: A Non-Autoregressive Pre-trained Language Model for Efficient and Effective Text Generation**. Li et al. EMNLP'22\
<https://arxiv.org/pdf/2210.13304>

## Training: Provide Negative Samples (in addition to MLE on Gold)

**NEURAL TEXT DEGENERATION WITH UNLIKELIHOOD TRAINING**. Welleck et al. ICLR'20\
Determine negative tokens at each step and minimize likelihood.\
Heuristics: use previous context tokens as negatives to avoid repentance.\
<https://openreview.net/pdf?id=0qSOodKmJaN>

**BRIO: Bringing Order to Abstractive Summarization**. Liu et al. ACL'22\
Scoring seq to calibrate good/bad: ROUGE with gold.\
Margin loss on sampled generations, with margin scaled by scoring.\
<https://aclanthology.org/2022.acl-long.207>

**CALIBRATING SEQUENCE LIKELIHOOD IMPROVES CONDITIONAL LANGUAGE GENERATION**. Zhao et al. ICLR'23\
Similar to BRIO but more comprehensive experiments.\
Margin loss on sampled generations; relative ordering per scores performs better than continuous scores directly.\
<https://openreview.net/pdf?id=0qSOodKmJaN>

**Click: Controllable Text Generation with Sequence Likelihood Contrastive Learning**. Zheng et al. ACL Findings'23\
Margin loss on negative sequence likelihood; with method to sample negative without gold references.\
<https://aclanthology.org/2023.findings-acl.65>
