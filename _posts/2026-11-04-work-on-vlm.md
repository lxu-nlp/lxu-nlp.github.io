---
layout: post
title: "Works on Visual Language Models"
date: 2026-11-04
categories: [LLM]
tags: [LLM, vlm, multimodal]
math: true
---

For pre-LLM multimodal works, see [Notes on multimodal](https://lxu-nlp.github.io/posts/work-on-multimodal/).

## Overview

Updated: **2026-11-04**. Focus: image/video understanding VLMs, especially training, data, perception, and reasoning. Audio and image generation are discussed only when they affect the training recipe.

**Main takeaways.**

- **有事实上的训练骨架，但没有统一的最优 recipe。** 对使用现成 LLM/ViT 的路线，常见流程为 visual initialization/alignment → multimodal joint training / mid-training → instruction tuning → optional distillation / preference optimization / RL；长 context 往往逐步引入。数据比例、冻结哪些模块、是否单独做 projector warm-up，以及 RL 算法，都还没有统一答案。参见 [Qwen3-VL](https://arxiv.org/abs/2511.21631)、[InternVL3.5](https://arxiv.org/abs/2508.18265)、[Molmo2](https://arxiv.org/abs/2601.10611)。
- **2025 的主要进展是训练和数据的系统化。** Dynamic resolution、OCR/document/grounding、interleaved data、text-only replay、多阶段训练和 reasoning post-training 成为代表模型的共同关注点。架构多数仍是 ViT–connector–LLM，但 visual token 的数量、位置、融合层和 attention mask 很重要。参见 [Qwen2.5-VL](https://arxiv.org/abs/2502.13923)、[Eagle 2](https://arxiv.org/abs/2501.14818)、[InternVL3](https://arxiv.org/abs/2504.10479)。
- **2026 更明确地走向 native joint pretraining、视觉依赖的训练目标，以及按预算分配视觉计算。** Kimi K3 从头训练视觉 encoder 并与语言联合优化；Gemma 4 12B 探索 encoder-free 输入；LLaVA-OneVision-2 探索 codec-aware video tokenization。这些是不同路线，不能合并成一套 recipe。参见 [Kimi K3 report](https://github.com/MoonshotAI/Kimi-K3/blob/main/k3_tech_report.pdf)、[Gemma 4](https://arxiv.org/abs/2607.02770)、[LLaVA-OneVision-2](https://arxiv.org/abs/2605.25979)。
- **训练瓶颈需要区分 perception、reasoning 和 visual reliance。** 模型可能看错图，也可能看到了却不在推理中使用；更长 CoT / 更强 text reasoning 不自动修复这两种问题。训练数据应检验“不给图还能不能做对”，后训练则需要 visual reward、counterfactual objectives 或视觉工具反馈。参见 [MME-CoT](https://proceedings.mlr.press/v267/jiang25n.html)、[Perception-R1](https://openreview.net/forum?id=KttCXdjj4w)、[PAPO](https://openreview.net/forum?id=izbBqTL8vb)、[VIG](https://openreview.net/forum?id=z1bSFIEexL)。
- **研究可复现性时，优先读公开 data / code / intermediate checkpoints 的项目。** Molmo2、LLaVA-OneVision-1.5/2 和 Eagle 对实际训练有较高参考价值；旗舰开权重模型的报告用于观察大规模训练趋势，但通常不能完整复现。具体开放程度见下文各条目。

**Reading order.**

1. 训练流程：Qwen3-VL → Molmo2 → LLaVA-OneVision-1.5 → Kimi K2.5/K3。
2. 训练设计和消融：Prismatic → MM1 → Eagle/Eagle 2 → MiniCPM-V 4.5 → SmolVLM。
3. 视觉推理：MME-CoT → VL-Rethinker → Perception-R1/PAPO → ICML'26 perception–reasoning staged training。
4. 2026 数据研究：DataComp-VLM → Scaling Laws for Multimodal Data Mixtures。两篇均已在 NeurIPS'26 官方论文列表中；截至本 note 更新时会议尚未召开。

**Evidence convention.** 论文的首发年份与录用年份分开标注；tech report 不默认视为顶会论文。标记 **（abstract-level）** 的条目核验了官方会议摘要与作者/录用信息，未读取完整正文，结论限于摘要所述范围。高影响基础工作按后续报告的反复引用和训练相关性选择，不给未经核验的 citation ranking。下文的 **My reading / Suggested recipe** 是基于这些来源的综合判断，区别于作者报告的实验结论。

## Training Pipeline and the De Facto Recipe

### What is actually optimized?

多数 understanding VLM 将图像/video 编码成 continuous visual embeddings，作为 LLM 的条件输入，预测文本 answer/caption/coordinates/code。**Next-token prediction 不等于预测离散 image tokens**；训练损失可以仅覆盖文本 token，但梯度仍能通过 connector 回传到 ViT。InternVL3 明确描述了这种 text-only loss、full-parameter joint optimization 的组合。  
<https://arxiv.org/abs/2504.10479>

对样本 $i$，一种便于讨论的 SFT / conditional-LM 写法是：

$$
\mathcal{L}=\frac{1}{Z}\sum_i a_i\sum_{t\in\mathcal{T}_i}b_{i,t}\left[-\log p_\theta(y_{i,t}\mid v_i,x_i,y_{i,<t})\right].
$$

- $v_i$: visual embeddings。
- $x_i$: prompt。
- $\mathcal{T}_i$: 有监督的 target text positions。
- $a_i$: 样本/task weight。
- $b_{i,t}$: token weight。
- $Z$: 归一化因子。

这是概念性写法；预训练全文建模和 SFT answer-only masking 需要分别配置。InternVL3 的 length weighting、Molmo2 的 task weighting/global normalization 说明：**相同 data mixture 不一定意味着相同有效优化目标。**  
<https://arxiv.org/abs/2504.10479>  
<https://arxiv.org/abs/2601.10611>

### Common stages

| Stage | Goal / data | Common trainable modules | Important qualifications |
|---|---|---|---|
| Visual initialization | CLIP/SigLIP-style visual pretraining，或 caption/OCR/grounding adaptation | ViT + training heads / small LM | 不是每个 VLM 项目都从头做；可直接复用 pretrained ViT；Kimi K3 从头用 NTP 训练 |
| Alignment / warm-up | 建立 visual-to-language interface；caption/OCR 等 | Connector only，或 ViT + connector | 常见但非必要；Qwen3-VL、Eagle 做显式 warm-up，Prismatic/Molmo 给出替代方案 |
| Joint multimodal training / mid-training | 学 visual knowledge、OCR、grounding、多图、video 和语言能力 | 通常更新 LLM + connector；大预算下常更新全部模块 | 混入 text-only；“mid-training”和“pretraining”的命名跨报告不统一 |
| Context extension | 多页文档、长 video、long trajectories | 依模型而定 | 通常先短后长，并混入短样本；position extension 本身不足以训练长距离依赖 |
| SFT / cold start | Instruction following、output schema、CoT、tool protocol | LLM + connector，或 full model | 可以分 thinking/non-thinking；Kimi K2.5 的 zero-vision SFT 是特殊结果 |
| Preference optimization / distillation / RL | 对齐偏好、reasoning、grounding、tool use | Policy trainable modules 按任务选 | 各家顺序不同；DPO/MPO/GRPO/GSPO/SAPO 均有使用，没有算法统一标准 |

Sources: [Qwen2.5-VL](https://arxiv.org/abs/2502.13923), [Qwen3-VL](https://arxiv.org/abs/2511.21631), [Eagle 2](https://arxiv.org/abs/2501.14818), [InternVL3.5](https://arxiv.org/abs/2508.18265), [Molmo2](https://arxiv.org/abs/2601.10611), [Kimi K2.5](https://arxiv.org/abs/2602.02276), [Kimi K3](https://github.com/MoonshotAI/Kimi-K3/blob/main/k3_tech_report.pdf).

### Representative complete recipes

| Model | Initialization / early training | Joint / context training | Post-training | What to learn |
|---|---|---|---|---|
| Qwen2.5-VL, 2025 | Custom ViT；报告中的 visual phase 为 1.5T tokens、8K，主要训 ViT | 2T at 8K + 0.6T at 32K，joint training | SFT → DPO；post-training 冻结 ViT | OCR/grounding 和 dynamic resolution；visual adaptation 不只训 projector |
| Qwen3-VL, 2025 | SigLIP-2-derived encoder + Qwen3；67B merger-only alignment at 8K | ~1T at 8K → ~1T at 32K → 100B at 256K；全部参数 | SFT → strong-to-weak distillation → reasoning/general RL | 清晰的长 context curriculum；text-only distillation 可迁移到 VL reasoning |
| InternVL3 / 3.5, 2025 | ViT–MLP–LLM，native multimodal continuation / joint optimization | InternVL3 报告约 200B tokens；text : multimodal = 1:3 | v3: SFT → MPO；v3.5: SFT → offline MPO → online GSPO | Text retention、square averaging、offline-to-online RL |
| LLaVA-OneVision-1.5, 2025 | RICE-ViT + Qwen3；558K connector alignment | 85M concept-balanced mid-training，full model | 22M instructions + FineVision，full model；后续独立 RL release | 开放 data/code/checkpoints；mid-training 数据规模与 concept balancing |
| Molmo2, 2026 report | Pretrained LLM + SigLIP2；caption/pointing/NLP，全部更新 | Multitask SFT at 16K；短 long-context stage at 36,864 | 发布流程为 pretraining → SFT → long-context SFT | 开放 video grounding 数据、message trees、loss weighting |
| Kimi K2.5, 2026 | Near-end K2 checkpoint；ViT caption adaptation + 很短的 projector alignment | 15T joint tokens at 4K；long-context mid-training 32K → 256K | Zero-vision text SFT → joint text/vision RL | Early low-ratio fusion；强 pretraining 下的跨模态迁移 |
| Kimi K3, 2026 | Native joint training；MoonViT-V2 从头训练，NTP objective | 8K → 64K pretraining；256K → 1M cooldown | SFT → domain/effort-specific RL → multi-teacher on-policy distillation；SFT 起 QAT | 视觉 encoder 初始化、训练稳定性、long-context tasks、policy consolidation |
| Gemma 3 / 4, 2025/26 | 多数变体用 frozen visual encoder；Gemma4-12B 为 encoder-free | Multimodal pretraining；Gemma3 明确采用 teacher distillation | Distillation / RL；Gemma4 引入 thinking | Frozen ViT 仍是有效路线；同一模型 family 内也未统一架构 |

Sources: 各模型原文见 [Open Model Reports](#open-model-reports-20252026)。Token budget 指各报告自己的统计口径，可能包括 visual/text tokens、重复曝光和不同阶段；不要直接用这些数值比较训练 FLOPs、unique data 或成本。

### A concrete reproducible example: Molmo2

The following values are **reported settings, not recommended defaults**. They are useful because the report and repo expose all three stages.

| Setting | Image pretraining | Multitask SFT | Long-context SFT |
|---|---|---|---|
| Steps / global batch size | 32K / 128 | 30K / 128 | 2K / 128 |
| Maximum sequence length | 2,560 | 16,384 | 36,864 |
| Peak LR: ViT / connector / LLM | `6e-6` / `2e-4` / `2e-4` | `5e-6` / `5e-6` / `1e-5` | Same as SFT |
| Warm-up steps: ViT / connector / LLM | 2,000 / 200 / 2,000 | 200 / 200 / 200 | Same optimizer settings as SFT |
| Optimizer | AdamW; betas `(0.9, 0.95)`, epsilon `1e-6` | Same | Same |
| Schedule / weight decay | Cosine to 10% of peak LR; no weight decay | Same | Same |
| Data | 60% caption / 30% pointing / 10% NLP | Task-balanced image/video/multi-image/text | Same mixture, larger frame budget |

Source: [Molmo2](https://arxiv.org/abs/2601.10611), Section 3.2, Table 12, Appendix A/B; [training code](https://github.com/allenai/molmo2). The pretrained backbones, target weighting, attention masks and data formats are integral to these values. Full-parameter training here does not mean identical learning rates or identical warm-up for every module.

### “Native multimodal” needs a precise definition

**Joint training / early fusion、from-scratch initialization、encoder-free input 是三个不同维度。**

- InternVL3 所称 native pretraining 是将大规模文本和多模态数据统一优化；不能据此推断所有 weights 都随机初始化。 <https://arxiv.org/abs/2504.10479>
- Kimi K2.5 明确从 near-end Kimi K2 checkpoint 继续训练。它的 early fusion 指该联合训练过程更早注入 vision，而不是重做 K2 全部语言预训练。 <https://arxiv.org/abs/2602.02276>
- Kimi K3 明确联合训练语言和视觉，MoonViT-V2 从头训练，不用 SigLIP initialization 或 post-hoc modality alignment；仍有 ViT + MLP 视觉路径。 <https://github.com/MoonshotAI/Kimi-K3/blob/main/k3_tech_report.pdf>
- Gemma 4 12B 是 encoder-free：raw image patches / audio chunks 经轻量 projection 进入 LLM。其他 Gemma 4 变体有 frozen encoders。 <https://arxiv.org/abs/2607.02770>

**My reading.** 调研时应记录 initialization、何时引入 vision、哪些模块更新、预测哪些 targets，避免只按 native / non-native 的发布标签分类。

## Training Insights

### Data mixture and effective supervision

#### 1. Data diversity, visual necessity, and task coverage matter more than raw sample count.

Eagle 2 从 180+ sources 收集数据，再做 source balancing、过滤和 selection；其更小但更均衡的 SFT subset 可以改善表现。LLaVA-OneVision-1.5 在固定下游 recipe 下展示 concept-balanced mid-training 的收益。Molmo/PixMo 则展示少量高密度、人类 visual descriptions 的价值。  
<https://arxiv.org/abs/2501.14818>  
<https://arxiv.org/abs/2509.23661>  
<https://arxiv.org/abs/2409.17146>

**My reading.** “Quality”需要拆成正确性、视觉依赖、信息密度、覆盖范围、target style 和任务适配。通用 quality score 高，不一定意味着对 OCR、pointing 或 temporal grounding 有价值。

#### 2. Filtering does not replace mixing.

DataComp-VLM 的受控实验强调：在其模型/预算/任务范围，instruction-heavy mixtures 比 caption-heavy mixtures 更能随规模增长；mixing 的作用大于它所比较的 filtering strategies。该结论应与 Molmo 的 dense-caption 结果一起读：数据阶段、caption 信息密度、基础模型和评估目标都不同。**（abstract-level）**  
<https://openreview.net/forum?id=s8J0YiyTuN>  
<https://arxiv.org/abs/2409.17146>

#### 3. Text-only replay is common, but there is no universal text : vision ratio.

InternVL3 报告 50B text + 150B multimodal tokens；Qwen2.5-VL 的 SFT 为约 50% text samples，Qwen3-VL 的 SFT 为约 1/3 text samples；Kimi K2.5 比较 early 10:90 vision:text、mid 20:80、late 50:50 的同总预算安排。**这些比例的单位和阶段不同。**  
<https://arxiv.org/abs/2504.10479>  
<https://arxiv.org/abs/2502.13923>  
<https://arxiv.org/abs/2511.21631>  
<https://arxiv.org/abs/2602.02276>

SmolVLM 进一步发现：直接复用 LLM 最终 SFT text mixture 会使小 VLM 退化；保留 text 数据不等于保留任意 text 数据。  
<https://arxiv.org/abs/2504.05299>

**My reading.** 同时追踪 source sampling probability、visual/input tokens、supervised answer tokens 和 FLOPs；sample-level 的“50% text”不能推导成 50% compute 或 50% gradient contribution。

#### 4. Response length and loss normalization silently change the recipe.

InternVL3 的每 token 权重：token averaging $1$、square averaging $1/\sqrt{n}$、sample averaging $1/n$，$n$ 为该样本受监督 token 数。Square averaging 是长短答案之间的折中，不等同于对各 dataset size 做 square-root sampling。  
<https://arxiv.org/abs/2504.10479>

Molmo2 对 video caption/pointing 使用较低 task weights，对其他任务用 $4/\sqrt{n}$；此外以跨 GPU 的平均 loss-token 数归一化每 GPU loss，避免 local token count 不同造成隐式 sample reweighting。它同时采用 dataset-size square-root sampling；这是另一层独立机制。  
<https://arxiv.org/abs/2601.10611>, Sections 3.2 and Appendix A/B.

**My reading.** 复现时记录 loss mask、task weights、length weights、DDP/gradient accumulation normalization；只抄 LR、batch size 和 dataset percentages 很可能无法复现有效目标。

### Alignment, encoder training, and scale

#### 5. A separate projector warm-up is a useful option, not a prerequisite.

Prismatic 在 LLaVA-scale data 下去掉 alignment stage 可维持/改善表现；Molmo 全参数 caption training 不做各模块分阶段冻结；Molmo2 用更高 connector LR 和更长 ViT/LLM warm-up，使最初的更新主要作用于 connector。Qwen3-VL 和 LLaVA-OneVision-1.5 仍采用显式 alignment。  
<https://proceedings.mlr.press/v235/karamcheti24a.html>  
<https://arxiv.org/abs/2409.17146>  
<https://arxiv.org/abs/2601.10611>  
<https://arxiv.org/abs/2511.21631>  
<https://arxiv.org/abs/2509.23661>

PORTAL 的 NeurIPS'26 官方摘要进一步报告：用 modality summary statistics 做 training-free projector initialization，在所测 backbone/benchmark 上可替代 paired-caption projector pretraining。其结果加强了“coarse alignment 可有多种实现”的判断，尚不能外推到所有规模。**（abstract-level）**  
<https://openreview.net/forum?id=HObX41utGv>

#### 6. “Freeze ViT” and “unfreeze ViT” are both supported under different conditions.

Prismatic 在其小数据设置下发现 full ViT fine-tuning 伤害性能，尤其 localization；Molmo、Molmo2、InternVL3、LLaVA-OneVision-1.5 在更充分数据下更新 ViT；Gemma3/4 多数变体冻结已适配的 vision encoder。Kimi K3 则展示从头训练 encoder 的稳定性与性能。  
<https://arxiv.org/abs/2402.07865>  
<https://arxiv.org/abs/2601.10611>  
<https://arxiv.org/abs/2504.10479>  
<https://arxiv.org/abs/2503.19786>  
<https://arxiv.org/abs/2607.02770>  
<https://github.com/MoonshotAI/Kimi-K3/blob/main/k3_tech_report.pdf>

**My reading.** 小规模 domain adaptation 可以先冻结 ViT；若错误来自 OCR、小物体、空间/时序表征，且有足够针对性监督，再比较低 LR 解冻、分阶段视觉训练和更强 encoder。不能把小预算消融推广成基础模型训练定律。

#### 7. Vision initialization affects optimization as well as feature quality.

Kimi K3 Figure 6 比较 SigLIP-initialized MoonViT-3D 与 from-scratch MoonViT-V2：后者 gradient norm 更低、spikes 更少，并匹配前者的 vision evaluation。作者据此认为，在其规模下 contrastive initialization 非必要。该比较也包含 encoder architecture/normalization 改动；不能解读为所有架构的 random init 都更好。  
<https://github.com/MoonshotAI/Kimi-K3/blob/main/k3_tech_report.pdf>, Section 2.4, Figure 6.

**My reading.** 选择 encoder 时同时看 dense feature/OCR/grounding 表现、joint-training stability 和 token cost；CLIP zero-shot retrieval 分数不足以独立决定 VLM backbone。

### Visual tokens, positions, and context

#### 8. The visual evidence budget is part of the training recipe.

MM1 强调 image resolution、visual token count 和 encoder 的影响，connector 类型的影响相对更小；MiniCPM-V 4.5、InternVL3.5-Flash 和 SmolVLM 用不同 compression 策略获得效率收益。更强压缩改变了能保留什么视觉证据，因此需要按 OCR/小目标/空间关系/video 等任务分别比较。  
<https://arxiv.org/abs/2403.09611>  
<https://arxiv.org/abs/2509.18154>  
<https://arxiv.org/abs/2508.18265>  
<https://arxiv.org/abs/2504.05299>

#### 9. Multi-level fusion and visual attention are meaningful design choices.

Qwen3-VL 的 DeepStack 将多个 ViT 层的特征经 merger 注入早期 LLM layers，不增加 context token 数；Molmo2 的 visual tokens 可彼此 bidirectionally attend，跨 frame/image 的 visibility 也会影响效果。它们处理的是 representation/access，而不只是 connector 参数量。  
<https://arxiv.org/abs/2511.21631>, Sections 2.2 and 5.12.2.  
<https://arxiv.org/abs/2601.10611>, Section 3.1 and ablations.

#### 10. Long-context capability needs long-range supervision and a curriculum.

Qwen3-VL: 8K → 32K → 256K；Eagle 2.5: progressive mixed post-training；Molmo2: 16K SFT 后短 36K stage；Kimi K3: 8K → 64K → 256K → 1M，并合成长任务，使 answer 需要 scattered evidence。不能只改 RoPE / max length 就视为已学会长 video/document reasoning。  
<https://arxiv.org/abs/2511.21631>  
<https://arxiv.org/abs/2504.15271>  
<https://arxiv.org/abs/2601.10611>  
<https://github.com/MoonshotAI/Kimi-K3/blob/main/k3_tech_report.pdf>

#### 11. Video training must allocate both spatial and temporal evidence.

Qwen2.5-VL 动态 FPS + absolute-time MRoPE；Qwen3-VL 改为 explicit textual timestamps 和 interleaved MRoPE。Kimi K2.5 对连续 frames 做 temporal compression；LLaVA-OneVision-2 依据 codec bit-cost/motion residual 分配 tokens，并指出 detail-sensitive queries 仍可能更适合 frame sampling。  
<https://arxiv.org/abs/2502.13923>  
<https://arxiv.org/abs/2511.21631>  
<https://arxiv.org/abs/2602.02276>  
<https://arxiv.org/abs/2605.25979>

**My reading.** Frames/sec、resolution、tokens/frame、timestamp scheme 和 sampling mode 都应固定或报告。增加 frame count 可能挤压每帧细节；长 video 的 temporal retrieval、dense motion 和 fine OCR 不一定共享最优采样策略。

### Reasoning, visual reliance, and RL

#### 12. Text reasoning can transfer to vision, provided alignment is already strong.

ICML'25 model merging 可以把同源 text reasoning weights 迁移到 VLM；Qwen3-VL 用 text-only strong-to-weak distillation 增强 multimodal reasoning；Kimi K2.5 用 zero-vision SFT 激活视觉 agent/tool behaviors，然后 joint RL。这些结果说明部分 reasoning skills 跨模态共享，但不说明可以跳过视觉预训练。  
<https://proceedings.mlr.press/v267/chen25cm.html>  
<https://arxiv.org/abs/2511.21631>  
<https://arxiv.org/abs/2602.02276>

#### 13. Outcome RL / longer CoT does not guarantee better perception.

MME-CoT 发现 perception-heavy tasks 可能被 CoT prompting 损害；Perception-R1 报告常规 RLVR 对 perception 的提升不足；ICML'26 staged training 用专门数据先学 perception 再学 reasoning，优于其比较的混合训练。上述结论限于各自 benchmark / checkpoints；不构成“RL 不能训练 perception”的结论。  
<https://proceedings.mlr.press/v267/jiang25n.html>  
<https://openreview.net/forum?id=KttCXdjj4w> **（abstract-level）**  
<https://openreview.net/forum?id=r7uOjvZdzO> **（abstract-level）**

#### 14. Reward / objective should make visual evidence useful.

Perception-R1 用 visual-reference consistency 给 perception reward；PAPO 对同一 rollout 在 original/corrupted image 下的分布引入 KL-based implicit perception loss；VIG 量化图像对预测不确定性的降低，并选择有视觉信息的 samples/tokens。三者分别作用于 reward、optimization objective 和 data/token selection。**（abstract-level）**  
<https://openreview.net/forum?id=KttCXdjj4w>  
<https://openreview.net/forum?id=izbBqTL8vb>  
<https://openreview.net/forum?id=z1bSFIEexL>

Qwen3-VL cold-start 数据直接剔除 text-only Qwen3 能在不给图时答对的 visual math 题；这是容易实现的 visual-necessity filtering baseline。  
<https://arxiv.org/abs/2511.21631>, Section 4.2.2.

**My reading.** 视觉依赖不能仅用 attention heatmap 判断。训练/评估可增加 no-image、wrong-image、caption/oracle-perception controls；构造 counterfactual 时应确保 image 改变确实影响 answer，避免把背景扰动误当作有效视觉监督。

#### 15. RL difficulty selection, reward reliability, and thinking budget interact.

VL-Rethinker 用 SSR 缓解 binary-reward group 全对/全错的 vanishing advantages，并用 forced rethinking 激活反思；InternVL3.5 用 offline MPO warm-up → online GSPO；GLM-4.1V 用 curriculum/difficulty-aware sampling 和多域 reward；MiniCPM-V 4.5 联合优化 short/long reasoning modes。  
<https://arxiv.org/abs/2504.08837>  
<https://arxiv.org/abs/2508.18265>  
<https://arxiv.org/abs/2507.01006>  
<https://arxiv.org/abs/2509.18154>

Qwen3-VL 按当前 model 的 pass rate 去掉过易问题，并筛掉 flagship 多次采样都失败的 queries；这些阈值属于其 recipe，难题不能被笼统视为“越难越好”。  
<https://arxiv.org/abs/2511.21631>, Section 4.4.1.

**My reading.** 先看 non-zero-advantage group 比例、reward agreement 和 task-specific accuracy，再解释训练是否停滞。可在已有 VLM 上直接 RL，也可先 cold-start SFT；是否需要 cold start 取决于 rollout 是否已有正确格式、基本 tool use 和成功轨迹。

#### 16. Training visual tool use changes the reasoning problem.

VTool-R1 在 RL 环境中让 VLM 用 Python 视觉编辑工具、返回中间图像，再继续推理；Kimi K2.5/K3 将 visual inspection 与 coding/tool trajectories 结合。监督对象因此是“获取/编辑证据 → 观察 → 决策”，不只是看一次图后输出更长文字。  
<https://openreview.net/forum?id=Idst6X6gmy> **（abstract-level）**  
<https://arxiv.org/abs/2602.02276>  
<https://github.com/MoonshotAI/Kimi-K3/blob/main/k3_tech_report.pdf>

### Training systems and evaluation

#### 17. Packing should balance both visual and language work.

Eagle 2 / LLaVA-OneVision-1.5 优化 packed sequences 和跨 GPU 负载；Molmo2 同时约束 LLM tokens 与 image crops，并用 message trees 共享一个 visual prefix 的多条 annotations。树状/packed attention mask 必须阻止不相关样本及 sibling answers 互相泄漏。  
<https://arxiv.org/abs/2501.14818>  
<https://arxiv.org/abs/2509.23661>  
<https://arxiv.org/abs/2601.10611>

FlexRL 将 decoding/preprocessing 分散到 worker，并按计算/显存成本拆分长序列，处理 VLM RL 的 I/O 和 workload skew。其最高 throughput speedup 依赖所测 hardware/workload，不能视作通用加速倍率。**（abstract-level）**  
<https://openreview.net/forum?id=aa14rlfR6k>

#### 18. Separate model learning from extra inference compute.

Qwen3-VL 明确披露 video 比较中的不同 API frame caps；InternVL3.5 的 parallel thinking、Kimi 的 tool-enabled vision evaluation、以及 thinking/non-thinking modes 都改变推理预算。报告模型性能时需要同时给出 input visual tokens/frames、output token cap、samples/query、tools 和 aggregation policy。  
<https://arxiv.org/abs/2511.21631>, Section 5.5.  
<https://arxiv.org/abs/2508.18265>  
<https://github.com/MoonshotAI/Kimi-K3>

**My reading.** 训练 recipe 的主评价至少分 perception、visual reasoning、language retention 和 cost；同预算单次回答与 extra-compute upper bound 分开报告。新增数据/RL 若只改善一个视觉数学榜单，不能自动推广成通用 VLM 提升。

## Papers on Training and Understanding

### Foundations with useful ablations

**Prismatic VLMs: Investigating the Design Space of Visually-Conditioned Language Models**. Karamcheti et al. ICML'24  
Controlled study of optimization, visual representation, LLM backbone, and training data.  
Single-stage connector + LLM training can replace explicit alignment in their setting.  
Fine-tuning ViT hurts under their data budget; fused DINOv2 + SigLIP features help.  
Insight: valuable ablation baseline, with conclusions tied to LLaVA-scale training.  
<https://proceedings.mlr.press/v235/karamcheti24a.html>

**MM1: Methods, Analysis & Insights from Multimodal LLM Pre-training**. McKinzie et al. 2024  
Systematic ablations of encoder, resolution, visual-token budget, connector, and data mix.  
Caption data supports zero-shot performance; interleaved and text-only data are important for few-shot / language ability.  
Encoder/resolution/token count are more consequential than connector variants in their comparison.  
Insight: data ratios must be optimized against the intended evaluation mode.  
<https://arxiv.org/abs/2403.09611>

**Cambrian-1: A Fully Open, Vision-Centric Exploration of Multimodal LLMs**. Tong et al. 2024  
Vision-centric evaluation, multiple visual encoders, spatial vision aggregation, and a curated instruction mixture.  
Insight: inspect visually demanding tasks separately; benchmark averages can obscure the vision bottleneck.  
<https://arxiv.org/abs/2406.16860>

**Eagle: Exploring The Design Space for Multimodal LLMs with Mixture of Encoders**. Shi et al. ICLR'25, Spotlight  
Compare complementary vision experts, resolutions, and fusion strategies.  
Simple concatenation is competitive with more complex mixing; expert pre-alignment helps language compatibility.  
Insight: test complementary representations before adding elaborate fusion.  
<https://openreview.net/forum?id=Y2RW9EVwhT>

**SigLIP 2: Multilingual Vision-Language Encoders with Improved Semantic Understanding, Localization, and Dense Features**. Tschannen et al. 2025  
Sigmoid image-text loss + decoder-based caption/localization objectives + self-distillation/masked prediction, with staged training.  
Native-aspect-ratio / variable-resolution variants; improved dense features and downstream VLM transfer.  
Insight: the ViT itself has a training recipe; global contrastive alignment is only one objective.  
<https://arxiv.org/abs/2502.14786>

### ICML 2025: reasoning transfer and training dynamics

**Bring Reason to Vision: Understanding Perception and Reasoning through Model Merging**. Chen et al. ICML'25  
Merge text reasoner weights into compatible VLMs; transfer reasoning without new training.  
In their analysis, perception concentrates more in early layers, reasoning in middle-to-late layers.  
Insight: reasoning transfer and preservation of visual interfaces deserve separate treatment; same-source backbone compatibility matters.  
<https://proceedings.mlr.press/v267/chen25cm.html>

**Diving into Self-Evolving Training for Multimodal Reasoning**. Liu et al. ICML'25  
M-STaR: study training method, reward model, and prompt variation in iterative self-training.  
Automatic balancing targets training saturation.  
Insight: self-generated CoT quality alone is insufficient; selection, rewards, and diversity shape the learning loop.  
<https://proceedings.mlr.press/v267/liu25aj.html>

**Generalizing from SIMPLE to HARD Visual Reasoning: Can We Mitigate Modality Imbalance in VLMs?**. Park et al. ICML'25  
Controlled Table Readout / Grid Navigation / Visual Analogy tasks with matched text versions.  
Image-to-text conversion helps simple-to-hard transfer; the conversion can be internalized. Gradient alignment is studied.  
Insight: matched visual/text tasks can distinguish a visual access problem from a reasoning problem.  
<https://proceedings.mlr.press/v267/park25j.html>

**Perception in Reflection**. Wei et al. ICML'25  
RePer + reflective preference learning for improved fine-grained perception and hallucination reduction.  
Insight: reflection should revisit visual evidence, rather than only elaborate an existing textual interpretation.  
<https://proceedings.mlr.press/v267/wei25r.html>

**MME-CoT: Benchmarking Chain-of-Thought in Large Multimodal Models for Reasoning Quality, Robustness, and Efficiency**. Jiang et al. ICML'25  
Evaluate reasoning quality, robustness, and efficiency across six domains.  
CoT prompting can degrade perception-heavy tasks; reflective models can be inefficient.  
Insight: monitor short visual questions and answer cost while training longer reasoning.  
<https://proceedings.mlr.press/v267/jiang25n.html>

### NeurIPS 2025: long video and RL

**Eagle 2.5: Boosting Long-Context Post-Training for Frontier Vision-Language Models**. Chen et al. NeurIPS'25  
Information-first sampling, progressive mixed post-training, Eagle-Video-110K with clip/video annotations.  
Preserve high-resolution evidence when packing image/video inputs; gradually extend context while retaining short examples.  
Insight: long-context schedule and evidence allocation must be designed together.  
<https://neurips.cc/virtual/2025/poster/117551>  
<https://arxiv.org/abs/2504.15271>

**VL-Rethinker: Incentivizing Self-Reflection of Vision-Language Models with Reinforcement Learning**. Wang et al. NeurIPS'25  
GRPO + Selective Sample Replay for vanishing advantages; forced rethinking in subsequent RL.  
Direct reasoning RL from an instruction-tuned VLM, without teacher CoT distillation in the reported method.  
Insight: better RL signal does not necessarily elicit reflection by itself; rollout design also matters.  
<https://neurips.cc/virtual/2025/poster/119918>  
<https://arxiv.org/abs/2504.08837>

### ICLR 2026: visual grounding of post-training

**Vision-R1: Incentivizing Reasoning Capability in Multimodal Large Language Models**. Huang et al. 2025 preprint; ICLR'26 **（accepted version: abstract-level）**  
200K multimodal cold-start CoT through modality bridging + filtering, followed by GRPO.  
Progressive Thinking Suppression Training addresses overthinking after cold start.  
Insight: cold-start traces and thinking-length optimization can be useful, but are separate from perception learning.  
<https://openreview.net/forum?id=UZIjskfbfU>  
<https://arxiv.org/abs/2503.06749>

**Perception-R1: Advancing Multimodal Reasoning Capabilities of MLLMs via Visual Perception Reward**. Xiao et al. ICLR'26 **（abstract-level）**  
Visual references extracted from CoT annotations; judging model rewards agreement with visual facts.  
Insight: add supervision for seeing correctly, beyond final-answer correctness; reference/judge reliability remains part of the recipe.  
<https://openreview.net/forum?id=KttCXdjj4w>

**Perception-Aware Policy Optimization for Multimodal Reasoning**. Wang et al. ICLR'26 **（abstract-level）**  
PAPO: original/corrupted image distributions over the same rollout → implicit perception KL loss + entropy regularization.  
No additional human annotations or teacher required in the reported method.  
Insight: visual dependence can be encouraged through the objective; corruption choice and optimization stability matter.  
<https://openreview.net/forum?id=izbBqTL8vb>

**VTool-R1: VLMs Learn to Think with Images via Reinforcement Learning on Multimodal Tool Use**. Wu et al. ICLR'26 **（abstract-level）**  
RL over Python visual editing tools; interleave text and intermediate visual observations.  
Outcome rewards; no process supervision required in the reported method.  
Insight: learn to acquire/manipulate evidence; results focus on structured charts/tables.  
<https://openreview.net/forum?id=Idst6X6gmy>

**Scaling Large Vision-Language Model RL Training via Efficient Load Balancing**. Wang et al. ICLR'26 **（abstract-level）**  
FlexRL: distributed metadata-driven visual loading + adaptive subsequence sharding.  
Insight: RL recipe includes input preprocessing and workload distribution, especially for heterogeneous image/video contexts.  
<https://openreview.net/forum?id=aa14rlfR6k>

### ICML 2026: visual necessity and capability curricula

**Focusing Where Vision Matters: Selective Training for Large Vision Language Models via Visual Information Gain**. Lee et al. ICML'26 **（abstract-level）**  
VIG measures the image's reduction of prediction uncertainty at sample/token granularity.  
Selective training favors visually informative data/tokens.  
Insight: visual necessity can be a training-selection criterion, rather than only an evaluation control.  
<https://openreview.net/forum?id=z1bSFIEexL>  
<https://icml.cc/virtual/2026/poster/60557>

**From Seeing to Thinking: Decoupling Perception and Reasoning Improves Post-Training of Vision-Language Models**. Wu et al. ICML'26 **（abstract-level）**  
Separate stages for visual perception, visual reasoning, and textual reasoning, with specialized data.  
Staged training outperforms merged training in their experiments; perception RL is effective.  
Insight: capability-based curricula are another axis besides problem difficulty.  
<https://openreview.net/forum?id=r7uOjvZdzO>  
<https://icml.cc/virtual/2026/poster/61345>

### NeurIPS 2026: data and alignment

The following works are listed in the official NeurIPS'26 paper program. **Accepted; conference upcoming as of 2026-11-04.** Findings below are based on official abstracts, not a full-paper reproduction review.

**DataComp-VLM: Improved open datasets for Vision-Language Models**. Farina et al. NeurIPS'26 **（abstract-level）**  
Standardize 160 datasets / 6T corpus tokens; controlled experiments at 1B–8B models and 6.25B–200B training tokens.  
Authors report instruction-heavy mixtures scale better than caption-heavy mixtures; benchmark mixing/filtering/formatting/sampling strategies.  
Insight: a benchmark for a reproducible data recipe, rather than a universal fixed data mixture.  
<https://openreview.net/forum?id=s8J0YiyTuN>  
<https://neurips.cc/virtual/2026/poster/138991>

**Scaling Laws for Multimodal Data Mixtures**. Khandelwal et al. NeurIPS'26 **（abstract-level）**  
268 MoE experiments, 457M–8.3B parameters, up to 150B tokens; study text/vision/audio mixing.  
Modality-specific compute optima; asymmetric cross-modal interference; limited gains from repeating vision/audio beyond 2× in their setting.  
Insight: fit mixtures under compute constraints; the reported repeat threshold is not a general law for all VLM SFT.  
<https://openreview.net/forum?id=xX1gcyTLbF>  
<https://neurips.cc/virtual/2026/poster/148475>

**Rethinking Projector Training in Multimodal LLMs**. Gröger et al. NeurIPS'26 **（abstract-level）**  
Most gains recovered with a fraction of projector data; projectors can be interpolated across datasets.  
PORTAL: statistics + optimal-transport initialization, skipping paired-caption projector pretraining.  
Insight: early projector training may mainly provide coarse feature-space compatibility.  
<https://openreview.net/forum?id=HObX41utGv>  
<https://neurips.cc/virtual/2026/poster/153944>

**Rosetta: Composable Native Multimodal Pretraining**. Liu et al. NeurIPS'26 **（abstract-level）**  
Global shared / modality-specific experts + momentum-anchored orthogonal gradient projection.  
Transfusion-style training with text NTP and visual diffusion; investigate forgetting under modality expansion.  
Insight: understanding + generation introduces additional objective conflicts; this recipe has a different scope from image-to-text VLM SFT.  
<https://openreview.net/forum?id=X7davuwy7B>  
<https://neurips.cc/virtual/2026/poster/151897>

## Open Model Reports, 2025–2026

### 2025: multimodal joint training and reasoning post-training

**Qwen2.5-VL Technical Report**. Qwen Team. 2025  
Custom ViT with native dynamic resolution, window attention, 2D-RoPE; unified image/video representation with temporal downsampling.  
Visual pretraining → full multimodal training → long-context pretraining; SFT → DPO with frozen ViT.  
Data: caption, interleaved documents, OCR, visual knowledge, STEM, grounding, video, agent trajectories; text-only data included.  
Read: Sections 2.2/2.3 and Table 2 for trainable components, context lengths, and data composition.  
Open weights / inference and fine-tuning code; full pretraining corpus is not provided by the report.  
<https://arxiv.org/abs/2502.13923>

**InternVL3: Exploring Advanced Training and Test-Time Recipes for Open-Source Multimodal Models**. Zhu et al. 2025  
Native multimodal pretraining with text-only + visual data; all modules jointly updated.  
~200B training tokens; language:multimodal = 1:3 in the reported best mixture.  
Square averaging for response-length balance; SFT → Mixed Preference Optimization.  
Additional GUI/tool/3D/video data; variable visual position encoding for long contexts.  
Read: Section 2.2 for loss, normalization, sampling; Section 2.3 for MPO.  
<https://arxiv.org/abs/2504.10479>

**InternVL3.5: Advancing Open-Source Multimodal Models in Versatility, Reasoning, and Efficiency**. Wang et al. 2025  
Cascade RL: offline MPO warm-up → online GSPO.  
MMPR preference data and pass-rate filtering for online RL; dense/MoE scales.  
Flash variant: Visual Consistency Learning → train Visual Resolution Router, choosing visual compression adaptively.  
Read: Section 2.3 and Section 3.15 ablations; distinguish training improvements from parallel thinking / deployment speedups.  
<https://arxiv.org/abs/2508.18265>

**Eagle 2: Building Post-Training Data Strategies from Scratch for Frontier Vision-Language Models**. 2025  
“Diversity first, then quality”: 180+ sources, rule/model filtering, subset balancing, clustering and augmentation.  
Connector alignment → full-model diverse training → full-model high-quality instruction tuning.  
Balanced packing; transparent development ablations and data-source lists.  
Read together with NeurIPS'25 Eagle 2.5 for progressive long-context training.  
<https://arxiv.org/abs/2501.14818>  
<https://github.com/NVlabs/Eagle>

**LLaVA-OneVision-1.5: Fully Open Framework for Democratized Multimodal Training**. LLaVA-OneVision Community. 2025  
RICE-ViT + Qwen3; 558K alignment → 85M concept-balanced mid-training → instruction tuning.  
22M instruction set plus FineVision; full-parameter mid-training/SFT.  
Offline parallel packing, base/instruct checkpoints, data and training code; useful reproduction starting point.  
“From scratch” here refers to building the VLM with pretrained vision/language backbones, not random-initializing the LLM.  
Reported dollar cost depends on hardware/rental assumptions and is not a universal budget estimate.  
<https://arxiv.org/abs/2509.23661>  
<https://github.com/EvolvingLMMs-Lab/LLaVA-OneVision-1.5>

**GLM-4.5V and GLM-4.1V-Thinking: Towards Versatile Multimodal Reasoning with Scalable Reinforcement Learning**. GLM Team. 2025  
AIMv2-based vision encoder + GLM; native resolution and video; knowledge-rich pretraining + domain-specific reasoning SFT.  
RL with Curriculum Sampling across math, OCR, grounding, video and other tasks.  
Insight: reward system correctness and dynamic difficulty selection are central to multi-domain RL.  
Base/thinking weights and VLM reward-system components released. The arXiv report has later versions; here training claims refer to the inspected v1.  
<https://arxiv.org/abs/2507.01006>  
<https://github.com/zai-org/GLM-4.1V-Thinking>

**MiniCPM-V 4.5: Cooking Efficient MLLMs via Architecture, Data, and Training Recipe**. Yu et al. 2025  
Warm-up resampler → unfreeze ViT with OCR/caption data while LLM frozen → full joint training with text/interleaved/video data.  
3D-Resampler compresses spatial/temporal tokens; package size/FPS augmentation during training.  
Document text corruption/reconstruction jointly teaches OCR and contextual knowledge.  
Hybrid short/long reasoning RL; RLAIF-V for alignment.  
Read: Section 2.2 for curriculum, 2.4 for RL, and architecture/data ablations.  
<https://arxiv.org/abs/2509.18154>

**SmolVLM: Redefining small and efficient multimodal models**. Hugging Face Team. 2025  
Compact VLM ablations: encoder/LLM balance, aggressive pixel shuffle, learned image-position tokens, query masking, text/CoT mixtures.  
Excessive CoT data or unsuitable LLM-SFT text mixtures can hurt small VLMs.  
Vision training → video training; small-model recipe depends strongly on capacity and token overhead.  
Read: Sections 2/3, particularly negative results and scale-specific effects.  
<https://arxiv.org/abs/2504.05299>  
<https://github.com/huggingface/smollm/tree/main/vision>

**Gemma 3 Technical Report**. Gemma Team. 2025  
Frozen adapted SigLIP encoder; pooled 256 visual tokens/image; Pan & Scan for aspect ratio / high resolution.  
Multimodal pretraining with knowledge distillation; late context extension; post-training distillation + RL.  
Insight: frozen image embeddings enable efficient training; full ViT updates are not required by every successful recipe.  
<https://arxiv.org/abs/2503.19786>

### 2026: native training, open video grounding, and visual compute

**Molmo2: Open Weights and Data for Vision-Language Models with Video Understanding and Grounding**. Ai2 Team. Report: 2026-01-15; model/data release began in 2025  
Caption/pointing/NLP full-parameter pretraining → multitask SFT → short long-context SFT.  
Pretraining mix: 60% caption, 30% image pointing, 10% NLP; SFT dataset-size square-root sampling plus manual category balancing.  
Pointing/tracking supervision, human-rich dense video captions, bidirectional visual attention, message-tree packing, global loss-token normalization.  
Weights at intermediate stages, data and training scripts released. Data uses their open-model pipelines; “no closed VLMs” does not mean “no synthetic data.”  
Read: Sections 2/3/5 and Appendix A/B; the most directly usable recipe for video grounding in this list.  
<https://arxiv.org/abs/2601.10611>  
<https://github.com/allenai/molmo2>

**Kimi K2.5: Visual Agentic Intelligence**. Kimi Team. 2026  
Near-end K2 initialization; native-resolution MoonViT-3D, shared image/video weights, temporal compression.  
ViT adaptation with caption cross-entropy → joint text/vision pretraining → high-quality long-context mid-training.  
Controlled early/mid/late vision-injection ablation favors early low-ratio fusion at fixed total budgets.  
Zero-vision text SFT activates visual tool use; joint text/vision RL; PARL trains the swarm orchestrator with frozen subagents.  
Read: Sections 2.1/2.2/4.3; avoid interpreting zero-vision SFT as zero-vision training overall.  
Open post-trained weights; the full training corpus/environment is not released in the report.  
<https://arxiv.org/abs/2602.02276>

**Kimi K3: Open Frontier Intelligence**. Kimi Team. 2026, official repository report  
Native joint pretraining; MoonViT-V2 from scratch using NTP; stable visual optimization without contrastive initialization.  
Pretraining 8K → 64K; cooldown 256K → 1M; synthesize tasks requiring scattered long-context evidence.  
Per-Head Muon, cosine schedule with 1% warm-up, weight decay 0.1; independent hyperparameter search when comparing cosine/WSD.  
SFT → specialized domain/effort RL → multi-teacher on-policy distillation; QAT from SFT onward.  
Scale up programmatic visual-code pairs: SVG, webpage, game, CAD/3D assets.  
Read: Sections 2.4/3/4.1. Report describes frontier-scale choices; corpus ratios/total training tokens are not fully specified in those sections.  
<https://github.com/MoonshotAI/Kimi-K3>  
<https://github.com/MoonshotAI/Kimi-K3/blob/main/k3_tech_report.pdf>

**LLaVA-OneVision-2: Towards Next-Generation Perceptual Intelligence**. Glint Lab / LLaVA-OneVision Community. 2026  
OneVision-Encoder shared across images, sampled frames, and codec-selected evidence.  
Video bootstrap → image/video instruction tuning → long-video training → denser codec/spatial/tracking refinement.  
Codec-stream tokens follow bit-cost / motion-residual evidence; mixed codec/frame input curriculum.  
Useful counterexample to uniform-frame-only video recipes; authors discuss static/fine-detail tasks where frame sampling remains preferable.  
Read: Sections 2/4/8; verify current released subsets/checkpoints against the repo rather than assuming all announced assets are downloadable.  
<https://arxiv.org/abs/2605.25979>  
<https://github.com/EvolvingLMMs-Lab/LLaVA-OneVision-1.5>

**OneVision-Encoder: Codec-Aligned Sparsity as a Foundational Principle for Multimodal Intelligence**. 2026  
Encoder-level codec patchification and shared spatiotemporal coordinates.  
Insight: evidence selection can be designed jointly with the encoder; downstream VLM training inherits this input distribution.  
<https://arxiv.org/abs/2602.08683>

**Gemma 4 Technical Report**. Gemma Team. Report: 2026-07-02  
Most variants: adapted frozen ViT with variable aspect ratio and visual-token budgets.  
12B variant: encoder-free raw image patches / audio chunks through lightweight projections.  
Thinking mode, context-efficient attention, post-training following Gemma3 with updated data/behaviors.  
Insight: native multimodal does not imply one encoder strategy; encoder-free is promising but not a settled VLM standard.  
<https://arxiv.org/abs/2607.02770>

### Latest release checks, as of 2026-11-04

Detailed older reports are often more useful for training than a newer release card. The following checks keep “latest” separate from “fully documented recipe.”

| Family / verified latest item | Primary source | Training relevance / caveat |
|---|---|---|
| Qwen3-VL report; Qwen3.5 → 3.6 → **3.8** releases | [Qwen official repo](https://github.com/QwenLM/Qwen3.5), [Qwen3-VL report](https://arxiv.org/abs/2511.21631) | Qwen3.5 release on 2026-02-16 describes trillion-token early-fusion multimodal training; Qwen3.8 cards/release logs are newer but disclose less of the complete recipe |
| **Qwen3.8-27B**, released 2026-08-14 | [Official model card](https://huggingface.co/Qwen/Qwen3.8-27B) | Open native image/video model. Do not confuse it with the flagship checkpoint below |
| **Qwen3.8-2.4T-A95B**, released 2026-08-12 | [Official model card](https://huggingface.co/Qwen/Qwen3.8-2.4T-A95B) | Released checkpoint is a causal language model; card reserves vision support as an additional capability of the hosted Qwen3.8-Max version |
| **Kimi K3** | [Official repo/report](https://github.com/MoonshotAI/Kimi-K3) | Latest verified Kimi item in this search; complete PDF provides more than the release summary |
| **GLM-5.3-Flash**, released 2026-08-26 | [GLM-V official repo](https://github.com/zai-org/GLM-4.1V-Thinking) | Official news describes native multimodality. Use GLM-4.1V report/reward release for detailed RL methodology; do not infer an unchanged recipe |
| **MiniCPM-V 4.7**, released 2026-10-12; OCR suite 2026-10-28 | [Official repo](https://github.com/OpenBMB/MiniCPM-V), [4.7 details](https://github.com/OpenBMB/MiniCPM-V/blob/main/docs/minicpm_v4dot7_en.md) | Mixed 4×/16× visual compression, intra-ViT early compression, deployment and self-replay fine-tuning. MiniCPM-V 4.5 remains the inspected full training report |
| **LLaVA-OneVision-2** | [Official repo](https://github.com/EvolvingLMMs-Lab/LLaVA-OneVision-1.5), [2026 report](https://arxiv.org/abs/2605.25979) | Repo has evolved beyond its 1.5 name; check report/model/data versions separately |

Latest here means **latest verified in the inspected primary sources**, not a claim that every release worldwide has been exhaustively covered. No unsupported “current SOTA” ranking is intended.

## Suggested Recipes and Research Priorities

### Starting from an existing VLM checkpoint

**Suggested recipe — synthesis, not an experimentally universal configuration.**

1. **先定义视觉错误和评估集。** 区分 OCR/grounding/temporal perception、reasoning、format/tool use；建立 target-domain holdout、text-retention tests，以及适用的 no-image / oracle-perception controls。固定 visual budget 和 generation settings。
2. **延续该 checkpoint 的 processor/interface。** 保持 crop/FPS/timestamps、coordinate format、chat template、special tokens 和 label mask 一致。先确认数据管线能准确保留视觉证据。
3. **用 task-balanced SFT 建 baseline。** 优先目标任务的可信标注，加入适量 general VL / text replay；控制重复、同图多 QA、long-answer 占比。已有完整 VLM 通常没有重新做 generic projector alignment 的理由。
4. **根据错误决定解冻。** 可先训练 LLM adapters / connector、冻结 ViT；若可观测的 perception bottleneck 仍在，再比较低 LR ViT 更新或专门视觉监督。LoRA 是预算选择，不能视作 frontier full-training 的替代标准。
5. **Reasoning/tool use 再做有目的的后训练。** 已有成功 rollouts 可直接 RL；无基本 protocol/correct traces 时先 cold-start SFT。可验证任务采用 outcome rewards；视觉依赖不足时考虑 visual necessity filtering、perception reward 或 counterfactual objectives。
6. **长 context 最后按需求扩展。** 混合短/长样本，在真正需要跨页/跨时间证据的任务上训练；记录实际 input/output tokens 和训练成本。只需短图任务时，无需复制基础模型的长 context pipeline。

Basis: [Molmo2](https://arxiv.org/abs/2601.10611), [Qwen3-VL](https://arxiv.org/abs/2511.21631), [Prismatic](https://arxiv.org/abs/2402.07865), [VL-Rethinker](https://arxiv.org/abs/2504.08837), [Perception-R1](https://openreview.net/forum?id=KttCXdjj4w), [PAPO](https://openreview.net/forum?id=izbBqTL8vb).

### Building a VLM with pretrained vision / language backbones

**Suggested recipe — start from a released end-to-end pipeline, then ablate.**

1. **选择一条可复现 baseline。** Image/general instruction 可从 LLaVA-OneVision-1.5 或 Eagle 开始；video pointing/tracking 可从 Molmo2 开始。复现它的 data format、loss、packing 和 eval，而不是拼接各报告的最佳单项超参。
2. **短 alignment 或 staged LR warm-up。** 新 connector 先建立基本映射；比较显式 warm-up 与 all-parameter training + differential LR，不预设前者必要。
3. **投入主要预算到 joint multimodal data。** 覆盖 caption/OCR/document/grounding/multi-image/video/knowledge 和 text-only；依据任务平衡 source，并记录 supervised-token/FLOP contribution。
4. **用高质量 instruction / reasoning data 激活能力。** 教 output schema、视觉引用、拒答/缺失目标、tool protocol；针对看图必要的题目做过滤，避免图像只成为装饰。
5. **按需求扩展视频/context，随后 preference/RL/distillation。** 模块更新和 data mixture 在阶段间重新评估。Text retention、perception 和 cost 是共同约束。

Basis: [Eagle 2](https://arxiv.org/abs/2501.14818), [LLaVA-OneVision-1.5](https://arxiv.org/abs/2509.23661), [Molmo2](https://arxiv.org/abs/2601.10611), [Qwen3-VL](https://arxiv.org/abs/2511.21631).

**若有从头预训练的 frontier 预算**，才进一步比较 early native fusion、随机视觉初始化、encoder-free、MoE/attention 形状与 modality mixing scaling laws。Kimi K3 的 cosine/Per-Head Muon 与 MiniCPM 的 WSD 都有各自条件；optimizer、scheduler、LR、batch size 需要联合调，不存在可跨规模直接搬运的最佳配置。  
<https://github.com/MoonshotAI/Kimi-K3/blob/main/k3_tech_report.pdf>  
<https://arxiv.org/abs/2607.02770>  
<https://openreview.net/forum?id=xX1gcyTLbF>  
<https://arxiv.org/abs/2509.18154>

### What looks settled, and what does not?

| Question | Evidence-based answer | Practical implication |
|---|---|---|
| ViT–connector–LLM as a starting architecture? | Broadly established for understanding VLMs; encoder-free alternatives emerging | Use a proven interface unless architecture itself is the research question |
| Diverse multimodal data + some text retention? | Common practice; transfer/interference depends on scale and mixture | Optimize ratios against both VL and text objectives |
| Projector-only warm-up required? | No; successful explicit and implicit alternatives exist | Ablate necessity and cost |
| Freeze or train ViT? | Depends on data budget, task, initialization and phase | Diagnose perception errors and validate targeted updates |
| Caption-heavy or instruction-heavy? | Depends on stage, supervision density and budget; recent controlled mixing study favors instruction-heavy mixtures in its setting | Compare under fixed compute and fixed evaluation suite |
| SFT then RL? | Common for reasoning/agents; direct RL from a suitable existing VLM also works | Check rollout support before choosing cold start |
| GRPO as the universal RL algorithm? | No; GRPO-family, GSPO, SAPO, MPO and hybrids used | Reward/data/rollout stability often deserve first attention |
| Long CoT always helps? | No; perception-heavy and small-model negative results exist | Train/evaluate thinking budgets and concise modes |
| Native multimodal from scratch? | Increasingly important at large scale; continuation and frozen-encoder recipes remain competitive alternatives | Choose based on available compute and existing backbone assets |
| One fixed data ratio / LR / epoch count? | No cross-model consensus supported by these reports | Publish assumptions and retune coupled hyperparameters |

Evidence: the comparative recipes and cited ablations above. This table summarizes the inspected literature, not an external standard.

### Experiments worth doing first

**My priorities for a training-focused VLM project.**

- **Visual necessity:** baseline mix vs no-image filtering / visual-gain selection, with matched compute; inspect gains on image-dependent questions.
- **Loss accounting:** token mean vs length-weighted loss, with correct global normalization; inspect both short answers and dense captions/point outputs.
- **Perception before reasoning:** merged training vs staged perception → reasoning, while keeping total data and training budget comparable.
- **Encoder adaptation:** frozen vs lower-LR trainable ViT, holding visual resolution/token count fixed; inspect OCR/grounding and text retention.
- **Evidence budget:** more spatial detail vs more temporal observations vs adaptive compression, at the same total visual tokens.
- **RL signal:** track non-zero-advantage groups, reward disagreement, success rate, reasoning length and tool calls; compare data/reward changes before attributing gains to an algorithm name.

These experiments follow the open questions in [Prismatic](https://arxiv.org/abs/2402.07865), [Molmo2](https://arxiv.org/abs/2601.10611), [VIG](https://openreview.net/forum?id=z1bSFIEexL), [ICML'26 staged post-training](https://openreview.net/forum?id=r7uOjvZdzO), [VL-Rethinker](https://arxiv.org/abs/2504.08837), and [LLaVA-OneVision-2](https://arxiv.org/abs/2605.25979).

## Reproduction Pointers

| Resource | Useful assets / purpose |
|---|---|
| [Molmo2](https://github.com/allenai/molmo2) | Training scripts, data, pretrain/SFT/long-context checkpoints, custom packing/message-tree implementation |
| [LLaVA-OneVision-1.5 / 2](https://github.com/EvolvingLMMs-Lab/LLaVA-OneVision-1.5) | Curated mid-training/instruction data, base/instruct models, training code and versioned release information |
| [Eagle](https://github.com/NVlabs/Eagle) | Encoder design and data strategy; Eagle2.5 code and long-context training docs |
| [Qwen3-VL](https://github.com/QwenLM/Qwen3-VL) | Processor, image/video handling, fine-tuning examples, model variants; read report for foundation training stages |
| [GLM-V](https://github.com/zai-org/GLM-4.1V-Thinking) | Base/thinking model history, VLM reward-system release, later native-model news |
| [MiniCPM-V](https://github.com/OpenBMB/MiniCPM-V) | Efficient visual representation, deployment/fine-tuning docs and newer model versions |
| [SmolVLM training](https://github.com/huggingface/smollm/tree/main/vision) | m4/experiments training code and compact-model design |
| [FineVision](https://huggingface.co/datasets/HuggingFaceM4/FineVision) | Public consolidated instruction datasets; inspect source-specific schemas and splits |
| [Prismatic](https://github.com/TRI-ML/prismatic-vlms) | Controlled architecture/optimization ablations and training implementation |

**Reading limitation.** Full HTML/PDF training text was inspected for the principal reports and older ablation papers. Several 2026 conference works were accessible here only through official abstracts; these are explicitly labeled. The note does not assert a verified citation-count ordering, reproduce training runs, or treat author-reported benchmark improvements as a same-budget leaderboard.
