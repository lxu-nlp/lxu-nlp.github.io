---
layout: post
title: "Works on Discourse"
date: 2023-04-03
categories: [NLP]
tags: [nlp, discourse]
math: true
---

## Conventional Discourse

**Attention, Intentions, and the Structure of Discourse**. Grosz and Sidner. 1986\
<https://aclanthology.org/J86-3001/>

**Rethorical Structure Theory: Toward a functional theory of text organization**. William et al. 1988\
RST.\
<https://www.sfu.ca/rst/05bibliographies/bibs/Mann_Thompson_1988.pdf>

**The Penn Discourse TreeBank 2.0** Prasad et al. LREC'08\
PDTB.\
<https://aclanthology.org/L08-1093/>

**QADiscourse - Discourse Relations as QA Pairs: Representation, Crowdsourcing and Baselines**. Pyatkin et al. EMNLP'20\
PDTB-style discourse represented by QA pairs.\
<https://aclanthology.org/2020.emnlp-main.224>

**Where Are We in Discourse Relation Recognition?**. Atwell et al. SigDial'21\
Good summary:\
Rhetorical Structure Theory (RST): hierarchical (tree) structure.\
Penn Discourse Treebank (PDTB): shallow, non-hierarchical triples.\
<https://aclanthology.org/2021.sigdial-1.34>

## Questions under Discussion (QUD)

**Questions Under Discussion: From Sentence to Discourse**. Benz and Jasinskaja. 2017\
QUD survey.\

**Towards automatically generating Questions under Discussion to link information and discourse structure**. De Kuthy et al. COLING'20\
<https://aclanthology.org/2020.coling-main.509>

**Inquisitive Question Generation for High Level Text Comprehension**. Ko et al. EMNLP'20\
Dataset to ask inquisitive questions (forward) on sentences.\
<https://aclanthology.org/2020.emnlp-main.530>

**Discourse Comprehension: A Question Answering Framework to Represent Sentence Connections**. Ko et al. EMNLP'22\
QUD dataset.\
To understand NON-FIXED sentence relationships and connections: annotate question for each sentence, as well as link to its logical antecedent sentence.\
Unlike human-defined discourse relations, questions are open-ended, more flexible, without restrictions.\
Experiments: use QA to identify discourse: given context, question, optionally antecedent sentence, identify the sentence.\
Human performs well without given antecedent, while model drops 20+% without antecedent.\
<https://aclanthology.org/2022.emnlp-main.806>

**Discourse Analysis via Questions and Answers: Parsing Dependency Structures of Questions Under Discussion**. Ko et al. ACL Findings'23\
QUD parser.\
Open-world discourse relations by questions.\
(source sentence, question (edge relation), target sentence), where question arises from source and is answered by target.\
<https://aclanthology.org/2023.findings-acl.710>

**QUDeval: The Evaluation of Questions Under Discussion Discourse Parsing**. Wu et al. EMNLP'23\
QuD automatic evaluation.\
<https://aclanthology.org/2023.emnlp-main.325>

## Other Discourse Relations: Entailment, ...

**Global Learning of Focused Entailment Graphs**. Berant et al. ACL'10\
Entailment graph construction.\
<https://aclanthology.org/P10-1124>

**Efficient Tree-based Approximation for Entailment Graph Learning**. Berant et al. ACL'12\
<https://aclanthology.org/P12-1013/>

**From Key Points to Key Point Hierarchy: Structured and Expressive Opinion Summarization**. Cattan et al. ACL'23\
Directional relation graph construction: 1) pairwise scores: NLI; distributional context similarity; 2) edge building by score threshold.\
<https://aclanthology.org/2023.acl-long.52>

**How Do We Answer Complex Questions: Discourse Structure of Long-form Answers**. Xu et al. ACL'22\
Human-defined FIXED sentence roles in the long answers (tagging on each sentence). Half of the sentences are not direct answers.\
<https://aclanthology.org/2022.acl-long.249>

**Connective Prediction for Implicit Discourse Relation Recognition via Knowledge Distillation**. Wu et al. ACL'23\
(1) First predict connectives between sentences based on template prompting; (2) use predicted connective distribution to predict discourse sense.\
<https://aclanthology.org/2023.acl-long.325>

**A Multi-Task Dataset for Assessing Discourse Coherence in Chinese Essays: Structure, Theme, and Logic Analysis**. Wu et al. EMNLP'23\
Fine-grained discourse relations for essay scoring.\
<https://aclanthology.org/2023.emnlp-main.412>

## Discourse Applications

Also see papers in summarization, etc.

**The Role of Discourse Units in Near-Extractive Summarization**. Li et al. SIGDIAL'16\
<https://aclanthology.org/W16-3617/>

**Structure-Discourse Hierarchical Graph for Conditional Question Answering on Long Documents**. Du et al. ACL Findings'23\
<https://aclanthology.org/2023.findings-acl.391>

**Exploring Discourse Structure in Document-level Machine Translation**. Hu and Wan. EMNLP'23\
Attention on RST tree.\
<https://aclanthology.org/2023.emnlp-main.857>
