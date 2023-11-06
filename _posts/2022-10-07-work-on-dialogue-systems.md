---
layout: post
title: "Dialogue-related"
date: 2022-10-07
categories: [NLP]
tags: [nlp, dialogue]
math: true
---

Out of date..

# Dialogue Systems

## Datasets

**EVA: An Open-Domain Chinese Dialogue System with Large-Scale Generative Pre-Training**. Zhou et al. 2021\
Repost, online forum, Q&A.\
<http://arxiv.org/abs/2108.01547>

**What would Harry say? Building Dialogue Agents for Characters in a Story**. Si et al. 2022\
Character response under a changing environment (relations to others, etc).\
Generation needs to be conditioned on the environment.\
<http://arxiv.org/abs/2211.06869>

**Doc2Bot: Accessing Heterogeneous Documents via Conversational Bots**. Fu et al. EMNLP'22\
Doc-grounded dialogues.\
<https://arxiv.org/pdf/2210.11060>

## Generation Approach

**BoB: BERT Over BERT for Training Persona-based Dialogue Models from Limited Personalized Data**. Song et al. ACL'21\
Persona consistency.\
<https://aclanthology.org/2021.acl-long.14>

## Retrieval Approach

**Learning Implicit User Profiles for Personalized Retrieval-Based Chatbot**. Qian et al. CIKM'21\
<https://arxiv.org/abs/2108.07935>

## Hybrid Approach

**Recipes for Building an Open-Domain Chatbot**. Roller et al. EACL'21\
Compare (1) retrieval (2) generative (3) retrieval-then-refine.\
<https://aclanthology.org/2021.eacl-main.24>

## Dialogue Evaluation

**Learning an Unreferenced Metric for Online Dialogue Evaluation**. Sinha et al. ACL'20\
Referenced metrics: correlate poorly to human judgement, as reasonable response can be versatile.\
Propose self-supervised unreferenced metric MADUE that correlate better with human: train model to score response on
large-scale corpus with negative examples.\
<https://aclanthology.org/2020.acl-main.220>

**MDD-Eval: Self-Training on Augmented Data for Multi-Domain Dialogue Evaluation**. Zhang et al. AAAI'22\
Good baselines with referenced and unreferenced metrics.\
Propose semi-supervised metric through self-learning with augmentation on unlabeled data; student model learns soft
pseudo labels.\
<http://arxiv.org/abs/2112.07194>

## Dialogue Management

**Recent Advances in Deep Learning Based Dialogue Systems: A Systematic Survey**. Ni et al. ArXiv'21\
<https://arxiv.org/abs/2105.04387>

**Neural, Neural Everywhere: Controlled Generation Meets Scaffolded, Structured Dialogue**. Chi et al. 2021\
System from Stanford in Alexa Prize 4; clear description on dialogue design.

**Multimodal Conversational AI A Survey of Datasets and Approaches**. Sundar and Heck. 2022
<https://aclanthology.org/2022.nlp4convai-1.12>

**Open-domain Dialogue Generation: What We Can Do, Cannot Do, And Should Do Next**. Kann et al. 2022\
<https://aclanthology.org/2022.nlp4convai-1.13>

**The Design and Implementation of XiaoIce, an Empathetic Social Chatbot**. Zhou et al. 2020\
XiaoIce DM:\
(1) top-level dialogue policy manages skill or general chat, coupled with topic manager to switch/suggest topics\
(2) user utterance: rewritten COREF and completion, recognize (intents, emotions, topics, opinions, persona)\
(3) response for chi-chat: retrieval-based or generation-based, determined by ranker that considers coherence and empathy\
<https://arxiv.org/abs/1812.08989>

## Thoughts

Thoughts for chi-chat: time-series structured dialogue state representation?
* Explicit structure knowledge (past history, external KB), instead of simple dialogue state tuple (ok for task)
* Inference/generation based on structures
* Conditioned on emotions, opinions, etc.
* Goal-oriented state-aware generation (hybrid)?

Emora: dialogue-flow schema
* humanly designed schema (similar to task-oriented but without specific goal), fully controls the topic; rigid, not scalable, precision than recall
* Not handling diverse user intents currently
* Handle opinion, emotion, experience, social in the schema design
* Schema: rules with slot -> runtime regex with slot filled
* Ontology: currently only for NLU slot, not for generation or inference
* Chi-chat: (1) not information oriented; (2) following properties are better: no self-contradiction, no common-sense contradiction, no repentance, being empathetic, talk about experience
* Chi-chat: thus, multi-turn capability matters
* Challenge: controllable hybrid approach (rule + generation); train generation with target state in-mind?
* Challenge: inference; need explicit structured knowledge/context representation?
