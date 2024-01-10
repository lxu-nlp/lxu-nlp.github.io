---
layout: post
title: "Works on Discourse"
date: 2023-04-03
categories: [NLP]
tags: [nlp, discourse]
math: true
---

**Where Are We in Discourse Relation Recognition?**. Atwell et al. SigDial'21\
Good summary:\
Rhetorical Structure Theory (RST): hierarchical (tree) structure.\
Penn Discourse Treebank (PDTB): shallow, non-hierarchical triples.\
<https://aclanthology.org/2021.sigdial-1.34>

**How Do We Answer Complex Questions: Discourse Structure of Long-form Answers**. Xu et al. ACL'22\
Human-defined FIXED sentence roles in the long answers (tagging on each sentence). Half of the sentences are not direct answers.\
<https://aclanthology.org/2022.acl-long.249>

**Discourse Comprehension: A Question Answering Framework to Represent Sentence Connections**. Ko et al. EMNLP'22\
To understand NON-FIXED sentence relationships and connections: annotate question for each sentence, as well as link to its logical antecedent sentence.\
Unlike human-defined discourse relations, questions are open-ended, more flexible, without restrictions.\
Experiments: use QA to identify discourse: given context, question, optionally antecedent sentence, identify the sentence.\
Human performs well without given antecedent, while model drops 20+% without antecedent.\
<https://aclanthology.org/2022.emnlp-main.806>

**Discourse Analysis via Questions and Answers: Parsing Dependency Structures of Questions Under Discussion**. Ko et al. ACL Findings'23\
Open-world discourse relations by questions.\
(source sentence, question (edge relation), target sentence), where question arises from source and is answered by target.\
<https://aclanthology.org/2023.findings-acl.710>

**Connective Prediction for Implicit Discourse Relation Recognition via Knowledge Distillation**. Wu et al. ACL'23\
(1) First predict connectives between sentences based on template prompting; (2) use predicted connective distribution to predict discourse sense.\
<https://aclanthology.org/2023.acl-long.325>


## Unconventional Discourse Relations: Entailment, ...

**Global Learning of Focused Entailment Graphs**. Berant et al. ACL'10\
Entailment graph construction.\
<https://aclanthology.org/P10-1124>

**Efficient Tree-based Approximation for Entailment Graph Learning**. Berant et al. ACL'12\
<https://aclanthology.org/P12-1013/>

**From Key Points to Key Point Hierarchy: Structured and Expressive Opinion Summarization**. Cattan et al. ACL'23\
Directional relation graph construction: 1) pairwise scores: NLI; distributional context similarity; 2) edge building by score threshold.\
<https://aclanthology.org/2023.acl-long.52>


## Discourse Applications

**Structure-Discourse Hierarchical Graph for Conditional Question Answering on Long Documents**. Du et al. ACL Findings'23\
<https://aclanthology.org/2023.findings-acl.391>

**Exploring Discourse Structure in Document-level Machine Translation**. Hu and Wan. EMNLP'23\
Attention on RST tree.\
<https://aclanthology.org/2023.emnlp-main.857>
