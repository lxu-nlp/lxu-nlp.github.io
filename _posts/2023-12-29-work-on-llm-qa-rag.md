---
layout: post
title: "Works on LLM QA and RAG"
date: 2023-12-29
categories: [NLP]
tags: [nlp, QA, LLM]
math: true
---


# Benchmark

### Long Context / Global Reasoning

**The NarrativeQA Reading Comprehension Challenge**. Kočiský et al. TACL'18\
Questions on long stories that require higher-level abstraction than superficial pattern matching: majority answers do not have spans directly in text.\
Approach: retrieval & concatenation to shorter doc -> QA.\
<https://aclanthology.org/Q18-1023>

**Fantastic Questions and Where to Find Them: FairytaleQA – An Authentic Dataset for Narrative Comprehension**. Xu et al. ACL'22\
FairytaleQA: similar to NarrativeQA with more diverse questions.\
<https://aclanthology.org/2022.acl-long.34>

**QuALITY: Question Answering with Long Input Texts, Yes!**. Pang et al. NAACL'22\
More abstractive questions: how, why, description.\
Multi-choice.\
<https://aclanthology.org/2022.naacl-main.391>

**True Detective: A Deep Abductive Reasoning Benchmark Undoable for GPT-3 and Challenging for GPT-4**. Del and Fishel. *SEM'23\
<https://aclanthology.org/2023.starsem-1.28>

**LONGBENCH: A BILINGUAL, MULTITASK BENCHMARK FOR LONG CONTEXT UNDERSTANDING**. Bai et al. 2023\
<https://arxiv.org/abs/2403.03514>

**LongBench v2: Towards Deeper Understanding and Reasoning on Realistic Long-context Multitasks**. Bai et al. 2024\
<https://arxiv.org/abs/2412.15204>

**XLBench: A Benchmark for Extremely Long Context Understanding with Long-range Dependencies**. Ni et al. 2024\
<https://arxiv.org/abs/2308.14508>

**LooGLE: Can Long-Context Language Models Understand Long Contexts?**. Li et al. 2024\
<https://arxiv.org/abs/2311.04939>

**CLongEval: A Chinese Benchmark for Evaluating Long-Context Large Language Models**. Qiu et al. 2024\
<https://arxiv.org/abs/2403.03514>

**NovelQA: A Benchmark for Long-Range Novel Question Answering**. Wang et al. 2024\
Novels > 50k. Designed question types and templates.\
<https://arxiv.org/abs/2403.12766>

**RULER: What’s the Real Context Size of Your Long-Context Language Models?** Hsieh et al. COLM'24\
Enhanced needle in a haystack.\
<https://openreview.net/forum?id=kIoBbc76Sy>

**∞ Bench: Extending Long Context Evaluation Beyond 100K Tokens**. Zhang et al. 2025\
<https://arxiv.org/abs/2402.13718>

**HELMET: How to Evaluate Long-Context Language Models Effectively and Thoroughly**. Yen et al. ICLR'25\
<https://openreview.net/forum?id=293V3bJbmE>

### RAG

**LaRA: Benchmarking Retrieval-Augmented Generation and Long-Context LLMs - No Silver Bullet for LC or RAG Routing**. Li et al. 2025\
Empirical comparison: Long Context vs. RAG.\
<https://arxiv.org/pdf/2502.09977>

---

# Method: Building Enhanced Index

**Walking Down the Memory Maze: Beyond Context Limit through Interactive Reading**. Chen et al. 2023\
Semantic similarity tree by summaries: zero-shot LLM to figure out the path from root.\
<https://arxiv.org/abs/2310.05029>

**RAPTOR: Recursive Abstractive Processing for Tree-Organized Retrieval**. Sarthi et al. ICLR'24\
Semantic similarity tree.\
<https://openreview.net/forum?id=GN921JHCRw>

**From Local to Global: A GraphRAG Approach to Query-Focused Summarization**. Edge et al. 2024\
GraphRAG: KG nodes and relations.\
<https://arxiv.org/abs/2404.16130>

**LightRAG: Simple and Fast Retrieval-Augmented Generation**. Guo et al. 2024\
KG-based.\
<https://arxiv.org/abs/2410.05779>

**KAG: Boosting LLMs in Professional Domains via Knowledge Augmented Generation**. Liang et al. 2024\
Complex KG-based.\
<https://arxiv.org/abs/2409.13731>

**From RAG to Memory: Non-Parametric Continual Learning for Large Language Models**. Gutiérrez et al. 2025\
HippoRAG v2: combine KG and passage nodes for contextualization.\
<https://arxiv.org/abs/2502.14802>

**SiReRAG: Indexing Similar and Related Information for Multihop Reasoning**. Zhang et al. ICLR'25\
Semantic similarity tree + entity proposition tree for relatedness.\
<https://openreview.net/forum?id=yp95goUAT1>

# Method: Map Query to Index

**Generate rather than Retrieve: Large Language Models are Strong Context Generators**. Yu et al. ICLR'23\
Use LLM to replace retrieval by relevant context generation.\
<https://openreview.net/forum?id=fB0hRu9GZUS>

**A Human-Inspired Reading Agent with Gist Memory of Very Long Contexts**. Lee et al. 2024\
Use LLM to select which segments to read and when to stop (on linear segment summaries).\
<https://arxiv.org/abs/2402.09727>

**LongAgent: Scaling Language Models to 128k Context through Multi-Agent Collaboration**. Zhao et al. 2024\
Use LLM: one leader for decision making and several members for discussion.\
<https://arxiv.org/abs/2402.11550>

**HippoRAG: Neurobiologically Inspired Long-Term Memory for Large Language Models**. Gutiérrez et al. NIPS'24\
Graph-based + page rank for related node/edge expansion.\
<https://openreview.net/forum?id=hkujvAPVsg>

# Method: Explicit In-Context Retrieval

**Rank1: Test-Time Compute for Reranking in Information Retrieval**. Weller et al. 2025\
Dataset of R1 COT for (query, doc) retrieval.\
<https://arxiv.org/pdf/2502.18418>

# Method: Implicit In-Context Retrieval

See Context Compression in post Works on LLM Memory.

# Method: RL-based Multi-step

**Search-R1: Training LLMs to Reason and Leverage Search Engines with Reinforcement Learning**. Jin et al. 2025\
<https://arxiv.org/abs/2503.09516>

**DeepRetrieval: Hacking Real Search Engines and Retrievers with Large Language Models via Reinforcement Learning**. Jiang et al. 2025\
<https://arxiv.org/abs/2503.00223>

---

# Task: Question Generation

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

# Task: Decontextualization

**Decontextualization: Making Sentences Stand-Alone**. Choi et al. TACL'21\
Augment local context: add coref, bridging, global modifier, ...\
Applications: passage retrieval, etc.\
<https://aclanthology.org/2021.tacl-1.27/>

**A Question Answering Framework for Decontextualizing User-facing Snippets from Scientific Documents**. Newman et al. EMNLP'23\
QG -> QA -> rewrite upon QA pairs (online manner).\
Evaluation on LLM prompting.\
<https://aclanthology.org/2023.emnlp-main.193>

# Task: LLM Retrieval

**Autoregressive Search Engines: Generating Substrings as Document Identifiers**. Bevilacqua et al. NIPS'22\
SEAL: train LM to generate distinguished ngrams upon query.\
Test time: constrained decoding using FM index.\
<https://openreview.net/forum?id=Z4kZxAjg8Y>
