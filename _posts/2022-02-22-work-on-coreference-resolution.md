---
layout: post
title: "Works on Coreference Resolution"
date: 2022-02-22
categories: [NLP]
tags: [nlp, coreference, discourse]
math: true
---

## Traditional Coref

**Sequence to Sequence Coreference Resolution**. Urbizu et al. CRAC'20\
<https://aclanthology.org/2020.crac-1.5>

**Incorporating Syntax and Semantics in Coreference Resolution with Heterogeneous Graph Attention Network**. Jiang and Cohn.
NAACL'21\
Construct syntactic graph by syntactic dependencies and semantic graph by SRL + GAT per graph + heterogeneous
integration.\
<https://aclanthology.org/2021.naacl-main.125/>

**Incorporating Constituent Syntax for Coreference Resolution**. Jiang and Cohn. AAAI'22\
<https://arxiv.org/pdf/2202.10710.pdf>

**PairSpanBERT: An Enhanced Language Model for Bridging Resolution**. Kobayashi et al. ACL'23\
Use heuristics and distant supervision to obtain coreferent pairs for pretraining.\
<https://aclanthology.org/2023.acl-long.383/>

**LingMess: Linguistically Informed Multi Expert Scorers for Coreference Resolution**. Otmazgin et al. EACL'23\
<https://aclanthology.org/2023.eacl-main.202>

## Sequence Generation

See two sequence works in Sentence Parsing from Liu et al.

**Seq2seq is All You Need for Coreference Resolution**. Zhang et al. EMNLP'23\
<https://aclanthology.org/2023.emnlp-main.704>

**Coreference Resolution through a seq2seq Transition-Based System**. Bohnet et al. TACL'23\
<https://aclanthology.org/2023.tacl-1.13>

## Multilingual Coref

**Neural Cross-Lingual Coreference Resolution And Its Application To Entity Linking**. Kundu et al. ACL'18\
<https://aclanthology.org/P18-2063/>

## Cross-Document Coref

**Focus on what matters: Applying Discourse Coherence Theory to Cross Document Coreference**. Held et al. EMNLP'21\
TODO\
<https://aclanthology.org/2021.emnlp-main.106>

**Sequential Cross-Document Coreference Resolution**. Allaway et al. EMNLP'21\
TODO\
<https://aclanthology.org/2021.emnlp-main.382>

## Ellipsis

**We Understand Elliptical Sentences, and Language Models should Too: A New Dataset for Studying Ellipsis and its Interaction with Thematic Fit**. Testa et al. ACL'23\
Dataset.\
<https://aclanthology.org/2023.acl-long.188/>

## COREF in Downstream Tasks

**Coreference Reasoning in Machine Reading Comprehension**. Wu et al. ACL'21\
TODO\
<https://aclanthology.org/2021.acl-long.448/>

## COREF Analysis

**OntoGUM: Evaluating Contextualized SOTA Coreference Resolution on 12 More Genres**. Zhu et al. ACL'21\
TODO\
<https://aclanthology.org/2021.acl-short.59/>

**Moving on from OntoNotes: Coreference Resolution Model Transfer**. Xia and Durme. EMNLP'21\
Domain adaptation by continued training.\
<https://aclanthology.org/2021.emnlp-main.425>

## Others

**Signed Coreference Resolution**. Yin et al. EMNLP'21\
TODO\
<https://aclanthology.org/2021.emnlp-main.405>
