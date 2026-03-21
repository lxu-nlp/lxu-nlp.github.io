---
layout: post
title: "Works on Multilingual Tasks"
date: 2022-02-15
categories: [NLP]
tags: [nlp, multilingual]
math: true
---


## Benchmark

**XGLUE: A New Benchmark Dataset for Cross-lingual Pre-training, Understanding and Generation**. Liang et al. EMNLP'20\
<https://www.aclweb.org/anthology/2020.emnlp-main.484>

**XTREME: A Massively Multilingual Multi-task Benchmark for Evaluating Cross-lingual Generalization**. Hu et al. ICML'20\
<https://proceedings.icml.cc/static/paper_files/icml/2020/4220-Paper.pdf>

**XTREME-R: Towards More Challenging and Nuanced Multilingual Evaluation**. Ruder et al. EMNLP'21\
Two new tasks: Multilingual Causal Reasoning, Retrieval from a Multilingual Pool.\
<https://aclanthology.org/2021.emnlp-main.802/>

## Multilingual Representation/Encoder
### Explicit Alignment Between Embedding Space:

**Exploiting Similarities among Languages for Machine Translation**. Mikolov et al. '13\
Target on supervised BLI: use linear transformation between source and target word embedding using a seed dictionary, minimizing the L2 distance as regression: $\sum_i \Vert Wx_i - z_i \Vert^2$.\
Evaluation: word and phrase translation on WMT11\
<https://arxiv.org/abs/1309.4168>

**Normalized Word Embedding and Orthogonal Transform for Bilingual Word Translation**. Xing et al. NAACL'15\
Target on supervised BLI: following Mikolov'13, add orthogonal constraint on linear transformation, making all word embedding as unit vector, closing the gap between training objective (distance) and evaluation (cosine similarity).\
<https://www.aclweb.org/anthology/N15-1104/>

**Generalizing and Improving Bilingual Word Embedding Mappings with a Multi-Step Framework of Linear Transformations**. Artetxe et al. AAAI'18\
Target on supervised BLI: a good summary and a general framework of previous supervised BLI approaches, that decouples, generalizes, and analyzes different methods. All are based on the orthogonal transformation under different processing.\
<http://ixa.si.ehu.eus/sites/default/files/dokumentuak/11455/aaai18.pdf>

**Loss in Translation: Learning Bilingual Word Mapping with a Retrieval Criterion**. Joulin et al. EMNLP'18\
Target on supervised BLI: integrate the inference as a retrieval problem into an end-to-end training process. Newly designed objective and optimization (a convex relaxation of new loss) is performed. SOTA supervised BLI before 2020.\
<https://www.aclweb.org/anthology/D18-1330>

**A robust self-learning method for fully unsupervised cross-lingual mappings of word embeddings**. Artetxe et al. ACL'18\
Target on unsupervised BLI: use iterative self-learning (VecMap). Most robust unsupervised BLI before 2019.\
<https://www.aclweb.org/anthology/P18-1073>

**WORD TRANSLATION WITHOUT PARALLEL DATA**. Conneau et al. ICLR'18\
Target on unsupervised BLI: use adversarial training with unsupervised model selection criterion (MUSE).\
Evaluation: word translation, word similarity, sentence translation retrieval\
<https://openreview.net/pdf?id=H196sainb>

**On the Limitations of Unsupervised Bilingual Dictionary Induction**. Søgaard et al. ACL'18\
Analysis on unsupervised BLI: analyze adversarial training BLI (Conneau'18) on its instability across languages, especially on distant language pairs.\
<https://www.aclweb.org/anthology/P18-1072>

**Revisiting Adversarial Autoencoder for Unsupervised Word Translation with Cycle Consistency and Improved Training**. Mohiuddin and Joty. NAACL'19\
Target on unsupervised BLI: enhance adversarial training, proposing (linear) autoencoder on BLI that maps source and target embeddings into latent space, emphasizing the benefits over original embedding space.\
<https://www.aclweb.org/anthology/N19-1386>

**Cross-Lingual Alignment of Contextual Word Embeddings, with Applications to Zero-shot Dependency Parsing**. Schuster et al. NAACL'19\
Use anchor points to align contextual word embeddings; can be either supervised or unsupervised.\
<https://www.aclweb.org/anthology/N19-1162/>

**Improving Bilingual Lexicon Induction for Low Frequency Words**. Huang et al. EMNLP'20\
Target on supervised BLI: use two statistics to identify accuracy drop for low-frequency words, and address each of them: (1) diminishing margin between cosine similarity (optimize by hinge loss to increase margin of non-correct over correct) (2) exacerbated hubness (optimize by linear assignment). Writing is very clear.\
<https://www.aclweb.org/anthology/2020.emnlp-main.100>

**Are All Good Word Vector Spaces Isomorphic?** Vulic et al. EMNLP'20\
Analysis on MWE: isomorphism does not hold for distant languages and monolingual corpora of small size; target language performance drastically decreases as language is more distant with smaller pretraining corpora.\
<https://www.aclweb.org/anthology/2020.emnlp-main.257>

**The Secret is in the Spectra: Predicting Cross-lingual Task Performance with Spectral Similarity Measures**. Dubossarsky et al. EMNLP'20\
Analysis on MWE: propose statistics of numerical analysis to measure isomorphism between MWE, which is shown to have strong correlation with performance on downstream tasks, suggesting this spectrum-based measurement is a strong predictor of cross-lingual transfer performance.\
<https://www.aclweb.org/anthology/2020.emnlp-main.186>

**LNMAP: Departures from Isomorphic Assumption in Bilingual Lexicon Induction Through Non-Linear Mapping in Latent Space**. Mohiuddin et al. EMNLP'20\
Target on semi-supervised BLI (small seed dictionary): (1) use semi-supervised that outperforms all previous supervised or unsupervised approaches (2) similar to their 19 work, use autoencoder to mapping source and target embedding into latent space, and learn transformation/mapping on latent space instead of original embedding space (3) use non-linearity in autoencoder and mapping (non-isomorphism).\
Optimization: (1) loss on reconstruction of same word per autoencoder on a language (2) loss on mapping loss (similar to previous work) on latent space (3) back-translation loss across two languages (4) loss on reconstruction on the back-translated latent embedding across two languages.\
Findings: (1) new SOTA by a good margin (2) perform better on low-resource languages (3) non-linearity does not help much on rich-resource languages.\
Good related work.\
<https://www.aclweb.org/anthology/2020.emnlp-main.215/>

**Semi-Supervised Bilingual Lexicon Induction with Two-way Interaction**. Zhao et al. EMNLP'20\
Target on semi-supervised BLI: a framework that uses interaction signals from both annotated and unannotated data. Outperform all previous BLI approaches. The framework can use other unsupervised methods.\
<https://www.aclweb.org/anthology/2020.emnlp-main.238>

**Interactive Refinement of Cross-Lingual Word Embeddings**. Yuan et al. EMNLP'20\
A method to quickly incorporate human feedback/annotation/interaction to refine CLWE on downstream classification task. Improving results on tasks of low-resource languages within one hour.\
<https://www.aclweb.org/anthology/2020.emnlp-main.482>

**Pre-tokenization of Multi-word Expressions in Cross-lingual Word Embeddings**. Otani et al. EMNLP'20\
Target on supervised BLI: focus on Multi-Word Expressions (MWE) in cross-lingual setting, which is important for downstream tasks: (1) build MWE lists of different languages using WordNet (2) pre-tokenize corpus using MWE list (3) train word-embedding with MWE per language (4) train mapping between two languages on the trained word-embedding without using MWE in dictionary (same training data as traditional BLI) (5) evaluation on MWE\
Findings: outperforms baseline (averaging tokens in MWE) on evaluation of MWE (2) no performance hurt for traditional single-word translation.\
<https://www.aclweb.org/anthology/2020.emnlp-main.360/>

**Cross-Lingual BERT Transformation for Zero-Shot Dependency Parsing**. Wang et al. EMNLP'19\
Target on contextualized BLI; learn contextualized word embedding alignment (linear transformation) between Eng BERT and mBERT; word alignment is obtained through existing tool. Evaluation: zero-shot dependency parsing using existing biaffine Eng parser; comparable with XLM.\
<https://www.aclweb.org/anthology/D19-1575/>

## Joint/Shared Embedding Space:

**XNLI: Evaluating Cross-lingual Sentence Representations**. Conneau, et al. EMNLP'18\
Joint multilingual sentence embedding w/ explicit alignment: use MWE + LSTM, and further force similar sentence embedding between parallel pairs and dissimilar for negative pairs.\
<https://www.aclweb.org/anthology/D18-1269.pdf>

**Unicoder: A Universal Language Encoder by Pre-training with Multiple Cross-lingual Tasks**. Huang et al. EMNLP'19\
Joint multilingual embedding w/ explicit alignment: similar to XLM, with multi-tasks in pretraining using silver explicit alignment information. Although perform worse than XLM.\
<https://www.aclweb.org/anthology/D19-1252/>

**Cross-lingual Language Model Pretraining**. Conneau and Lample. NIPS'19\
Joint multilingual embedding w/o explicit alignment: dubbed XLM, (1) shared vocabulary with BPE, sampling that alleviates bias towards high-resource languages (2) two monolingual objectives: traditional left-to-right (CLM); masked Cloze task (MLM) (3) multilingual objective: translation LM (TLM) using parallel data.\
Evaluation: XNLI, UNMT, NMT\
<https://papers.nips.cc/paper/8928-cross-lingual-language-model-pretraining.pdf>

**Unsupervised Cross-lingual Representation Learning at Scale**. Conneau et al. ACL'20\
Joint ME w/o explicit alignment: dubbed XLM-R following Conneau'19, with slight modification: (1) fix undertuned by using RoBERTa (2) more languages, larger dataset.\
Observation: (1) more model capacity helps with "the curse of multilinguality" (2) larger shared vocabulary helps with performance.\
Evaluation: XNLI, NER, QA, GLUE\
<https://www.aclweb.org/anthology/2020.acl-main.747.pdf>

**Do Explicit Alignments Robustly Improve Multilingual Encoders?** Wu and Dredze. EMNLP'20\
Joint ME w/ explicit alignment: experiment four different explicit alignment techniques during pretraining, including token-level linear alignment (direct) and contrastive alignment (relative); however, results show no improvement over implicit alignment such as XLM.\
<https://www.aclweb.org/anthology/2020.emnlp-main.362>

**Low-Resource Sequence Labeling via Unsupervised Multilingual Contextualized Representations**. Bao et al. EMNLP'19\
Joint ME w/ explicit alignment: explicit distribution alignment on joint ME pretraining for closer joint embedding space, e.g. identical string, mean and variance, etc.\
Tagging task: joint ME as features + LSTM +CRF.\
<https://www.aclweb.org/anthology/D19-1095.pdf>

**MULTILINGUAL ALIGNMENT OF CONTEXTUAL WORD REPRESENTATIONS**. Cao et al. ICLR'20\
Joint ME w/ explicit alignment: silver explicit word alignment on joint ME pretraining, which minimizes L2 distance with finetuning (not learning a transformation) and regularization to keep as much original pretraining embeding. Perform worse than XLM.\
<https://arxiv.org/abs/2002.03518>

**CROSS-LINGUAL ALIGNMENT VS JOINT TRAINING: A COMPARATIVE STUDY AND A SIMPLE UNIFIED FRAMEWORK**. Wang et al. ICLR'20\
Joint ME w/ explicit alignment: (1) compare downstream performance between  alignment-based MWE and jointly-training-based ME (2) a general framework to do both joint training and utilizing explicit alignment.\
Evaluation on BLI, NER, MT; however, only 3 lang for NER and 2 lang for MT.\
<https://arxiv.org/abs/1910.04708>

**Improving Multilingual Models with Language-Clustered Vocabularies**. Chung et al. EMNLP'20\
Joint ME w/o explicit alignment: focus on vocabulary and use regular XLM training. Argue that using overal frequencies to determine subword vocabulary is suboptimal; group languages by k-mean clustering on language similarity (one-hot representation for each language), then allocate subwords based on cluster size.\
Evaluation on QA, XNLI, NER shows improvement with good margin; the only change is the vocabulary.\
<https://www.aclweb.org/anthology/2020.emnlp-main.367>

**CROSS-LINGUAL ABILITY OF MULTILINGUAL BERT: AN EMPIRICAL STUDY**. K et al. ICML'20\
Analysis of mBERT: using a methodology that creating Fake-English language to analyze the important factors of cross-lingual ability in mBERT (more accurately, bilingual BERT (B-BERT) in the analysis).\
Findings: (1) shared vocabulary (overlapping word-pieces) has little impact, surprisingly; B-BERT performs well even with no shared vocabulary (2) depth and total parameters are crucial for BERT's performance, while multi-head is not significant (3) character-level and word-level tokenization perform much worse than word-piece level.\
<https://openreview.net/pdf/1499e19238fd9d7ee8a9c7e7bb6f9e2c9e6a0adf.pdf>

**On the Cross-lingual Transferability of Monolingual Representations**. Artetxe et al. ACL'20\
Analysis of mBERT: (1) shared vocabulary is not necessary; disjoint vocabularies perform as well (2) an important factor is the effective vocabulary size per language (3) large numbers of languages do not perform better (also mentioned in Conneau'20) (4) joint multilingual pretraining is not essential for cross-lingual generalization; monolingual models already show generalized abstraction in layers.\
<https://www.aclweb.org/anthology/2020.acl-main.421>

**Emerging Cross-lingual Structure in Pretrained Language Models**. Conneau et al. ACL'20\
Analysis of mBERT: (1) again, shared vocabulary has trivial performance impact; still shows multilinguality with no overlapping vocabulary (2) sharing parameters is important for cross-lingual performance (3) monolingual models also show multilinguality (same finding as Artetxe'20) (4) similar to BLI, contextual word-level embedding from monolingual BERT can be aligned with simple orthogonal linear mapping (isomorphism).\
<https://www.aclweb.org/anthology/2020.acl-main.536/>

**Identifying Elements Essential for BERT’s Multilinguality**. Dufter and Schu ̈tze. EMNLP'20\
Analysis of mBERT: (1) shared position embeddings, shared special tokens, common masking strategy contribute to multilinguality (2) word order is an important factor; reversing word order disrupts multilinguality.\
Good related work on multilinguality.\
<https://www.aclweb.org/anthology/2020.emnlp-main.358/>

**A Call for More Rigor in Unsupervised Cross-lingual Learning**. Artetxe et al. ACL'20\
Retrospect: (1) argue there are plenty parallel data in real world, hence UCL is not as necessary; still advocate UCL as important research task, as it is not obvious why UCL should be possible at all, since humans cannot align two unknown languages without any grounding (2) current UCL utilizes vocabulary more or less.\
<https://arxiv.org/pdf/2004.14958.pdf>

**A Simple Approach to Learning Unsupervised Multilingual Embeddings**. Jawanpuria et al. EMNLP'20\
Joint multilingual embedding through MWE: without learning language models; approach is very simple. It provides insights on how to align two existing embeddings on a shared space.\
<https://www.aclweb.org/anthology/2020.emnlp-main.240/>

**ERNIE-M: Enhanced Multilingual Representation by Aligning Cross-lingual Semantics with Monolingual Corpora**. Ouyang et al. EMNLP'21\
Upon XLM-R, adding two objectives for force shared alignment: (1) TLM but only using target sentences rather than both source & target (2) back-translation, by generating pseudo target sentence from source through filling MASK placeholder, then use it to predict source MASK.\
<https://aclanthology.org/2021.emnlp-main.3>

## Machine Translation

**Google’s Multilingual Neural Machine Translation System: Enabling Zero-Shot Translation**. Johnson et al. TACL'17\
Target on supervised joint multilingual MT: improve low-resource and enable zero-shot translation by using a single multilingual encoder-decoder trained with shared vocab and all multilingual parallel data consisting of multiple language pairs at once, just adding a special token in the front to indicate target language.\
Evaluation: different setup, one-to-many, many-to-one, many-to-many pairs.\
<https://www.aclweb.org/anthology/Q17-1024/>

**Massively Multilingual Neural Machine Translation**. Aharoni et al. NAACL'19\
Target on supervised joint multilingual MT: use same model as Johnson'17, scale up to 100+ languages. Evaluate many-to-many, one-to-many, many-to-one performance under different conditions (# languages involved).\
<https://aclanthology.org/N19-1388/>

**UNSUPERVISED MACHINE TRANSLATION USING MONOLINGUAL CORPORA ONLY**. Lample et al. ICLR'18\
Target on unsupervised MT: also propose unsupervised model selection.\
<https://arxiv.org/abs/1711.00043>

**UNSUPERVISED NEURAL MACHINE TRANSLATION**. Artetxe et al. ICLR'18\
Target on unsupervised MT: use shared encoder-decoder, pretrained multilingual embedding, and utilize back-translation.\
<https://openreview.net/pdf?id=Sy2ogebAW>

**Phrase-Based & Neural Unsupervised Machine Translation**. Lample et al. EMNLP'18\
Summary on unsupervised MT: (1) initialization, roughly aligned distributions (2) reinforce language model (3) back-translation. Training loss includes (2) and (3).\
<https://www.aclweb.org/anthology/D18-1549.pdf>

**Improving Massively Multilingual Neural Machine Translation and Zero-Shot Translation**. Zhang et al. ACL'20\
Target on unsupervised MT: (1) improve zero-shot MT by utilize back-translation (2) propose OPUS-100 dataset.\
<https://www.aclweb.org/anthology/2020.acl-main.148/>

**Bilingual Dictionary Based Neural Machine Translation without Using Parallel Sentences**. Duan et al. ACL'20\
Target on unsupervised MT: without any parallel corpus, but use a ground-truth bilingual dictionary; similar to UMT, utilizing back-translation, but replace some tokens with the corresponding token in the other language, serving as anchor points. Words are also replaced during pretraining.\
Evaluation shows that it has similar performance on rich resource lang pair with UMT, but better performance on low resource lang pair since it uses ground-truth dict.\
<https://www.aclweb.org/anthology/2020.acl-main.143/>

**Pre-training Multilingual Neural Machine Translation by Leveraging Alignment Information**. Lin et al. EMNLP'20\
Target on MT "pretraining": train MT objective on English-centric pairs with Random Aligned Substitution, randomly replacing a source word to a different language of the same meaning by using BLI. Then train on downstream language pairs.\
It performs better than directly training on downstream pairs, although this feels more like data augmentation than pretraining.\
<https://www.aclweb.org/anthology/2020.emnlp-main.210>

## Transfer on Downstream Tasks

### Only Source Language in Training

**From Zero to Hero: On the Limitations of Zero-Shot Language Transfer with Multilingual Transformers**. Lauscher et al. EMNLP'20\
Retrospect: analyze when the zero-shot transfers of Transformers fail: distant languages with smaller training corpora; suggest using few-shot transfer that gives much gain with little annotation on target languages. Evaluate on both low-level and higher-level downstream tasks.\
<https://www.aclweb.org/anthology/2020.emnlp-main.363>

**Don’t Use English Dev: On the Zero-Shot Cross-Lingual Evaluation of Contextual Embeddings**. Keung et al. EMNLP'20\
Retrospect: for tasks including MLDoc, MLQA, XNLI, etc., there exists large performance inconsistency between Eng dev and other languages' dev/test; Eng dev itself is not a good representation/evaluation for zero-shot transfer on other languages. For some languages, the directional agreement between Eng dev and other target's dev is even worse than random. See Figure 1 and Table 3.\
Suggestion: find the best checkpoint on target language's dev. (Why don't few-shot if target dev is available?..)\
<https://www.aclweb.org/anthology/2020.emnlp-main.40>

**On Difficulties of Cross-Lingual Transfer with Order Differences: A Case Study on Dependency Parsing**. Ahmad et al. NAACL'19\
Use existing MWE as feature, and propose Transformers decoder that enforces order-free representation for better generalization across languages, using order-free position embeddings. Evaluate on zero-shot dependency parsing, and perform better than regular Transformers or RNN.\
<https://www.aclweb.org/anthology/N19-1253/>

### w/ Unlabeled Target Language (Not Necessarily Parallel Corpus)

**A Robust Self-Learning Framework for Cross-Lingual Text Classification**. Dong and Melo. EMNLP'19\
Make use of unlabelled target language data; joint ME as encoder.\
Use self-learning on zero/few-shot cross-lingual classification, and add high-confident predicted labels on unlabeled
dataset into the training set. Their evaluation shows some non-marginal improvement.\
<https://www.aclweb.org/anthology/D19-1658.pdf>

**PPT: Parsimonious Parser Transfer for Unsupervised Cross-Lingual Adaptation**. Kurniawan et al. EACL'21\
Self-learning for unlabeled structured training.\
<https://aclanthology.org/2021.eacl-main.254/>

**Zero-Resource Cross-Lingual Named Entity Recognition**. Bari et al. AAAI'20\
Make use of unlabelled target language data; MWE + LSTM + CRF.\
Train on source language, sample predictions on target language as silver labels; training (gold) source and (silver)
target language together.\
<https://arxiv.org/pdf/1911.09812.pdf>

**Single-/Multi-Source Cross-Lingual NER via Teacher-Student Learning on Unlabeled Data in Target Language**. Wu et al.
ACL'20\
Make use of unlabelled target language data; joint ME as encoder.\
Distill the model (with shared embedding) trained on source languages as teacher on the target language, by using MSE to
align on logits. Minor improvement.\
Why don't they also try other tasks?\
<https://arxiv.org/abs/2004.12440>

**Alignment-free Cross-lingual Semantic Role Labeling**. Cai and Lapata. EMNLP'20\
Make use of unlabelled parallel corpus; joint ME as encoder.\
(1) project source annotation on parallel corpus to get silver target annotation (2) a specific model for SRL that
regularizes on parallel corpus in the training. Writing is not that clear.\
<https://www.aclweb.org/anthology/2020.emnlp-main.319>

**Multilingual AMR-to-Text Generation**. Fan and Gardent. EMNLP'20\
Make use of unlabelled parallel corpus; joint ME.\
(1) get silver annotation for training on parallel corpus by using existing AMR parsing on source and project on
target (2) use the same Eng AMR encoder, embedded language indicator for decoder (3) use multilingual embedding (XLM) as
decoder initialization.\
<https://www.aclweb.org/anthology/2020.emnlp-main.231>

**Smelting Gold and Silver for Improved Multilingual AMR-to-Text Generation**. Ribeiro et al. EMNLP'21\
Strategy to combine (1) gold AMR with translated sentences (2) generated AMR, as noisy labels for target languages.\
<https://aclanthology.org/2021.emnlp-main.57>

**Unsupervised Cross-Lingual Part-of-Speech Tagging for Truly Low-Resource Scenarios**. Eskander et al. EMNLP'20\
Make use of unlabelled parallel corpus; joint ME as features + LSTM as encoder.\
(1) obtain silver annotation for training on parallel corpus: use existing Eng parsing tool on source, and use existing
word alignment tool and heuristic to project POS from source on target (2) further, use brown cluster as feature.\
<https://www.aclweb.org/anthology/2020.emnlp-main.391>

**XL-AMR: Enabling Cross-Lingual AMR Parsing with Transfer Learning Techniques**. Blloshmi et al. EMNLP'20\
Make use of unlabelled parallel corpus; MWE + LSTM.\
(1) annotation project + translation.\
<https://www.aclweb.org/anthology/2020.emnlp-main.195>

### w/ Small Labeled Target Language

**Multi-View Cross-Lingual Structured Prediction with Minimum Supervision**. Hu et al. ACL'21\
Learned weighted views through small labeled target samples, while regularizing multi-view to be similar.\
<https://aclanthology.org/2021.acl-long.207>

### w/ External Knowledge

**Cross-Lingual Named Entity Recognition via Wikification**. Tsai et al. EMNLP'19\
Make use of external knowledge (multilingual entities).\
Build language-independent NER model by wikification, linking every n-gram to wiki titles, and using n-gram types from
FreeBase and Wiki categories as features. Experiments shown that it helps with both monolingual NER and zero-shot
cross-lingual NER.\
<https://www.aclweb.org/anthology/K16-1022.pdf>

**Mind the Gap: Cross-Lingual Information Retrieval with Hierarchical Knowledge Enhancement**. Zhang et al. AAAI'22\
Use neighboring entities of multilingual KG to enrich cross-lingual query-pair representation on mBERT.\
<https://arxiv.org/pdf/2112.13510>

### w/ Adversarial Training or Language-agnostic Representation

**Adversarial Learning with Contextual Embeddings for Zero-resource Cross-lingual Classification and NER**. Keung et al.
EMNLP'19\
Use adversarial training to get language-agnostic representation; joint ME as encoder with explicit embedding space
alignment using adversarial training.\
A discriminator to force language-independent representation from mBERT.\
<https://www.aclweb.org/anthology/D19-1138.pdf>

**Cross-lingual Multi-Level Adversarial Transfer to Enhance Low-Resource Name Tagging**. Huang et al. NAACL'19\
Use adversarial training to get language-agnostic representation; WE + CNN as encoder with explicit embedding space
alignment using adversarial training.\
Both word-level and sentence-level adversarial training. Tagging: BiLSTM+CRF.\
<https://www.aclweb.org/anthology/N19-1383/>

**Language-Agnostic Representation from Multilingual Sentence Encoders for Cross-Lingual Similarity Estimation**.
Tiyajamorn et al. EMNLP'21\
Decompose meaning embedding and language embedding, making use of parallel corpus.\
<https://aclanthology.org/2021.emnlp-main.612>

### w/ Other Techniques

**Learn to Cross-lingual Transfer with Meta Graph Learning Across Heterogeneous Languages**. Li et al. EMNLP'20\
General cross-lingual transfer: apply metric-based meta-learning in downstream tasks to learn similarity of language
pairs (language as task). Then use this similarity as a graph and perform GCN to refine node.\
<https://www.aclweb.org/anthology/2020.emnlp-main.179>

**Zero-Shot Cross-Lingual Transfer with Meta Learning**. Nooralahzadeh et al. EMNLP'20\
General cross-lingual transfer: apply optimization-based meta-learning to learn the parameters that can be finetuned
rapidly on different languages.\
<https://www.aclweb.org/anthology/2020.emnlp-main.368>

### w/ w/ Translation or Other Baselines

**Enhancing Answer Boundary Detection for Multilingual Machine Reading Comprehension**. Yuan et al. ACL'20\
Make use of translation; joint ME as encoder.\
(1) translate questions in source language to target languages and mix together; train on this mixed QA pair (2) mine
knowledge phrases in different languages on web and add masked phrases in pretraining.\
<https://www.aclweb.org/anthology/2020.acl-main.87/>

**Zero-Shot Crosslingual Sentence Simplification**. Mallinson et al. EMNLP'20\
Make use of translation (seems like labels are also available for target language?).\
(1) partially-shared encoder (2) language-specific decoder so that encoder representation is universal for input (3)
multi-task learning in training and adversarial training.\
<https://www.aclweb.org/anthology/2020.emnlp-main.415>

**Learning with Limited Data for Multilingual Reading Comprehension**. Lee et al. EMNLP'19\
Make use of translation; MWE + BiDAF, or joint ME as encoder.\
Propose weakly supervised method to generate new data for new language on QA task using Question Generator and Answer
Extractor. Translation alignment is important for their approach.\
<https://www.aclweb.org/anthology/D19-1283/>

**Cross-Lingual Machine Reading Comprehension**. Cui et al. EMNLP'19\
Make use of translation; joint ME as encoder.\
Propose Dual-BERT, which consider the context-query in both languages with proposed Self Adaptive Attention.\
<https://www.aclweb.org/anthology/D19-1169.pdf>

**Zero-Shot Cross-Lingual Opinion Target Extraction**. Jebbara and Cimiano. NAACL'19\
Baseline on aspect-based sentiment analysis. Encoder: multilingual word embedding aligned by either supervised
dictionary or unsupervised adversarial training. Decoder: CNN tagging.\
<https://www.aclweb.org/anthology/N19-1257/>

**Zero-shot Reading Comprehension by Cross-lingual Transfer Learning with Multi-lingual Language Representation Model**.
Hsu et al. EMNLP'19\
Baseline on cross-lingual QA. Evaluate translation approach and zero-shot approach on QA in En, Chn, Kor. Finding: (1)
translation degrades the performance easily (2) mBert learns generalized representation.\
<https://www.aclweb.org/anthology/D19-1607.pdf>

**Neural Cross-Lingual Relation Extraction Based on Bilingual Word Embedding Mapping**. Ni and Florian. EMNLP'19\
Baseline on relation extraction: use bilingual word embedding mapping to project target embedding to source embedding on
existing PCNN relation extraction model.\
<https://www.aclweb.org/anthology/D19-1038.pdf>

**Document Translation vs. Query Translation for Cross-Lingual Information Retrieval in the Medical Domain**. Saleh and
Pecina. ACL'20\
MT application on IR. Conclusion: (1) NMT is better than SMT (2) Query Translation (QT) is much better than Document
Translation (DT).\
<https://www.aclweb.org/anthology/2020.acl-main.613/>

**Multilingual Offensive Language Identification with Cross-lingual Embeddings**. Ranasinghe and Zampieri. EMNLP'20\
Make use of other domains/tasks; joint ME as encoder.\
Simple use of XLM-R to "pretrain" on Eng data, then train on other languages/domains/tasks to achieve transfer on
offense identification task.\
<https://www.aclweb.org/anthology/2020.emnlp-main.470/>

**A Supervised Word Alignment Method based on Cross-Language Span Prediction using Multilingual BERT**. Nagata et al.
EMNLP'20\
Word alignment problem; joint ME as encoder.\
Frame as QA, a span prediction problem given two input. Use basic span prediction approach with mBERT.\
<https://www.aclweb.org/anthology/2020.emnlp-main.41>

**End-to-End Slot Alignment and Recognition for Cross-Lingual NLU**. Xu et al. EMNLP'20\
<https://www.aclweb.org/anthology/2020.emnlp-main.410>

**Cross-lingual Spoken Language Understanding with Regularized Representation Alignment**. Liu et al. EMNLP'20\
<https://www.aclweb.org/anthology/2020.emnlp-main.587>

**Cross-lingual Transfer Learning for Japanese Named Entity Recognition**. Johnson et al. NAACL'19\
Direct weight transfer; WE as encoder.\
Instead of using multilingual embedding, transfer weights in char-level, word-level, dense-level from Eng to Jpn; propose to romanize Jpn char on input.\
<https://www.aclweb.org/anthology/N19-2023/>

**Cross-lingual Structure Transfer for Relation and Event Extraction**. Subburathinam et al. EMNLP'19\
Use POS and parsing features with GCN on zero-shot extraction task.\
<https://www.aclweb.org/anthology/D19-1030/>
