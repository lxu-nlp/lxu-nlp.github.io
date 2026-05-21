---
layout: post
title: "Works on LLM Latent Thinking and Generation"
date: 2026-03-11
categories: [LLM]
tags: [nlp, LLM, latent, training]
math: true
---


## Latent Thinking

**Let’s Think Dot by Dot: Hidden Computation in Transformer Language Models**. Pfau et al. COLM'24\
Add fixed-length DOT token as thinking.\
<https://openreview.net/forum?id=NikbrdtYvG>

**Think before you speak: Training Language Models With Pause Tokens**. Goyal et al. ICLR'24\
Add fixed-length PAUSE token to pretraining+SFT+inference to allow for planning and compression.\
Note: injecting in pretraining is important; otherwise little improvement.\
<https://openreview.net/forum?id=ph04CRkPdC>

**Training Large Language Models to Reason in a Continuous Latent Space**. Hao et al. COLM'25\
Coconut: learn to compress then generate: step by step replacing explicit partial thinking to latent token, till all latent.\
<https://openreview.net/forum?id=Itxz7S4Ip3>

**LightThinker: Thinking Step-by-Step Compression**. Zhang et al. EMNLP'25\
Learn to generate then compress, repeatedly.\
<https://aclanthology.org/2025.emnlp-main.673>

**Compressed Chain of Thought: Efficient Reasoning through Dense Representations**. Cheng et al. 2024\
Distill to match interleaved hidden states.\
<https://arxiv.org/abs/2412.13171>

**CODI: Compressing Chain-of-Thought into Continuous Space via Self-Distillation**. Shen et al. EMNLP'25\
Distill to match the hidden state of last thinking token.\
<https://aclanthology.org/2025.emnlp-main.36>

**Reasoning by Superposition: A Theoretical Perspective on Chain of Continuous Thought**. Zhu et al. NIPS'25\
Latent thinking in continuous space is more expressive than discrete thinking.\
<https://openreview.net/forum?id=UdOEZgWJLc>

**Latent Chain-of-Thought as Planning: Decoupling Reasoning from Verbalization**. Wang et al. 2026\
<https://arxiv.org/abs/2601.21358>

## Soft Thinking

**Soft Thinking: Unlocking the Reasoning Potential of LLMs in Continuous Concept Space**. Zhang et al. NIPS'25\
Test time: at each thinking step, input embedding weighted by last step vocab prob.\
Termination: when exceeding consecutive steps of low entropy.\
<https://openreview.net/forum?id=ByQdHPGKgU>

**LLMs are Single-threaded Reasoners: Demystifying the Working Mechanism of Soft Thinking**. Wu et al. ICLR'26\
Vanilla soft-thinking: generation is dominated by the majority component of the Soft Token (characterizing greedy decoding).\
Method: add randomness to logits.\
<https://openreview.net/forum?id=ASLuOoP78o>

**Soft Tokens, Hard Truths**. Butt et al. ICLR'26\
Similar to above: add noise to embedding (exploration); train with grpo.\
<https://openreview.net/forum?id=9JjKTp8Jmy>
