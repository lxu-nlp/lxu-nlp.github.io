---
layout: post
title: "Works on LLM Training"
date: 2025-02-25
categories: [LLM]
tags: [nlp, LLM, training, RL]
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

**For Better or for Worse, Transformers Seek Patterns for Memorization**. Panwar et al. NIPS'25\
LLMs memorize and generalize on easier patterns first.\
<https://openreview.net/pdf?id=98NrkXPRZ9>


# Lora

**Parameter-Efficient Transfer Learning for NLP**. Houlsby et al. ICML'19\
Bottleneck lor-rank adapters.\
<https://proceedings.mlr.press/v97/houlsby19a.html>

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

General RL:
- Why using log trick in policy gradient? To make the trajectory probability computable by converting $\prod$ to $\sum$, so to achieve derivative of sum == sum of derivative; otherwise derivative of prod is not traceable.
- BTW: log trick can derive the general product rule
- REINFORCE (policy gradient) is strictly on-policy; we can introduce mini-batch update to improve sample efficiency, where PPO uses relative prob and clipping to combat the off-policy drift.
  - Relative prob: even though this data is slightly old, here is how likely the current model is to take that action compared to the old one
  - Clipping: if the model drifts too far, the clipping kills the gradient
- RL is essentially steering the desired alignment
- RL does exploration; for trajectory reward, both pos+neg provide contrastive signals for robust generalization

Details:
- Original PPO: use critic estimation to provide per-step score
- Reference model: use as KL loss or reward penalty to control drifting
- PPO/GRPO loss: Relative Prob * Trajectory Advantage + Entropy + KL


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

### RL Training Ingredients

**DAPO: An Open-Source LLM Reinforcement Learning System at Scale**. Yu et al. NIPS'25\
<https://openreview.net/forum?id=2a36EMSSTp>

**Part I: Tricks or Traps? A Deep Dive into RL for LLM Reasoning**. Liu et al. 2025\
<https://arxiv.org/abs/2508.08221>

**A Comedy of Estimators: On KL Regularization in RL Training of LLMs**. Shah et al. 2025\
KL: use K1 in Reward.\
<https://arxiv.org/abs/2512.21852>

[**JustRL**](https://iclr-blogposts.github.io/2026/blog/2026/justrl/): many tricks are not necessary


# Training: Add Negative Signals

Not just on gold sequences.

### in SFT

Usually scale loss or do contrastive.

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

### in RL

Natural pos+neg optimization.

**The Surprising Effectiveness of Negative Reinforcement in LLM Reasoning**. Zhu et al. NIPS'25\
When neg reward: only penalize incorrect rollout, redistribute probs from sampled trajectories to unsampled, while reserving overall prior.\
Unlike pos reward, which keeps pushing probs of sampled trajectories (as in SFT), leading to losing entropy/generalization/exploration.\
Essentially, neg reward learns slower in solution space by telling which not do, but keeps healthy entropy/generalization; pos reward learns faster by remembering what to do, but prone to losing other potential solutions through exploration.\
Note that GRPO naturally avoids this problem through rollout reward normalization.\
<https://arxiv.org/abs/2506.01347>
