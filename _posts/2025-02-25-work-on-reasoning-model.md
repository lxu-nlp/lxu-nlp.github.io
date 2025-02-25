---
layout: post
title: "Works on Reasoning Model"
date: 2025-02-25
categories: [NLP]
tags: [nlp]
math: true
---

RL spinup:
- <https://spinningup.openai.com/en/latest/spinningup/rl_intro.html>
- <https://rail.eecs.berkeley.edu/deeprlcourse-fa17/f17docs/lecture_4_policy_gradient.pdf>


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
