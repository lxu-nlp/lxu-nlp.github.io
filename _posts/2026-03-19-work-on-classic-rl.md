---
layout: post
title: "Works on Classic Reinforcement Learning"
date: 2026-03-19
categories: [RL]
tags: [RL, training]
math: true
---

Also see LLM-related RL in separate [post](https://lxu-nlp.github.io/posts/work-on-llm-training/).

**Neural Episodic Control**. Pritzel et al. ICML'17\
To estimate (state, action), instead of parameterized estimation, do KNN lookup: maintain (action, state, Q) list.\
Get Q estimation on a state for an action: average Q weighted by state distance, such that Q estimation is unparameterized.\
Rely on assumption: if two states are close, then the action policy is close.\
Advantages: quick convergence; but eventually underperform parameterized Q estimation.\
<https://proceedings.mlr.press/v70/pritzel17a.html>
