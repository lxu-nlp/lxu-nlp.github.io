---
layout: post
title: "Works on Multi-Modal"
date: 2022-03-20
categories: [NLP]
tags: [nlp, multimodal]
math: true
---

Key: alignment
* Naive alignment (similar to cross-lingual alignment): projection between separate feature space
* Mutual alignment (similar to multilingual embedding): mutual **interaction** on shared feature space
  * Encoder explicit alignment: direct interaction by pairing image-text (discriminative objective e.g. classification, contrastive)
  * Generation-based implicit alignment: indirect interaction by designing generation objective

**Clinical-BERT: Vision-Language Pre-Training for Radiograph Diagnosis and Reports Generation**. Yan and Pei. AAAI'22\
<https://aaai-2022.virtualchair.net/poster_aaai4013>

## Vision Related Work

**AN IMAGE IS WORTH 16X16 WORDS: TRANSFORMERS FOR IMAGE RECOGNITION AT SCALE**. Dosovitskiy et al. ICLR'21\
ViT: Vision Transformers (w/o CNN).\
Input: flattened fixed-size patches with position emb; each patch is projected as a token embedding.\
<https://arxiv.org/pdf/2010.11929>

More vision Transformers:
* <https://hub.baai.ac.cn/view/21957>
* Challenges: (1) 2D/3D spacial encoding

**Masked Autoencoders Are Scalable Vision Learners**. He et al. CVPR'22\
Pretraining ViT: mask patches and recover.\
<https://arxiv.org/pdf/2111.06377>

**BEIT: BERT Pre-Training of Image Transformers**. Bao et al. ICLR'22\
Use auto-denoising on vision pretraining: represent images as discrete tokens, and pretrain similar to MLM (labels are image tokens).\
Discrete tokens are obtained by another VAE that reconstructs original images given tokens.\
<http://arxiv.org/abs/2106.08254>

## Direct Interaction (Encoder-Based)

**Learning Transferable Visual Models From Natural Language Supervision**. Radford et al. ICML'21\
CLIP.\
Simple contrastive learning on image-text pairs.
New dataset: entity query + searched images.\
**Alignment**: linear projection on both image & text embedding.\
<https://proceedings.mlr.press/v139/radford21a.html>

**Scaling Up Visual and Vision-Language Representation Learning With Noisy Text Supervision**. Jia et al. ICML'21\
Simple contrastive learning on image-text pairs.\
New dataset: image + alt text (simple frequency-based filtering).\
<https://proceedings.mlr.press/v139/jia21b.html>

**UNIMO-2: End-to-End Unified Vision-Language Grounded Learning**. Li et al. ACL Findings'22\
Mechanism to leverage unpaired image and text corpus (large-scale), in addition to aligned image-text pairs (limited),\
by mapping image and text to a shared vocab that serves as anchors in unpaired image/text pretraining.\
**Alignment**: mutual interactions that only happen through the shared vocab in Transformers to make sure it learns alignment.\
Observations: (1) shared vocab is critical (2) unaligned pretraining is useful.\
Why straightforward interaction (w/o shared) + paired&unpaired training has lower performance? Because the mapping uses topK shared vocab that emphasizes most important semantics?
<https://aclanthology.org/2022.findings-acl.251/>

**Image as a Foreign Language: BEIT Pretraining for All Vision and Vision-Language Tasks**. Wang et al. 2022\
BEIT-3: Similar to BEIT that auto-denoises image tokens, with modification: (1) multiway-Transformers for general multi-modal encoding (2) add text as well (analogy to multilingual parallel corpus) (3) better image tokenization.\
One MLM pretraining objective.
<http://arxiv.org/abs/2106.08254>

## Indirect Interaction (Generation-Based)

**Unifying Vision-and-Language Tasks via Text Generation**. Cho et al. ICML'21\
Similar to T5, pack multiple multimodal tasks as sequence generation.\
Pretraining: several designed tasks to learn alignment.\
<https://arxiv.org/abs/2102.02779>

**SIMVLM: SIMPLE VISUAL LANGUAGE MODEL PRETRAINING WITH WEAK SUPERVISION**. Wang et al. ICLR'22\
Simple pretraining objective (see slides).\
<https://arxiv.org/pdf/2108.10904>

**Language Is Not All You Need: Aligning Perception with Language Models**. Huang et al. 2023\
MLLM: same as LLM but accepting multimodal input (can be interleaved); output is always languages.\
Add language-only instruction tuning, achieving zero-shot and cross-modal transfer.\
Input representation: flattened sequence.\
<http://arxiv.org/abs/2302.14045>
