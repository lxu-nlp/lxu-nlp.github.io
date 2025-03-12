---
layout: post
title: "Works on LLM Basics"
date: 2023-04-25
categories: [NLP]
tags: [nlp, LLM]
math: true
---


## LLM Basics

**Language Models are Few-Shot Learners**. Brown et al. NIPS'20\
GPT-3: introduce few-shot prompting paradigm.\
<https://papers.nips.cc/paper/2020/hash/1457c0d6bfcb4967418bfb8ac142f64a-Abstract.html>

**FINETUNED LANGUAGE MODELS ARE ZERO-SHOT LEARNERS**. Wei et al. ICLR'22\
Introduce instruction tuning paradigm (finetuning task prompts of various tasks); zero-shot capability of unseen tasks.\
Flan: NLP tasks verbalized via natural language instruction templates.\
<https://arxiv.org/pdf/2109.01652>

**Scaling Instruction-Finetuned Language Models**. Chung et al. 2022\
Upscale 1.8k tasks to Flan (instruction tuning with either zero-shot or few-shot instructed prompts).\
Adding chain-of-thought (CoT) task prompts in instruction tuning.\
<http://arxiv.org/abs/2210.11416>

**MULTITASK PROMPTED TRAINING ENABLES ZERO-SHOT TASK GENERALIZATION**. Sanh et al. ICLR'22\
P3: similar to Flan, prompts of various tasks.\
<http://arxiv.org/abs/2110.08207>

**The Flan Collection: Designing Data and Methods for Effective Instruction Tuning**. Longpre et al. 2023\
Specifically investigate data and methods for instruction tuning; evaluate on held-in tasks, held-out tasks, CoT tasks.\
(1) finetuning with mixed zero-shot and few-shot prompts improves both held-in and held-out\
(2) adding more tasks is beneficial to held-out tasks\
(3) input-inversion is helpful as task instruction augmentation\
(4) certain instruction datasets are more effective\
<http://arxiv.org/abs/2301.13688>

**PaLM: Scaling Language Modeling with Pathways**. Chowdhery et al. 2022\
PaLM: similar to GPT3 (decoder-only), few-shot performance steeply increases as scaled.\
<http://arxiv.org/abs/2204.02311>

**GLM-130B: AN OPEN BILINGUAL PRE-TRAINED MODEL**. ICLR'23\
Scale GLM to 130B.\
<http://arxiv.org/abs/2210.02414>

**Large Language Models Encode Clinical Knowledge**. Singhal et al. 2022\
Based on Flan-PaLM + continuous few-shot prompting for medical domain.\
<http://arxiv.org/abs/2212.13138>

**Training Compute-Optimal Large Language Models**. Hoffmann et al. 2022\
Investigate the best parameter size and training tokens given fixed FLOPs.\
Approach: use loss to determine the best model, by fixing/varying size, tokens, FLOPs, etc.\
Chinchilla: 70B parameters, 1.4T tokens (based on the optimal prediction)\
<http://arxiv.org/abs/2203.15556>

**LLaMA: Open and Efficient Foundation Language Models**. Touvron et al. 2023\
Similar to PaLM/Chinchilla: train with more data on smaller models.\
<http://arxiv.org/abs/2302.13971>

**Mixture-of-Experts Meets Instruction Tuning: A Winning Combination for Large Language Models**. Shen et al. 2023\
<https://arxiv.org/abs/2305.14705>

**Mixtral of Experts**. Jiang et al. 2024\
Sparse gating: constant computation cost while scaling up parameters.\
<http://arxiv.org/abs/2401.04088>

## Instruction Alignment

**Training language models to follow instructions with human feedback**. Ouyang et al. 2022\
InstructGPT: alignment with human intent.\
Supervised finetuning on golds -> output on unlabeled data -> human feedback w/ trained reward model -> RL w/ reward model.\
<http://arxiv.org/abs/2203.02155>

**Improving alignment of dialogue agents via targeted human judgements**. Glaese et al. 2022\
Sparrow: supervised finetuning + train rewards based on designed rules = RL on larger unlabeled inputs (self-play/learning)\
Train to decide when to retrieve evidence and how good the evidence is.\
Reward models can be used for re-ranking during inference.\
<http://arxiv.org/abs/2209.14375>

**Constitutional AI: Harmlessness from AI Feedback**. Bai et al. 2022\
Goal: align with certain principles without human efforts.\
(1) Supervised bootstrapping: (prompt, response) -GPT-> (prompt, revision), by giving principles and self-revise. Then finetune a model on the revision for bootstrapping.\
(2) RL: use another independent LM to provide the score for two response comparison based on given principles. Score: get the log-likelihood of response ID.\
<http://arxiv.org/abs/2212.08073>

**Self-Instruct: Aligning Language Model with Self Generated Instructions**. Wang et al. ACL'23\
Generated instructions from LLM (GPT3) by iterative bootstrapping: seed task instructions -> new task instructions -> input/output instructions.\
<http://arxiv.org/abs/2212.10560>

**INSTRUCTION TUNING WITH GPT-4**. Peng et al. 2023\
Generated instructions from LLM (GPT4), with rating from GPT4 as well for reward training.\
<http://arxiv.org/abs/2304.03277>

**LIMA: Less Is More for Alignment**. Zhou et al. 2023\
Hypothesis: user-intent alignment is a simple process to merely learn the format or style to expose acquired knowledge in pretraining.\
(1) Finetuning LLaMA on 1000 single-response high-quality examples: good quality response; good OOD performance.\
(2) Multi-turn dialogue: ok zero-shot performance within 3 turns; adding only 30 dialogue chains in training gives significant gains.\
<https://arxiv.org/abs/2305.11206>

**The RefinedWeb Dataset for Falcon LLM: Outperforming Curated Corpora with Web Data, and Web Data Only**. Penedo et al. 2023\
RefinedWeb high-quality web corpus: rather than human curation, use filtering heuristics can obtain better performance.\
Ablations: filtering heuristics and deduplication contribute a lot; better URL filtering.\
<http://arxiv.org/abs/2306.01116>

**Platypus: Quick, Cheap, and Powerful Refinement of LLMs**. Lee et al. 2023\
High-quality and small (25k) instruction set on multiple specialized subjects.\
<https://arxiv.org/abs/2308.06259>

**CHINESE OPEN INSTRUCTION GENERALIST: A PRELIMINARY RELEASE**. Zhang et al. 2023\
Diverse instruction-tuning tasks in general, human-value alignment, code, exams.\
<https://arxiv.org/pdf/2304.07987>

**Self-Alignment with Instruction Backtranslation**. Li et al. 2023\
Instruction generation by semi-supervised + back-verification: (instruction, doc) seed pairs -> (doc, pseudo-instruction) by backward model -> curation by forward model.\
Observations: quality > quantity; more quantity only helps when quality is good.\
<http://arxiv.org/abs/2308.06259>

## Position Interpolation

**RoFormer: Enhanced Transformer with Rotary Position Embedding**. Su et al. 2021\
RoPE formulation: as a kernel, when q/k can be encoded by each position separately, and their multiplication represents the relative difference.\ 
Regard hidden as d/2 partitions where each partition is 2-dim, then rotate each partition by \theta based on position.\
Essential kernel: Re(p1 - p2) = f(q, p1) * f(q, p2)\
<http://arxiv.org/abs/2104.09864>

**EXTENDING CONTEXT WINDOW OF LARGE LANGUAGE MODELS VIA POSITION INTERPOLATION**. Chen et al. 2023\
Position Interpolation (PI): linearly reduce rotation angle in RoPE (does not have to be integer) proportional to length; with bound proof.\
Observation: RoPE attention score delays as length increases, could exploding when relative position > max length.\
<https://arxiv.org/abs/2305.10601>

**Effective Long-Context Scaling of Foundation Models**. Xiong et al. 2023\
(1) continuous pretraining with longer max len limit; reduce rotation angle similar to PI, but fixed.\
(2) SFT with constructed long context examples: build completion from LLM based on short chunk, but use full context for SFT.\
For long example SFT, also enable loss on the long context itself, not only completion.\
<http://arxiv.org/abs/2309.16039>

**YaRN: Efficient Context Window Extension of Large Language Models**. Peng et al. 2023\
<https://arxiv.org/abs/2309.00071>

**Beyond the Limits: A Survey of Techniques to Extend the Context Length in Large Language Models**. Wang et al. 2024\
<https://arxiv.org/abs/2402.02244>

## Reasoning

**Chain-of-Thought Prompting Elicits Reasoning in Large Language Models**. Wei et al. NIPS'22\
Introduce few-shot CoT in various reasoning tasks; teaching the problem solving (rough algorithm) rather than through few-shot guessing.\
<http://arxiv.org/abs/2201.11903>

**Large Language Models are Zero-Shot Reasoners**. Kojima et al. NIPS'22\
Zero-shot CoT.\
<https://openreview.net/pdf?id=e2TBb5y0yFf>

**Tree of Thoughts: Deliberate Problem Solving with Large Language Models**. Yao et al. 2023\
<https://arxiv.org/abs/2305.10601>

**Limitations of Language Models in Arithmetic and Symbolic Induction**. Qian et al. 2022\
LLM cannot perfectly generalize three simple reasoning tasks of any inputs: copying, reversing, and addition; especially when input has repeating characters.\
Introduce prompting that encodes the exact algorithm (strict action sequence) to teach specific reasoning generalization.\
<http://arxiv.org/abs/2208.05051>

**SELF-CONSISTENCY IMPROVES CHAIN OF THOUGHT REASONING IN LANGUAGE MODELS**. Wang et al. ICLR'23\
Give CoT prompt, generate multiple paths and aggregate the final answers (e.g. majority vote).\
<https://openreview.net/forum?id=1PL1NIMMrw>

**A Multitask, Multilingual, Multimodal Evaluation of ChatGPT on Reasoning, Hallucination, and Interactivity**. Bang et al. 2023\
A systematic evaluation ChatGPT on various types of tasks, including reasoning, multimodality, factuality/hallucination.\
<http://arxiv.org/abs/2302.04023>

**Decomposed Prompting: A Modular Approach for Solving Complex Tasks**. Khot et a. 2022\
In-context few-shot CoT to generate reasoning steps (algorithms), where each step is handled by a specific LLM/program for this sub-problem.\
<http://arxiv.org/abs/2210.02406>

**Successive Prompting for Decomposing Complex Questions**. Dua et al. EMNLP'22\
Task decomposition: generate intermediate states as QA pairs sequentially, each step conditioned on past QA pairs; the model decides when to stop and give the final answer.\
Benefits: modularity of sub-problems, able to involve other solvers.\
Decide in-context decomposition examples for the current problem: searched from index.\
<https://aclanthology.org/2022.emnlp-main.81>

**Least-to-Most Prompting Enables Complex Reasoning in Large Language Models**. Zhou et al. 2022\
In-context few-shot CoT for compositional tasks. Similar to above.\
<http://arxiv.org/abs/2205.10625>

**SOLVING CHALLENGING MATH WORD PROBLEMS USING GPT-4 CODE INTERPRETER WITH CODE-BASED SELF-VERIFICATION**. Zhou et al. 2023\
Leverage Code-Interpreter to verify results real-time and self-revise.\
<http://arxiv.org/abs/2308.07921>

**LARGE LANGUAGE MODELS ARE HUMAN-LEVEL PROMPT ENGINEERS**. Zhou et al. ICLR'23\
(1) Use LLM to generate prompt candidates based on input-output (2) use LLM likelihood to score candidates (3) use LLM to generate more variants for top candidates.\
<https://openreview.net/pdf?id=92gvk82DE->

**Prompting with Pseudo-Code Instructions**. Mishra et al. EMNLP'23\
Function-style with doc-string comments as instructions, supporting few-shot.\
<https://aclanthology.org/2023.emnlp-main.939>

---

## Analysis

**Emergent Abilities of Large Language Models**. Wei et al. TMLR'22\
Quantify specific tasks and prompting techniques regarding emergent capability on different models.\
<https://arxiv.org/pdf/2206.07682>

**DISSOCIATING LANGUAGE AND THOUGHT IN LARGE LANGUAGE MODELS: A COGNITIVE PERSPECTIVE**. Mahowald et al. 2022\
Decomposition: 1) formal linguistic competence (linguistic rules and statistical regularities); 2) functional linguistic competence (formal cognitive reasoning).\
<http://arxiv.org/abs/2301.06627>

**From Word Models to World Models: Translating from Natural Language to the Probabilistic Language of Thought**.\
LLM to convert natural languages to program languages + use program languages as probabilistic inference for reasoning.\
<http://arxiv.org/abs/2202.12837>

**Rethinking the Role of Demonstrations: What Makes In-Context Learning Work?**. Yao et al. EMNLP'22\
(1) Important: input distribution, label space, format; (2) does not rely on the ground truth input-label mapping (little gap with random labels).\
<http://arxiv.org/abs/2202.12837>

**Do Large Language Models Know What They Don’t Know?**. Yin et al. 2023\
Dataset: answerable and unanswerable questions.\
<http://arxiv.org/abs/2305.18153>

**Lost in the Middle: How Language Models Use Long Contexts**. Liu et al. TACL'23\
<https://arxiv.org/abs/2307.03172>

**Case-Based or Rule-Based: How Do Transformers Do the Math?**. Hu et al. ICML'24\
Design math dataset that controls case similarity, then train & evaluate.\
Conclusion: LLMs rely on similar math cases.\
<https://openreview.net/forum?id=4Vqr8SRfyX>

**Scaling Exponents Across Parameterizations and Optimizers**. Everett et al. ICML'24\
<https://openreview.net/forum?id=0ksNeD1SJT>

**Why are Sensitive Functions Hard for Transformers?**. Hahn and Rofin. ACL'24\
<http://arxiv.org/abs/2402.09963>

## Analysis: Bias

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

## Analysis: Self-Reasoning

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

## Analysis: Hallucination

Also see post about Factuality.

**Halo: Estimation and Reduction of Hallucinations in Open-Source Weak Large Language Models**. Elaraby et al. 2023\
Use SUMMAC directly; new dataset constructed by GPT, with approach to reduce hallucination.\
<http://arxiv.org/abs/2308.11764>

**SELF-CONTRADICTORY HALLUCINATIONS OF LLMS: EVALUATION, DETECTION AND MITIGATION**. Mündler et al. 2023\
Induce LLM to generate two statements regarding the same context, then use another LLM to detect inconsistency.\
<http://arxiv.org/abs/2305.15852>

**CHAIN-OF-VERIFICATION REDUCES HALLUCINATION IN LARGE LANGUAGE MODELS**. Dhuliawala et al. 2023\
QA-based verification for detecting non-factual query response.\
<https://arxiv.org/pdf/2309.11495.pdf>

**DOLA: DECODING BY CONTRASTING LAYERS IMPROVES FACTUALITY IN LARGE LANGUAGE MODELS** Chuang et al. 2023\
<https://arxiv.org/pdf/2309.03883.pdf>

## Analysis: Attack

**Universal and Transferable Adversarial Attacks on Aligned Language Models**. Zou et al. 2023\
(1) For attacking, the desired output starts with its corresponding affirmative response, e.g. Sure, here is how to ...\
(2) Search a fixed suffix of the user prompt, such that the loss (negative log likelihood) of the affirmative response is low\
(3) How to search: greedy search; selecting top tokens that achieve largest negative gradient at each suffix position INDEPENDENTLY (as if Gibbs sampling); then sample from these candidate tokens and select the sequence with lowest target loss (as if Monte-Carlo).\
(4) Repeat this search process on multiple prompts incrementally, to find the final universal suffix that works for all prompts\
<http://arxiv.org/abs/2307.15043>

---

## Applications

**Constrained Language Models Yield Few-Shot Semantic Parsers**. Shin et al. EMNLP'21\
(1) semantic parsing as paraphrasing natural sentences into a sublanguage/canonical expression that is close to natural languages (2) sublanguage-constrained decoding. Same few-shot performance but much less data.\
<https://aclanthology.org/2021.emnlp-main.608>

**CINS: Comprehensive Instruction for Few-shot Learning in Task-oriented Dialog Systems**. Mi et al. AAAI'22\
Intent Classification, Dialogue State Tracking, NLG.\
<https://arxiv.org/pdf/2109.04645>

**Generating Datasets with Pretrained Language Models**. Schick and Schütze. EMNLP'21\
How to generate labeled classification dataset.\
<https://aclanthology.org/2021.emnlp-main.555>

**Co-Writing Screenplays and Theatre Scripts with Language Models An Evaluation by Industry Professionals**. MIROWSKI et al. 2022\
Few-shot prompting for writing long coherent text: designed hierarchical dialogue generation by each scene\
Example + logline = titles; Example + logline = characters description; Example + logline + characters = brief plots with scenes; Example + characters + place + plot = detailed dialogue\
Coherence: protected by the shared short description in prompting\
<http://arxiv.org/abs/2209.14958>

**DOC: Improving Long Story Coherence With Detailed Outline Control**. Yang et al. 2022\
Hierarchical generation.\
<http://arxiv.org/abs/2212.10077>

