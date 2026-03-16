---
layout: post
title: "Works on LLM Agents and Memory"
date: 2025-07-31
categories: [NLP]
tags: [nlp, LLM, agent]
math: true
---


## Experiential Memory

### Explicit Design w/o Training

**ExpeL: LLM Agents Are Experiential Learners**. Zhao et al. AAAI'24\
Abstraction (insights) on pos and neg trajectories: 1) reflect pos and neg for the same task; 2) reflect pos trajectories across tasks.\
Inference: add full task insights to context.\
<https://arxiv.org/abs/2308.10144>

**MemoryBank: Enhancing Large Language Models with Long-Term Memory**. Zhong et al. AAAI'24\
Simple mem update with decay.\
<https://arxiv.org/abs/2305.10250>

**Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory**. Chhikara et al. 2025\
Simple mem update with graph database.\
<https://arxiv.org/pdf/2504.19413>

**A-Mem: Agentic Memory for LLM Agents**. Xu et al. NIPS'25\
Memory notes.\
Link across notes: select top-k notes to generate relations.\
<https://openreview.net/forum?id=FiM0M8gcct>

**G-Memory: Tracing Hierarchical Memory for Multi-Agent Systems**. Zhang et al. NIPS'25\
<https://openreview.net/forum?id=mmIAp3cVS0>

**CAM: A Constructivist View of Agentic Memory for LLM-Based Reading Comprehension**. Li et al. NIPS'25\
<https://openreview.net/forum?id=ACSOnSHiWe>

### Latent Experiential Memory

**Agentic Memory: Learning Unified Long-Term and Short-Term Memory Management for Large Language Model Agents**. Yu et al. 2026\
<https://arxiv.org/abs/2601.01885>

**MemGen: Weaving Generative Latent Memory for Self-Evolving Agents**. Zhang et al. ICLR'26\
RL trained lora: interleave latent hidden states as experiential memory.\
<https://openreview.net/forum?id=vI56m4Iu4e>


## General Agent Training

**DeepResearcher: Scaling Deep Research via Reinforcement Learning in Real-world Environments**. Zheng et al. 2025\
RL agent with web search, with a keep-appending short-term memory repository.\
<https://arxiv.org/abs/2504.03160>

**Encouraging Divergent Thinking in Large Language Models through Multi-Agent Debate**. Liang et al. EMNLP'24\
Spur exploration by forcing different stances and roles explicitly.\
<https://aclanthology.org/2024.emnlp-main.992>


## Evolving

**EvolveR: Self-Evolving LLM Agents through an Experience-Driven Lifecycle**. Wu et al. 2025\
Offline: generate trajectories and form insights/principles.\
Inference: retrieve principles.\
RL training to utilize and integrate principles.\
<https://arxiv.org/abs/2510.16079>
