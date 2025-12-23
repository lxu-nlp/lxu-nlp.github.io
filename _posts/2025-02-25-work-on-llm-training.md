---
layout: post
title: "Works on LLM Training"
date: 2025-02-25
categories: [NLP]
tags: [nlp, LLM, training]
math: true
---


# General Training Dynamics

**Weak-to-Strong Generalization: Eliciting Strong Capabilities With Weak Supervision**. Burns et al. ICML'24\
strong models trained with weak supervision can often generalize to a substantially higher performance than the weak model itself.\
<https://openreview.net/forum?id=BEqFYCDU0u>

**Grokking: Generalization Beyond Overfitting on Small Algorithmic Datasets**. Power et al. 2022\
Generalize after overfitting.\
<https://arxiv.org/abs/2201.02177>

**Scaling Exponents Across Parameterizations and Optimizers**. Everett et al. ICML'24\
<https://openreview.net/forum?id=0ksNeD1SJT>


# Efficient Training: Low-Rank

**Parameter-Efficient Transfer Learning for NLP**. Houlsby et al. ICML'19\
Bottleneck lor-rank adapters.\
<https://proceedings.mlr.press/v97/houlsby19a.html>

**Prefix-Tuning: Optimizing Continuous Prompts for Generation**. Li and Liang. ACL'21\
Prefix-tuning: optimize a continuous prompt prefix per task, instead of discrete task prompt.\
Close to finetuning, and better than finetuning when data is small.\
<https://aclanthology.org/2021.acl-long.353>

**The Power of Scale for Parameter-Efficient Prompt Tuning**. Lester et al. EMNLP'21\
Prompt-tuning: similar to prefix-tuning, only optimizing a continuous prompt prefix.\
(1) Prompt length can be critical.\
(2) Initializing prompt by class-label vocab performs the best.\
(3) Prompt tuning becomes more competitive with scale (same performance as finetuning).\
<https://aclanthology.org/2021.emnlp-main.243>

**LORA: LOW-RANK ADAPTATION OF LARGE LANGUAGE MODELS**. Hu et al. ICLR'22\
Instead of inserting layers (adapters) as parts of parameters, learn a separate low-rank parameter delta.\
<https://arxiv.org/abs/2106.09685>

**Towards a Unified View of Parameter-Efficient Transfer Learning**. He et al. ICLR'22\
Connecting adapter, prefix-tuning and lora.\
<https://openreview.net/forum?id=0RDcd5Axok>

**DoRA: Weight-Decomposed Low-Rank Adaptation**. Liu et al. ICML'24\
(1) Analyzing the delta change of weights after finetuning, by decomposing into magnitude and direction.\
(2) Different delta dynamics between Lora and full full finetuning.\
(3) Perform update by separating magnitude and direction of Lora, to be more stable and mimic full finetuning dynamic.\
<https://openreview.net/forum?id=3d5CIRG1n2>

**LONGLORA: EFFICIENT FINE-TUNING OF LONG CONTEXT LARGE LANGUAGE MODELS**. Chen et al. 2023\
Same local window size for each attention head. But, half heads on original local regions, while the other half on shifted local regions with overlapping, to enable communication between locals.\
Thus, although receiving local k/v at each attention layer, global attention can still be achieved by stacked attention layers.\
<https://arxiv.org/pdf/2309.12307>

**ReFT: Representation Finetuning for Language Models**. Wu et al. NIPS'24\
<https://openreview.net/forum?id=fykjplMc0V>


# RL Training

RL spinup:
- <https://spinningup.openai.com/en/latest/spinningup/rl_intro.html>
- <https://rail.eecs.berkeley.edu/deeprlcourse-fa17/f17docs/lecture_4_policy_gradient.pdf>

Q&A
- Why using log trick in policy gradient? To make the trajectory probability computable by converting $\prod$ to $\sum$, so to achieve derivative of sum == sum of derivative; otherwise derivative of prod is not traceable.
- BTW: log trick can derive the general product rule

- GRPO vs. PPO? GRPO is REINFORCE variant, only have sequence end score; while PPO uses critic estimation to provide per-step score.
- PPO/GRPO vs. vanilla REINFORCE? PPO adopts relative prob estimation (rather than raw trajectory prob) to achieve more stable update (along with clipping).
- Reference model in PPO/GRPO? A fixed model, used as KL loss or reward penalty to control exploration.
- PPO/GRPO loss? Relative Prob * Trajectory Advantage + Entropy + KL
- Mini update step? Split a batch to mini batches to accelerate convergence (mini batches use the same sampled trajectories)

- What does RL essentially do for LLM? To steer alignment of desired behaviors upon the existing language model distribution.
- Why RL is more robust compared to SFT? RL receives negative signals during exploration, while SFT has no exploration with no negative signals; RL has tried more distribution space.


**There May Not be Aha Moment in R1-Zero-like Training — A Pilot Study**. Liu et al.\
Reflection exists in pretraining; RL (as alignment) strengthens effective reflection.\
Reasoning length is not an indicator of performance.\
<https://oatllm.notion.site/oat-zero>

**Let's Verify Step by Step**. Lightman et al. ICLR'24\
Train reward model to score steps (token-level scoring) for RL supervision.\
<https://openreview.net/forum?id=v8L0pN6EOi>

**Demystifying Long Chain-of-Thought Reasoning**. Yang et al. ICML'25\
(1) SFT is not strictly necessary but significantly simplifies efficiency\
(2) reasoning capabilities are not guaranteed, making reward shaping essential for stabilizing CoT length growth\
<https://openreview.net/forum?id=OLodUbcWjB>

**CoT-Valve: Length-Compressible Chain-of-Thought Tuning**. Ma et al. ACL'25\
Train lora to control length.\
<https://aclanthology.org/2025.acl-long.300>


# SFT Training: Add Negative Signals

Not just vanilla MLE on gold sequences.

**NEURAL TEXT DEGENERATION WITH UNLIKELIHOOD TRAINING**. Welleck et al. ICLR'20\
Determine negative tokens at each step and minimize likelihood.\
Heuristics: use previous context tokens as negatives to avoid repentance.\
<https://openreview.net/pdf?id=0qSOodKmJaN>

**BRIO: Bringing Order to Abstractive Summarization**. Liu et al. ACL'22\
Scoring seq to calibrate good/bad: ROUGE with gold.\
Margin loss on sampled generations, with margin scaled by scoring.\
<https://aclanthology.org/2022.acl-long.207>

**CALIBRATING SEQUENCE LIKELIHOOD IMPROVES CONDITIONAL LANGUAGE GENERATION**. Zhao et al. ICLR'23\
Similar to BRIO but more comprehensive experiments.\
Margin loss on sampled generations; relative ordering per scores performs better than continuous scores directly.\
<https://openreview.net/pdf?id=0qSOodKmJaN>

**Click: Controllable Text Generation with Sequence Likelihood Contrastive Learning**. Zheng et al. ACL Findings'23\
Margin loss on negative sequence likelihood; with method to sample negative without gold references.\
<https://aclanthology.org/2023.findings-acl.65>
