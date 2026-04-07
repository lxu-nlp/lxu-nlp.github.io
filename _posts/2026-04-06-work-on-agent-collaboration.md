---
layout: post
title: "Works on Agent Collaboration"
date: 2026-04-06
categories: [LLM]
tags: [nlp, LLM, agent]
math: true
---

## Benchmarks

**MultiAgentBench : Evaluating the Collaboration and Competition of LLM agents**. Zhu et al. ACL'25\
<https://aclanthology.org/2025.acl-long.421>


## Self-Reflection

**Reflexion: language agents with verbal reinforcement learning**. Shinn et al. NIPS'23\
<https://openreview.net/forum?id=vAElhFcKW6>

**Self-Refine: Iterative Refinement with Self-Feedback**. Madaan et al. NIPS'23\
<https://openreview.net/forum?id=S37hOerQLB>

**Large Language Models Cannot Self-Correct Reasoning Yet**. Huang et al. ICLR'24\
Self-correction without external info does not work, after examining three previous techniques:\
(1) reflexion (2) multi-agent debate (3) self-refine.\
LLM cannot judge the correctness of reasoning. For the same budget, simple majority voting is more effective than discussions.\
<https://openreview.net/forum?id=IkmD3fKBPQ>


## Collaboration and Debating

**Improving Factuality and Reasoning in Language Models through Multiagent Debate**. Du et al. ICML'24\
<https://openreview.net/forum?id=zj7YuTE4t8>

**ReConcile: Round-Table Conference Improves Reasoning via Consensus among Diverse LLMs**. Chen et al. ACL'24\
<https://aclanthology.org/2024.acl-long.381/>

**Rethinking the Bounds of LLM Reasoning: Are Multi-Agent Discussions the Key?**. Wang et al. ACL'24\
Self-discussion does not bring improvement; may help marginally when without demonstration.\
<https://aclanthology.org/2024.acl-long.331>

**Encouraging Divergent Thinking in Large Language Models through Multi-Agent Debate**. Liang et al. EMNLP'24\
Spur exploration by forcing different stances and roles explicitly.\
<https://aclanthology.org/2024.emnlp-main.992>

**Scaling Large Language Model-based Multi-Agent Collaboration**. Qian et al. ICLR'25\
Topologies: chain, tree, and graph.\
Collaborative scaling law: S-shape.\
<https://openreview.net/forum?id=K3n5jPkrU6>
