---
layout: post
title: "Works on Sequence Modeling"
date: 2022-02-18
categories: [NLP]
tags: [nlp, language model]
math: true
---

This article is being updated.

Transformers-based **pre-LLM** architectures. LLMs and multilingual models are addressed in separate posts.


## Benchmark

**MEASURING MASSIVE MULTITASK LANGUAGE UNDERSTANDING**. Hendrycks et al. ICLR'21\
MMLU: 57 tasks (requiring knowledge, reasoning) across STEM, social sciences, etc.\
<https://arxiv.org/abs/2009.03300>

**BEYOND THE IMITATION GAME: QUANTIFY-ING AND EXTRAPOLATING THE CAPABILITIES OF LANGUAGE MODELS**. Srivastava et al. 2022\
Big-Bench: 204 harder tasks (requiring reasoning, math, knowledge) covering diverse topics, with 20\% interactive tasks.\
<http://arxiv.org/abs/2206.04615>

## Encoder-based

**Attention Is All You Need**. Vaswani et al. NIPS'17.\
The Transformers architecture.\
<https://arxiv.org/abs/1706.03762>

**BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding**. Devlin et al. NAACL'19\
BERT, the first pre-trained model on Transformers with Masked LM (global bidirectional context compared to vanilla auto-regressive LM).\
<https://www.aclweb.org/anthology/N19-1423/>

**RoBERTa: A Robustly Optimized BERT Pretraining Approach**. Liu et al. 2019\
Use dynamic masking in pretraining; use large batch size and long sequence without NSP.\
<https://arxiv.org/abs/1907.11692>

**ALBERT: A Lite BERT for Self-supervised Learning of Language Representations**. Lan et al. ICLR'20\
(1) share parameters across layers but with widen hidden states (2) factorized embedding to reduce memory (3) add sentence-order prediction objective. Overall, ALBERT gives better performance under the same size of parameters (shared layers but larger hidden states).\
<https://arxiv.org/abs/1909.11942>

**SpanBERT: Improving Pre-training by Representing and Predicting Spans**. Joshi et al. TACL'20\
Better for span-selection tasks such as extractive QA or coreference resolution: use MLM + Span Boundary Objective (mask a span and predict span tokens using boundary tokens) in pretraining.\
<https://www.aclweb.org/anthology/2020.tacl-1.5/>

**ELECTRA: PRE-TRAINING TEXT ENCODERS AS DISCRIMINATORS RATHER THAN GENERATORS**. Clark et al. ICLR'20\
More efficient pre-training objective: a discriminator to predict whether a token is replaced with a plausible alternative by a small MLM model (generator). Although similar to GAN, generator is trained independently. More efficient since prediction is on all tokens, leading to better performance under the same condition compared with MLM.\
<https://arxiv.org/pdf/2003.10555.pdf>

**SAS: Self-Augmentation Strategy for Language Model Pre-training**. Xu et al. AAAI'22\
Encoder; similar to ELECTRA, but combine generator inside the model as a special head.\
<http://arxiv.org/abs/2106.07176>

**ROFORMER: ENHANCED TRANSFORMER WITH ROTARY POSITION EMBEDDING**. Su et al. 2022\
Relative position encoding; summary of adding relative pos into query$\cdot$key.\
<http://arxiv.org/abs/2104.09864>

## Generation-based

**Improving Language Understanding by Generative Pre-Training**. Radford et al. 2018\
Decoder-only; GPT, the first pre-trained model on Transformers with auto-regressive LM.\
<https://www.semanticscholar.org/paper/Improving-Language-Understanding-by-Generative-Radford/cd18800a0fe0b668a1cc19f2ec95b5003d0a5035>

**Unified Language Model Pre-training for Natural Language Understanding and Generation**. Dong et al. NIPS'19\
Encoder-Decoder; handle both NLU and generation.\
Decoder pretraining: recover only the corrupted span.\
<https://arxiv.org/pdf/1905.03197.pdf>

**UNILMv2: Pseudo-Masked Language Models for Unified Language Model Pre-Training**. Bao et al. 2020\
<https://arxiv.org/abs/2002.12804>

**BART: Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension**. Lewis et al. 2019\
Encoder-Decoder; combining denoising encoder and auto-regressive decoder. Better for generation tasks.\
Pretraining: recover the ENTIRE corrupted input sequence (unlike UniLM or T5).\
<https://arxiv.org/abs/1910.13461>

**Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer**. Raffel et al. JMLR'20\
T5 Encoder-Decoder: cast different downstream tasks as S2S generation tasks.\
Pretraining objective: can recover the entire sequence or only corrupted span (Figure 5). Authors find only recovering corrupted span performs good.\
<https://arxiv.org/abs/1910.10683>

**GLM: General Language Model Pretraining with Autoregressive Blank Infilling**. Du et al. ACL'22\
Similar to T5, with differences: 1) enable full mask before generation (still decoder, not a separate encoder); 2) use position encoding to distinguish multiple masked spans, with shuffling for interactions.\
<https://aclanthology.org/2022.acl-long.26>

**WeLM: A Well-Read Pre-trained Language Model for Chinese**. Su et al. 2022\
Decoder-only; (1) pretrain on high-quality corpus (2) finetune with designed prompts of different tasks to enable zero-shot task (model learns tasks from prompts).\
<https://arxiv.org/pdf/2209.10372>

## Retrieval-based

**Nonparametric Masked Language Modeling**. Min et al. 2022\
Nonparametric: output decided by data, not by parameters.\
Encoder on masked text + nearest-neighbor on phrases in corpus.\
No softmax on finite vocabulary.\
<http://arxiv.org/abs/2212.01349>

## Incorporate External Knowledge Source

**Atlas: Few-shot Learning with Retrieval Augmented Language Models**. Izacard et al. 2022\
Decouple knowledge from LM, rather than memorization within LM parameters.\
Model: recognize relevant docs regarding input query + LM decoding based on docs and query.\
Pretraining retrieval: relevant doc distribution as latent distribution (no annotations), designing self-supervised objective leveraging LM signals (discussed four candidates)\
Pretraining LM: auto-denoising spans, similar to T5\
Jointly train; use pre-computed doc embedding for document collections, and update regularly.\
Evaluate: in-context prompting, or few-shot finetuning.\
<http://arxiv.org/abs/2208.03299>

## For Long Sequence

**Transformer-XL: Attentive Language Models beyond a Fixed-Length Context**. Dai et al. ACL'19\
Recurrent-based.\
Encode variable length input (auto-regressive): use a fixed-length Transformers segment and apply recurrence on the input; utilize relative position in self-attention to address cross-segment distance.\
<https://www.aclweb.org/anthology/P19-1285>

**Recurrent Memory Transformer**. Bulatov et al. NIPS'22\
Past hidden + current text -> current hidden.\
<https://openreview.net/forum?id=Uynr3iPhksa>

**XLNet: Generalized Autoregressive Pretraining for Language Understanding**. Yang et al. NeurIPS'19\
Recurrent-based.\
Enhanced model upon Transformer-XL: use Permutation Language Modeling in auto-regressive LM but with permuted sequence to encode bidirectionally. Also some adaptations in the implementation to cope with permutation.\
<https://arxiv.org/pdf/1906.08237.pdf>

**Reformer: The Efficient Transformer**. Kitaev et al. ICLR'20\
Bucket-based.\
Encode large sequence (auto-regressive): (1) use shared QK matrices (2) use Locality Sensitive Hashing (LSH) in the self-attention, to approximately find a subset of K with high prob on Q, without computing full QK (3) use reversible residual connections in the attention and feedforward layer to eliminate full copy of forward values for back-propagation.\
Ablations: (1) reversible layers do not really hurt performance (2) hash bucket size is a trade-off between performance and complexity.\
<https://arxiv.org/abs/2001.04451>

**Longformer: The Long-Document Transformer**. Beltagy et al. 2020\
Window-based.\
Enable long sequence at thousands level (either MLM or auto-regressive): (1) use sliding-window local attention (similar to CNN), therefore $O(n)$ instead of $O(n^2)$; full attention is implicitly obtained by stacking layers; varying window size and dilation at each layer (2) task specific global attention (viewed as constant).\
<https://arxiv.org/abs/2004.05150>

**Linformer: Self-Attention with Linear Complexity**. Wang et al. 2020\
Low-rank on attention.\
<https://arxiv.org/abs/2006.04768>

**Big Bird: Transformers for Longer Sequences**. Zaheer et al. NIPS'20\
Window-based.\
<https://arxiv.org/abs/2007.14062>

**Poolingformer: Long Document Modeling with Pooling Attention**. Zhang et al. ICML'21\
Window-based.\
<https://arxiv.org/abs/2105.04371>

**Informer: Beyond Efficient Transformer for Long Sequence Time-Series Forecasting**. Zhou et al. AAAI'21\
<https://arxiv.org/abs/2012.07436>

**MEMORIZING TRANSFORMERS**. Wu et al. ICLR'22\
Attention on entire past context -> local context (current chunk sequence) + top-k (kNN search by QK dot-product) past token position stored as external KV.\
Training: for each chunk, regard past stored KV as given (not backprop), resulting in distributional shift for long document (need training strategy).\
<https://openreview.net/pdf?id=TrjbxzRcnf->

**Focused Transformer: Contrastive Training for Context Scaling**. Tworkowski et al. 2023\
Same as above, but add negative sampling when performing top-k search to force distinguishing important positions.\
<http://arxiv.org/abs/2307.03170>

<https://huggingface.co/blog/long-range-transformers>

## For Distillation

**Analyzing Multi-Head Self-Attention: Specialized Heads Do the Heavy Lifting, the Rest Can Be Pruned**. Voita et al. ACL'\
<https://aclanthology.org/P19-1580>

**TinyBERT: Distilling BERT for Natural Language Understanding**. Jiao et al. EMNLP-Findings'20\
Pre-training objective (on each layer): (1) MSE on raw attention score (2) MSE on FFNN (3) MSE on embedding\
Finetuning objective: temperature softmax on prediction\
Finetuning data augmentation: replace words with synonyms\
Finding: on downstream tasks, two finetuning techniques have more impact than pre-training objective.\
<https://www.aclweb.org/anthology/2020.findings-emnlp.372>

**MINILM: Deep Self-Attention Distillation for Task-Agnostic Compression of Pre-Trained Transformers**. Wang et al. NeurIPS'20\
Pre-training objective (only on last layer): (1) KL on attention prob distribution (2) KL on value relations\
No other objectives.\
<https://arxiv.org/abs/2002.10957>

**Adversarial Data Augmentation for Task-Specific Knowledge Distillation of Pre-Trained Transformers**. Zhang et al. AAAI'22\
Good related work.\
Task-specific distillation: (1) standard KD (2) Adversarial Data Augmentation using embedding perturbation.\
Benefits: embedding augmentation helps low-resource and generalization.\
<https://aaai-2022.virtualchair.net/poster_aaai1872>

**ZipLM: Inference-Aware Structured Pruning of Language Models**. Kurtić et al. NIPS'23\
<https://arxiv.org/abs/2302.04089>

## For Incorporating Knowledge

**ERNIE: Enhanced Language Representation with Informative Entities**. Zhang et al. ACL'19\
Fuse entity embedding: pretraining objective to predict entity given aligned tokens.\
<https://arxiv.org/abs/1905.07129>

**ERNIE 2.0: A Continual Pre-Training Framework for Language Understanding**. Sun et al. AAAI'20\
Design other tasks (w/o human annotation) to further pretrain the model.\
Continual learning without forgetting: allocate training times per task throughout the stages.\
<http://arxiv.org/abs/1907.12412>

**ERNIE 3.0: LARGE-SCALE KNOWLEDGE ENHANCED PRE-TRAINING FOR LANGUAGE UNDERSTANDING AND GENERATION**. Sun et al. 2021\
Leverage explicit KG in pretraining, adding another mask objective to fuse KG, in addition to other NLU and NLG objectives.\
Model: shared bottom layers + (small) task-specific layers (w/ both NLU and NLG).\
<http://arxiv.org/abs/2107.02137>

**K-BERT: Enabling Language Representation with Knowledge Graph**. Liu et al. AAAI'20\
Inject triplets into the sequence with soft position embedding and masked attention.\
<https://arxiv.org/abs/1909.07606>

**DKPLM: Decomposable Knowledge-enhanced Pre-trained Language Model for Natural Language Understanding**. Zhang et al. AAAI'22\
Inject by replacing entity with repr of lexical head/tail/relation and entity description (no need to require separate KG embedding).\
<https://arxiv.org/pdf/2112.01047>

## For Distilling Knowledge

**Symbolic Knowledge Distillation: from General Language Models to Commonsense Models**. West et al. NAACL'22\
Distill common-sense knowledge by few-shot prompts from general GPT3 to a student model.\
<https://aclanthology.org/2022.naacl-main.341>

**BertNet: Harvesting Knowledge Graphs from Pretrained Language Models**. Hao et al. 2022\
Distill KG given few-shot: paraphrase (by GPT3) relation prompts + search entities on generated auto-prompts of the same relation semantic.\
<http://arxiv.org/abs/2206.14268>

## LM Editing/Update

Goal: 1) efficacy of editing; 2) efficacy of generalization; 3) no forgetting.

Gradient-based:

**Editing Factual Knowledge in Language Models**. De Cao et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.522/>

**Fast Model Editing at Scale**. Mitchell et al. ICLR'22\
<https://arxiv.org/abs/2110.11309>

Locate-then-edit:

**Transformer feed-forward layers are key-value memories**. Geva et al. EMNLP'21\
Argue that linear operations, e.g. MLP, can operate as a kv storage.\
<https://aclanthology.org/2021.emnlp-main.446/>

**Transformer feed-forward layers build predictions by promoting concepts in the vocabulary space**. Geva et al. EMNLP'22\
<https://aclanthology.org/2022.emnlp-main.3/>

**Locating and Editing Factual Associations in GPT**. Meng et al. NIPS'22\
A new kv pair can be inserted in MLP by solving a constrained least-squares problem.\
<https://arxiv.org/abs/2202.05262>

**Mass-Editing Memory in a Transformer**. Meng et al. ICLR'23\
<https://arxiv.org/abs/2210.07229>

**Editing models with task arithmetic**. Ilharco et al. ICLR'23\
<https://openreview.net/forum?id=6t0Kwf8-jrj>

Cache-based:

**Memory-Based Model Editing at Scale**. Mitchell et al. ICML'22\
Pseudo-editing: not touching parameters; simply store update examples, and train a binary classifier to check if input is related to any examples.\
<http://arxiv.org/abs/2206.06520>

## For Analysis

**Perturbed Masking: Parameter-free Probing for Analyzing and Interpreting BERT**. Wu et al. ACL'20\
Unsupervised probing w/o training to induce sent/doc structure: obtain a matrix of perturbation impact.\
<https://aclanthology.org/2020.acl-main.383>

**SparseBERT: Rethinking the Importance Analysis in Self-attention**. Shi et al. ICML'21\
Diagonal positions in attention map are not as important compared to other positions.\
<https://arxiv.org/pdf/2102.12871>

**Probing Linguistic Information for Logical Inference In Pre-Trained Language Models**. Chen and Gao. AAAI'22\
<https://arxiv.org/abs/2112.01753>

**Visualizing and Measuring the Geometry of BERT**. Coenen et al. NIPS'19\
(1) By linear probing, both attentions and embeddings are able to encode syntactic information.\
(2) BERT can show word-sense clusters, with good accuracy by NN probing.\
(3) By linear transformation probing (to smaller dimension) on embedding, word-sense is more drastic, and earlier layers retrain more semantic info.\
(4) Another word-sense (opposite sense concatenation) towards context change shows that earlier layers are more stable than final layers against context change.\
(5) The semantic against context change is "soft" rather than "hard": indiscriminately absorb meaning from all neighbors.\
<http://arxiv.org/abs/1906.02715>

**How Contextual are Contextualized Word Representations? Comparing the Geometry of BERT, ELMo, and GPT-2 Embeddings**. Ethayarajh et al. EMNLP'19\
anisotropic: averaged cosine-similarity (randomly sampled) is high; self-similarity: contextualization sensitivity.\
(1) contextualized representations are anisotropic, especially in higher layers (2) more context-specific in higher layers.\
<https://aclanthology.org/D19-1006/>

**Let's Play Mono-Poly: BERT Can Reveal Words' Polysemy Level and Partitionability into Senses**. Garí Soler and Apidianaki. TACL'21\
Investigate the impact of context variation on token representation (through cosine similarity comparison):\
(1) BERT can distinguish mono and poly words.\
(2) BERT can distinguish word sense for poly words.\
(3) In earlier layers, context variation makes representations more dissimilar; in last layers, information about the input token is recovered for LM prediction.\
<https://aclanthology.org/2021.tacl-1.50>

**Ditto: A Simple and Efficient Approach to Improve Sentence Embeddings**. Chen et al. EMNLP'23\
Weighted token hidden state by pooling attention scores of different layers.\
<https://aclanthology.org/2023.emnlp-main.359>

**The Inductive Bias of In-Context Learning: Rethinking Pretraining Example Design**. Levine et al. ICLR'22\
In pretraining, directly seen > separately seen (requires generalization).\
<http://arxiv.org/abs/2110.04541>

**Large Models are Parsimonious Learners: Activation Sparsity in Trained Transformers**. Li et al. 2022\
Sparsity: only 2.7% activated neurons in MLPs of NLP/vision Transformers; likely related to training dynamics, as this is true for any datasets.\
<http://arxiv.org/abs/2210.06313>

**Pre-Training to Learn in Context**. Gu et al. ACL'23\
Pack sentences of similar tasks for pretraining.\
<https://aclanthology.org/2023.acl-long.267>

## Recent Survey

**Pre-Trained Models: Past, Present and Future**. Han et al. arXiv'21\
<https://arxiv.org/abs/2106.07139>
