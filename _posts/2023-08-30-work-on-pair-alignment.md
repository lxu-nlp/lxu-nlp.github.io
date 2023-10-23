---
layout: post
title: "Work on Pair Alignment"
date: 2023-08-30
categories: [NLP]
tags: [nlp, alignment]
math: true
---

Pair alignment: bilingual sentence alignment, context-summary alignment, ...

# Bilingual Sentence Alignment

Need finer granularity: 1-1 over 3-3.

**A program for Aligning Sentences in Bilingual Corpora**. Gale and Church. 1993\
DP algorithm: search over all possible alignment (search space: pruned by length).\
<https://aclanthology.org/J93-1004/>

**Fast and accurate sentence alignment of bilingual corpora**. Moore. 2002\
DP algorithm (BleuAlign): dynamic length pruning (rather than the limited fixed pruning).\
<https://aclanthology.org/2002.amta-papers.14/>

**Vecalign: Improved Sentence Alignment in Linear Time and Space**. Thompson and Koehn. EMNLP'19\
Linear DP approximation.\
<https://aclanthology.org/D19-1136>
