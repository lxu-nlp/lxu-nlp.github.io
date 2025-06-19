---
layout: post
title: "Works on LLM Evaluation"
date: 2025-06-11
categories: [NLP]
tags: [nlp, LLM]
math: true
---


See the separate [post](https://lxu-nlp.github.io/posts/work-on-llm-qa-rag/) for general QA-related evaluation.


## Length

**RULER: What’s the Real Context Size of Your Long-Context Language Models?** Hsieh et al. COLM'24\
Enhanced needle in a haystack.\
<https://openreview.net/forum?id=kIoBbc76Sy>

**BABILong: Testing the Limits of LLMs with Long Context Reasoning-in-a-Haystack**. Kuratov et al. NIPS'24\
20 tasks; evidence scattered across long natural context.\
<https://openreview.net/forum?id=u7m2CG84BQ>

**Same Task, More Tokens: the Impact of Input Length on the Reasoning Performance of Large Language Models**. Levy et al. ACL'24\
Add irrelevant context.\
<https://aclanthology.org/2024.acl-long.818>


## Number

Analysis on number-related limitations.

**Transformers need glasses! Information over-squashing in language tasks**. Barbero et al. NIPS'24\
Theoretical: limitation on counting or copying due to representational collapse.\
<https://openreview.net/forum?id=93HCE8vTye>

**When Can Transformers Count to n?**. Yehudai et al. 2024\
<https://arxiv.org/abs/2407.15160>

**Can We Count on LLMs? The Fixed-Effect Fallacy and Claims of GPT-4 Capabilities**. Ball et al. TMLR'24\
Evaluate on count/max/sort/multiply... tasks.\
<https://openreview.net/forum?id=qt4d0EGZsK>

**LLM The Genius Paradox: A Linguistic and Math Expert’s Struggle with Simple Word-based Counting Problems**. Xu and Ma. NAACL'25\
Count words.\
<https://aclanthology.org/2025.naacl-long.172>


## In-Context Learning

**Long-context LLMs Struggle with Long In-context Learning**. Li et al. TMLR'25\
Long in-context learning in extreme-label classification with up to 174 classes.\
<https://openreview.net/forum?id=Cw2xlg0e46>

**In-Context Learning with Long-Context Models: An In-Depth Exploration**. Bertsch et al. NAACL'25\
Scaling examples works.\
<https://aclanthology.org/2025.naacl-long.605/>

**Many-Shot In-Context Learning**. Agarwal et al. NIPS'24\
Experiments on traditional NLP, math/algorithmic task, as well as vector classification, parity.\
Scaling in-context examples: 1) using auto-gen rationale; 2) using only inputs (still works by locating relevant knowledge).\
Observations: both surpass few-shot; auto-rationale usually performs better.\
<https://openreview.net/forum?id=AB6XpMzvqH>

**Stress-Testing Long-Context Language Models with Lifelong ICL and Task Haystack**. Xu et al. NIPS'24\
Scaling tasks.\
<https://proceedings.neurips.cc/paper_files/paper/2024/hash/1cc8db5884a7474b4771762b6f0c8ee1-Abstract-Datasets_and_Benchmarks_Track.html>

#### Analysis

**Larger language models do in-context learning differently**. Wei et al. 2023\
Intrinsic knowledge vs. in-context label mapping: label mapping supersedes intrinsic knowledge, and can scale by model size.\
<https://arxiv.org/abs/2303.03846>

**Why Larger Language Models Do In-context Learning Differently?**. Shi et al. ICML'24\
Theoretical: larger model learns more features but is more prone to noises.\
Good related work.\
<https://openreview.net/forum?id=WOa96EG26M>


## Global Evidence Coverage

**Is It Really Long Context if All You Need Is Retrieval? Towards Genuinely Difficult Long Context NLP**. Goldman et al. EMNLP'24\
Define properties:\
(1) dispersion: difficulty of information finding\
(2) scope: quantity of information needed to solve the task\
<https://aclanthology.org/2024.emnlp-main.924>

**Ada-LEval: Evaluating long-context LLMs with length-adaptable benchmarks**. Wang et al. NAACL'24\
Tasks: 1) SORT segments; 2) identify BEST answers from many candidates.\
<https://aclanthology.org/2024.naacl-long.205>

**BAMBOO: A Comprehensive Benchmark for Evaluating Long Text Modeling Capacities of Large Language Models**. Dong et al. LREC'24\
Tasks: 1) multi-evidence QA; 2) SORT segments, or sort summaries given full context; 3) ...\
<https://aclanthology.org/2024.lrec-main.188>

**Leave No Document Behind: Benchmarking Long-Context LLMs with Extended Multi-Doc QA**. Wang et al. EMNLP'24\
Tasks: 1) simple localization; 2) multi-piece COMPARISON; 3) multi-piece clustering; 4) multi-piece dependency and deduction, e.g. temporal or citation chain.\
<https://aclanthology.org/2024.emnlp-main.322>

**ETHIC: Evaluating Large Language Models on Long-Context Tasks with High Information Coverage**. Lee et al. NAACL'25\
Tasks: 1) recall ALL entities; 2) summarize EACH segment; 3) SORT segment summaries; 4) attribute relevant segments.\
<https://aclanthology.org/2025.naacl-long.283>


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
