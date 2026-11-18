---
layout: post
title: "Works on Data Mixture"
date: 2026-11-19
categories: [LLM]
tags: [LLM, data]
math: true
---

## Overview

## Current Progress and Insights

### 1. 有常见处理骨架，但没有脱离目标和预算的“最优数据”。

常见流程为 extraction / language identification → 基础清洗 → 去重 → 质量与领域标注 → selection / mixing → 阶段性调整；各步骤的顺序仍需消融。RefinedWeb 证明认真处理的 web-only 可以胜过当时的 curated mixtures；FineWeb-Edu、DCLM、FineMath 又表明，针对学习内容的筛选可以继续提高数据效率。**同一篇文档对不同任务、模型和训练阶段的价值不同。**  
Sources: [RefinedWeb](https://arxiv.org/abs/2306.01116), [FineWeb](https://arxiv.org/abs/2406.17557), [DCLM](https://arxiv.org/abs/2406.11794), [SmolLM2](https://arxiv.org/abs/2502.02737).

### 2. Extraction 是数据质量的一部分，不能只研究 filter。

WARC HTML 重新提取常优于直接采用 WET text；boilerplate removal 与正文 recall 都重要。通用 article extractor 可能损伤 code、math、tables；Llama 3 / FineMath 保留公式结构，Qwen3 用 VLM OCR 扩展 PDF 数据。**删除 markup 的通用经验不能直接套到 LaTeX、代码缩进或文档结构。**  
Sources: [The Pile](https://arxiv.org/abs/2101.00027), [DCLM](https://arxiv.org/abs/2406.11794), [Llama 3](https://arxiv.org/abs/2407.21783), [SmolLM2](https://arxiv.org/abs/2502.02737), [Qwen3](https://arxiv.org/abs/2505.09388).

### 3. Cleanliness、educational value、relevance、coverage 应分别测量。

重复行、异常符号、菜单碎片属于基础噪声；是否解释概念、展示推导属于学习内容；与目标任务的分布接近属于 relevance。DCLM 的 instruction / ELI5 正例优于传统 Wikipedia / books 正例，且 human quality judgments 对训练收益的指示有限；SmolLM 的不同 audience / style 分别帮助不同 benchmarks。**漂亮或专业的文本不自动等于更有效的训练文本。**  
Sources: [DCLM](https://arxiv.org/abs/2406.11794), [SmolLM](https://huggingface.co/blog/smollm), [DSIR](https://arxiv.org/abs/2302.03169).

### 4. Deduplication 通常有价值；“重复越少越好”没有普遍证据。

Exact substring、near-document、URL、semantic duplicates 是不同对象；去重可降低 memorization、减少 train–test overlap，并改善大量 web 噪声。FineWeb 的 global MinHash 方案却改变了保留数据的分布，使旧 crawl 剩下更多低质量独有文本；FineWeb2 则采用 global dedup + quality-informed rehydration。**意外重复应清理，优质内容的曝光次数可以再主动优化。**  
Sources: [Deduplicating Training Data](https://aclanthology.org/2022.acl-long.577/), [FineWeb](https://arxiv.org/abs/2406.17557), [FineWeb2](https://arxiv.org/abs/2506.20920), [Olmo 3](https://arxiv.org/abs/2512.13961).

### 5. Quality filtering 本身就是隐式 domain / format mixing。

WebOrganizer 将网页分成 topic 和 format 两个维度，发现 quality filters 会显著改变两者的比例；显式 domain mixing 还能改善已 quality-filtered 的数据。SmolLM2 的 FineWeb-Edu / DCLM 互补也是实例。**“筛选器有效”可能同时反映去噪、风格选择和领域重分配，需检查这些因素。**  
Sources: [WebOrganizer](https://arxiv.org/abs/2502.10341), [SmolLM2](https://arxiv.org/abs/2502.02737).

### 6. Domain transfer / interference 使最佳比例无法按单域质量排序推导。

DoReMi 在 downweight 某些域时仍改善其 perplexity；DoGE / Skill-it 利用跨域或跨技能依赖；RegMix 发现一些按常识“优质”的来源并非最有利。Data Mixing Laws 用跨域系数预测这种关系。**只测每个来源单独训练的分数，不足以确定组合中的价值。**  
Sources: [DoReMi](https://arxiv.org/abs/2305.10429), [DoGE](https://arxiv.org/abs/2310.15393), [Skill-it](https://arxiv.org/abs/2307.14430), [RegMix](https://arxiv.org/abs/2407.01492), [Data Mixing Laws](https://arxiv.org/abs/2403.16952).

### 7. 小模型可以指导大模型，但 ranking transfer 与 metric choice 必须验证。

RegMix 支持低成本 proxy swarm → regression → large-model mixture。DataDecide 的 25 corpora 对照进一步发现：小规模的直接 ranking 已是强 baseline，复杂 scaling-law 方法未超过其 compute–decision frontier；continuous likelihood metrics 比接近随机 / 零分的离散 benchmark 更早提供信号。**准确预测 loss 数值与准确选出最佳数据是两个问题。**  
Sources: [RegMix](https://arxiv.org/abs/2407.01492), [DataDecide](https://arxiv.org/abs/2504.11393), [Olmix](https://arxiv.org/abs/2602.12237).

### 8. Mixture 应同时优化训练 horizon、有限数据和阶段安排。

严格筛选可能适合短预算，却让长训练过度重复；高质量重复也存在收益递减。Olmix 将每域 available tokens 和 repetition cap 放入最终优化约束；ADO / Aioli 则在线更新比例。Llama 3、OLMo 2、SmolLM3、Qwen3 常在后期增加 math / code / reasoning。**阶段性重配很常见，但 dynamic mixing 并不在所有实验中优于最佳 static mixture。**  
Sources: [Data-Constrained Scaling](https://arxiv.org/abs/2305.16264), [Olmix](https://arxiv.org/abs/2602.12237), [ADO](https://arxiv.org/abs/2410.11820), [Aioli](https://arxiv.org/abs/2411.05735), [SmolLM3](https://huggingface.co/blog/smollm3).

### 9. Synthetic data 的主要变量是内容设计、覆盖和验证，不只有 generator size。

SmolLM 使用相同 prompt 换更强 generator 未获显著提升，改进 topics、web seeds 和 audience 却有效；Phi-1、OLMo 2、Nemotron-CC 展示解释性重写 / exercises 的价值。Nemotron-CC 未验证重写后的 factual fidelity；model-collapse 研究提醒递归生成可丢失分布尾部。**生成来源、正确性、seed diversity 和真实数据覆盖需要一起追踪。**  
Sources: [SmolLM](https://huggingface.co/blog/smollm), [Phi-1](https://arxiv.org/abs/2306.11644), [OLMo 2](https://arxiv.org/abs/2501.00656), [Nemotron-CC](https://arxiv.org/abs/2412.02595), [Model Collapse](https://www.nature.com/articles/s41586-024-07566-y).

### 10. 文档 sampling ratio 不等于有效监督或计算比例。

按 documents、tokens、bytes 或 source size 采样会产生不同分布；Rho-1 进一步只对选中 tokens 计算 loss。Packing、截断、FIM、context length、SFT loss mask 也会改变训练目标与成本。**报告 mixture 时至少说明单位、unique / seen tokens、重复次数、阶段、packing 和 loss positions。**  
Sources: [Rho-1](https://arxiv.org/abs/2404.07965), [DeepSeek-V3](https://arxiv.org/abs/2412.19437), [SmolLM3](https://huggingface.co/blog/smollm3).

### 11. 多语言需要适配语言统计，过滤还会改变社会与文化覆盖。

英文的 word-length、punctuation、LID confidence 阈值无法原样推广；FineWeb2 用 language-specific statistics 和 low-resource precision checks 自动适配。C4 audit 发现 word blocklist 不成比例地移除少数群体相关文本。**数据质量评估应包含“被删掉了什么”，而不只统计 retained tokens 和平均分。**  
Sources: [FineWeb2](https://arxiv.org/abs/2506.20920), [C4 Audit](https://aclanthology.org/2021.emnlp-main.98/).

### 12. 可复现的训练消融与独立 holdout 比单个最终模型排名更有信息。

DCLM 固定 architecture / training recipe 来比较数据；OLMo 2 区分 development 与 held-out evaluations；DataDecide 跨 scale / seed 测决策准确率。Decontamination、同预算对照和未用于调 mixture 的评估都重要。**“我们的完整模型更强”不能单独证明其中某个 filtering / mixing 组件有效。**  
Sources: [DCLM](https://arxiv.org/abs/2406.11794), [OLMo 2](https://arxiv.org/abs/2501.00656), [DataDecide](https://arxiv.org/abs/2504.11393).

**仍未统一的选择。** 最优教育分数阈值、跨 crawl 去重强度、static vs dynamic mixture、重复上限、synthetic ratio、通用 validation loss 能否代表目标能力，以及 proxy 到不同架构 / 更大 scale 的迁移，都仍依赖设置。**My reading:** 更可靠的方向是把 quality、domain、format、repetition 和 time 分开控制，再用目标能力与独立 holdout 联合验证。

## Papers on Corpora and Data Quality

### Foundations and data auditing

**The Pile: An 800GB Dataset of Diverse Text for Language Modeling**. Gao et al.; EleutherAI. 2020 preprint / 2021 release  
Method: Combines 22 sources spanning web text, books, academic writing, code, and forums; extracts Pile-CC from WARC with jusText and assigns sampling weights by source.  
Insight: Source diversity improves cross-domain performance in its comparisons and enables evaluation by source. The results support diversity, while leaving the optimal mixture weights open.  
<https://arxiv.org/abs/2101.00027>

**Scaling Language Models: Methods, Analysis & Insights from Training Gopher**. Rae et al.; DeepMind. 2021  
Method: Applies MassiveText rules for word length, stopwords, symbols, and repeated lines / paragraphs / n-grams, alongside near-document deduplication and benchmark-overlap filtering.  
Insight: Turns common forms of noise into inexpensive, reproducible statistical rules. Gopher / MassiveText filters became important baselines for RefinedWeb, FineWeb, and Dolma.  
<https://arxiv.org/abs/2112.11446>

**Deduplicating Training Data Makes Language Models Better**. Lee et al.; Google Research et al. 2021 preprint; ACL'22  
Method: Combines exact substring matching with approximate document deduplication; measures memorization and train–validation overlap.  
Insight: Tested models emit memorized text about 10× less frequently and need fewer steps for comparable or better performance. Deduplication improves both training efficiency and evaluation validity.  
<https://aclanthology.org/2022.acl-long.577/>

**Documenting Large Webtext Corpora: A Case Study on the Colossal Clean Crawled Corpus**. Dodge et al.; AI2 et al. EMNLP'21  
Method: Audits C4's source domains, machine-generated content, benchmark overlap, and text retained or removed by filtering.  
Insight: Word blocklists disproportionately remove text from and about minority individuals. Cleaning decisions shape representation, so audits should report what was removed.  
<https://aclanthology.org/2021.emnlp-main.98/>

### Web cleaning and learned selection

**The RefinedWeb Dataset for Falcon LLM: Outperforming Curated Corpora with Web Data, and Web Data Only**. Penedo et al.; TII. 2023  
Method: Uses Trafilatura extraction, language identification, Gopher-style quality / repetition rules, and line cleaning; combines word 5-gram MinHash, exact substring deduplication above 50 tokens, and URL deduplication across processing partitions. Filtering is heuristic apart from language identification.  
Insight: Carefully processed web-only data outperforms contemporary curated corpora; filtering and deduplication both contribute. Cutting, loss-masking, or dropping documents containing exact duplicate spans shows no significant zero-shot difference in its experiments.  
<https://arxiv.org/abs/2306.01116>

**Dolma: An Open Corpus of Three Trillion Tokens for Language Model Pretraining Research**. Soldaini et al.; AI2. 2024  
Method: Builds a multi-source corpus with Gopher / C4 rules, toxicity classifiers, PII masking, and Bloom-filter-based URL / document / paragraph deduplication; releases the pipeline and data ablations.  
Insight: Rules remove different kinds of content, making their combination an empirical question. PII removal versus masking has no measurable performance effect in its ablations, where little data is affected; broader filtering costs remain setting-dependent.  
<https://arxiv.org/abs/2402.00159>

**The FineWeb Datasets: Decanting the Web for the Finest Text Data at Scale**. Penedo et al.; Hugging Face. 2024  
Method: Ablates extraction, per-crawl MinHash, and filter groups through training. Rejects documents with punctuation-line fraction ≤0.12, duplicated-line character fraction ≥0.1, or short-line fraction ≥0.67. FineWeb-Edu uses Llama-3-70B labels for 460K pages, a linear regressor on frozen embeddings, and a retention threshold of ≥3.  
Insight: Global MinHash leaves lower-quality data than per-crawl deduplication in its setting; educational filtering substantially improves MMLU / ARC. Thresholds are selected for training utility, making removal rate alone an inadequate quality metric.  
<https://arxiv.org/abs/2406.17557>

**DataComp-LM: In Search of the Next Generation of Training Sets for Language Models**. Li et al.; multi-institution team. 2024  
Method: Fixes training and evaluation protocols across scales; combines Resiliparse extraction, heuristics, and deduplication with a fastText bigram classifier trained on OpenHermes 2.5 and highly rated ELI5 positives, retaining the top 10% of documents.  
Insight: Positive-example selection matters more than classifier complexity in its comparisons; the lightweight classifier beats more elaborate alternatives. Human quality judgments have limited correspondence with downstream utility. DCLM provides a controlled platform for testing data interventions.  
<https://arxiv.org/abs/2406.11794>

**RedPajama: An Open Dataset for Training Large Language Models**. Weber et al.; Together AI, Stanford, EleutherAI et al. 2024 report  
Method: V1 reproduces a LLaMA-style multi-source corpus; V2 releases raw web text, metadata, and 46 quality signals covering repetition, length, language, and learned quality scores.  
Insight: Exposing signals and metadata enables alternative quality criteria to be combined and ablated. A scored data pool still requires selection before it becomes a filtered training corpus.  
<https://arxiv.org/abs/2411.12372>

**FineWeb2: One Pipeline to Scale Them All — Adapting Pre-Training Data Processing to Every Language**. Penedo et al.; Hugging Face / EPFL. 2025  
Method: Adapts word segmentation, language-ID confidence, and filtering thresholds to language-specific statistics. Global MinHash within each language retains cluster sizes for quality-informed rehydration using filtering rates; low-resource languages receive additional high-affinity word checks.  
Insight: Duplicate frequency and quality have a nonmonotonic relationship: singleton and very large clusters often fare worse, while intermediate-frequency content can merit upsampling. Deduplication can be followed by deliberate restoration of a useful sampling distribution.  
<https://arxiv.org/abs/2506.20920>

## Papers on Selection, Mixing, and Curriculum

### Target matching, transfer, and learnability

**Data Selection for Language Models via Importance Resampling**. Xie et al.; Stanford. 2023  
Method: Estimates $p_{target}(x)/p_{raw}(x)$ in a hashed unigram / bigram feature space and uses importance resampling to match an unlabeled target distribution.  
Insight: Resampling outperforms top-score selection in its general-domain comparisons; distribution matching can replace some expert curation. Evidence primarily comes from RoBERTa-style pretraining, leaving transfer to decoder LLMs to be validated.  
<https://arxiv.org/abs/2302.03169>

**DoReMi: Optimizing Data Mixtures Speeds Up Language Model Pretraining**. Xie et al.; Stanford / Google DeepMind. 2023  
Method: Trains a small reference model, then a proxy with Group DRO over worst-domain excess loss; averages domain weights over time and uses the resulting static mixture for a larger model. Excess loss is proxy loss − reference loss, measuring learnable headroom.  
Insight: On The Pile, a 280M proxy guides an 8B model to +6.5 points in average few-shot accuracy and baseline accuracy with 2.6× fewer steps. Even downweighted domains improve, demonstrating the importance of cross-domain transfer.  
<https://arxiv.org/abs/2305.10429>

**DoGE: Domain Reweighting with Generalization Estimation**. Fan et al.; EPFL. 2023 preprint / 2024 revision  
Method: Approximates generalization gain through alignment between training-domain gradients and target-loss gradients; learns domain weights with bilevel proxy optimization and transfers them to a larger model.  
Insight: Captures inter-domain dependencies and improves target perplexity even when the target domain is absent from the training corpus. Training utility depends on transfer relationships beyond domain names or standalone scores.  
<https://arxiv.org/abs/2310.15393>

**Skill-it! A Data-Driven Skills Framework for Understanding and Training Language Models**. Chen et al.; Stanford, University of Chicago, UW–Madison et al. 2023  
Method: Estimates a directed dependency graph from how training one skill improves loss on others, then samples online using the graph and current losses.  
Insight: Learning prerequisite skills can reduce the data needed for a target skill. Semantic labels alone do not guarantee a useful skill graph; nearly complete or empty dependency structures limit curriculum gains.  
<https://arxiv.org/abs/2307.14430>

**Rho-1: Not All Tokens Are What You Need**. Lin et al.; Microsoft, Xiamen, Tsinghua et al. 2024  
Method: Trains a reference LM on high-quality target data and computes token-level reference losses. Selective Language Modeling optimizes high excess-loss tokens while retaining their context.  
Insight: A single document contains already-learned, learnable, and noisy tokens. Selection can act on loss positions within documents; the reported gains include math continued pretraining.  
<https://arxiv.org/abs/2404.07965>

### Predicting and searching mixtures

**Data Mixing Laws: Optimizing Data Mixtures by Predicting Language Modeling Performance**. Ye et al.; Fudan / Shanghai AI Lab. 2024 preprint / 2025 revision  
Method: Fits domain losses as exponential functions of mixture proportions, $L_i(r)=c_i+k_i\exp(\sum_j t_{ij}r_j)$, and nests model-size / training-step scaling laws to predict larger-model performance.  
Insight: Mixture effects on domain loss are quantitatively predictable. In its 1B / 100B-token RedPajama comparison, the learned mixture matches the default mixture trained for 48% more steps; the framework also estimates forgetting boundaries in continued pretraining.  
<https://arxiv.org/abs/2403.16952>

**RegMix: Data Mixture as Regression for Language Model Pre-Training**. Liu et al.; Sea AI Lab et al. 2024 preprint / 2025 revision  
Method: Trains many tiny proxy models on random mixtures, fits regressors such as LightGBM, predicts untested mixtures, and trains a larger model on the selected proportions.  
Insight: A swarm of 512 1M proxies identifies the best mixture among 64 1B candidate models, supporting ranking transfer across scales. Web data correlates more positively with performance than some conventionally high-quality sources, including Wikipedia, in its tested settings.  
<https://arxiv.org/abs/2407.01492>

**Organize the Web: Constructing Domains Enhances Pre-Training Data Curation**. Wettig et al.; Princeton / AI2 / UW / Berkeley. 2025  
Method: Defines 24 topics × 24 formats with WebOrganizer; distills Llama-3.1-405B annotations into 140M classifiers and uses RegMix-style search to learn sampling across both dimensions.  
Insight: Topic and format provide complementary information, and their combination further improves quality-filtered datasets. Quality filters implicitly select different domain mixtures; organizing the web makes these effects easier to interpret.  
<https://arxiv.org/abs/2502.10341>

**Nemotron-CLIMB: CLustering-based Iterative Data Mixture Bootstrapping for Language Model Pre-training**. Diao et al.; NVIDIA / Georgia Tech. 2025  
Method: Discovers domains through semantic embeddings and clustering; proxy runs produce mixture–performance pairs for a predictor, followed by iterative proposal, pruning, and validation of candidate mixtures.  
Insight: General-purpose and domain-specific mixtures can be optimized without manual source labels, and preferred proportions change with the target. Iterative mixture search does not by itself imply online reweighting at every step of the final training run.  
<https://arxiv.org/abs/2504.13161>

**DataDecide: How to Predict Best Pretraining Data with Small Experiments**. Magnusson et al.; AI2 / UW et al. 2025  
Method: Releases controlled experiments across 25 corpora, up to 1B parameters / 100B tokens, and 3 seeds; compares small-scale rankings, 8 scaling-law baselines, and continuous likelihood proxies.  
Insight: Direct ranking at 150M predicts roughly 80% of pairwise decisions at the 1B target scale; tested scaling-law methods do not exceed its compute–decision frontier. Metrics with early signal make small experiments more useful than benchmarks stuck near zero.  
<https://arxiv.org/abs/2504.11393>

**Olmix: A Framework for Data Mixing Throughout LM Development**. Chen et al.; AI2 / Stanford / UW. 2026  
Method: Studies proxy swarms, regressors, metrics, solvers, and repetition constraints. Full / Partial Mixture Reuse preserves relative weights among unaffected domains and reoptimizes updated domains plus selected related domains.  
Insight: Required swarm size grows roughly linearly with domain count. Constraints belong in final mixture optimization, and regularization helps avoid overoptimizing noisy surrogates. Across 5 domain updates, full reuse captures 95% of full recomputation's gains with 74% fewer proxy runs.  
<https://arxiv.org/abs/2602.12237>

### Online adaptation and finite-data budgets

**Adaptive Data Optimization: Dynamic Sample Selection with Scaling Laws**. Jiang et al.; CMU / Stanford / Princeton. 2024  
Method: Fits per-domain scaling curves during training and updates the mixture using estimated learning potential and credit assignment, with temporal averaging and minimum sampling probabilities; requires no separate proxy LM.  
Insight: Online loss dynamics support inexpensive reweighting, while high absolute loss alone does not imply training value. Added wall time is <0.4% in its 1.3B experiments, although learning-potential estimates remain approximate.  
<https://arxiv.org/abs/2410.11820>

**Aioli: A Unified Optimization Framework for Language Model Data Mixing**. Chen et al.; Stanford / NYU / Genentech. 2024 preprint / 2025 revision  
Method: Unifies mixing methods as optimization over a loss–mixture response model; uses short exploration phases to reestimate mixing-law parameters online and update sampling.  
Insight: Existing methods do not consistently beat stratified sampling on average test perplexity in its comparisons; inaccurate response models help explain failures. Aioli beats this baseline on all 6 tested datasets, highlighting the need to validate mixing assumptions.  
<https://arxiv.org/abs/2411.05735>

**Scaling Data-Constrained Language Models**. Muennighoff et al.; Hugging Face, Harvard et al. 2023  
Method: Fits a scaling law for diminishing returns from repeated tokens using 400+ controlled runs; compares repetition, code augmentation, perplexity filtering, and deduplication.  
Insight: Under its fixed-compute, finite-data settings, repetition differs little from unique data through roughly 4 epochs, with diminishing returns thereafter. Filtering helps noisy corpora more; aggressively shrinking clean, scarce data can be counterproductive. The 4-epoch observation is setting-specific.  
<https://arxiv.org/abs/2305.16264>

## Papers on Synthetic Data and Rewriting

**Textbooks Are All You Need**. Gunasekar et al.; Microsoft. 2023  
Method: Uses GPT-4 to label educational value for about 100K code samples, then CodeGen embeddings and a random forest to select about 6B tokens. GPT-3.5 generates textbooks / exercises, with topic, audience, and function-name constraints encouraging diversity.  
Insight: Compact, explanatory Python data can train strong small models; content organization and task-aligned exercises matter. Reported style and API-coverage limitations constrain generalization to broader LLM capabilities.  
<https://arxiv.org/abs/2306.11644>

**SmolLM / SmolLM-Corpus**. Ben Allal, Lozhkov, Bakouch et al.; Hugging Face. 2024 author blog / dataset release  
Method: Combines FineWeb-Edu-Dedup, Python-Edu, and Cosmopedia v2. A scorer trained on 500K Llama3 code labels retains scores ≥4; synthetic generation uses 34K+ topics, relevant web seeds, and controlled audiences / styles.  
Insight: Python-Edu reaches the reported 16% HumanEval pass@1 target over 3× faster. Stronger generators with unchanged prompts yield no significant gain, while better topics, seeds, and prompts help. Middle-school textbooks suit most tested tasks; stories help commonsense.  
<https://huggingface.co/blog/smollm>  
<https://huggingface.co/datasets/HuggingFaceTB/smollm-corpus>

**Nemotron-CC: Transforming Common Crawl into a Refined Long-Horizon Pretraining Dataset**. Su et al.; NVIDIA. 2024 preprint / 2025 revision  
Method: Takes the maximum percentile bucket across three quality classifiers and groups quality tiers using training results. Rewrites low-quality pages for clarity and derives varied QA / presentations from high-quality pages, alongside a larger pool of unique original text.  
Insight: Low recall from a single strict scorer constrains long-horizon training. Its HQ subset beats DCLM by 5.6 MMLU points in matched 8B / 1T experiments; the full corpus balances quality and supply. Rewrite fidelity is unverified, and pipeline ablations are incomplete.  
<https://arxiv.org/abs/2412.02595>

**AI Models Collapse When Trained on Recursively Generated Data**. Shumailov et al.; Oxford, Cambridge et al. Nature'24  
Method: Uses theoretical distribution models and generative-model experiments to study error accumulation when successive models repeatedly learn from predecessor outputs.  
Insight: Recursive generation can first lose distribution tails and then degrade diversity. These recursive, indiscriminate learning settings do not establish that synthetic augmentation with real seeds, filtering, and verification is universally harmful.  
<https://www.nature.com/articles/s41586-024-07566-y>

## Open Model Reports and Complete Recipes

**The Llama 3 Herd of Models**. Meta. 2024 technical report  
Method: Preserves math / code through custom HTML extraction; applies URL / document / line deduplication and general-quality / code / STEM classifiers. Knowledge categories and small-model scaling experiments guide mixing, followed by quality-heavy annealing.  
Insight: Integrates broad coverage, capability-specific selection, and annealing. The reported mix is about 50% general knowledge / 25% math-reasoning / 17% code / 8% multilingual; these proportions describe its recipe and require validation for new models.  
<https://arxiv.org/abs/2407.21783>

**2 OLMo 2 Furious**. OLMo Team; AI2. 2025 technical report  
Method: Follows large-scale pretraining with high-quality Dolmino mid-training. Inexpensive microanneals from the same checkpoint test sources, mixtures, repetition, and rewriting, with separate development and holdout tasks.  
Insight: Small math-data proportions already help substantially, and modest repetition adds gains. Tiny-GSM code solutions perform poorly with its low-code baseline; rewriting them as natural-language problem solving helps, showing the importance of matching representation to the checkpoint.  
<https://arxiv.org/abs/2501.00656>

**SmolLM2: When Smol Goes Big — Data-Centric Training of a Small Language Model**. Ben Allal et al.; Hugging Face. 2025 report (model released in 2024)  
Method: Ablates web / math / code sources and stage-specific proportions. FineMath discovers useful domains, expands crawling, reextracts pages preserving LaTeX, and selects step-by-step reasoning; Stack-Edu filters educational code, with specialized high-quality data added later.  
Insight: A 60/40 FineWeb-Edu / DCLM mix balances educational and commonsense benchmarks in its comparisons. FineMath4+ achieves about 2× GSM8K / 6× MATH scores over InfiMM-WebMath in annealing experiments, emphasizing worked reasoning and appropriate content level.  
<https://arxiv.org/abs/2502.02737>

**DeepSeek-V3 Technical Report**. DeepSeek-AI. 2024 preprint / 2025 revision  
Method: Increases math / programming proportions and language coverage while reducing redundancy and preserving diversity; uses document-preserving packing without cross-document attention masking and applies FIM preprocessing at 10%.  
Insight: Data representation and objective transformations belong in the recipe. Building on DeepSeekCoder-V2's FIM findings, it supports both infilling and next-token prediction. Limited disclosure of cleaning details and source weights restricts reproducibility.  
<https://arxiv.org/abs/2412.19437>

**Qwen3 Technical Report**. Qwen Team; Alibaba. 2025  
Method: Expands PDF text with Qwen2.5-VL OCR and Qwen2.5 refinement; synthesizes data with specialized math / coder models and annotates educational value, fields, and domains. Training proceeds from general coverage to STEM / code / reasoning emphasis, then long context.  
Insight: Document acquisition, capability classification, and stage-specific reweighting jointly shape the corpus. Roughly 30T general-stage and 5T reasoning-stage tokens describe the complete recipe; the report does not isolate each data intervention's causal contribution.  
<https://arxiv.org/abs/2505.09388>

**SmolLM3: Smol, Multilingual, Long-Context Reasoner**. Hugging Face. 2025 author report  
Method: Uses 3B-model data ablations; shifts web / code / math proportions from 85/12/3 → 75/15/10 → 63/24/13, progressively adding Stack-Edu and stronger math data. Separate long-context and reasoning mid-training stages have published configurations.  
Insight: Mixtures evolve with training stage and data quality. Extra upsampling of books, repositories, and long web pages yields no further gain in its long-context ablations; reasoning mid-training later degrades long-context performance, revealing capability interference.  
<https://huggingface.co/blog/smollm3>

**Olmo 3**. Olmo Team; AI2. 2025 report / 2026 revision  
Method: Builds Dolma 3 with topic / programming-language partitions, 30M proxy swarms, per-task BPB regression, token constraints, and conditional mixing; adds quality-aware upsampling within domains, followed by targeted mid-training and decontamination.  
Insight: Mixing across domains and quality sampling within domains are separate optimization layers. Repeating top-quality data can beat flat top-k filtering under finite token budgets. Reusing mixtures as sources evolve is a practical development requirement, later studied systematically in Olmix.  
<https://arxiv.org/abs/2512.13961>

## Reproduction Pointers

| Resource | Useful assets / purpose |
|---|---|
| [Datatrove](https://github.com/huggingface/datatrove) | Extraction、heuristics、MinHash / exact dedup 等大规模 pipeline components |
| [FineWeb](https://huggingface.co/datasets/HuggingFaceFW/fineweb) / [FineWeb-Edu](https://huggingface.co/datasets/HuggingFaceFW/fineweb-edu) | Broad web 与 educational subsets；按实际版本核对 metadata 和 token counts |
| [FineWeb2 pipeline](https://github.com/huggingface/fineweb-2) | Language-specific processing、dedup metadata、rehydration 与 training / evaluation code |
| [DCLM](https://github.com/mlfoundations/dclm) | 固定训练 / 评估的 filtering 和 mixing benchmark |
| [Dolma](https://github.com/allenai/dolma) | 多来源 curation、质量与重复处理、公开 corpus tooling |
| [RegMix](https://github.com/sail-sg/regmix) / [Mixing Laws](https://github.com/yegcjs/mixinglaws) | Proxy swarm、response fitting、candidate-mixture search |
| [WebOrganizer](https://weborganizer.allen.ai/) | Topic / format taxonomy、classifiers、带领域标注的 web 数据 |
| [DataDecide](https://github.com/allenai/datadecide) | 跨 corpora / scale / seed 的 checkpoints 与预测决策实验 |
| [SmolLM](https://github.com/huggingface/smollm) | SmolLM2/3 training configs 与各阶段数据；Python-Edu 的 code retrieval 需另按 dataset card 操作 |
| [Olmix](https://github.com/allenai/olmix) | Constrained mixing、配置消融与 domain-update reuse |
