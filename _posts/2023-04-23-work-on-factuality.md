---
layout: post
title: "Works on Summarization Evaluation and Factuality"
date: 2023-04-23
categories: [NLP]
tags: [nlp, factuality]
math: true
---

Summarization: intrinsic multi-dimension evaluation
* Reference-based: granularity (information selection), style, ... (conditioned on input/task requirement)
* Reference-free: coherence, fluency, consistency, ...

For EACH dimension, evaluation can be:
* Human judgement
* Automatic (e.g. automatic factuality evaluation); may or may not reflect/correlate human judgement well

Also include some hallucination work.

## General Evaluation

**What Have We Achieved on Text Summarization?** Huang et al. EMNLP'20\
<https://aclanthology.org/2020.emnlp-main.33/>

**SummEval: Re-evaluating Summarization Evaluation**. Fabbri et al. TACL'21\
Good consolidation and analysis of previous summ evaluation metrics.\
SummEval: fine-grained human evaluation on Coherence, Consistency, Fluency, Relevance.\
Analysis of correlation between evaluation metrics and human judgement.\
<https://aclanthology.org/2021.tacl-1.24>

**Towards a Unified Multi-Dimensional Evaluator for Text Generation**. Zhong et al. EMNLP'22\
To have one evaluator for all dimensions, use boolean QA-based format as the common interface to unify each dimension (Coherence, Consistency, Fluency, Relevance).\
Train the evaluator: convert existing evaluation dimensions to QAs; also train with other auxiliary tasks.\ 
<https://aclanthology.org/2022.emnlp-main.131>

## Dimension: Factuality

Need: dataset with annotations of fine-grained error type taxonomy.

**Entity-level Factual Consistency of Abstractive Text Summarization**. Nan et al. EACL'21\
IE-based factuality evaluation.\
<https://aclanthology.org/2021.eacl-main.235/>

**Understanding Factuality in Abstractive Summarization with FRANK: A Benchmark for Factuality Metrics**. Pagnoni et al. NAACL'21\
FRANK: fine-grained human annotation of factuality from 1) semantic frame 2) discourse 3) content errors.\ 
Analysis: existing factuality auto-evaluation do not correlate well with human judgement.\
<https://aclanthology.org/2021.naacl-main.383>

**QuestEval: Summarization Asks for Fact-based Evaluation**. Scialom et al. EMNLP'21\
QA-based factuality evaluation that tries to correlate with human judgement, without using gold summaries.
(1) Precision: for each noun phrases in given summary, generate questions by QG model (self-consistent on summary), verify on context by QA model.\
(2) Recall: for each noun phrases in context, generate questions and verify on summary; assign learned weight for each QA pair based on importance.\
<https://aclanthology.org/2021.emnlp-main.529>

**QAFactEval: Improved QA-Based Factual Consistency Evaluation for Summarization**. Fabbri et al. NAACL'22\
Good summary/comparison of existing QA-based factuality work.\
Similar to QuestEval but only focusing on precision.\
(1) QG is critical, and QA is not bottleneck; (2) different ways to determine answer overlapping.\
<https://aclanthology.org/2022.naacl-main.187>

**SUMMAC: Re-Visiting NLI-based Models for Inconsistency Detection in Summarization**. Laban et al. TACL'22\
Entailment-based factuality evaluation that tries to correlate with human judgement, without using gold summaries.\
As NLI works on sentence-level, split context and summary and have pairwise grid scores, then aggregate.\
SUMMAC: a benchmark consisting of 6 previous consistency datasets.\
<https://aclanthology.org/2022.tacl-1.10/>

**Annotating and Modeling Fine-grained Factuality in Summarization**. Goyal and Durrett. NAACL'21\
Study on automatic factuality evaluator by training on synthetic data.\
Comparing two approaches: (1) entity-perturbed data; (2) generation beam. Both synthetic data can localize errors on dependency arc.\
(1) Synthetic data cannot reflect the real factuality error distribution, especially different datasets also show different error type distribution.\
(2) Annotate and train an auto-evaluator DAE: fine-grained factuality annotation (error localization) significantly improves auto-checker.\
(3) Further leverage DAE and mask out loss on non-factual tokens: improvement on factuality based on human judgement.\
<https://aclanthology.org/2021.naacl-main.114>

**Evaluating Factuality in Text Simplification**. Devaraj et al. ACL'22\
Factuality metric: information insertion/deletion/substitution.\
Correlation with human judgement:\
(1) Existing simplification metric SARI: weak correlation.\
(2) Edit distance: ok with deletion, but not insertion/substitution.\
(3) Semantic similarity: well with deletion, ok with insertion, poor with substitution.\
(4) Existing factuality metrics Fact-CC & DAE: even poorer than semantic similarity.\
(5) More annotation and train a classifier for automatic evaluation: still challenging.\
<http://arxiv.org/abs/2204.07562>

**Understanding Factual Errors in Summarization: Errors, Summarizers, Datasets, Error Detectors**. Tang et al. 2022\
AggreFACT: benchmark upon SUMMAC.\
Fine-grained error types: taxonomy.\
Evaluation of previous factuality approaches: error localization; QA-based; entailment-based.\
Findings: (1) factuality evaluators have poorer performance on SOTA (Transformer-based); (2) no systems can capture all error types.\
<http://arxiv.org/abs/2205.12854>

**MQAG: Multiple-choice Question Answering and Generation for Assessing Information Consistency in Summarization**. Manakul et al. AACL'23\
QA-based: instead of span, use multi-choice questions (QG, QA models trained on RACE, a multi-choice QA dataset).\
<http://arxiv.org/abs/2301.12307>

**Improving Abstractive Dialogue Summarization with Speaker-Aware Supervised Contrastive Learning**. Geng et al. COLING'22\
Factuality analysis: BART has ~50% speaker factual errors by probing test on token hidden.\
Enforce token/turn contrastive by speakers (but trivial impact).\
<https://aclanthology.org/2022.coling-1.569>

**CONFIT: Toward Faithful Dialogue Summarization with Linguistically-Informed Contrastive Fine-tuning**. Tang et al. NAACL'22\
Objective: design rule-based negative samples of different factual error types, and perform contrastive learning as additional supervision.\  
<https://aclanthology.org/2022.naacl-main.415/>

**Less is More for Long Document Summary Evaluation by LLMs**. Wu et al. 2023\
For long doc: extract, then verify (by LLM directly).\
<https://arxiv.org/pdf/2309.07382>

## Dimension: Coherence

**SNaC: Coherence Error Detection for Narrative Summarization**. Goyal EMNLP'22\
Focus on the coherence of long summaries of movies and books: human-defined error types and annotations.\
Compare automatic coherence evaluation: (1) unsupervised (2) trained on synthetic data (3) trained on annotations.\
Observation: current summ has good amount of coherence errors, while traditional evaluators are not able to identify.\
<https://aclanthology.org/2022.emnlp-main.29>
