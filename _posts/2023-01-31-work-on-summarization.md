---
layout: post
title: "Work on Summarization"
date: 2023-01-31
categories: [NLP]
tags: [nlp, summarization]
math: true
---

This article mainly focuses on dialogue summarization, but also include general summ work.

Dialogue characteristics: 1) different viewpoints / personal perspectives; 2) speaker role change; 3) interactive information, coreference; 4) topic flow/switch.

Approach characteristics: utilize dialogue-specific features, e.g. topic flow, discourse structure, speaker interaction, coreference information, etc.

## Datasets

**The AMI meeting corpus**. McCowan et al. 2005\
AMI: 137 virtual product design meetings.\
<https://research.utwente.nl/en/publications/the-ami-meeting-corpus>

**The ICSI Meeting Corpus**. Janin et al. ICASSP'03\
ICSI: 59 research meetings of a speech group.
<https://ieeexplore.ieee.org/document/1198793>

**QMSum: A New Benchmark for Query-based Multi-domain Meeting Summarization**. Zhong et al. NAACL'21\
QMSum: 1.8k long meetings (AMI, ICSI, committee meetings), with query-based summarization.\
<https://aclanthology.org/2021.naacl-main.472/>

**SQuALITY: Building a Long-Document Summarization Dataset the Hard Way**. Wang et al. EMNLP'22\
SQuALITYL story summarization (long input), with new annotation protocal to ensure high quality.\
<https://aclanthology.org/2022.emnlp-main.75>

**Storytelling with Dialogue: A Critical Role Dungeons and Dragons Dataset**. Rameshkumar and Bailey. ACL'20\
CRD3: 35k dialogues in role-playing games.\
<https://aclanthology.org/2020.acl-main.459/>

**SAMSum Corpus: A Human-annotated Dialogue Dataset for Abstractive Summarization**. Gliwa et al. 2019\
SAMSum: 16k daily conversation on messengers; formal or informal with slangs.\
High-quality, annotated by linguistics; short dialogues.\
<https://aclanthology.org/D19-5409/>

**MSAMSum: Towards Benchmarking Multi-lingual Dialogue Summarization**. Feng et al. 2022\
MSAMSum: six language settings from SAMSum, using translation.\
<https://aclanthology.org/2022.dialdoc-1.1>

**CLIDSUM: A Benchmark Dataset for Cross-Lingual Dialogue Summarization**. Wang et al. EMNLP'22\
ClidSum: 56k English dialogues + Chinese/German summaries from SAMSum MediaSum.\
<https://aclanthology.org/2022.emnlp-main.526>

**DIALOGSUM: A Real-Life Scenario Dialogue Summarization Dataset**. Chen et al. ACL Findings'21\
DialogSum: 13k real-life spoken dialogues, with various daily topics.\
Similar to SAMSum, but: 40% longer dialogues, more business/task topics, more abstractive.\
<https://aclanthology.org/2021.findings-acl.449>

**ConvoSumm: Conversation Summarization Benchmark and Improved Abstractive Summarization with Argument Mining**. Fabbri et al. ACL'21\
ConvoSumm: 1k dev/test for conversations 1) online discussion 2) email threads 3) comments 4) QA.\
<https://aclanthology.org/2021.acl-long.535>

**MEDIASUM: A Large-scale Media Interview Dataset for Dialogue Summarization**. Zhu et al. NAACL'21\
MediaSum: large-scale 460k tv/radio interviews with silver summaries.\
Corpus: more turns, more speakers, longer context; but summaries are rather short.\
Not really helpful as pretraining.\
<https://aclanthology.org/2021.naacl-main.474>

**100,000 Podcasts: A Spoken English Document Corpus**. Clifton et al. COLING'20\
Spotify Podcast Dataset: large-scale transcribed speech text, characterized: 1) ASR errors 2) diverse format/topics 3) long context.\
<https://aclanthology.org/2020.coling-main.519>

**Summarizing Medical Conversations via Identifying Important Utterances**. Song et al. COLING'20\
45k online medical inquiry conversations between doctors and patients; separate summaries by patients and doctors.\
<https://aclanthology.org/2020.coling-main.63>

**Dr. Summarize: Global Summarization of Medical Dialogue by Exploiting Local Structures**. Joshi et al. EMNLP Findings'20\
17k telemedicine dialogue snippets; summaries written by doctors.\
<https://aclanthology.org/2020.findings-emnlp.335>

**Automatic Dialogue Summary Generation for Customer Service**. Liu et al. KDD'19\
Didi customer service dataset (not publicly available).\
<https://dl.acm.org/doi/pdf/10.1145/3292500.3330683>

**Topic-Oriented Spoken Dialogue Summarization for Customer Service with Saliency-Aware Topic Modeling**. Zou et al. AAAI'21\
19k non-readable (anonymized) customer service data.\
Approach: topic modeling to distinguish salient works in summary and other words; two-stage paradigm (extract-abstract).\
<https://arxiv.org/pdf/2012.07311>

**CSDS: A Fine-Grained Chinese Dataset for Customer Service Dialogue Summarization**. Lin et al. EMNLP'21\
CSDS: 11k customer service dialogues, with annotated summaries in explicit speaker & topic structure.\
Summary consists of Query-Agent pairs; each pair is a topic, and the pair order represents topic flow.\
<https://aclanthology.org/2021.emnlp-main.365>

**SummScreen: A Dataset for Abstractive Screenplay Summarization**. Chen et al. ACL'22\
SummScreen: 4.3k/22.5k episodes in TV series: transcripts & episode recaps.\
Corpus: very long dialogues, many speakers; main plots are often indirect (not in direct transcripts) and many subplots are scattered.\
<https://aclanthology.org/2022.acl-long.589>

**Are We Summarizing the Right Way? A Survey of Dialogue Summarization Data Sets**. Tuggener et al. 2022\
A systematic survey on datasets: 1) meetings 2) interviews 3) customer/patient support 4) spontaneous Conversations.\
<https://aclanthology.org/2021.newsum-1.12>

#### Non-Dialogue Datasets

**Paragraph-level Simplification of Medical Texts**. Devaraj et al. NAACL'21\
Rephrasing medical text for easier understanding.\
Adjust decoding NLL to down-weight certain tokens.\
<https://aclanthology.org/2021.naacl-main.395>

**BOOKSUM: A Collection of Datasets for Long-form Narrative Summarization**. Kryscinski et al. EMNLP Findings'22\
Hierarchical: paragraph/chapter/book-level summarization.\
<https://aclanthology.org/2022.findings-emnlp.488>

**HIBRIDS: Attention with Hierarchical Biases for Structure-aware Long Document Summarization**. Cao et al. ACL'22\
Hierarchical summary based on saliency, with training set.\
<https://aclanthology.org/2022.acl-long.58>

## Approach: General

**PEGASUS: Pre-training with Extracted Gap-sentences for Abstractive Summarization**. Zhang et al. ICML'20\
Similar to T5, but mask multiple whole sentences of top importance based on ROUGE score.\
<https://arxiv.org/abs/1912.08777>

**DialogLM: Pre-trained Model for Long Dialogue Understanding and Summarization**. Zhong et al. AAAI'22\
Dialogue-specific corruption: disrupt content and order of speakers and utterances.\
<https://arxiv.org/abs/2109.02492>

**Fast Abstractive Summarization with Reinforce-Selected Sentence Rewriting**. Chen and Bansal. ACL'18\
FastRL: TODO\
<https://aclanthology.org/P18-1063>

**Unsupervised Abstractive Meeting Summarization with Multi-Sentence Compression and Budgeted Submodular Maximization**. Shang et al. ACL'18\
TODO\
<https://aclanthology.org/P18-1062/>

**A Bag of Tricks for Dialogue Summarization**. Khalifa et al. EMNLP'21\
(1) Speakers: replace with more common names seen in pretraining; (2) common-sense: finetuning with other tasks.\
<https://aclanthology.org/2021.emnlp-main.631>

**Back to the Future: Bidirectional Information Decoupling Network for Multi-turn Dialogue Modeling**. Li et al. EMNLP'22\
TODO\
<https://aclanthology.org/2022.emnlp-main.177>

**Learning to summarize from human feedback**. Stiennon et al. NIPS'20\
RLHF in summarization (see InstructGPT).\
Reward model: found to be better than automatic metrics; more closely follow the desired intent/behavior.\
<https://proceedings.neurips.cc/paper/2020/hash/1f89885d556929e98d3ef9b86448f951-Abstract.html>

## Approach: Long Input

[一文速览长文本摘要进展](https://mp.weixin.qq.com/s/qjLl69uTyitsyb2mkqh0JA)

**An Exploratory Study on Long Dialogue Summarization: What Works and What’s Next**. Zhang et al. EMNLP Findings'21\
Three types of models: 1) LongFormer; 2) retrieve-summarize pipeline; 3) hierarchical encoding.\
Observations: 1) retrieve-summarize pipeline performs well, even with weak retrieval; 2) CNN/Dailymail is good for pretraining.\
<https://aclanthology.org/2021.findings-emnlp.377>

**A Hierarchical Network for Abstractive Meeting Summarization with Cross-Domain Pretraining**. Zhu et al. EMNLP Findings'20\
TODO\
<https://aclanthology.org/2020.findings-emnlp.19/>

**Recursively Summarizing Books with Human Feedback**. Wu et al. 2021\
Recursive summarization (similar as below); each summarizer using RLHF.\
<http://arxiv.org/abs/2109.10862>

**Summ$^N$: A Multi-Stage Summarization Framework for Long Input Dialogues and Documents**. Zhang et al. ACL'22\
Recursive summarization/compression (coarse-to-fine), without information loss due to truncation or extraction.\
Obtain the intermediate segment-summary pairs for training, by greedy ROUGE-1.\
SOTA on multiple datasets.\
<https://aclanthology.org/2022.acl-long.112>

**Towards Abstractive Grounded Summarization of Podcast Transcripts**. Song et al. ACL'22\
Summarize on chunks sequentially, with chunk selection learned.\
<https://aclanthology.org/2022.acl-long.302>

**BooookScore: A systematic exploration of book-length summarization in the era of LLMs**. Chang et al. 2023\
(1) how to prompt LLMs to summarize long doc: hierarchical, or online\
(2) how to use LLMs to evaluate summary coherence, with similar human judgement\
<http://arxiv.org/abs/2310.00785>

## Approach: Topic, Discourse Structure

**A Discourse-Aware Attention Model for Abstractive Summarization of Long Documents**. Cohan et al. NAACL'18\
Hierarchical RNN encoding + decoding with attention on sections.\
New dataset: ArXiv, PubMed (abstract as gold)\.
<https://aclanthology.org/N18-2097/>

**Improving Abstractive Dialogue Summarization with Graph Structures and Topic Words**. Zhao et al. COLING'20\
Encoding: concat sequence encoding + structure encoding by GNN (node: topic words, utterance; edges: connectivity).\
<https://aclanthology.org/2020.coling-main.39>

**Multi-View Sequence-to-Sequence Models with Conversational Structure for Abstractive Dialogue Summarization**. Chen and Yang. EMNLP'20\
Objective: modeling designed topic flow by chunking (each chunk as a stage), as additional supervision.\
In decoding, attend on chunk repr (special stage token) and reweight attention by chunks.\
1% improvement.\
<https://aclanthology.org/2020.emnlp-main.336/>

**Structure-Aware Abstractive Conversation Summarization via Discourse and Action Graphs**. Chen and Yang. NAACL'21\
Objective: inject discourse-structure as additional supervision.\
In encoding, build discourse graph (each utterance as node) and action graph by external models, encoded by GAT.\
In decoding, obtain separate discourse/action cross-attention and fuse to single decoder hidden.\
< 1% improvement.\
<https://aclanthology.org/2021.naacl-main.109/>

**Discourse-Aware Neural Extractive Text Summarization**. Xu et al. ACL'20\
Objective: inject discourse-structure (RST & Coref graph) as additional supervision.\
Formulate extractive summarization as sequence labeling on discourse unit: encode unit + GAT + tagging.\
<https://aclanthology.org/2020.acl-main.451>

**Dialogue Discourse-Aware Graph Model and Data Augmentation for Meeting Summarization**. Feng et al. IJCAI'21\
Objective: inject discourse-structure (to LSTM-based generation to better model long-range dependency).\
Encoding: original + node from GNN.\
<https://arxiv.org/abs/2012.03502>

**Incorporating Distributions of Discourse Structure for Long Document Abstractive Summarization**. Pu et al. ACL'23\
Objective: inject soft discourse distribution in Longformer's attention.\
<https://aclanthology.org/2023.acl-long.306>

**Improving Abstractive Dialogue Summarization with Hierarchical Pretraining and Topic Segment**. Qi et al. EMNLP Findings'21\
TODO\
<https://aclanthology.org/2021.findings-emnlp.97>

**ConvoSumm: Conversation Summarization Benchmark and Improved Abstractive Summarization with Argument Mining**. Fabbri et al. ACL'21\
Approach: 1) vanilla S2S; 2) use existing argument-mining model to prune out non-informative sentences; 3) further build argument graph structure (arg-unit/sent as nodes, NLI to build relation edges) and linearize as input.\
Transform input by argument-mining can generally give improvement for long docs with multiple topics.\
<https://aclanthology.org/2021.acl-long.535>

**Guiding Generation for Abstractive Text Summarization Based on Key Information Guide Network**. Li et al. NAACL'18\
Extract salient spans as additional inputs, then generate while attending on salient tokens.\
<https://aclanthology.org/N18-2009>

**Controllable Abstractive Dialogue Summarization with Sketch Supervision**. Wu et al. ACL Findings'21\
Objective: modular generation per segment with additional heuristic weak supervision (learn to generation sketch as part of output).\
Generation: one sentence per segment, able to control length by controlling segments.\
Segment supervision: designed similarity to train chunking point classifier;\
Heuristic weak supervision: (utterance intent, key phrases) for each segment.\
<https://aclanthology.org/2021.findings-acl.454>

## Approach: Role/Speaker Aspect or Interaction

**Entity-based De-noising Modeling for Controllable Dialogue Summarization**. Liu and Chen. SIGDial'22\
Objective: personal NER alignment in dialogue.\
context + gold with masked and permuted personal NER => train a separate S2S denoiser, which can: 1) validate/edit generation; 2) data augmentation.\
Dataset: SAMSum\
<https://aclanthology.org/2022.sigdial-1.40>

**Controllable Neural Dialogue Summarization with Personal Named Entity Planning**. Liu and Chen. EMNLP'21\
Conditional generation on personal entities: modular generation by entities.\
<https://aclanthology.org/2021.emnlp-main.8>

**Other Roles Matter! Enhancing Role-Oriented Dialogue Summarization via Role Interactions**. Lin et al. ACL'22\
Objective: decoding per role to achieve role-aspect generation.\
Approach: whole context encoding & decoding => role-only context + role-context hidden cross attention on encoder & decoder.\
Flaw: only handle fixed roles (same decoder for each role); weak baseline.\
<https://aclanthology.org/2022.acl-long.182>

**Towards Modeling Role-Aware Centrality for Dialogue Summarization**. Liang et al. AACL'22\
Conditional generation on speaker/role.\
Objective: to be more role-aware, reweight encoder hidden per utterance, by manipulating utterance interactions.\
Trivial effects.\
<https://aclanthology.org/2022.aacl-short.6>

## Approach: Coreference (on named entities)

**Coreference-Aware Dialogue Summarization**. Liu et al. SIGDial'21\
Inject silver coreference explicitly in encoding.\
<https://aclanthology.org/2021.sigdial-1.53/>

## Alignment

**Summary-Source Proposition-level Alignment: Task, Datasets and Supervised Baseline**. Ernst et al. CoNLL'21\
Alignment between proposition-spans in summary and sentences in context.\
New dataset: use BERT similarity, NLI entailment score, ROUGE score to filter alignment + human filtering.\
Supervised model (as binary classification) gives better results as expected.\
<https://aclanthology.org/2021.conll-1.25>

## Knowledge

**Mind the Gap! Injecting Commonsense Knowledge for Abstractive Dialogue Summarization**. Kim et al. COLING'22\
TODO\
<https://aclanthology.org/2022.coling-1.548>

## Analysis

**Training Dynamics for Text Summarization Models**. Goyal et al. ACL Findings'22\
(1) Model initially learns copy (easier), gradually improving abstractiveness; (2) training longer cannot reduce factuality (actually increasing with abstractiveness)\
<https://aclanthology.org/2022.findings-acl.163>

## Others

**An End-to-End Dialogue Summarization System for Sales Calls**. Asi et al. NAACL Industry'22\
<https://aclanthology.org/2022.naacl-industry.6>

## Approach: LLM

**News Summarization and Evaluation in the Era of GPT-3**. Goyal et al. 2022\
Study on auto metrics for summarization, especially concerning the zero-shot sum by LLM prompts without emulating the references by training.\
(1) Overall: LLM performs much better than traditional finefined models by designed human study, while being more versatile.\
(2) Reference-based metrics: poor correlation with human judgement\
(3) Reference-free metrics (quality, factuality): poor correlation with human\
Suggesting: all these metrics are completely ineffective at evaluating zero-shot LLM prompted summaries.
<http://arxiv.org/abs/2209.12356>

**Benchmarking Large Language Models for News Summarization**. Zhang et al. 2023\
(1) Poor reference quality on XSUM; low correlation with human.\
(2) instruction tuning LLM is more crucial than model size\
(3) LLM performs on par with writers by human study.\
<http://arxiv.org/abs/2301.13848>

**Exploring the Limits of ChatGPT for Query or Aspect-based Text Summarization**. Yang et al. 2023\
Evaluation ChatGPT on aspect-based summarizaiton of different domains, especially long input.\
Observation: ChatGPT zero-shot prompting is comparable to finetuned methods.\
<http://arxiv.org/abs/2302.08081>

**Cross-Lingual Summarization via ChatGPT**. Wang et al. 2023\
(1) static prompts; (2) interaction prompts to control summary granularity.\
<http://arxiv.org/abs/2302.14229>
