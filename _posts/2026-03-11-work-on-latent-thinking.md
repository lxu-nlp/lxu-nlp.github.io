---
layout: post
title: "Works on LLM Latent Thinking and Generation"
date: 2026-03-11
categories: [NLP]
tags: [nlp, LLM, training]
math: true
---


# Latent Thinking

**Let’s Think Dot by Dot: Hidden Computation in Transformer Language Models**. Pfau et al. COLM'24\
Add fixed-length DOT token as thinking.\
<https://openreview.net/forum?id=NikbrdtYvG>

**Think before you speak: Training Language Models With Pause Tokens**. Goyal et al. ICLR'24\
Add fixed-length PAUSE token to pretraining+SFT+inference to allow for planning and compression.\
Note: injecting in pretraining is important; otherwise little improvement.\
<https://openreview.net/forum?id=ph04CRkPdC>

**Soft Thinking: Unlocking the Reasoning Potential of LLMs in Continuous Concept Space**. Zhang et al. NIPS'25\
Each thinking step: input an embedding weighted by last step vocab prob.\
Termination: when exceeding consecutive steps of low entropy.\
<https://openreview.net/forum?id=ByQdHPGKgU>

**Training Large Language Models to Reason in a Continuous Latent Space**. Hao et al. 2025\
Multi-stage training: step by step replacing explicit partial thinking to latent token, till all latent.\
<https://arxiv.org/abs/2412.06769>

**Compressed Chain of Thought: Efficient Reasoning through Dense Representations**. Cheng et al. 2024\
Distill to match interleaved hidden states.\
<https://arxiv.org/abs/2412.13171>

**CODI: Compressing Chain-of-Thought into Continuous Space via Self-Distillation**. Shen et al. EMNLP'25\
Distill to match the hidden state of last thinking token.\
<https://aclanthology.org/2025.emnlp-main.36>

**Latent Chain-of-Thought as Planning: Decoupling Reasoning from Verbalization**. Wang et al. 2026\
<https://arxiv.org/abs/2601.21358>
