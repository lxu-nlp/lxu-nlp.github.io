---
layout: post
title: "Works on LLM Decoding"
date: 2024-08-05
categories: [NLP]
tags: [nlp, LLM, decoding]
math: true
---


# Speculative Decoding

**Fast Inference from Transformers via Speculative Decoding**. Leviathan et al. ICML'23\
n-times autoregression by smaller draft model + LLM verification once that recovers original distribution through rejection sampling strategies.\
Easier positions are now handled by the smaller draft model.\
<https://openreview.net/forum?id=C9NEblP8vS>

**EAGLE: Speculative Sampling Requires Rethinking Feature Uncertainty**. Li et el. ICML'24\
(1) learn a small draft model that mimics the final hidden states (features) generated from the original LLM, to be consumed directly by the LM head of original LLM\
(2) draft model: (latest token, previous feature) -> latest feature, next token; same effect as original LLM: (latest token, past kv) -> latest kv, next token\
(3) draft tree rather than chain; LLM verification once on the tree.\
<https://openreview.net/forum?id=1NdN7eXyb4>
