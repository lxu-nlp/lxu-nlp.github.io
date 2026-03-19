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
To estimate (state, action), instead of parameterized estimation, do KNN lookup: maintain (action, state, val) list.\
For an action, get Q estimation weighted by current state distance, such that only state is parameterized but Q estimation is unparameterized.\
Advantages: quick convergence; but eventually underperform parameterized Q estimation.\
<https://proceedings.mlr.press/v70/pritzel17a.html>
