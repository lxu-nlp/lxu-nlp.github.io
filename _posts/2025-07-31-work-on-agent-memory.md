---
layout: post
title: "Works on Agents and Memory"
date: 2025-07-31
categories: [LLM]
tags: [nlp, LLM, agent, memory]
math: true
---


## Evaluation

**Evaluating Very Long-Term Conversational Memory of LLM Agents**. Maharana et al. ACL'24\
LoCoMo: dialogue generation by LLM.\
Task type: single/multi-hop, temporal, commonsense, unanswerable, etc.\
<https://aclanthology.org/2024.acl-long.747>

**LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory**. Wu et al. ICLR'25\
(question, answer, evidence) -> embed evidence to session -> embed session to irrelevant sessions.\
<https://openreview.net/forum?id=pZiyCaVuti>

**Evaluating Memory in LLM Agents via Incremental Multi-Turn Interactions**. Hu et al. ICLR'26\
Task types: retrieval, learning (similar to ICL), QA and sum, etc.\
<https://openreview.net/forum?id=DT7JyQC3MR>


## Memory Organization

### Explicit Organization (w/o Training)

**MemoryBank: Enhancing Large Language Models with Long-Term Memory**. Zhong et al. AAAI'24\
Simple mem update with decay.\
<https://arxiv.org/abs/2305.10250>

**Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory**. Chhikara et al. 2025\
Simple mem update with graph database.\
<https://arxiv.org/pdf/2504.19413>

**A-Mem: Agentic Memory for LLM Agents**. Xu et al. NIPS'25\
Memory notes.\
Link across notes: select top-k notes to generate relations.\
<https://openreview.net/forum?id=FiM0M8gcct>

**G-Memory: Tracing Hierarchical Memory for Multi-Agent Systems**. Zhang et al. NIPS'25\
<https://openreview.net/forum?id=mmIAp3cVS0>

**CAM: A Constructivist View of Agentic Memory for LLM-Based Reading Comprehension**. Li et al. NIPS'25\
<https://openreview.net/forum?id=ACSOnSHiWe>

**SimpleMem: Efficient Lifelong Memory for LLM Agents**. Liu et al. 2026\
<https://arxiv.org/abs/2601.02553>

### Explicit Organization (w/ Training)

**The Pensieve Paradigm: Stateful Language Models Mastering Their Own Context**. Liu et al. ICLR'26\
StateLM: learn tool-use regarding context management (build index, take notes), along with learning context pruning.\
<https://openreview.net/forum?id=GymjF88oGQ>

**Memory-R1: Enhancing Large Language Model Agents to Manage and Utilize Memories via Reinforcement Learning**. Yan et al. 2025\
Similar to StateLM.\
<https://arxiv.org/abs/2508.19828>

**MemAgent: Reshaping Long-Context LLM with Multi-Conv RL-based Memory Agent**. Yu et al. ICLR'26\
Read long context by maintaining fixed-length memory (RL-trained).\
<https://openreview.net/forum?id=k5nIOvYGCL>

### Latent Organization (w/ Training)

**M+: Extending MemoryLLM with Scalable Long-Term Memory**. Wang et al. ICML'25\
Good related work; bad writing.\
Separate long and short-term memory models to hold memory tokens; load into context during inference.\
<https://openreview.net/forum?id=OcqbkROe8J>

**MemGen: Weaving Generative Latent Memory for Self-Evolving Agents**. Zhang et al. ICLR'26\
Interleave latent hidden states as experiential memory: learned trigger to determine where to insert latent memory.\
<https://openreview.net/forum?id=vI56m4Iu4e>

**Agentic Memory: Learning Unified Long-Term and Short-Term Memory Management for Large Language Model Agents**. Yu et al. 2026\
<https://arxiv.org/abs/2601.01885>

**Dynamic Long Context Reasoning over Compressed Memory via End-to-End Reinforcement Learning**. Chen et al. 2026\
Similar to MemAgent (maintain textual working memory), but using compressed latent long term memory (interleaved gists).\
Long term memory gist can be selected (by a binary classifier) by current state (query and working memory), and utilized in generation.\
<https://arxiv.org/abs/2602.08382>

## Self-Evolving with Experiential Memory

**ExpeL: LLM Agents Are Experiential Learners**. Zhao et al. AAAI'24\
Abstraction (insights) on pos and neg trajectories: 1) reflect pos and neg for the same task; 2) reflect pos trajectories across tasks.\
Inference: add full task insights to context.\
No parameterization.\
<https://arxiv.org/abs/2308.10144>

**Synapse: Trajectory-as-Exemplar Prompting with Memory for Computer Control**. Zheng et al. ICLR'24\
Abstraction on pos and neg trajectories; retrieve relevant abstract trajectories in-context.\
No parameterization.\
<https://openreview.net/forum?id=Pc8AU1aF5e>

**ReasoningBank: Scaling Agent Self-Evolving with Reasoning Memory**. Ouyang et al. 2025\
Similar to Synapse. Abstraction: consolidate memory notes on trajectories through parallel or sequential scaling.\
<https://arxiv.org/abs/2509.25140>

**MemEvolve: Meta-Evolution of Agent Memory Systems**. Zhang et al. 2025\
<https://arxiv.org/abs/2512.18746>

**EvolveR: Self-Evolving LLM Agents through an Experience-Driven Lifecycle**. Wu et al. 2025\
Offline: generate trajectories and form insights/principles.\
Inference: retrieve principles similar to query.\
Parameterization on procedures to utilize principles, via RL training.\
<https://arxiv.org/abs/2510.16079>

**MemRL: Self-Evolving Agents via Runtime Reinforcement Learning on Episodic Memory**. Zhang et al. 2026\
Experience is the estimation target (like action is the target in classic RL).\
Rely on assumption: if two intents (queries) are close, then the experience utility is close.\
Design: pair experience with intent. Maintain: [intent -> (experience, Q) list]; just like [state -> (action, Q) list]\
Rank/Retrieval policy: 1) use query-intent similarity as initial filtering; 2) learn additional Q estimation.\
No parameterization: update Q by general TD (or regression, as if last step TD) (EM paradigm).\
<https://arxiv.org/abs/2601.03192>

**SkillRL: Evolving Agents via Recursive Skill-Augmented Reinforcement Learning**. Xia et al. 2026.\
Similar to EvolveR: skill consolidation, retrieval and utilization.\
Parameterization on procedures to utilize skills.\
<https://arxiv.org/abs/2602.08234>

**Group-Evolving Agents: Open-Ended Self-Improvement via Experience Sharing**. Weng et al. 2026\
i) select diverse agents (dissimilar success among tasks); ii) share experience.\
Essentially: aggregate diverse traces.\
<https://arxiv.org/abs/2602.04837>


## Other Agent Training

**Search-R1: Training LLMs to Reason and Leverage Search Engines with Reinforcement Learning**. Jin et al. 2025\
<https://arxiv.org/abs/2503.09516>

**DeepRetrieval: Hacking Real Search Engines and Retrievers with Large Language Models via Reinforcement Learning**. Jiang et al. 2025\
<https://arxiv.org/abs/2503.00223>

**DeepResearcher: Scaling Deep Research via Reinforcement Learning in Real-world Environments**. Zheng et al. 2025\
RL agent with web search, with a keep-appending short-term memory repository.\
<https://arxiv.org/abs/2504.03160>

**Search Self-Play: Pushing the Frontier of Agent Capability without Supervision**. Lu et al. ICLR'26\
<https://openreview.net/forum?id=ZmGirmNJqE>

**R-Zero: Self-Evolving Reasoning LLM from Zero Data**. Huang et al. ICLR'26\
Similar to GAN: 1) one LLM is trained to generate questions uncertain to the other one; 2) the other is trained to learn these questions.\
Two LLMs are separately updated.\
<https://openreview.net/forum?id=96apU6YzSO>
