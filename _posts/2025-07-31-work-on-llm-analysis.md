---
layout: post
title: "Works on LLM Analysis"
date: 2025-07-31
categories: [NLP]
tags: [nlp, LLM]
math: true
---


## Learnability, Generalization, Composition

**Overcoming a Theoretical Limitation of Self-Attention**. Chiang and Cholak. ACL'22\
Parity can be learned by Transformers with simple modification.\
<https://aclanthology.org/2022.acl-long.527>

**Exploring Length Generalization in Large Language Models**. Anil et al. NIPS'22\
Parity learning is not generalized by SFT, but scratchpad helps.\ 
<https://openreview.net/forum?id=zSkYVeX7bC4>

**Simplicity Bias in Transformers and their Ability to Learn Sparse Boolean Functions**. Bhattamishra et al. ACL'23\
Experiments on model architectures vs. sensitivity (k-sparse parity).\
<https://aclanthology.org/2023.acl-long.317>

**Why are Sensitive Functions Hard for Transformers?**. Hahn and Rofin. ACL'24\
<https://aclanthology.org/2024.acl-long.800>

**Sub-Task Decomposition Enables Learning in Sequence to Sequence Tasks**. Wies et al. ICLR'23\
Theoretical: adding intermediate decomposition makes unlearnable problem learnable.\
<https://openreview.net/forum?id=BrJATVZDWEH>

**Faith and Fate: Limits of Transformers on Compositionality**. Dziri et al. NIPS'23\
Controlled complexity: explicit computational graph: multiplication, puzzle, DP.\
Cannot generalize multi-hop solving procedure.\
LLMs solve compositional tasks by reducing multi-step compositional reasoning into linearized subgraph matching.\
<https://openreview.net/forum?id=Fkckkr3ya8>

**The Coverage Principle: A Framework for Understanding Compositional Generalization**. Chang et al. 2025\
Compositional generalization comes from: functional equivalence.\
Task: different compute graph.\
<https://arxiv.org/abs/2505.20278>

**How Far Can Transformers Reason? The Globality Barrier and Inductive Scratchpad**. Abbe et al. NIPS'24\
Formal on task difficulty: global degree.\
Can only learn constant degree task by regular training.\
Scratchpad (decomposition by low global degree) can help.\
<https://openreview.net/forum?id=FoGwiFXzuN>

**Grokking of Implicit Reasoning in Transformers: A Mechanistic Journey to the Edge of Generalization**. Wang et al. NIPS'24\
Examine implicit reasoning on composition task (two-hop tail entity prediction), with aspects on: grokking, causal tracing, etc.\
<https://openreview.net/forum?id=D4QgSWxiOb>

**The Parallelism Tradeoff: Limitations of Log-Precision Transformers**. Merrill and Sabharwal. TACL'23\
Transformers can only express O(1)-parallel complexity.\
<https://aclanthology.org/2023.tacl-1.31/>

**Chain of Thought Empowers Transformers to Solve Inherently Serial Problems**. Li et al. ICLR'24\
CoT allows serial (multi-hop) computation. Without CoT, it is then bounded by transformer depth.\
<https://openreview.net/forum?id=3EWTEy9MTM>

**Auto-Regressive Next-Token Predictors are Universal Learners**. Malach. ICML'24\
CoT can approximate any function efficiently computed by a Turing machine.\
Length complexity: the number of intermediate tokens in a CoT sequence required to approximate some target function.\
<https://dl.acm.org/doi/10.5555/3692070.3693470>

**Limits of Deep Learning: Sequence Modeling through the Lens of Complexity Theory**. Zubic et al. ICLR'25\
Cannot efficiently perform function composition (multi-hop); even with CoT, requiring large intermediate steps.\
<https://openreview.net/forum?id=DhdqML3FdM>

**Generalizing Reasoning Problems to Longer Lengths**. Xiao and Liu. ICLR'25\
Theoretical: length generalization only with CoT.\
(n-r) consistency enables generalization.\
<https://openreview.net/forum?id=zpENPcQSj1>


## CoT / Thinking

**Why think step by step? Reasoning emerges from the locality of experience**. Prystawski et al. NIPS'23\
Key training data hop property: intermediate steps are only helpful when the training data is locally structured with respect to dependencies between variables.\
<https://openreview.net/forum?id=rcXXNFVlEn>

**Case-Based or Rule-Based: How Do Transformers Do the Math?**. Hu et al. ICML'24\
Design math dataset that controls case similarity, then train & evaluate.\
Observations: LLMs rely on similar math cases.\
<https://openreview.net/forum?id=4Vqr8SRfyX>

**Does Reinforcement Learning Really Incentivize Reasoning Capacity in LLMs Beyond the Base Model?**. Yue et al. NIPS'25\
RL training enables new patterns or just alignment of base model?\
Observation: the latter.\
<https://openreview.net/forum?id=4OsgYD7em5>

### CoT Robustness

**GSM-Symbolic: Understanding the Limitations of Mathematical Reasoning in Large Language Models**. Mirzadeh et al. ICLR'25\
GSM-stype math problem through generation with controlled complexity.\
CoT is not robust to problem variation.\
<https://openreview.net/forum?id=AjXkRZIvjB>

**The Illusion of Thinking: Understanding the Strengths and Limitations of Reasoning Models via the Lens of Problem Complexity**. Shojaee et al. NIPS'25\
CoT is not always reliable, such that it could underperform on simple cases and fail on harder cases.\
<https://arxiv.org/abs/2506.06941>

### CoT Saliency Identification / CoT Efficiency

**Beyond the 80/20 Rule: High-Entropy Minority Tokens Drive Effective Reinforcement Learning for LLM Reasoning**. Wang et al. NIPS'25\
Expected: 20\% tokens are high entropy, which usually decide reasoning path and are critical to the final performance.\
<https://openreview.net/forum?id=yfcpdY4gMP>

**Compressing Chain-of-Thought in LLMs via Step Entropy**. Li et al. 2025\
80\% of low-entropy intermediate steps can be pruned.\
<https://arxiv.org/abs/2508.03346>

**Understanding Chain-of-Thought in LLMs through Information Theory**. Ton et al. ICML'25\
Quantify the information gain at each reasoning step.\
<https://openreview.net/forum?id=IjOWms0hrf>

**Forking Paths in Neural Text Generation**. Bigelow et al. ICLR'25\
Identify critical tokens that significantly impact subsequent generation.\
<https://openreview.net/forum?id=8RCmNLeeXx>

**Enhancing Chain-of-Thought Reasoning with Critical Representation Fine-tuning**. Huang et al. ACL'25\
Use information flow to identify critical positions to finetune.\
<https://aclanthology.org/2025.acl-long.1129>

**Do LLMs Encode Functional Importance of Reasoning Tokens?**. Singh and Hakkani. 2025\
Token-level sequential pruning that preserves reasoning likelihood.\
<https://arxiv.org/abs/2601.03066>

**Thought Anchors: Which LLM Reasoning Steps Matter?**. Bogdan et al. 2025\
Identify salient sentences by sampling without each sentence.\
<https://arxiv.org/abs/2506.19143>

### CoT Necessity

**Do NOT Think That Much for 2+3=? On the Overthinking of Long Reasoning Models**. Chen et al. ICML'25\
Quantify overthinking on math.\
<https://openreview.net/forum?id=MSbU3L7V00>

**Mind Your Step (by Step): Chain-of-Thought can Reduce Performance on Tasks where Thinking Makes Humans Worse**. Liu et al. ICML'25\
When CoT brings negative.\
<https://openreview.net/forum?id=J3gzdbYZxS>

**To CoT or not to CoT? Chain-of-thought helps mainly on math and symbolic reasoning**. Sprague et al. ICLR'25\
Cot mostly benefits symbolic (multi-hop) tasks, but still underperforms actual symbolic solvers.\
<https://openreview.net/forum?id=w6nlcS8Kkn>

**When Reasoning Meets Its Laws**. Zhang et al. 2025\
Quantify task difficulty + new benchmark.\
Finetune to control CoT length in proportional to task difficulty.\
<https://arxiv.org/abs/2512.17901>

### CoT Planning / Internal Planning

**Future Lens: Anticipating Subsequent Tokens from a Single Hidden State**. Pal et al. CoNLL'23\
Hidden state transferred to another prompt can yield the same results, thus current hidden state contains subsequent info.\
<https://aclanthology.org/2023.conll-1.37/>

**Patchscopes: A Unifying Framework for Inspecting Hidden Representations of Language Models**. Ghandeharioun et al. ICML'24\
Similar to Future Lens: move hidden state to another prompt to derive the meaning of hidden state.\
<https://openreview.net/forum?id=5uwBzcn885>

**The Internal State of an LLM Knows When It’s Lying**. Azaria and Mitchell. EMNLP Finding'23\
Probing on the last input hidden state by binary label (truthfulness).\
Prober gives more accuracy estimation compared to rollout probability.\
<https://aclanthology.org/2023.findings-emnlp.68>

**Enhancing Language Model Factuality via Activation-Based Confidence Calibration and Guided Decoding**. Liu et al. EMNLP'24\
Train truthfulness classifier on hidden states.\
<https://aclanthology.org/2024.emnlp-main.583>

**Estimating Knowledge in Large Language Models Without Generating a Single Token**. Gottesman and Geva. EMNLP'24\
Similar to above.\
<https://aclanthology.org/2024.emnlp-main.232>

**Knowing Before Saying: LLM Representations Encode Information About Chain-of-Thought Success Before Completion**. Afzal et al. ACL Finding'25\
Probing on the last input hidden state by binary label (correctness).\
<https://aclanthology.org/2025.findings-acl.662>

**Emergent Response Planning in LLMs**. Dong et al. ICML'25\
Probe on goal-oriented task that requires sequential actions.\
<https://openreview.net/forum?id=Ce79P8ULPY>

**On Reasoning Strength Planning in Large Reasoning Models**. Sheng et al. NIPS'25\
Obtain mean vector of different difficulties, which could steer reasoning length.\
<https://arxiv.org/abs/2506.08390>


## Probing Methods

**Eliciting Latent Predictions from Transformers with the Tuned Lens**. Belrose et al. 2023\
Middle-layer hidden states -> last-layer hidden states, which can then be decoded to vocab.\
<https://arxiv.org/abs/2303.08112>

**Dissecting Recall of Factual Associations in Auto-Regressive Language Models**. Geva et al. EMNLP'23\
Attention blocking.\
<https://aclanthology.org/2023.emnlp-main.751>

**Unlocking the Future: Exploring Look-Ahead Planning Mechanistic Interpretability in Large Language Models**. Men et al. EMNLP'24\
Probe and attention blocking.\
<https://aclanthology.org/2024.emnlp-main.440>

**Internal Chain-of-Thought: Empirical Evidence for Layer-wise Subtask Scheduling in LLMs**. Yang et al. EMNLP'25\
Two-hop across layers. (but did not investigate max hops)\
<https://aclanthology.org/2025.emnlp-main.1147>

**Deep Hidden Cognition Facilitates Reliable Chain-of-Thought Reasoning**. Chen et al. AAAI'26\
Train confidence predictor on hidden state of each CoT step.\
<https://arxiv.org/abs/2507.10007>


## Inference with Planning

**Successive Prompting for Decomposing Complex Questions**. Dua et al. EMNLP'22\
Task decomposition: generate intermediate states as QA pairs sequentially, each step conditioned on past QA pairs; the model decides when to stop and give the final answer.\
Benefits: modularity of sub-problems, able to involve other solvers.\
Decide in-context decomposition examples for the current problem: searched from index.\
<https://aclanthology.org/2022.emnlp-main.81>

**Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning by Large Language Models**. Wang et al. ACL'23\
<https://aclanthology.org/2023.acl-long.147/>

**Decomposed Prompting: A Modular Approach for Solving Complex Tasks**. Khot et a. ICLR'23\
In-context few-shot CoT to generate reasoning steps (algorithms), where each step is handled by a specific LLM/program for this sub-problem.\
<https://openreview.net/forum?id=_nGgzQjzaRy>

**Planning in Natural Language Improves LLM Search for Code Generation**. Wang et al. ICLR'25\
<https://openreview.net/forum?id=48WAZhwHHw>


## In-Context Learning (ICL)

**Rethinking the Role of Demonstrations: What Makes In-Context Learning Work?**. Yao et al. EMNLP'22\
(1) Important: input distribution, label space, format; (2) does not rely on the ground truth input-label mapping (little gap with random labels).\
<https://arxiv.org/abs/2202.12837>

**Data Distributional Properties Drive Emergent In-Context Learning in Transformers**. Chan et al. NIPS'22\
ICL emerges when the training data exhibits particular distributional properties such as burstiness (items appear in clusters rather than being uniformly distributed over time).\
<https://openreview.net/forum?id=lHj-q9BSRjF>

**Larger language models do in-context learning differently**. Wei et al. 2023\
Intrinsic knowledge vs. in-context label mapping: label mapping supersedes intrinsic knowledge, and can scale by model size.\
<https://arxiv.org/abs/2303.03846>

**Why Larger Language Models Do In-context Learning Differently?**. Shi et al. ICML'24\
Theoretical: larger model learns more features but is more prone to noises.\
Good related work.\
<https://openreview.net/forum?id=WOa96EG26M>

**Label Words are Anchors: An Information Flow Perspective for Understanding In-Context Learning**. Wang et al. EMNLP'24\
Analysis by attention-based saliency: labels are anchors in ICL.\
<https://aclanthology.org/2023.emnlp-main.609>

**Why In-Context Learning Models are Good Few-Shot Learners?**. Wu et al. ICLR'25\
Theoretical: ICL functions as a new learning algorithm.\
<https://openreview.net/forum?id=iLUcsecZJp>


## Context Utilization

**Lost in the Middle: How Language Models Use Long Contexts**. Liu et al. TACL'23\
<https://aclanthology.org/2024.tacl-1.9/>


## Bias

**Language Models Don’t Always Say What They Think: Unfaithful Explanations in Chain-of-Thought Prompting**. Turpin et al. NIPS'23\
Positional bias and social bias.\
<https://proceedings.neurips.cc/paper_files/paper/2023/hash/ed3fea9033a80fea1376299fa7863f4a-Abstract-Conference.html>

**Large Language Models are not Fair Evaluators**. Wang et al. ACL'24\
Determine when model is uncertain and human should step in.\
Uncertain calibration mainly by swapping orders.\
<https://aclanthology.org/2024.acl-long.511/>

**Large Language Models Sensitivity to The Order of Options in Multiple-Choice Questions**. Pezeshkpour and Hruschka. NAACL Findings'23\
Conjecture order sensitivity caused by: uncertainty + positional bias. Identify patterns that amplify/mitigate bias.\
<https://aclanthology.org/2024.findings-naacl.130>

**Large Language Models Are Not Robust Multiple Choice Selectors**. Zheng et al. ICLR'24\
PriDe:\
(1) bias comes from token bias, rather than position bias.\
(2) compute prior token bias on a small set of samples (via permuting options), which is later used for debiasing directly.\
<https://openreview.net/forum?id=shr9PXz7T0>

**“My Answer is C”: First-Token Probabilities Do Not Match Text Answers in Instruction-Tuned Language Models**. Wang et al. ACL Findings'24\
More capable model -> better instruction understanding -> better alignment (less mismatch) between single-token prob and text response.\
<https://aclanthology.org/2024.findings-acl.441>

**Look at the Text: Instruction-Tuned Language Models are More Robust Multiple Choice Selectors than You Think**. Wang et al. 2024\
For both accuracy & selection bias & robustness by perturbation: text answer > debiased first token prob > direct first token prob.\
<https://arxiv.org/abs/2404.08382>

**A Peek into Token Bias: Large Language Models Are Not Yet Genuine Reasoners**. Jiang et al. EMNLP'24\
Quantify token bias (e.g. widely seen entities) through perturbation.\
<https://aclanthology.org/2024.emnlp-main.272>


## Self-Reasoning

**Reflexion: language agents with verbal reinforcement learning**. Shinn et al. NIPS'23\
<https://openreview.net/forum?id=vAElhFcKW6>

**Self-Refine: Iterative Refinement with Self-Feedback**. Madaan et al. NIPS'23\
<https://openreview.net/forum?id=S37hOerQLB>

**Improving Factuality and Reasoning in Language Models through Multiagent Debate**. Du et al. ICML'24\
<https://openreview.net/forum?id=zj7YuTE4t8>

**ReConcile: Round-Table Conference Improves Reasoning via Consensus among Diverse LLMs**. Chen et al. ACL'24\
<https://aclanthology.org/2024.acl-long.381/>

**Large Language Models Cannot Self-Correct Reasoning Yet**. Huang et al. ICLR'24\
Self-correction without external info does not work, after examining three previous techniques:\
(1) reflexion (2) multi-agent debate (3) self-refine.\
LLM cannot judge the correctness of reasoning. For the same budget, simple majority voting is more effective than discussions.\
<https://openreview.net/forum?id=IkmD3fKBPQ>

**Rethinking the Bounds of LLM Reasoning: Are Multi-Agent Discussions the Key?**. Wang et al. ACL'24\
Self-discussion does not bring improvement; may help marginally when without demonstration.\
<https://aclanthology.org/2024.acl-long.331>


## Attack

**Universal and Transferable Adversarial Attacks on Aligned Language Models**. Zou et al. 2023\
(1) For attacking, the desired output starts with its corresponding affirmative response, e.g. Sure, here is how to ...\
(2) Search a fixed suffix of the user prompt, such that the loss (negative log likelihood) of the affirmative response is low\
(3) How to search: greedy search; selecting top tokens that achieve largest negative gradient at each suffix position INDEPENDENTLY (as if Gibbs sampling); then sample from these candidate tokens and select the sequence with lowest target loss (as if Monte-Carlo).\
(4) Repeat this search process on multiple prompts incrementally, to find the final universal suffix that works for all prompts\
<https://arxiv.org/abs/2307.15043>


## Data Contamination

**Benchmarking Benchmark Leakage in Large Language Models**. Xu et al. 2024\
Metrics: compare perplexity and n-gram between existing corpus and synthesis corpus.\
<https://arxiv.org/abs/2404.18824>


## Conceptual Grounding / World Representation

**Mapping Language Models to Grounded Conceptual Spaces**. Patel and  Pavlick. ICLR'22\
LLM can not only learn to ground the concepts that it is explicitly taught, but appears to generalise to several instances of unseen concepts as well.\
<https://openreview.net/forum?id=gJcEM8sxHK>

**Emergent World Representations: Exploring a Sequence Model Trained on a Synthetic Task**. Li et al. ICLR'23\
Othello GPT: probe board state.\
<https://openreview.net/forum?id=DeG07_TcZvT>

**From Tokens to Thoughts: How LLMs and Humans Trade Compression for Meaning**. Shani et al. 2025\
<https://arxiv.org/abs/2505.17117>


## Others

**From Word Models to World Models: Translating from Natural Language to the Probabilistic Language of Thought**.\
LLM to convert natural languages to program languages + use program languages as probabilistic inference for reasoning.\
<https://arxiv.org/abs/2202.12837>
