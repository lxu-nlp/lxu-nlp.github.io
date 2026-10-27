---
layout: post
title: "Notes on Training Parallelism"
date: 2026-10-27
categories: [Notes]
tags: [optimization, ML]
math: true
---

## 0. 一分钟定位

并行的一切设计都可以归到一个问题：**把什么切开，以及用什么通信把它补回来**。这篇讲原理、术语与框架选型；显存侧（model state 16 B/param、ZeRO/FSDP 各 stage 切什么、FSDP 的 all-gather 生命周期与 Root/Leaf 显存陷阱）详见 [Note on Training](https://lxu-nlp.github.io/posts/note-on-training) 的第 1 节。MoE 侧的最小术语集见 [Note on LLM MOE](https://lxu-nlp.github.io/posts/note-on-moe)。

- [1. 并行维度总表](#1-并行维度总表)
- [2. 通信原语](#2-通信原语)
- [3. 组合与 Parallel Folding](#3-组合与-parallel-folding)
- [4. MoE 侧的并行](#4-moe-侧的并行)
- [5. 框架对比](#5-框架对比)
- [6. 怎么选](#6-怎么选)
- [7. 常见误解](#7-常见误解)

---

## 1. 并行维度总表

| 缩写 | 全称 | 切什么 | 主要通信 | 解决什么 |
|---|---|---|---|---|
| DP / DDP | Data Parallelism | batch（每卡一整份模型） | all-reduce 梯度 | 吞吐 |
| TP | Tensor Parallelism | 张量维度（head 数、intermediate 维） | all-reduce（或前向 all-gather + 反向 reduce-scatter） | 单层放不下 |
| SP | Sequence Parallelism | 序列维的**激活**（与 TP 配套，切 layernorm / dropout / 残差这些 TP 没切到的部分） | all-gather / reduce-scatter | 省激活显存 |
| CP | Context Parallelism | 序列 / context（注意力跨卡取 KV，如 ring attention） | 环状 P2P | 超长上下文 |
| PP | Pipeline Parallelism | 层（按 stage 切，micro-batch 流水） | P2P 传激活 | 层数太多放不下 |
| **EP** | **Expert Parallelism** | **专家（按整数个切）** | **all-to-all 传 token** | MoE 专家权重放不下 |
| **ETP** | **Expert Tensor Parallelism** | **专家内部的张量维度** | all-gather + reduce-scatter | 单个专家放不下（**一般设 1**） |
| EDP | Expert Data Parallelism | MoE 侧的 batch（复制专家换吞吐） | all-reduce | 提吞吐 |
| ZeRO / FSDP | — | 参数 / 梯度 / 优化器状态（**不切 activation**） | all-gather + reduce-scatter | 省 model state 显存 |

三条最容易混的：

- **TP vs EP**：TP 切**矩阵维度**，每卡只持有权重的一部分，靠 all-reduce 把输出补回完整张量；EP 切**"专家"这个整数单位**，矩阵不动，但 token 要搬到目标卡再搬回来。
- **SP vs CP**：都跟序列有关但不是一个东西。SP 是 TP 的附属品（切 TP 没切到的激活）；CP 是独立的长上下文方案（切序列本身，注意力要跨卡取 KV）。文献里两者经常被混叫，看上下文判断。
- **ZeRO/FSDP 不在"切计算"这一族**：它只切 model state（参数/梯度/优化器状态），**每卡的 forward 仍然要临时拼回完整权重算完整的一层**。所以它解决的是"模型放不下"，不是"一层算不动"。

---

## 2. 通信原语

| 原语 | 语义 | 谁在用 |
|---|---|---|
| **all-reduce** | 所有人求和，且每个人都拿到完整结果 | DP 的梯度同步、TP 的输出合并 |
| **all-gather** | 把各卡的分片拼成完整张量，每卡都拿到全量 | FSDP 前向取参数、TP 的 sequence parallel |
| **reduce-scatter** | 归约后只把结果拆开、每卡留自己那片 | FSDP 反向的梯度、ZeRO |
| **all-to-all** | 每个人给每个人发**不同的一块** | MoE 的 token dispatch / combine |
| **P2P send/recv** | 点对点 | PP 相邻 stage 传激活 |

一句话记忆：**DP/TP 是"大家算同一件事的不同部分，最后合并"（collective 归约类）；EP 是"大家算不同的东西，所以数据要重新分发"（all-to-all）**。

---

## 3. 组合与 Parallel Folding

传统布局强制 attention 层和 MoE 层共用一套并行配置，副作用是 **EP 不超过 DP**（EP 组只能从 DP 组里切出来）。

Megatron-Core 的 **Parallel Folding** 把两者解耦：

- attention 层用 $\text{TP}\times\text{CP}\times\text{DP}\times\text{PP}$
- MoE 层用 $\text{ETP}\times\text{EP}\times\text{EDP}\times\text{PP}$
- **唯一约束**：PP 的分组与成员在两侧必须一致，否则梯度流断掉

收益举例：attention 配 TP=4 / CP=2 / DP=8 共 256 卡，传统布局下 EP 最多 8；折叠后 MoE 侧可以做到 EP=64、ETP=1、EDP=1，EP 提升 8 倍，而 attention 仍保持自己的最优配置。另一个副作用是**降低最小卡数**：CP=8 且 EP=8 的传统布局至少要 64 卡，折叠后 CP 与 EP 共用同一组 GPU，8 卡就够。

配套的业界惯例：

- **EP 尽量留在节点内**（NVLink 域）。跨节点 IB 会显著放大 all-to-all 开销；要跨节点就得靠 DualPipe 这类通信重叠。
- **ETP 一般设 1**。NVIDIA 调优指南原话是避免对 MLP GEMM 施加 TP 带来的显著通信开销；MoE 的专家本来就窄，再切一次张量并行得不偿失。
- **MoE 块内没有 CP**：token 已按专家重排，序列维失去意义，Megatron 在 MoE 块内用 DP 取代 CP。

---

## 4. MoE 侧的并行

- **变长 all-to-all**：MoE 的 dispatch / combine 是**变长**的——每个专家收到的 token 数由路由决定、运行时才知道，且各卡不均。这比定长 all-to-all 难优化得多，是 MoE 通信瓶颈的根因。
- **permute / un-permute**：all-to-all 前后把 token 按专家编号排序 / 还原，让同一专家的 token 在内存里连续，才能喂给 GEMM。
- **node-limited routing**（DeepSeek-V3，每 token 最多发 $M=4$ 个节点）：主动牺牲一点路由自由度换跨机通信量下降。
- **DualPipe**：双向流水调度，让 all-to-all 与 forward / backward 的计算重叠。
- **Grouped GEMM**：每个专家只处理分给自己的那批 token，于是"一个大的矩阵乘"变成"每个专家一个小矩阵乘"，且各专家的 token 数不一样多、但权重形状完全相同。Grouped GEMM 把它们塞进一次 kernel launch 里算完。DeepGEMM 只在"token 数"这一轴分组正是吃准了权重形状一致这一点（CUTLASS 的通用 grouped GEMM 允许三个维度都变，但配置开销更大）；它提供 contiguous（prefill / 训练前向）与 masked（decode，可进 CUDA graph）两种布局。维度记法（M / N / K）与完整算子清单见 [Note on LLM MOE](https://lxu-nlp.github.io/posts/note-on-moe) 的第八节。

---

## 5. 框架对比

| | **Megatron-Core** | **FSDP2（PyTorch 原生）** | **DeepSpeed ZeRO** | **HF Trainer / Accelerate** |
|---|---|---|---|---|
| 本质 | 一整套并行栈，**含自己的模型实现** | 分片式数据并行（等价 ZeRO-3） | 分片式数据并行 + offload | 上层封装，**不是并行实现** |
| 提供维度 | DP / TP / SP / CP / PP / EP / ETP / EDP | DP（TP/CP/SP 需外挂） | DP（ZeRO-1/2/3）+ MoE 的 EP | 取决于后端：DDP / FSDP / DeepSpeed |
| 切 activation | 是（TP / SP / CP） | **否**，只切 model state | 否 | 同后端 |
| 模型侵入 | 高（要用 Megatron 版模型定义） | 无（`fully_shard` 套任意 module） | 无 | 无 |
| 通信重叠 | 最强（fused kernel + pipeline schedule + DualPipe） | 中等（有 prefetch，但不如 Megatron 精细） | 一般 | 同后端 |
| 低精度 | TransformerEngine，FP8 一等公民 | 依赖 torch 原生 | 有 | 依赖后端 |
| 典型场景 | 千卡级预训练、MoE、超长上下文 | 微调 / 中小规模预训练、快速实验 | 同上 + 显存不够要 offload | 所有 HF 生态任务 |

### Megatron-Core（NVIDIA）

覆盖维度最全、通信与计算重叠做得最极致，是千卡级预训练和所有开源 MoE（DeepSeek / Qwen / Kimi）训练侧的事实标准。**代价是侵入性**：它不用 HF 的模型类，而是自己维护一份模型定义（以及配套的算子，如 TransformerEngine 的 FP8 kernel 与 grouped GEMM）。所以"换框架"往往意味着"换模型代码 + 换权重转换脚本"。

### FSDP2（PyTorch 原生）

零侵入，`fully_shard(module)` 套上就能用，与 HF Trainer / Accelerate 深度集成，是微调与中小规模预训练的默认选择。它**只做 DP 维度上的分片**，概念上等价于 ZeRO-3：参数 / 梯度 / 优化器状态各留 1/N，forward 前 all-gather 出完整权重、算完即释放，反向 reduce-scatter 梯度。

NVIDIA 调优指南给出的偏好条件值得记：**小模型 + 长序列时 FSDP 优于 TP+PP+DP**，因为参数的 all-gather 与梯度的 reduce-scatter 能被计算有效隐藏；但**activation 显存仍是 FSDP 场景的主要瓶颈**（它只分片 model state），而且需要足够大的 per-GPU activation 才能把 FSDP 的通信藏住。

FSDP 的分片语义、`forward_pre_hook` 生命周期、以及 Root（Embeddings / LM Head 全程常驻）带来的显存陷阱，详见 [Note on Training](https://lxu-nlp.github.io/posts/note-on-training)。FSDP1 与 FSDP2 的区别主要在底层表示：FSDP1 用 flat parameter 分组，FSDP2 按 per-parameter 的 DTensor 分片、不做 flatten 打包。

### DeepSpeed ZeRO

提出了 ZeRO 这套语言（stage 1/2/3），并额外提供 ZeRO-Offload / Infinity（把分片状态卸到 CPU / NVMe），以及 DeepSpeed-MoE 的专家并行和 Ulysses 序列并行。近两年在新项目里的存在感被 Megatron 与 FSDP2 分流，但 **"ZeRO-1/2/3 切什么"仍然是描述所有分片方案的通用词汇**。

### HF Trainer / Accelerate

不是并行实现，而是把上面几套后端包成 `TrainingArguments` 的几个开关（`fsdp` / `deepspeed` / 分布式启动）。选它就等于选了后端。

一个值得知道的新变化：transformers v5 的 `Trainer` **已经感知 TP / CP / SP**——`get_total_train_batch_size()` 里明确写了 $dp = world\_size / (tp \cdot cp \cdot sp)$，并注释 CP 走 ring attention（FSDP2 路径）、SP 走 ALST/Ulysses（DeepSpeed 路径）。但这些维度**不在 `TrainingArguments` 里**（已核对 v5 main，`training_args.py` 无 `tp_size` 等字段），而是从 `model._tp_size` 或 accelerate 的 `ParallelismConfig(tp_size=..., cp_size=..., sp_size=..., dp_shard_size=...)` 读取。**EP 则完全没有**——HF 参考实现连 grouped GEMM 都没有，是 `for expert_idx in expert_hit` 的 Python 循环。

---

## 6. 怎么选

- **模型放得下，只是想跑快点** → DDP。每步只有一次梯度 all-reduce，几乎与 backward 完全 overlap，比 FSDP 快 5–20%，且每卡都有完整模型、最好调试。
- **模型放不下，但激活还行** → FSDP2 / ZeRO-3。零侵入，先试试能不能跑起来。
- **激活也放不下（长序列 / 大模型）** → 需要 TP / SP / CP，FSDP 帮不上忙（它不切 activation）。此时上 Megatron，或者 FSDP2 + torch 原生的 DTensor TP/CP 组合。
- **MoE，且专家权重启用 EP** → Megatron（EP / ETP / EDP + Parallel Folding + grouped GEMM）。FSDP 只能按 DP 思路把专家权重分掉，拿不到 EP 的语义。
- **千卡以上、要把 MFU 抠到极致** → Megatron。通信重叠、fused kernel、FP8 支持都在这里。
- **只想在 HF 生态里做 SFT / LoRA** → Trainer + FSDP（或 DDP），别碰 Megatron。

---

## 7. 常见误解

1. **"FSDP 切了参数，所以激活也省了"** —— 没有。FSDP1 / FSDP2 都只 shard params / grads / optimizer states，**永不碰 activations**。管激活的正确工具是 activation checkpointing、FlashAttention，以及（超长 context）TP / SP / CP。
2. **"TP 和 EP 是一类东西"** —— 不是。TP 切维度、通信是归约类；EP 切整数个专家、通信是 all-to-all。两者的瓶颈完全不同（前者怕 all-reduce 带宽，后者怕变长 all-to-all 的延迟与不均衡）。
3. **"SP 就是 CP"** —— 不是，见第 1 节。文献里混叫很常见。
4. **"EP 开越大越好"** —— 不是。EP 越大，all-to-all 跨的节点越多；业界惯例是优先把 EP 留在 NVLink 域内，ETP 保持 1。
