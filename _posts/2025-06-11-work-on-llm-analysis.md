---
layout: post
title: "Works on LLM Analysis"
date: 2025-07-31
categories: [NLP]
tags: [nlp, LLM]
math: true
---


## Learnability and Expressibility

**Why are Sensitive Functions Hard for Transformers?**. Hahn and Rofin. ACL'24\
<https://aclanthology.org/2024.acl-long.800>


## Generalization

**Is Chain-of-Thought Reasoning of LLMs a Mirage? A Data Distribution Lens**. Zhao et al. 2025\
<https://arxiv.org/abs/2508.01191>


## Context Utilization

**Lost in the Middle: How Language Models Use Long Contexts**. Liu et al. TACL'23\
<https://aclanthology.org/2024.tacl-1.9/>


## In-Context Learning

**Rethinking the Role of Demonstrations: What Makes In-Context Learning Work?**. Yao et al. EMNLP'22\
(1) Important: input distribution, label space, format; (2) does not rely on the ground truth input-label mapping (little gap with random labels).\
<https://arxiv.org/abs/2202.12837>

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


## Bias

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


## Hallucination

Also see post about Factuality.

**Do Large Language Models Know What They Don’t Know?**. Yin et al. 2023\
Dataset: answerable and unanswerable questions.\
<https://arxiv.org/abs/2305.18153>

**Halo: Estimation and Reduction of Hallucinations in Open-Source Weak Large Language Models**. Elaraby et al. 2023\
Use SUMMAC directly; new dataset constructed by GPT, with approach to reduce hallucination.\
<https://arxiv.org/abs/2308.11764>

**SELF-CONTRADICTORY HALLUCINATIONS OF LLMS: EVALUATION, DETECTION AND MITIGATION**. Mündler et al. 2023\
Induce LLM to generate two statements regarding the same context, then use another LLM to detect inconsistency.\
<https://arxiv.org/abs/2305.15852>

**CHAIN-OF-VERIFICATION REDUCES HALLUCINATION IN LARGE LANGUAGE MODELS**. Dhuliawala et al. 2023\
QA-based verification for detecting non-factual query response.\
<https://arxiv.org/pdf/2309.11495.pdf>

**DOLA: DECODING BY CONTRASTING LAYERS IMPROVES FACTUALITY IN LARGE LANGUAGE MODELS** Chuang et al. 2023\
<https://arxiv.org/pdf/2309.03883.pdf>


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


## Probing

**Eliciting Latent Predictions from Transformers with the Tuned Lens**. Belrose et al. 2023\
Middle-layer hidden states -> last-layer hidden states, which can then be decoded to vocab.\
<https://arxiv.org/abs/2303.08112>

**Future Lens: Anticipating Subsequent Tokens from a Single Hidden State**. Pal et al. CoNLL'23\
Hidden state transferred to another prompt can yield the same results, thus current hidden state contains subsequent info.\
<https://aclanthology.org/2023.conll-1.37/>

**Dissecting Recall of Factual Associations in Auto-Regressive Language Models**. Geva et al. EMNLP'23\
Attention blocking.\
<https://aclanthology.org/2023.emnlp-main.751>

**Unlocking the Future: Exploring Look-Ahead Planning Mechanistic Interpretability in Large Language Models**. Men et al. EMNLP'24\
Probe and attention blocking.\
<https://aclanthology.org/2024.emnlp-main.440>

**Emergent Response Planning in LLMs**. Dong et al. ICML'25\
Probe on goal-oriented task that requires sequential actions.\
<https://openreview.net/forum?id=Ce79P8ULPY>

**Internal Chain-of-Thought: Empirical Evidence for Layer-wise Subtask Scheduling in LLMs**. Yang et al. 2025\
<https://arxiv.org/abs/2505.14530>

**On Reasoning Strength Planning in Large Reasoning Models**. Sheng et al. 2025\
Obtain mean vector of different difficulties, which could steer reasoning length.\
<https://arxiv.org/abs/2506.08390>

**Enhancing Chain-of-Thought Reasoning with Critical Representation Fine-tuning**. Huang et al. ACL'25\
Use information flow to identify critical positions to finetune.\
<https://aclanthology.org/2025.acl-long.1129>


## Others

**Emergent Abilities of Large Language Models**. Wei et al. TMLR'22\
Quantify specific tasks and prompting techniques regarding emergent capability on different models.\
<https://arxiv.org/pdf/2206.07682>

**From Word Models to World Models: Translating from Natural Language to the Probabilistic Language of Thought**.\
LLM to convert natural languages to program languages + use program languages as probabilistic inference for reasoning.\
<https://arxiv.org/abs/2202.12837>
