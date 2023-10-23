---
layout: post
title: "Work on Neural Topic Modeling"
date: 2022-08-07
categories: [NLP]
tags: [nlp, topic modeling]
math: true
---

See another post for Bayes-related.

**Neural Variational Inference for Text Processing**. Miao et al. ICML'16\
Document modeling: apply VAE on document to learn latent document representation (latent document semantics), dubbed **NVDM**.\
Note that no topics are explicitly modeled; NOT topic modeling.\
Event: likelihood to reconstruct document given latent document representation.\
Posteriori to learn: latent document representation of the input document.\
<https://arxiv.org/abs/1511.06038>

**Discovering Discrete Latent Topics with Neural Variational Inference**. Miao et al. ICML'17\
Topic modeling: variational inference to learn approximated topic distribution, dubbed **GSM**.\
Event: likelihood to generate the input document given topic distribution.\
Posteriori to learn: the latent topic distribution of the input document.\
<https://arxiv.org/abs/1706.00359>

**Topic Modeling in Embedding Spaces**. Dieng et al. TACL'20\
Similar to Miao et al., 2017.\
<https://aclanthology.org/2020.tacl-1.29/>

**Autoencoding variational inference for topic models**. Srivastava and Sutton. ICLR'17\
ProdLDA: replace mixture of topics in LDA with a product of experts.\
<https://arxiv.org/abs/1703.01488>

**Coherence-Aware Neural Topic Modeling**. Ding et al. EMNLP'18\
Incorporate explicit topic coherence regularization in VI training.\
<https://aclanthology.org/D18-1096>
