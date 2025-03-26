---
layout: post
title: "Works on Coreference Resolution"
date: 2022-02-22
categories: [NLP]
tags: [nlp, coreference, discourse]
math: true
---

## Traditional Coref

**Incorporating Syntax and Semantics in Coreference Resolution with Heterogeneous Graph Attention Network**. Jiang and Cohn. NAACL'21\
Construct syntactic graph by syntactic dependencies and semantic graph by SRL + GAT per graph + heterogeneous integration.\
<https://aclanthology.org/2021.naacl-main.125/>

**Incorporating Constituent Syntax for Coreference Resolution**. Jiang and Cohn. AAAI'22\
<https://arxiv.org/pdf/2202.10710.pdf>

**PairSpanBERT: An Enhanced Language Model for Bridging Resolution**. Kobayashi et al. ACL'23\
Use heuristics and distant supervision to obtain coreferent pairs for pretraining.\
<https://aclanthology.org/2023.acl-long.383/>

**LingMess: Linguistically Informed Multi Expert Scorers for Coreference Resolution**. Otmazgin et al. EACL'23\
<https://aclanthology.org/2023.eacl-main.202>

**Maverick: Efficient and Accurate Coreference Resolution Defying Recent Trends**. Martinelli et al. ACL'24\
Enhanced E2E with binary CE loss on both mention detection and coreference link.\
<https://aclanthology.org/2024.acl-long.722>

## Incremental

**Incremental Neural Coreference Resolution in Constant Memory**. Xia et al. EMNLP'20\
<https://aclanthology.org/2020.emnlp-main.695/>

**Sentence-Incremental Neural Coreference Resolution**. Grenander et al. EMNLP'22\
<https://aclanthology.org/2022.emnlp-main.28/>

## Sequence Generation

See two sequence works in Sentence Parsing from Liu et al.

**Sequence to Sequence Coreference Resolution**. Urbizu et al. CRAC'20\
<https://aclanthology.org/2020.crac-1.5>

**Seq2seq is All You Need for Coreference Resolution**. Zhang et al. EMNLP'23\
Generate the original document with mention markup.\
<https://aclanthology.org/2023.emnlp-main.704>

**Coreference Resolution through a seq2seq Transition-Based System**. Bohnet et al. TACL'23\
Link-Append.\
<https://aclanthology.org/2023.tacl-1.13>

## LLM-based Zero-Shot

**Assessing the Capabilities of Large Language Models in Coreference: An Evaluation**. Gan et al. LREC'24\
QA: current mention as query, and select from prior candidates.\
<https://aclanthology.org/2024.lrec-main.145/>

**Are Language Models Robust Coreference Resolvers?**. Le and Ritter. COLM'24\
Generate the original document with mention markup.\
<https://openreview.net/forum?id=MmBQSNHKUl>


## Multilingual Coref

**Neural Cross-Lingual Coreference Resolution And Its Application To Entity Linking**. Kundu et al. ACL'18\
<https://aclanthology.org/P18-2063/>

## Cross-Document Coref

**Focus on what matters: Applying Discourse Coherence Theory to Cross Document Coreference**. Held et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.106>

**Sequential Cross-Document Coreference Resolution**. Allaway et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.382>

## Ellipsis

**We Understand Elliptical Sentences, and Language Models should Too: A New Dataset for Studying Ellipsis and its Interaction with Thematic Fit**. Testa et al. ACL'23\
Dataset.\
<https://aclanthology.org/2023.acl-long.188/>

## COREF in Downstream Tasks

**Coreference Reasoning in Machine Reading Comprehension**. Wu et al. ACL'21\
<https://aclanthology.org/2021.acl-long.448/>

## COREF Analysis

**OntoGUM: Evaluating Contextualized SOTA Coreference Resolution on 12 More Genres**. Zhu et al. ACL'21\
<https://aclanthology.org/2021.acl-short.59/>

**Moving on from OntoNotes: Coreference Resolution Model Transfer**. Xia and Durme. EMNLP'21\
Domain adaptation by continued training.\
<https://aclanthology.org/2021.emnlp-main.425>

## Others

**Signed Coreference Resolution**. Yin et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.405>
