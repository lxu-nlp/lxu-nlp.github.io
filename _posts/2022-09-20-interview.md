---
layout: post
title: "Interview"
date: 2022-09-20
categories: [Basics]
tags: [machine learning]
math: true
---

## Basics: Data Structure / Algorithm

**Tree**: connected and $|E| = |V| - 1$, or equivalently, there always exists one unique path between two nodes.
* MST: greedy algorithm (Kruskal) using disjoint set (implementation).

**Search**: see gitrepo.
* Shortest path (with negative, all-pair).

**Recursion**: master theorem.

**Randomized**:
* Partition: select kth largest / median (linear time).

**DP**: see gitrepo.

**Sort**: different sort, bubble.

---

## Basics: Calculus, Optimization, Linear Algebra

See post "basics-recap".

---

## Basics: Probability, Bayes

See post "basics-bayes".

---

## Basics: Uncertainty

See post "basics-uncertainty".

---

## Causality

[多篇顶会看个体因果推断（ITE）的前世今生](https://mp.weixin.qq.com/s/68ywogVbYmRP591ertzcTw)

## Basics: Machine Learning

Bias-Variance decomposition of MSE: E(error) = bias^2 + variance + irreducible noise

Also see law-of-total-variance in uncertainty post.

How to avoid overfit? (1) reduce model complexity, add regularization (or add parameter priori from Bayesian pov) (2) data augmentation ...

Inductive bias: a learning algorithm to prioritize one solution (or interpretation) over another, independent of the observed data. E.g. priori, regularization, architecture.

Bootstrap: sampling WITH replacement (~63.2%); can provide the estimated distribution.

Feature selection:
* correlation with labels, mutual information with labels, ...
* L1 reg, random forest feature importance

PCA: linear transformation on data that captures max variance (each principal component is orthogonal)
* [Tutorial](http://www.cs.otago.ac.nz/cosc453/student_tutorials/principal_components.pdf)

Metrics:
* AUC: TPR vs. FPR (how well the model can distinguish/separate two classes); AUC = 任意正样本的score大于负样本的score的概率(the probability that a uniformly drawn random positive has a higher score than a uniformly drawn random negative)
* AUPR: better for class imbalance (AUC being too optimistic; see [AUC](https://towardsdatascience.com/imbalanced-data-stop-using-roc-auc-and-use-auprc-instead-46af4910a494))

[Optimizer](https://www.cnblogs.com/guoyaohua/p/8542554.html):
* Momentum: add previous direction to reduce oscillation
* Adagrad/RMSprop: normalize by previous norm per parameter, to achieve adaptive LR per parameter (less frequent has higher weight, good for sparse)
* Adam: combine Adagrad & Momentum

LayerNorm & BatchNorm
* BatchNorm: normalize on each feature across batch; normalization is sample-dependent, prone to outliners for small batch size; harder for distributed models
* LayerNorm: normalize on single-input features; each normalization is independent

[Weight decay](https://benihime91.github.io/blog/machinelearning/deeplearning/python3.x/tensorflow2.x/2020/10/08/adamW.html):
* Instead of modify loss (e.g. L2 reg), modify update step: w = w - learning_rate * gradients - learning_rate * lamdba * w
* Weight decay != L2 Reg; only equivalent under SGD, and become different when adding momentum etc.

KL divergence: can be seen as $KL(q \Vert p) = H(p, q) - H(q)$

Mutual information: $KL(P(X, Y) \Vert P(X)P(Y))$

[EM](https://en.wikipedia.org/wiki/Expectation%E2%80%93maximization_algorithm#Description): MLE on parameters $\theta$ with latent variables $Z$ (e.g. mixture weights in GMM); $p(X|\theta) = \int p(X, Z | \theta) dZ$
1. Expectation (of latent variables): estimate latent variables based on current parameters.
2. Maximization (on parameters): update parameters by MLE based on currently estimated latent variables.

Variational Inference: estimate posterior distribution indirectly - use another (easier) distribution to approximate it.

HMM:
* Forward algorithm: marginal probability at a step (marginalize over hidden states at each step)
* Viterbi algorithm: joint probability over a sequence (DP that searches over previous sequence; taking single sequence w/o marginalization (max vs. sum))

Linear-chain CRF: similar to HMM that maximizes the **sequence probability** (each step not independent), where transition probability (priori) and emission probability (likelihood) are both estimated by model directly.
* See my implementation; operate on log-space
* Each forward step: first obtain priori (by applying transition), then normalize by likelihood (emission)

Naive Bayes:
* No parameters; calculate statistics of priori and likelihood
* Likelihood of multi-feature representation (e.g. words): assuming feature independence (limitation)
* Discrete likelihood: counting & smoothing; Continuous likelihood: variational

Pattern matching/searching (search patterns in string):
* Rabin-Karp: compare hashing between pattern and substring
* Prefix-suffix preprocessing: KMP (single pattern, 1D version of AC), Aho-Corasick (multi-pattern using Trie tree)
* Boyer–Moore

Decision trees: for training, choose a split on a variable that achieves better label purity, until stopping criteria.
* For continuous variable, choose certain candidate splits based on training set statistics

* Boosting: each learner is sequentially conditioned (increase model complexity); see blog
  * Gradient boosting: each learner focuses on previous's residual only (e.g. xgboost)
  * Forward stagewise boosting: each learner focuses on global optimization directly (e.g. adaboost)
* Bagging: each learner is INDEPENDENT (reduce variance); see blog

Tokenization:
* BPE: iteratively merge frequent consecutive tokens; a greedy way for compression
* Dictionary tokenization: based on dictionary, for each token: for start (to right) for end (to left) try
* [中文分词](https://zhuanlan.zhihu.com/p/50444885); 正向/反向最大匹配分词算法

Transformers:
* MLM why random 10% unchanged? Make sure the final hidden state to utilize itself as well (no train/test gap), in addition to context; nevertheless, the real objective is to obtain high-quality representation, not only the MLM classification accuracy
* MLM why random 10% replacement (not MASK)? Make sure to always use context, even without MASK (close train/test gap when w/ mask or w/o mask)
* Self-attention why scaled by sqrt(d)? Each raw attention element is from d*d, therefore the softmax scale will be large (variance proportional to d); need to normalize

RL:
* [Policy gradient](http://rail.eecs.berkeley.edu/deeprlcourse-fa17/f17docs/lecture_4_policy_gradient.pdf)

DL Framework (PyTorch 2.0): accelerate on eager model: to reduce I/O and achieve high FLOPS utilization
* Use refined atomic operators that pack multiple ops at once without transferring between devices back-and-forth
* Use graph model that captures blocks of subgraphs: code -> \[TorchDynamo, backend by TorchInductor that compiles using OpenAI Triton or OpenMP\] -> Hybrid FX graph that runs on GPU or CPU directly (maintain eager-mode capabilities)

---

## Research

Remaining challenging points for e.g. COREF

Characteristics of IE on certain domains e.g. dialogues, medical\
Challenges on keyphrase extraction: granularity/ambiguity (subjective)

Thoughts:
* Giant LM: impressive pseudo-reasoning by pattern matching, but not the ultimate solution of AGI
* Promising: instructive S2S for task-oriented (Flan-T5), open-domain / zero-shot (ChatGPT)
* Challenge: giant LM, how to better derive and utilize knowledge?
* Challenge: controllability & credibility; solution: structure/context-grounded inference?
* Challenge: lifelong learning
* Metric learning centroid vs. classification boundary

Things to try (engineering, LM, etc.):
* SwiGLU activations

## Industry

### Healthcare 医疗

[真实世界医疗知识图谱及临床事件图谱构建](https://mp.weixin.qq.com/s/mV039Jp9rUmO9Xy31-5zLg)
* Two knowledge bases: (1) 患者 (2) 疾病知识
* 应用：智能搜索，因果分析挖掘，患者病例自动描述，智能问答诊断
* 挑战：数据，可解释性(图谱概率推理)，DL overfit

标准化：手术，诊断(hardest)，药品。挑战：diverse expression，细粒度问题

[阿里医疗NLP实践与思考](https://mp.weixin.qq.com/s/FjhV0LgO-YNkAIx3575vvA): 医疗实体抽取，etc.

### Finance 金融

[图机器学习在蚂蚁集团安全风控场景的应用](https://mp.weixin.qq.com/s/UVQ8w6MEz1_OhW1RukKo8w)

### 智能客服

[美团智能客服技术实践](https://mp.weixin.qq.com/s/Dp4rlR3Znwv_2gbYCJu8cg)
* 2C: Taskbot (task flow/schema defined, to match user utterances with flows)
* 2C: QABot: KBQA / MRC
* 2C: Chatbot: retrieval / generation
* 2B: response suggestion, summarization

[美团知识图谱问答技术实践与探索](https://mp.weixin.qq.com/s/kF41iqbkU8Wy8bEKs2qMCA)\
[如何构造小爱智能问答的关键技术](https://mp.weixin.qq.com/s/Ys6aFq9Wl_asT2ycHZUp5w): multi-hop KBQA, retrieval-based QA\
[知识图谱在NLU与推荐中，是如何发挥作用的](https://mp.weixin.qq.com/s/2sUfI3SoWuVhPkMiz3TEgQ)\

### Search 搜索

[电商搜索中的Query理解与处理](https://mp.weixin.qq.com/s/8Zk-wxZO_vhJXPj0UhDxGQ)

### Weak Supervision

[Airbnb Embedding的实践和思考](https://mp.weixin.qq.com/s/Xx4KK9tId-b5X_svKgszJg)

### 落地，前景应用

* Instructive generation
* Instructive editing (replacing hard labor)

## BQ

Keep in mind:
* 基于给予对方认同的角度下提出个人想法
* 对现有情况的掌握，对未来的vision

为什么回国:
* 在美较久，对中美各有了解，不同生活方式，各有优劣，回国主要是因为..
* 虽然基本面来说.., 但头部有不错机会，未来发展整体看好，tradeoff的决定

Career development, motivation:
* Research-oriented development, as NLP is still not solved and product requires research
* My research vision
* 合适机会寻找产品

Failures:
* Challenges and failures are inevitable; as grown-up we realize things are expected to NOT go with the plan
* Thus, regard them as expected and necessary steps, focus on what new info is learned to complete the horizon (just like gradual gradient descent)
* Meanwhile, pivot the directions and adjust the plans based on the past steps
* Example: pivot HOI to analysis, pivot weak to seed set

Challenges examples:
* obtaining weak supervision, pivot to seed

Different opinions: depending on scenarios
* Pure technical perspective: different directions with teammates; research discussion with references and prelim experiments, list good/bad
* Direction perspective: bigger directions with management; understand what drives to a specific decision: because lack of clarity/knowledge, or due to other factors e.g. business needs?

Opinions on previous companies:
* Amazon: huge, lucky to have two good teams, OWNERSHIP; many teams doing the same thing, lacking coordination
* HA: top-down, in contrast to G/F engineering oriented; all about balance

最满意的成果?
做什么有成就感?
还有什么不足?

职位选择：
* 招收门槛，nlp团队合作者背景，平台高度
* 领导者，方向
* 过去成果
* 从事课题，未来课题规划
* 未来职业发展
* 晋升

