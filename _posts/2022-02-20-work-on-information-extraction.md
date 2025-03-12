---
layout: post
title: "Works on Document Information Extraction"
date: 2022-02-20
categories: [NLP]
tags: [nlp, information extraction, IE]
math: true
---

# Task: Doc-Level RE
---

## Dataset

**DocRED: A Large-Scale Document-Level Relation Extraction Dataset**. Yao et al. ACL'19\
Multi-label classification on entity-pair. Predicting: bilinear on entity pair\
<https://aclanthology.org/P19-1074/>

**Does Recommend-Revise Produce Reliable Annotations? An Analysis on Missing Instances in DocRED**. Huang et al. ACL'22\
<https://aclanthology.org/2022.acl-long.432>

**DWIE: an entity-centric dataset for multi-task document-level information extraction**. Zaporojets et al. 2021\
Dataset similar to DocRED, news articles with entity linking to Wiki.\
<https://arxiv.org/pdf/2009.12626.pdf>

**HacRED: A Large-Scale Relation Extraction Dataset Toward Hard Cases in Practical Applications**. Cheng et al. ACL
Findings'21\
Hard cases selected by a trained classifier and verified by human.\
<https://aclanthology.org/2021.findings-acl.249>

**Multi-Task Identification of Entities, Relations, and Coreference for Scientific Knowledge Graph Construction**. Luan et
al. EMNLP'18\
SciERC dataset: similar to DocRED but on scientific abstract.\
<https://aclanthology.org/D18-1360/>

**SCIREX: A Challenge Dataset for Document-Level Information Extraction**. Jain et al. ACL'20\
SciREX dataset: similar to  SciERC but on very long documents. Approach: separate out COREF as clustering (probably due to too long doc).\
<https://aclanthology.org/2020.acl-main.670>

**CodRED: A Cross-Document Relation Extraction Dataset for Acquiring Knowledge in the Wild**. Yao et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.366/>

**Dialogue-Based Relation Extraction**. Yu et al. ACL'21\
On Friends data.\
<https://aclanthology.org/2020.acl-main.444>

**DocEE: A Large-Scale and Fine-grained Benchmark for Document-level Event Extraction**. Tong et al. NAACL'22\
Doc-level event extraction.\
<https://aclanthology.org/2022.naacl-main.291>

**BioCreative V CDR task corpus: a resource for chemical disease relation extraction**. Li et al. 2016\
<https://academic.oup.com/database/article/doi/10.1093/database/baw068/2630414>

## Approach: Focus on Joint

**A Hierarchical Multi-task Approach for Learning Embeddings from Semantic Tasks**. Sanh et al. AAAI'19\
Hierarchical repr
for each level among e2e multi-tasks (mention + coref + RE); each level is BiLSTM taking [orig; prev level]; hierarchy
order matters. RE is observed to benefit from NER and coref. Only share encoding; representation receives no task
interference.\
<https://arxiv.org/pdf/1811.06031.pdf>

**A General Framework for Information Extraction using Dynamic Span Graphs**. Luan et al. NAACL'19\
DYGIE: span-based
pipeline + refinement/propagation on both coref and RE; coref prop (higher-order) helps very trivially; RE propagation
helps a bit more. Share encoding; representation has task interference from each task propagation. However, still
pipeline and no explicit task interaction.\
<https://aclanthology.org/N19-1308/>

**Entity, Relation, and Event Extraction with Contextualized Span Representations**. Wadden et al. EMNLP'19\
DYGIE++: (1) use
BERT (2) add event extraction and its propagation.\
<https://aclanthology.org/D19-1585/>

**An End-to-end Model for Entity-level Relation Extraction using Multi-instance Learning**. Eberts and Ulges. EACL'21\
Span-based pipeline + multi-instance RE (substitute entity-pair repr with localized mention context).\
<https://aclanthology.org/2021.eacl-main.319>

**Injecting Knowledge Base Information into End-to-End Joint Entity and Relation Extraction and Coreference Resolution**.
Verlinden et al. ACL Findings'21\
Span-based pipeline + Concat mention with attended KB candidate repr.\
<https://aclanthology.org/2021.findings-acl.171>

**Document-Level Event Argument Extraction by Conditional Generation**. Li et al. NAACL'21\
<https://aclanthology.org/2021.naacl-main.69>

**Document-level Entity-based Extraction as Template Generation**. Huang et al. EMNLP'21\
Benefits as generation: (1)
generation decoding captures entity dependencies (2) naturally n-ary without exponential combinations of entities.\
<https://aclanthology.org/2021.emnlp-main.426>

## Approach: Focus on Joint (Non-Doc-Level)

### Separate Modules w/o Interactions

**A Novel Cascade Binary Tagging Framework for Relational Triple Extraction**. Wei et al. ACL'20\
Two-step tagging: (1) tag subjects and associated relations (2) for each subject, tag objects.\
<https://aclanthology.org/2020.acl-main.136>

### Separate Modules w/ Interactions

**GraphRel: Modeling Text as Relational Graphs for Joint Entity and Relation Extraction**. Fu et al. ACL'19\
<https://aclanthology.org/P19-1136/>

**A Frustratingly Easy Approach for Entity and Relation Extraction**. Zhong and Chen. NAACL'21\
Encode per span pair: insert span marker in text.\
<https://aclanthology.org/2021.naacl-main.5/>

**TAGPRIME: A Unified Framework for Relational Structure Extraction**. Hsu et al. ACL'23\
Encode per given condition (trigger, head, etc.): simply append condition in text + BIO sequence tagging.\
<https://aclanthology.org/2023.acl-long.723>

**Cross-Task Instance Representation Interactions and Label Dependencies for Joint Information Extraction with Graph
Convolutional Networks**. Nguyen et al. NAACL'21\
<https://aclanthology.org/2021.naacl-main.3>

**PRGC: Potential Relation and Global Correspondence Based Joint Relational Triple Extraction**. Zheng et al. ACL'21\
<https://aclanthology.org/2021.acl-long.486>

**A Partition Filter Network for Joint Entity and Relation Extraction**. Yan et al. EMNLP'21\
Task-interaction in LSTM encoder.\
<https://aclanthology.org/2021.emnlp-main.17/>

**A Joint Neural Model for Information Extraction with Global Features**. Lin et al. ACL'20\
OneIE: consider task/entity interactions in a global graph, with human-designed dependency features.\
Optimization is on global graph (scores combining local and global features).\
<https://aclanthology.org/2020.acl-main.713>

### One-shot Module: S2S

**Joint Entity and Relation Extraction with Set Prediction Networks**. Sui et al. ArXiv'20\
S2S with non-autoregressive decoder + set prediction loss.\
Generation: one triple hidden state at each step; then decode into actual triple.\
Benefits over autoregressive: (1) avoid train/inference gap (2) avoid sequential decoding; now bidirectional decoding (3) avoid order.\
<https://arxiv.org/pdf/2011.01675>

**Extracting relational facts by an end-to-end neural model with copy mechanism**. Zeng et al. ACL'18\
CopyRE: first S2S on joint RE; baseline of CopyMTL.\
<https://aclanthology.org/P18-1047/>

**CopyMTL: Copy Mechanism for Joint Extraction of Entities and Relations with Multi-Task Learning Authors**. Zeng et al. AAAI'20\
S2S generation: one triple every three steps; w/ tagging to recover whole mentions.\
<https://arxiv.org/pdf/1911.10438>

**Effective Modeling of Encoder-Decoder Architecture for Joint Entity and Relation Extraction**. Nayak and Ng. AAAI'20\
S2S generation: (1) either lexical form generation, or (2) one triple hidden state at each step.\
<https://arxiv.org/pdf/1911.09886>

**Minimize Exposure Bias of Seq2Seq Models in Joint Entity and Relation Extraction**. Zhang et al. EMNLP Findings'20\
S2S generation: use tree-decoder that decodes triples in parallel at each step to avoid ordered output.\
<https://aclanthology.org/2020.findings-emnlp.23>

**Contrastive Triple Extraction with Generative Transformer**. Ye et al. AAAI'21\
<https://arxiv.org/abs/2009.06207>

**REBEL: Relation Extraction By End-to-end Language generation**. Cabot and Navigli. EMNLP Findings'21\
S2S on sentence-level or doc-level joint RE in lexical form generation, w/ pretraining on a designed wiki dataset.\
<https://aclanthology.org/2021.findings-emnlp.204/>

**GenerativeRE: Incorporating a Novel Copy Mechanism and Pretrained Model for Joint Entity and Relation Extraction**. Cao and Ananiadou. EMNLP Findings'21\
<https://aclanthology.org/2021.findings-emnlp.182/>

**Unified Structure Generation for Universal Information Extraction**. Lu et al. ACL'22\
S2S:\
(1) Add prompt as input to guide what to extract, enabling zero-shot task (model learns tasks from prompts)\
(2) Add "Rejection Mechanism" to alleviate exposure bias of auto-regressive and reject misleading generation.\
(3) Pretrain on aligned wiki to adapt encoding and decoding schema.\
(4) Open-domain / zero-shot capability from pretraining knowledge.\
<https://aclanthology.org/2022.acl-long.395>

**Dynamic Prefix-Tuning for Generative Template-based Event Extraction**. Liu et al. ACL'22\
First train S2S template event generation. Input: event type + template + context.\
Then further apply prefix-tuning: fix LM, and learn prefix conditioned on event type, and another prefix conditioned on both type and input context, prepended to the input.\
Adding prefix-tuning gives ~1-2% improvement.\
<https://aclanthology.org/2022.acl-long.358>

**STRUCTURED PREDICTION AS TRANSLATION BETWEEN AUGMENTED NATURAL LANGUAGES**. Paolini et al. ICLR'21\
S2S generation on many tasks, generating augmented lexicons with original text.\
Output original text to avoid duplicates and boost a little bit performance.\
<http://arxiv.org/abs/2101.05779>

### One-shot Module: Table Filling or Tagging Scheme

**Joint Extraction of Entities and Relations Based on a Novel Tagging Scheme**. Zheng et al. ACL'17\
Sequence tagging (not TABLE tagging): <entity BIO, relation, position>; cannot handle overlapping.\
Entities are squares on diagonal, relations are rectangles off diagonal (interactions).\
Equivalent to entity tagging along fused with learned relation bias.\
<https://aclanthology.org/P17-1113/>

**Modeling Joint Entity and Relation Extraction with Table Representation**. Miwa and Sasaki. EMNLP'14\
First table filling work: separate entity and relation labels in the table; relation label at head-tail intersection; cannot handle overlapping.\
<https://aclanthology.org/D14-1200/>

**Table Filling Multi-Task Recurrent Neural Network for Joint Entity and Relation Extraction**. Gupta et al. COLING'16\
<https://aclanthology.org/C16-1239/>

**End-to-End Neural Relation Extraction with Global Optimization**. Zhang et al. EMNLP'17\
<https://aclanthology.org/D17-1182>

**Two are Better than One: Joint Entity and Relation Extraction with Table-Sequence Encoders**. Wang and Lu. EMNLP'20\
Add a table encoder to directly consume table, in addition to the sequence encoding.\
<https://aclanthology.org/2020.emnlp-main.133/>

**UNIRE: A Unified Label Space for Entity Relation Extraction**. Wang et al. ACL'21\
Table tagging (unified entity and relation label space).\
<https://aclanthology.org/2021.acl-long.19>

**OneRel: Joint Entity and Relation Extraction with One Module in One Step**. Shang et al. AAAI'22\
Table tagging per relation but different from table filling scheme: include triple boundary in tagging scheme.\
<https://arxiv.org/pdf/2203.05412>

**Universal Information Extraction as Unified Semantic Matching**. Lou et al. AAAI'23\
USM: token-token interaction; incorporate type by verbalizing type semantics, thus only need linking, no need for tagging/classification.\
Unify different IE tasks into linking; support multi-task, pretraining, zero/few-shot.\
Compare to UIE: different formulation, but all convert/unify IE to automic ops, enabling pretraining and zero-shot.\
<http://arxiv.org/abs/2301.03282>

## Approach: focus on RE only (given Entities)

### Sequence-based RE

**HIN: Hierarchical Inference Network for Document-Level Relation Extraction**. Tang et al. PAKDD'20\
Hierarchical encoding:
doc repr of entity pair, which is attended sent repr conditioned on entity pair repr.\
<https://arxiv.org/pdf/2003.12754>

**Document-Level Relation Extraction with Adaptive Thresholding and Localized Context Pooling**. Zhou et al. AAAI'21\
(1) Marginalized pos and neg relation prediction by using a learned threshold\
(2) gather entity-level self-attention to concat as a new context emb for entity pair.\
<https://arxiv.org/pdf/2010.11304>

**Document-Level Relation Extraction with Adaptive Focal Loss and Knowledge Distillation**. Tan et al. ACL Findings'22\
(1) Add entity interaction by attention (2) modified adaptive loss for class imbalance (3) semi-supervised pseudo label.\
<https://aclanthology.org/2022.findings-acl.132>

**Entity Structure Within and Throughout: Modeling Mention Dependencies for Document-Level Relation Extraction**. Xu et al.
AAAI'21\
Structural encoding (token-level coref and co-occurance structure) into self-attention.\
<https://arxiv.org/pdf/2102.10249>

**Learning Logic Rules for Document-level Relation Extraction**. Ru et al. EMNLP'21\
TODO\
<https://aclanthology.org/2021.emnlp-main.95>

**Modular Self-Supervision for Document-Level Relation Extraction**. Zhang et al. EMNLP'21\
TODO\
<https://aclanthology.org/2021.emnlp-main.429>

**Document-Level Relation Extraction with Reconstruction**. Xu et al. AAAI'21\
<https://arxiv.org/abs/2012.11384>

### Graph-based RE

**Connecting the Dots: Document-level Neural Relation Extraction with Edge-oriented Graphs**. Christopoulou et al. EMNLP'19\
Heterogeneous graph.\
<https://aclanthology.org/D19-1498/>

**Global Context-enhanced Graph Convolutional Networks for Document-level Relation Extraction**. Zhou et al. COLING'20\
<https://aclanthology.org/2020.coling-main.461>

**Global-to-Local Neural Networks for Document-Level Relation Extraction**. Wang et al. EMNLP'20\
<https://aclanthology.org/2020.emnlp-main.303>

**Double Graph Based Reasoning for Document-level Relation Extraction**. Zeng et al. EMNLP'20\
<https://aclanthology.org/2020.emnlp-main.127>

**Reasoning with Latent Structure Refinement for Document-Level Relation Extraction**. Nan et al. ACL'21\
Similar to
higher-order propagation: compute scores to construct a latent structure for entities, then GCN to obtain entity repr.\
<https://aclanthology.org/2020.acl-main.141>

**Discriminative Reasoning for Document-level Relation Extraction**. Xu et al. ACL Findings'21\
TODO\
<https://aclanthology.org/2021.findings-acl.144>

**SIRE: Separate Intra- and Inter-sentential Reasoning for Document-level Relation Extraction**. Zeng et al. ACL Findings'21\
TODO\
<https://aclanthology.org/2021.findings-acl.47>

**MRN: A Locally and Globally Mention-Based Reasoning Network for Document-Level Relation Extraction**. Li et al. ACL
Findings'21\
TODO\
<https://aclanthology.org/2021.findings-acl.117>

**Joint Extraction of Entities, Relations, and Events via Modeling Inter-Instance and Inter-Label Dependencies**. Nguyen et al. NAACL'22\
Explicitly adding dependencies on instances/labels for global decoding/optimization.\
<https://aclanthology.org/2022.naacl-main.324>

## Approach: prompting

## Approach: focus on Pretraining

**Matching the Blanks: Distributional Similarity for Relation Learning**. Livio et al. ACL'19\
Sent-level.\
<https://aclanthology.org/P19-1279/>

**Learning from Context or Names? An Empirical Study on Neural Relation Extraction**. Peng et al. EMNLP'20\
Sent-level.\
Analysis: context is the main source to support classification, entity type also provides critical information; mention
itself may provide shallow clues without need to understand context. Approach: contrastive learning utilizing KB; help
with low-resource\
<https://aclanthology.org/2020.emnlp-main.298>

**Coreferential Reasoning Learning for Language Representation**. Ye et al. EMNLP'20\
Doc-level. Pretrain with Mention
Reference Prediction (in addition to MLM): mask one mention and optimize the marginal likelihood of repeated mentions (
heuristic assumption: repeated noun mentions in a sequence refer to each other). Improvement is dataset-specific;
trivial for DocRED.\
<https://aclanthology.org/2020.emnlp-main.582.pdf>

**Denoising Relation Extraction from Document-level Distant Supervision**. Xiao et al. EMNLP'20\
TODO\
<https://aclanthology.org/2020.emnlp-main.300>

**ERICA: Improving Entity and Relation Understanding for Pre-trained Language Models via Contrastive Learning**. Qin et al.
ACL'21\
Doc-level. Contrastive learning on two pretraining tasks on entity and relation representation; help with
low-resource\
<https://aclanthology.org/2021.acl-long.260>

## Approach: focus on Robustness/Evidence

**Three Sentences Are All You Need: Local Path Enhanced Document Relation Extraction**. Huang et al. ACL'21\
Heuristic rules
to select evidence sentences (on average < 3 sentences) and discard others.\
<https://aclanthology.org/2021.acl-short.126>

**Entity and Evidence Guided Relation Extraction for DocRED**. Huang et al. 2020\
<https://arxiv.org/abs/2008.12283>

**EIDER: Evidence-enhanced Document-level Relation Extraction**. Xie et al. ACL'21\
Train a simple sentence-evidence binary
classier per entity pair, then use predicted evidence to predict relations. Final results use fusion between full doc
and evidence only.\
<https://arxiv.org/pdf/2106.08657>

**Document-Level Relation Extraction with Sentences Importance Estimation and Focusing**. Xu et al. NAACL'22\
<https://aclanthology.org/2022.naacl-main.212>

**Does it Really Generalize Well on Unseen Data? Systematic Evaluation of Relational Triple Extraction Methods**. Lee et al. NAACL'22\
RE model cannot really generalize well on unseen data.\
Propose to randomly replace entity with completely noisy words to avoid memorizing entities.\
<https://aclanthology.org/2022.naacl-main.282>

**Evidence-aware Document-level Relation Extraction**. Xu et al. CIKM'22\
Regard evidence extraction as sequential decision problem, solve by Deep Q RL.\
To reduce search space, convert document to entity graph, with co-occurrence sentences as edges, as search candidates.\
<https://dl.acm.org/doi/10.1145/3511808.3557313>

# Task: for other Datasets
---

**Enhancing Dialogue-based Relation Extraction by Speaker and Trigger Words Prediction**. Zhao et al. ACL Findings'21\
On DialogRE.\
<https://aclanthology.org/2021.findings-acl.402>

**Selecting Optimal Context Sentences for Event-Event Relation Extraction**. Trong et al. AAAI'22\
Pairwise scoring on given event spans: use REINFORCE to select important sentences and compress documents.\
<https://www.aaai.org/AAAI22Papers/AAAI-3912.ManH.pdf>

**Text-to-Table: A New Way of Information Extraction**. Wu et al. ACL'22\
S2S.\
<https://aclanthology.org/2022.acl-long.180>

# Task: n-ary RE
---

**Cross-Sentence N-ary Relation Extraction with Graph LSTMs**. Peng et al. TACL'17\
TODO\
<https://aclanthology.org/Q17-1008/>

**Document-Level N-ary Relation Extraction with Multiscale Representation Learning**. Jia et al. NAACL'19\
Entity-centric n-ary relation extraction.\
<https://aclanthology.org/N19-1370/>

# Task: Few-Shot IE
---

**FewRel 2.0: Towards More Challenging Few-Shot Relation Classification**. Gao et al. EMNLP'19\
<https://aclanthology.org/D19-1649/>

**Entity Concept-enhanced Few-shot Relation Extraction**. Yang et al. ACL'21\
<https://aclanthology.org/2021.acl-short.124>

**Exploring Task Difficulty for Few-Shot Relation Extraction**. Han et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.204>

**Label Verbalization and Entailment for Effective Zero and Few-Shot Relation Extraction**. Sainz et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.92>

**Towards Realistic Few-Shot Relation Extraction**. Brody et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.433/>

# Task: Open-IE
---

## Cluster-based

**Open Relation Extraction: Relational Knowledge Transfer from Supervised Data to Unsupervised Data**. Wu et al. EMNLP'19\
<https://aclanthology.org/D19-1021>

**SelfORE: Self-supervised Relational Feature Learning for Open Relation Extraction**. Hu et al. EMNLP'20\
<https://aclanthology.org/2020.emnlp-main.299>

**Element Intervention for Open Relation Extraction**. Liu et al. ACL'21\
<https://aclanthology.org/2021.acl-long.361>

**Unsupervised Relation Extraction: A Variational Autoencoder Approach**. Yuan and Eldardiry. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.147>

**A Relation-Oriented Clustering Method for Open Relation Extraction**. Zhao et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.765/>

## Predicate-based

**Supervised Open Information Extraction**. Stanovsky et al. NAACL'18\
<https://aclanthology.org/N18-1081>

**IMOJIE: Iterative Memory-Based Joint Open Information Extraction**. Kolluru et al. ACL'20\
<https://aclanthology.org/2020.acl-main.521>

**OpenIE6: Iterative Grid Labeling and Coordination Analysis for Open Information Extraction**. Kolluru et al. EMNLP'20\
<https://aclanthology.org/2020.emnlp-main.306/>

**Multi2OIE: Multilingual Open Information Extraction Based on Multi-Head Attention with BERT**. Ro et al. EMNLP Findings'20\
<https://arxiv.org/abs/2009.08128>

**LSOIE: A Large-Scale Dataset for Supervised Open Information Extraction**. Solawetz and Larson. EACL'21\
<https://aclanthology.org/2021.eacl-main.222>

**Zero-Shot Information Extraction as a Unified Text-to-Triple Translation**. Wang et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.94>

**DetIE: Multilingual Open Information Extraction Inspired by Object Detection**. Vasilkovsky et al. AAAI'22\
<https://www.aaai.org/AAAI22Papers/AAAI-8073.VasilkovskyM.pdf>

# Task: Continual RE
---

**Sentence Embedding Alignment for Lifelong Relation Extraction**. Wang et al. NAACL'19

**Continual Relation Learning via Episodic Memory Activation and Reconsolidation**. Xu et al. ACL'20

**Curriculum-Meta Learning for Order-Robust Continual Relation Extraction**. Wu et al. AAAI'21

# Task: KG Completion
---

**Inductive Relation Prediction by BERT**. Zha et al. AAAI'22\
Linearize each local subgraph individually and encode by BERT, paired with target triple to do MIL scoring.\
Thereafter to achieve both semantic representation and implicit rule inference.\
<https://arxiv.org/pdf/2103.07102>

# Analysis
---

**More Data, More Relations, More Context and More Openness: A Review and Outlook for Relation Extraction**. Han et al. AACL'20\
Directions: (1) more utilization of distant/self-supervision (2) more domains (3) more complex context.\
<https://aclanthology.org/2020.aacl-main.75>

**Automatic Error Analysis for Document-level Information Extraction**. Das et al. ACL'22\
<https://aclanthology.org/2022.acl-long.274/>

# LLM
---

**Grammar-Constrained Decoding for Structured NLP Tasks without Finetuning**. Geng et al. EMNLP'23\
Unify decoding constraints of multiple structured output tasks, by formal language (grammar) description.\
<https://aclanthology.org/2023.emnlp-main.674>
