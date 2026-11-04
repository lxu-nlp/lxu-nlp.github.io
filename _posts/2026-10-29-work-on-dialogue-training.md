---
layout: post
title: "Works on Dialogue Training"
date: 2026-10-29
categories: [LLM]
tags: [LLM, dialogue]
math: true
---


## Reading Guide

A dialogue history contains all previous user and assistant messages, ending with the latest user message. The **current response** is the assistant reply being trained or evaluated; it need not be the last reply of a naturally completed conversation.

**Multi-turn data, current-response optimization.** Collect a conversation, then treat each selected user message and its preceding history as a separate prompt. Generate or compare only the assistant reply to that prompt. For example, when training the second reply, the first exchange is already fixed; the model is not asked to generate the user's third message.

**Interactive multi-turn training.** Generate an assistant reply, give that actual reply to a user simulator, and let the simulator produce the next user message. Continue the conversation and use later outcomes to evaluate earlier decisions. A fixed simulator means its model weights are not updated, not that its messages are pre-written.

**Two separate questions.** How much conversation is generated for scoring, and which assistant replies receive training loss? A method can simulate several future exchanges but update only one earlier reply. Likewise, a user simulator used to build offline SFT data is not necessarily used during RL.


## Training Recipes

### Classic Dialogue Alignment

**Training a Helpful and Harmless Assistant with Reinforcement Learning from Human Feedback**. Bai et al. 2022\
Data collection: a human writes a message, compares two model replies, chooses one to continue with, and writes a follow-up based on that reply. This produces genuine multi-turn histories and preference labels for individual replies.\
Reward modeling: the preference model (PM, i.e., RM) sees the entire history plus a candidate reply, but learns to judge that candidate rather than the outcome of future exchanges.\
PPO: take a recorded history ending in a user message, generate one new assistant reply, score it with the RM, and update the policy. There is no next user message in that training sample; the human feedback was collected beforehand.\
Iterated online RLHF refreshes human conversations and preference data using newer models, rather than connecting a live human to every PPO update.\
Read: Sec. 2, Sec. 4.5, Appendix A.2 and B.1.\
<https://arxiv.org/abs/2204.05862>

**Llama 2: Open Foundation and Fine-Tuned Chat Models**. Touvron et al. 2023\
Collect human comparisons between replies to the same prompt and train reward models. The prompt can include previous dialogue, so a reply is judged in context rather than as an isolated sentence.\
Rejection sampling: generate several candidate replies, use the RM to select the best, and train on that selected reply as an SFT target. PPO instead samples a reply from the current policy and uses the RM score as reward.\
Ghost Attention addresses a separate multi-turn problem: an instruction given at the start should still apply after later user messages. It builds synthetic dialogue examples and trains the final assistant reply while keeping earlier messages as context, with no loss on those earlier messages.\
Do not interpret these methods as evidence that PPO dynamically generates future user messages; the disclosed conditional-generation objective does not establish that loop.\
Read: Sec. 3.2-3.3.\
<https://arxiv.org/abs/2307.09288>

**The Llama 3 Herd of Models**. Dubey et al. 2024\
Humans hold multi-turn conversations with models and compare candidate replies at each turn. The conversations are then split into training examples: all messages up to a selected user question become the prompt, and the preferred/non-preferred replies become the DPO pair.\
For a three-exchange conversation, this can produce separate examples for the first, second and third assistant replies. Later examples retain earlier exchanges in their prompts; splitting does not remove the history.\
Run six rounds of SFT followed by DPO, collecting new data from improved models between rounds. The report does not disclose continuing each candidate reply with a dynamic user to measure its future consequences in ordinary chat.\
A reply that is fixed history in one example can be the training target in another. Reported results are for Llama 3.1.\
Read: Table 6, Sec. 4.1-4.2.\
<https://arxiv.org/abs/2407.21783>

**Constitutional AI: Harmlessness from AI Feedback**. Bai et al. 2022\
SFT data: ask the model to critique a reply against written behavioral principles, revise it, and use the revision as the training target. These critique/revision steps are not new user turns.\
RL data: generate two replies to the same dialogue prompt and ask another model which better follows the principles. Train an RM on these AI preference labels, then use it for RL; human helpfulness feedback is also retained.\
The AI evaluator replaces part of the preference labeling process, not the user who would respond to the assistant. The paper does not introduce a dynamic user loop into the inherited one-response RLHF pipeline.\
Read: Sec. 3-4.\
<https://arxiv.org/abs/2212.08073>

**Improving Alignment of Dialogue Agents via Targeted Human Judgements**. Glaese et al. 2022\
Sparrow collects human judgments of reply quality and whether the agent follows explicit conversational rules. Train preference and rule reward models, then optimize generated statements with A2C, an actor-critic policy-gradient method.\
Each training episode starts from an existing dialogue history and generates one statement. The model can play either the user or the assistant; it does not have to wait for a real human to continue the conversation.\
Valid generated statements are appended to the history and stored for later episodes. Thus, model-generated user and assistant messages can build a multi-turn conversation across updates, although one episode still contains only one newly generated statement.\
This differs from both permanently fixed logged histories and generating a whole dialogue before a single update. It does not establish credit assignment from later dialogue outcomes to earlier statements.\
Read: Sec. 2.3, 2.5 and 2.8; Fig. 7. Focus on dialogue training rather than search.\
<https://arxiv.org/abs/2209.14375>

**LaMDA: Language Models for Dialog Applications**. Thoppilan et al. 2022\
Humans hold multi-turn conversations and label replies for safety and quality: whether they make sense, are specific to the context, and are interesting. These labels train classifiers that assess candidate replies.\
Use the classifiers to filter dialogue data, then fine-tune a generator to predict a reply given its history, with loss on the reply rather than the history.\
At inference, generate candidates, discard unsafe ones, and rank the rest using the quality classifiers. SSI is the combination of sensibleness, specificity and interestingness; these scores are used for selection, not an RL update.\
A useful example of training a dialogue system with SFT and candidate selection rather than RLHF.\
Read: Sec. 5-6.1.\
<https://arxiv.org/abs/2201.08239>

### Recent Model Reports and Production Recipes

**CharacterFlywheel: Scaling Iterative Improvement of Engaging and Steerable LLMs in Production**. Nie et al. Meta, 2026\
Deploy a chat model, collect real-user conversations, obtain human preference labels and rewrites, train reward models, and improve the model with SFT, DPO and online RL. Repeat after deploying the improved model.\
During RL, take a partial conversation ending in a user message as a fixed prompt. GRPO generates and scores only new replies to that prompt; it does not ask the user to respond to each candidate and continue the dialogue.\
Near-policy prompts are histories collected with a recent model rather than a much older one. Refreshing them makes training contexts closer to those the updated model will encounter, but they remain fixed within an RL sample.\
The production-data loop is dynamic across model versions; the individual RL sample is still current-response optimization. The report also discusses biased user feedback and RM gains that fail to translate into better real-user engagement.\
Read: Sec. 2.4.3 and 3.5.\
<https://arxiv.org/abs/2603.01973>

**Nemotron 3 Ultra: Open, Efficient Mixture-of-Experts Hybrid Mamba-Transformer Model for Agentic Reasoning**. NVIDIA, 2026\
Chat SFT data: start with a user question, generate candidate replies, select a good reply, and use an LLM to act as the user and ask a follow-up based on the conversation. User prompts encourage behaviors such as requesting clarification or challenging an assumption.\
Train only the final assistant reply in each SFT example. Earlier replies can deliberately be lower-quality candidates, so the model learns to give a good answer even when its preceding conversation was imperfect.\
Separately, train a chat teacher with RLHF using a generative reward model (GenRM): a model that reasons about candidate reply quality and outputs scores. The report describes scoring replies in context, but does not say that this RLHF stage dynamically generates the next user message.\
The final Ultra model learns from the improved teacher through multi-teacher on-policy distillation (MOPD): the student generates text and learns from teacher-provided token-level guidance. This is distinct from the teacher's own RLHF training.\
Read: Sec. 3.1.1 (Chat) and 3.3.2 (Chat Teacher).\
<https://arxiv.org/abs/2606.15007>

**Introducing NVIDIA Nemotron 3.5 Bolt E4B 4GB**. NVIDIA, 2026-10-23. Blog\
An implementation entry point linking the model's post-training recipe and datasets, including RLHF rewards from a generative evaluator and data for following instructions across multiple exchanges.\
Multi-turn instruction-following data can test whether a reply respects requirements stated in earlier messages. Its name alone does not tell us whether those earlier messages are fixed or generated during RL.\
The blog does not establish a loop where the current assistant reply triggers a simulated user's next message. Useful for locating concrete recipes, not sufficient evidence of interactive conversation-level RLHF.\
Read: Training Methodologies & Open Recipes.\
<https://research.nvidia.com/labs/nemotron/NVIDIA-Nemotron-3.5-Bolt-E4B-4GB>

**How People Ask Claude for Personal Guidance**. Anthropic, 2026-04-30. Blog\
Analyze real conversations to find cases where users challenge an answer and the assistant becomes overly agreeable. Convert these interaction patterns into synthetic training scenarios.\
For each scenario, generate two candidate replies and have another Claude instance score how well they follow the behavioral principles in Claude's constitution. The reported goal is to improve responses under user pushback, not merely make the first reply polite.\
The blog describes behavior training for Opus 4.7 and Mythos Preview, but does not specify the optimizer or whether the simulated user continues after each candidate reply during training.\
Its stress test supplies an old problematic conversation as fixed history and checks whether the new model's next reply corrects course. That is evaluation, not proof of a dynamic-user RL rollout.\
Read: Improving Claude's behavior in relationship guidance.\
<https://www.anthropic.com/research/claude-personal-guidance>

### Interactive Multi-Turn RL

**Regressing the Relative Future: Efficient Policy Optimization for Multi-turn RLHF**. Gao et al. 2024\
REFUEL (Setting One) replaces live users with a fixed user LLM. After each actual assistant reply, that LLM reads the updated history and generates the user's next question; the assistant then answers it.\
At a randomly selected assistant turn, try two different replies from the same history. Continue both branches with the user simulator and the assistant policy, and score the resulting conversations with an RM.\
Use the difference in final scores to train the two replies at the branching turn. Rather than just labeling one reply as better, the regression loss fits the difference in how much the new policy favors the two replies, relative to the policy that generated them, to their observed future-reward gap.\
A single training sample directly updates only that branching reply, even though several future exchanges were generated to score it. Different samples choose different branching turns.\
Read: Algorithm 1 and Setting One. Setting Two instead uses questions chosen before the interaction, which cannot react to a changed assistant answer.\
<https://arxiv.org/abs/2410.04612>

**CollabLLM: From Passive Responders to Active Collaborators**. Wu et al. ICML'25\
For a candidate assistant reply, let an LLM-simulated user respond and continue the conversation. Repeat this continuation several times, then average the scores of the resulting interactions to estimate the candidate's value.\
The default scoring window contains at most two future user-assistant exchanges. For document co-creation, scores combine similarity to a target document, the amount of text the user must read/write, and a judge's assessment of interaction quality.\
Use these future-aware scores to select preferred replies for DPO or provide rewards for online PPO. Offline data generation uses a teacher assistant; online training uses the assistant being trained to generate its continuations.\
Intuition: a clarifying question can beat an immediate full draft if the user's answer leads to a better document with less rework. Even a current-reply DPO pair can therefore encode future consequences rather than only immediate reply quality.\
Read: Sec. 3 and Appendix B/D. Focus on document co-creation rather than code/math rewards.\
<https://arxiv.org/abs/2502.00640>

**CollabLLM: Teaching LLMs to Collaborate with Users**. Microsoft Research, 2025. Blog\
A companion explanation of the paper above: try candidate replies, simulate how the user and assistant continue, and reward replies that lead to more effective collaboration.\
Useful for the motivation: asking a question now can save the user from correcting a long, irrelevant answer later. Also summarizes the document co-creation study with real users; this study evaluates the trained model rather than supplying users for its RL updates.\
<https://www.microsoft.com/en-us/research/blog/collabllm-teaching-llms-to-collaborate-with-users/>

**Enhancing Personalized Multi-Turn Dialogue with Curiosity Reward**. Wan et al. 2025\
CURIO's educational setting gives a simulated student a learning preference that the assistant does not initially know. The student responds dynamically to the assistant, allowing it to learn the preference through conversation.\
A separate predictor reads the history and estimates the student's preference. Intermediate rewards encourage exchanges that improve this estimate; an RM also scores conversation quality at the end.\
Train the assistant with multi-turn policy gradients using these rewards, so an early question can receive credit for what it helps reveal later. This is not just SFT on dialogues that already contain the correct preference.\
The user simulator generates the next student message; the predictor estimates hidden traits for reward calculation. They perform different jobs, and better prediction is only a proxy for useful personalization.\
Read: Education Dialogue, Sec. 3 and Appendix A.\
<https://arxiv.org/abs/2504.03206>


## Dialogue Data Construction

**Enhancing Chat Language Models by Scaling High-quality Instructional Conversations**. Ding et al. EMNLP'23\
UltraChat builds synthetic conversations without collecting human queries: create diverse starting requests, then alternate generated user follow-ups and assistant answers to form 1.5M multi-turn dialogues.\
Fine-tune UltraLM on the completed dialogues. The assistant learns how to answer a later user message given the preceding exchanges, but those exchanges are already recorded when SFT begins.\
The user-generation process constructs offline training data; it is not a user environment that continues responding to the changing assistant policy during RL.\
<https://aclanthology.org/2023.emnlp-main.183/>

**PlatoLM: Teaching LLMs in Multi-Round Dialogue via a User Simulator**. Kong et al. ACL'24\
First train Socratic, a user simulator, to predict real humans' questions from dialogue history. This teaches user behavior directly instead of relying only on an assistant model prompted to pretend to be a user.\
Let Socratic and a teacher assistant alternate messages: Socratic asks a question or follow-up based on their conversation, and the teacher answers. Save the generated dialogues and use them to SFT the final assistant, PlatoLM.\
Dynamic interaction happens while constructing the dataset. The final assistant is then trained on fixed conversations, not by interacting with Socratic during RL.\
<https://aclanthology.org/2024.acl-long.424/>


## Evaluation and Insights

**LLMs Get Lost in Multi-Turn Conversation**. Laban et al. 2025\
Compare giving all task requirements in one message with having a simulated user reveal them across several exchanges. The simulator reads the current dialogue and chooses which remaining information to provide next, under constraints on what it can reveal.\
Models often commit to an answer before all requirements are known, then keep modifying that earlier answer rather than reconsidering the task. Thus, an early mistake can become part of the history that shapes later replies.\
The findings motivate training for clarification and recovery from imperfect histories. They do not prove that splitting RLHF data by turn causes the failures, or that interactive RL will automatically fix them.\
Read: Sec. 3 and 6-7. Evaluation and failure analysis, not a training recipe.\
<https://arxiv.org/abs/2505.06120>

**Flipping the Dialogue: Training and Evaluating User Language Models**. Naous et al. 2025\
UserLM reverses the usual SFT target: instead of predicting the assistant answer, predict the next real user message from a high-level user goal and the preceding conversation. Training data comes from WildChat's human-assistant conversations.\
The resulting model can ask follow-ups after a new assistant reply, reveal its goal gradually, and end a conversation. A goal describes what the user wants, not a script specifying every message.\
The paper finds that being a stronger assistant does not necessarily make a model a more realistic simulated user: helpful, exhaustive assistant behavior can be unlike how people actually ask questions.\
It trains and evaluates the user model, not an assistant RL policy. Such a model could provide the missing next-user-message generator for interactive training, but that downstream RL loop is not demonstrated here.\
<https://arxiv.org/abs/2510.06552>
