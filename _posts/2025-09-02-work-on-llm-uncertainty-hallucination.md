---
layout: post
title: "Works on LLM Uncertainty and Hallucination"
date: 2025-09-02
categories: [NLP]
tags: [nlp, LLM, uncertainty]
math: true
---

## Hallucination

Also see post about Factuality.

**Do Large Language Models Know What They Don’t Know?**. Yin et al. 2023\
Dataset: answerable and unanswerable questions.\
<https://arxiv.org/abs/2305.18153>

**Halo: Estimation and Reduction of Hallucinations in Open-Source Weak Large Language Models**. Elaraby et al. 2023\
Use SUMMAC directly; new dataset constructed by GPT, with approach to reduce hallucination.\
<https://arxiv.org/abs/2308.11764>

**SELF-CONTRADICTORY HALLUCINATIONS OF LLMS: EVALUATION, DETECTION AND MITIGATION**. Mündler et al. 2023\
Induce LLM to generate two statements regarding the same context, then use another LLM to detect inconsistency.\
<https://arxiv.org/abs/2305.15852>

**CHAIN-OF-VERIFICATION REDUCES HALLUCINATION IN LARGE LANGUAGE MODELS**. Dhuliawala et al. 2023\
QA-based verification for detecting non-factual query response.\
<https://arxiv.org/pdf/2309.11495.pdf>

**DOLA: DECODING BY CONTRASTING LAYERS IMPROVES FACTUALITY IN LARGE LANGUAGE MODELS** Chuang et al. 2023\
<https://arxiv.org/pdf/2309.03883.pdf>


## Uncertainty

### via Hidden/Attention

**INSIDE: LLMs' Internal States Retain the Power of Hallucination Detection**. Chen et al. ICLR'24\
Metric: covariance det of averaged token hidden states of multiple sampled responses.\
<https://openreview.net/forum?id=Zj12nzlQbz>

**Detecting hallucinations in large language models using semantic entropy**. Farquhar et al. Nature'24\
<https://www.nature.com/articles/s41586-024-07421-0>

**LLM-Check: Investigating Detection of Hallucinations in Large Language Models**. Sriramanan et al. NIPS'24\
<https://openreview.net/forum?id=LYx4w3CAgy>

**Scalable Best-of-N Selection for Large Language Models via Self-Certainty**. Kang et al. NIPS'25\
Distribution-level per step: similar to entropy.\
<https://openreview.net/forum?id=29FRqmVQK8>

**Think Deep, Not Just Long: Measuring LLM Reasoning Effort via Deep-Thinking Tokens**. Chen et al. 2026\
<https://arxiv.org/abs/2602.13517>
