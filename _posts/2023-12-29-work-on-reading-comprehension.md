---
layout: post
title: "Works on Reading Comprehension"
date: 2023-12-29
categories: [NLP]
tags: [nlp]
math: true
---

Reading comprehension other than the standard QA.

**Natural Language Decompositions of Implicit Content Enable Better Text Representations**. Hoyle et al. EMNLP'23\
Generate implicit (and simplified) propositions inferred by LLM through few-shot prompting.\
<https://aclanthology.org/2023.emnlp-main.815>

**Multi-level Contrastive Learning for Script-based Character Understanding**. Li et al. EMNLP'23\
<https://aclanthology.org/2023.emnlp-main.366>

**Drilling Down into the Discourse Structure with LLMs for Long Document Question Answering**. Nair et al. EMNLP'23\
LLM to find relevant sections (header + summary) -> LLM to pick relevant paragraphs -> QA.\
<https://aclanthology.org/2023.findings-emnlp.972>

## Benchmark

For long context.

**LONGBENCH: A BILINGUAL, MULTITASK BENCHMARK FOR LONG CONTEXT UNDERSTANDING**. Bai et al. 2023\
<https://arxiv.org/abs/2403.03514>

**XLBench: A Benchmark for Extremely Long Context Understanding with Long-range Dependencies**. Ni et al. 2024\
<https://arxiv.org/abs/2308.14508>

**CLongEval: A Chinese Benchmark for Evaluating Long-Context Large Language Models**. Qiu et al. 2024\
<https://arxiv.org/abs/2403.03514>

**NovelQA: A Benchmark for Long-Range Novel Question Answering**. Wang et al. 2024\
Novels > 50k. Designed question types and templates.\
<http://arxiv.org/abs/2403.12766>

## Question Generation

**Semantic Graphs for Generating Deep Questions**. Pan et al. ACL'20\
SRL or dependency graph -> augmented node representation by GNN -> question.\ 
<https://aclanthology.org/2020.acl-main.135>

**Generating Questions for Reading Comprehension using Coherence Relations**. Desai et al. 2018\
Question types according to Bloom’s Taxonomy in cognitive science.\
discourse relation -> template type -> question.\
<https://aclanthology.org/W18-3701>

**SkillQG: Learning to Generate Question for Reading Comprehension Assessment**. Wang et al. ACL'23 Findings'\
Question types according to Bloom’s Taxonomy.\
(context, answer from IE or given, skill) -> question.\
Downstream task: augment context for FairyTale QA.\
<https://aclanthology.org/2023.findings-acl.870>

**Guiding the Growth: Difficulty-Controllable Question Generation through Step-by-Step Rewriting**. Cheng et al. ACL'21\
context -> reasoning graph -> iterative question rewriting based on reasoning steps.\
<https://aclanthology.org/2021.acl-long.465>

## Decontextualization

**Decontextualization: Making Sentences Stand-Alone**. Choi et al. TACL'21\
Augment local context: add coref, bridging, global modifier, ...\
Applications: passage retrieval, etc.\
<https://aclanthology.org/2021.tacl-1.27/>

**A Question Answering Framework for Decontextualizing User-facing Snippets from Scientific Documents**. Newman et al. EMNLP'23\
QG -> QA -> rewrite upon QA pairs (online manner).\
Evaluation on LLM prompting.\
<https://aclanthology.org/2023.emnlp-main.193>

## (Interactive) Reading Agent

**Walking Down the Memory Maze: Beyond Context Limit through Interactive Reading**. Chen et al. 2023\
Summary tree: zero-shot LLM to figure out the path from root.\
<https://arxiv.org/abs/2310.05029>

**A Human-Inspired Reading Agent with Gist Memory of Very Long Contexts**. Lee et al. 2024\
Segment summaries (linear): zero-shot LLM to select which segments to read and when to stop.\
<http://arxiv.org/abs/2402.09727>

**LongAgent: Scaling Language Models to 128k Context through Multi-Agent Collaboration**. Zhao et al. 2024\
One leader for decision making and several members for discussion.\
<http://arxiv.org/abs/2402.11550>
