---
layout: post
title: "Theory of Computing (8): Mapping Reducibility, Turing Reducibility, Kolmogorov Complexity"
date: 2019-05-07
categories: [Theory of Computing]
tags: [reducibility]
math: true
---

This article is part of my review notes of “Theory of Computation” course. In this post we will focus on two kinds of reduction: **Mapping Reducibility** and **Turing Reducibility**. Their properties are useful to prove recognizable languages or decidable languages, as we have seen in the previous post about undecidable languages. In addition, a side topic of **Kolmogorov Complexity** is also introduced.

## Computable Functions

Let's get to the definition on the textbook.

A function $f: \Sigma^\ast\longrightarrow \Sigma^\ast$ is a computable function if some Turing machine $M$, on every input, halts with just $f(w)$ on its tape.

The computable function always halts on any input. The concept of it is similar to the general program on the computer.

## Mapping Reducibility

Definition: language $A$ is **mapping reducible** to language $B$, written $A \leq_m B$, if there is a computable function $f: \Sigma^\ast\longrightarrow \Sigma^\ast$, where for every $w$,
$$w \in A \iff f(w) \in B$$
The function $f$ is called the reduction from $A$ to $B$.

![reduce](/assets/img/legacy/reduce1.png)

To prove a function $f$ is a reduction from $A$ to $B$, two properties need to be satisfied:
1. $w \in A \to f(w) \in B$
2. $f(w) \in B \to w \in A$, or equivalently, $w \notin A \to f(w) \notin B$

Notice that this is a "many-to-one" reduction from $A$ to $B$. The reverse direction $B \leq_m A$ is not correct unless the reduction is "one-to-one".

The intuition of the mapping reducibility $A \leq_m B$ is, since $A$ can be reduced to $B$, then $A$ is no harder than $B$. "$\leq_m$" orders languages by "hardness".

Some properties of mapping reducibility $A \leq_m B$:
* B is recognizable/decidable $\to$ A is recognizable/decidable
* A is not recognizable/decidable $\to$ B is not recognizable/decidable
* Equivalent to $\bar{A} \leq_m \bar{B}$
* Transitive property: if $A \leq_m B$ and $B \leq_m C$, then $A \leq_m C$

### Theorems Using Mapping Reducibility

**Theorem**: $EQ_{TM}$ is neither recognizable or co-recognizable.

It can be shown that $A_{TM} \leq_m EQ_{TM}$ and $A_{TM} \leq_m \overline{EQ_{TM}}$ (details are on textbook). Since $\overline{A_{TM}}$ is not recognizable (see previous post), neither $EQ_{TM}$ or $\overline{EQ_{TM}}$ is recognizable. Hence the theorem.

**Theorem**: $REGULAR_{TM}$ is neither recognizable or co-recognizable.

Proof: similar to $EQ_{TM}$, we can also show the reduction $A_{TM} \leq_mREGULAR_{TM}$ and $A_{TM} \leq_m \overline{REGULAR_{TM}}$.

## Turing Reducibility

### Oracle Turing Machine

An "oracle Turing machine" is a modified Turing machine that has the additional capability to query an oracle. Denote $M^B$ as an oracle Turing machine that has an oracle for language $B$.

### Turing Reducibility

Language $A$ is Turing reducible to language $B$, written in $A \leq_T B$, if there exists an oracle Turing machine $M^B$ that decides $A$.

**Theorem**: if $A \leq_m B$, then $A \leq_T B$.

We can let $M^B$ = On input $x \in A$,
1. Compute $y = f(x) \in B$.
2. Query the oracle for $B$. If $y \in B$, accept; else, reject.

This theorem shows that Turing Reduction is more general than Mapping Reduction.

## Kolmogorov Complexity

This is a side topic that is not strongly related with reduction. Here we are interested in the minimal description length. We describe a binary string $x$ with a Turing machine $M$ and a binary input $w$, where $M$ runs on $w$ and generates the output $x$. The length of the description of $x$ is $< M, w>$, and we just treat it as $< M >w$.

Special care is needed when we encode $M$, since we need to distinguish $< M >$ and $w$. A naive encoding scheme is to double every bit of $< M >$, use a delimiter "$01$" which is followed by $w$.

Definition: the minimal description of binary string $x$, denoted by $d(x)$, is the shortest string $\vert M, w>$ where TM $M$ on input $w$ halts with $x$ on its tape. If several such strings exist, select the lexicographically first among them. The Kolmogorov complexity or description complexity of $x$, denoted by $K(x)$, is $K(x) = \vert d(x)\vert $.

In other words, Kolmogorov complexity of $x$ is the length of the best compressed string of $x$.

**Theorem**: there exists $c$ such that for any $x$, $K(x) \leq \vert x\vert  + c$

Proof: Let $M$ be a TM that accepts any string directly. $K(x) = \vert d(x)\vert  = \vert <M, x>\vert  = \vert \vert  M >x\vert  = \vert < M >\vert  + \vert x\vert $. Therefore, we can pick $c = \vert  M \vert $.

This theorem gives an upper bound of the descriptive length of any string $x$.

## Incompressible Strings

Definition: we say a binary string $x$ is c-compressible if
$$K(x) \leq \vert x\vert  - c$$
If $x$ is not c-compressible, we say that $x$ is incompressible by c.
If $x$ is incompressible by 1, we say that $x$ is incompressible.

**Theorem**: incompressible strings of every length exist.

Proof:
The number of possible strings of length $n$ is $2^n$.
The number of all possible strings with length $0$ to $n-1$ is $\sum 2^i = 1 + 2 +... + 2^{n-1} = 2^n - 1$.
Since it requires one-to-one mapping, at least one string of length $n$ is not compressible.

The probability of a binary string $x$ being c-compressible $\leq \dfrac{2^{n-c+1}}{2^n} <  2^{1-c}$.

**Theorem**: the Kolmogorov complexity $K(x)$ is not computable.

Proof: skipped for now.
