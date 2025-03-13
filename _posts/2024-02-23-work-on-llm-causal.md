---
layout: post
title: "Works on LLM Causality"
date: 2024-02-23
categories: [NLP]
tags: [nlp, causal, LLM]
math: true
---

## Evaluation and Analysis

**Causal Reasoning and Large Language Models: Opening a New Frontier for Causality**. Kıcıman et al. 2023\
LLM is at least as good as previous models at 1) causal discovery on existing benchmarks; 2) actual causality.\
(LLM is a good domain expert; though, those tasks may not reflect true causal inference capability; see works below)\
<https://arxiv.org/abs/2305.00050>

**CLADDER: Assessing Causal Reasoning in Language Models**. Jin et al. NIPS'23\
Assess formal causal reasoning, by verbalized symbolic inference, minimizing effects from memorization, common senses, etc.\
Dataset: (causal graph, query type) -> solution by Causal Inference Engine -> verbalization (symbolic -> natural langauge story)\
Designed COT: story -> determine causal graph and type in symbolics -> inference.\
<https://arxiv.org/abs/2312.04350>

**Causal Parrots: Large Language Models May Talk Causality But Are Not Causal**. Zečević et al. TMLR'23\
Core argument: LLM could occasionally give correct causal facts likely they are seen in pretraining (causal parrots), but not real causal inference.\
(1) a model that simply obtains its knowledge from various Wikipedia statements will also learn untrue statements, statements that are not facts, thus explaining behavior that is correct sometimes and wrong other times.\
(2) they are fundamentally different forms of representation in that learning from the physical measurements can be argued to be what we mean by ‘understanding’, whereas simply reading up on the textual article lacks exactly that component.\
(3) LLM can never truly perform causal inference, if only observational data is given but no physical manipulation/intervention is performed.\
<https://openreview.net/forum?id=tv46tCzs83>

**Understanding Causality with Large Language Models: Feasibility and Opportunities**. Zhang et al. 2023\
Similar to above: LLM can answer causal questions with existing causal knowledge, but not good enough for discovering new knowledge.\
<https://arxiv.org/abs/2304.05524>

**Can Large Language Models Infer Causation from Correlation?**. Jin et al. ICLR'24\
Assess causal discovery from correlations/dependencies (deductive reasoning).\
Basics: causation determines correlation but not reverse (one-to-many mapping); thus given correlation, need to reason causation.\
Dataset: unique causal DAGs -> determine d-separation set -> determine correlation -> assign causation if all corresponding causal DAGs have this edge.\
Observations: even after finetuning, the LLM prediction is not robust.\
<https://openreview.net/forum?id=vqIH0ObdqL>

**Bridging Causal Discovery and Large Language Models: A Comprehensive Survey of Integrative Approaches and Future Directions**. Wan et al. 2024\
<https://arxiv.org/abs/2402.11068>

## Causal Discovery

**Causal Structure Learning Supervised by Large Language Model**. Ban et al. 2023\
<https://arxiv.org/abs/2402.11068>

**Causal Inference Using LLM-Guided Discovery**. Vashishtha et al. 2024\
Related work.\
<https://arxiv.org/abs/2310.15117>

## Causal Inference
