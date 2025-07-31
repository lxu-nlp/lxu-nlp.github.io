---
layout: post
title: "Works on Embedding Representation"
date: 2025-03-12
categories: [NLP]
tags: [nlp, embedding]
math: true
---

## Benchmark

**MTEB: Massive Text Embedding Benchmark**. Muennighoff et al. EACL'23\
<https://aclanthology.org/2023.eacl-main.148/>

**MMTEB: Massive Multilingual Text Embedding Benchmark**. Enevoldsen et al. ICLR'25\
<https://openreview.net/forum?id=zl3pfz4VCV>

---

## Embedding by Non-LLM

**Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks**. Reimers and Gurevych. EMNLP'19\
S-BERT: first dual encoder.\ 
<https://aclanthology.org/D19-1410/>

**SimCSE: Simple Contrastive Learning of Sentence Embeddings**. Gao et al. EMNLP'21\
SimCSE: dropout as unsupervised negatives.\
<https://aclanthology.org/2021.emnlp-main.552/>

**Unsupervised Dense Information Retrieval with Contrastive Learning**. Izacard et al. TMLR'22\
Contriever: unsupervised.\
<https://openreview.net/forum?id=jKN1pXi7b0>

**Text and Code Embeddings by Contrastive Pre-Training**. Neelakantan et al. 2022\
OpenAI embedding: unsupervised + supervised.\
Large batch size: big improvement; large model size: ok improvement.\
<https://arxiv.org/abs/2201.10005>

**Large Dual Encoders Are Generalizable Retrievers**. Ni et al. EMNLP'22\
Increasing model size: mediocre improvement.\ 
<https://aclanthology.org/2022.emnlp-main.669/>

**Text Embeddings by Weakly-Supervised Contrastive Pre-training**. Wang et al. 2022\
E5: unsupervised + supervised.\
<https://arxiv.org/abs/2212.03533>

**Multilingual E5 Text Embeddings: A Technical Report**. Wang et al. 2023\
mE5.\
<https://arxiv.org/abs/2402.05672>

**C-Pack: Packed Resources For General Chinese Embeddings**. Xiao et al. SIGIR'24\
BGE: unsupervised + supervised.\
<https://arxiv.org/abs/2309.07597>

**Towards General Text Embeddings with Multi-stage Contrastive Learning**. Li et al. 2023\
GTE: unsupervised + supervised.\
<https://arxiv.org/abs/2308.03281>

**mGTE: Generalized Long-Context Text Representation and Reranking Models for Multilingual Text Retrieval**. Zhang et al. EMNLP Industry'24\
GTE with enhanced Transformers: match BGE-M3.\
<https://aclanthology.org/2024.emnlp-industry.103>

## Embedding by LLM

**Improving Text Embeddings with Large Language Models**. Wang et al. ACL'24\
LLM-generated corpus + hidden state from the last LLM position.\
<https://aclanthology.org/2024.acl-long.642>

**LLM2Vec: Large Language Models Are Secretly Powerful Text Encoders**. BehnamGhader et al. COLM'24\
Bidirectional + contrastive.\
<https://openreview.net/forum?id=IW1PR7vEBf>

**BeLLM: Backward Dependency Enhanced Large Language Model for Sentence Embeddings**. Li and Li. NAACL'24\
Bidirectional on last layer + contrastive.\
<https://arxiv.org/pdf/2311.05296>

**NV-Embed: Improved Techniques for Training LLMs as Generalist Embedding Models**. Lee et al. ICLR'25\
Latent pooling: hidden state from the last LLM position (Q) attended over latent dictionaries (K).\
<https://arxiv.org/abs/2405.17428>

## Other Representation by LLM

**Scaling Sentence Embeddings with Large Language Models**. Jiang et al. 2023\
Use template for CLM to obtain the last hidden state: This sentence: \[text\] means in one word:\
<https://arxiv.org/abs/2307.16645>

**Meaning Representations from Trajectories in Autoregressive Models**. Liu et al. ICLR'24\
Core idea: Wittgenstein’s use theory of meaning (meaning is use).\
Meaning representation: the likelihood of the continuation in language space.\
Similarity: sampling continuation equal times from both two inputs as the approximated continuation space.\
<https://arxiv.org/abs/2310.18348>

## Training Paradigms

**EASE: Entity-Aware Contrastive Learning of Sentence Embedding**. Nishikawa et al. NAACL'22\
<https://aclanthology.org/2022.naacl-main.284>

## Embedding Granularity

**Simple Entity-Centric Questions Challenge Dense Retrievers**. Sciavolino et al. EMNLP'21\
Recognize entity in entity-centric queries: unable to generalize to long-tail entities unless the query pattern is seen.\
<https://aclanthology.org/2021.emnlp-main.496>

**Salient Phrase Aware Dense Retrieval: Can a Dense Retriever Imitate a Sparse One?**. Chen et al. EMNLP Findings'22\
Able to mimic BM25 by dense retriever (but does it generalize?)\
Combine with DPR by concatenation works.\
<https://aclanthology.org/2022.findings-emnlp.19>

## Embedding Privacy

**Sentence Embedding Leaks More Information than You Expect: Generative Embedding Inversion Attack to Recover the Whole Sentence**. Li et al. ACL Findings'23\
<https://aclanthology.org/2023.findings-acl.881/>

**Text Embeddings Reveal (Almost) As Much As Text**. Morris et al. EMNLP'23\
Train a generator to recover input of its embedding from a black-box encoder.\
(which can be a strong argument that auto-encoder != semantic comprehension, as embedding performance is not perfect on downstream tasks)\
<https://aclanthology.org/2023.emnlp-main.765>
