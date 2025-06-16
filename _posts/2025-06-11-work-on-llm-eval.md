---
layout: post
title: "Works on LLM Eval"
date: 2025-06-11
categories: [NLP]
tags: [nlp, LLM]
math: true
---

See separate post for QA-related evaluation.


## General

**LiveBench: A Challenging, Contamination-Limited LLM Benchmark**. White et al. ICLR'25\
<https://openreview.net/forum?id=sKYHBTAxVa>

**GPQA: A Graduate-Level Google-Proof Q&A Benchmark**. Rein et al. COLM'24\
<https://openreview.net/forum?id=Ti67584b98>


## Algorithmic/Mathematics Reasoning

**Faith and Fate: Limits of Transformers on Compositionality**. Dziri et al. NIPS'23\
Controlled complexity: explicit computational graph.\
Problems: multiplication, puzzle, DP.\
Observations: hard to generalize even trained with COT; memorization.\
<https://openreview.net/forum?id=Fkckkr3ya8>

**Case-Based or Rule-Based: How Do Transformers Do the Math?**. Hu et al. ICML'24\
Design math dataset that controls case similarity, then train & evaluate.\
Observations: LLMs rely on similar math cases.\
<https://openreview.net/forum?id=4Vqr8SRfyX>

**GSM-Symbolic: Understanding the Limitations of Mathematical Reasoning in Large Language Models**. Mirzadeh et al. 2024\
GSM-stype math problem through generation with controlled complexity.\
<https://arxiv.org/abs/2410.05229>

**The Illusion of Thinking: Understanding the Strengths and Limitations of Reasoning Models via the Lens of Problem Complexity**. Shojaee et al. 2025\
Logic problems with controlled complexity, e.g. Hanoi tower.\
<https://arxiv.org/pdf/2506.06941>

**PUZZLES: A Benchmark for Neural Algorithmic Reasoning**. Estermann et al. NIPS'24\
A collection of RL-oriented logical puzzles.\
<https://neurips.cc/virtual/2024/poster/97818>

**Alice in Wonderland: Simple Tasks Showing Complete Reasoning Breakdown in State-Of-the-Art Large Language Models**. Nezhurina et al. SciDL'24\
Very simple template-based problem.\
<https://arxiv.org/abs/2406.02061>


## NLP Context Reasoning

**MuSR: Testing the Limits of Chain-of-thought with Multistep Soft Reasoning**. Sprague et al. ICLR'24\
<https://openreview.net/forum?id=jenyYQzue1>


## Instruction Following / Preference

**A User-Centric Multi-Intent Benchmark for Evaluating Large Language Models**. Wang et al. EMNLP'24\
Assistant questions.\
<https://aclanthology.org/2024.emnlp-main.210>


## Vision

**Embodied Agent Interface: Benchmarking LLMs for Embodied Decision Making**. Li et al. NIPS'24\
<https://arxiv.org/abs/2410.07166>

**EmbodiedBench: Comprehensive Benchmarking Multi-modal Large Language Models for Vision-Driven Embodied Agents**. Yang et al. ICML'25\
<https://arxiv.org/abs/2502.09560>


## Domain-Specific

**Evaluating LLMs’ Mathematical Reasoning in Financial Document Question Answering**. Srivastava et al. ACL Findings'24\
<https://aclanthology.org/2024.findings-acl.231>
