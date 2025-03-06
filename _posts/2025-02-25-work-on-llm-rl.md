---
layout: post
title: "Works on LLM RL"
date: 2025-02-25
categories: [NLP]
tags: [nlp]
math: true
---

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

**Open-Reasoner-Zero: An Open Source Approach to Scaling Up Reinforcement Learning on the Base Model**. Hu et al.\
PPO is robust and effective.\
Minimal reward is not only sufficient but optimal.\
<https://github.com/Open-Reasoner-Zero/Open-Reasoner-Zero>

**Logic-RL: Unleashing LLM Reasoning with Rule-Based Reinforcement Learning**. Xie et al.\
Reward: format + answer\
REINFORCE++ >= PPO > GRPO.\
Thinking tokens can be performance indicator, but not reasoning length.\
Reflection is gradual, not sudden, likely just a result of alignment upon pretraining.\
RL generalizes better than SFT.\
<https://arxiv.org/abs/2502.14768>
