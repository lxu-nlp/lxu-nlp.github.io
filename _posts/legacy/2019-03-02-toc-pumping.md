---
layout: post
title: "Theory of Computing (4): The Pumping Lemma and Myhill–Nerode Theorem"
date: 2019-03-02
categories: [Theory of Computing]
tags: [regular language]
math: true
---

This article is part of my review notes of "Theory of Computation" course. It discusses the **Pumping Lemma** for regular language; **Myhill–Nerode Theorem** is also introduced as a more powerful way to prove regular language.

We can use the pumping lemma to prove a certain language is not regular language. However, there is limitation; when the pumping lemma doesn't work, we can use Myhill–Nerode Theorem as an universal way to prove regularity.

## The Pumping Lemma for Regular Language

![pumping-lemma](/assets/img/legacy/pump1.png)

$p$ is called the **pumping length**. The existence of $p$ is tightly bound with DFA, as any regular language has an underlying DFA representation.

![pumping-lemma](/assets/img/legacy/pump2.png)

The proof is straightforward: for any regular language, if we set the $p$ to be the number of states in DFA, then any computation path with length $p$ must have a repeating state; then we have just pump the string between the repeating state by any times.

According to the pumping lemma, we have the following two theorems:
* regular language $\to$ $p$ exists
* $p$ doesn't exist $\to$ irregular language

By using the second theorem, we can claim a language is not regular language if such $p$ doesn't exist. However, notice that the following is NOT true: p exists $\to$ regular language, which means the pumping lemma might fail for some irregular languages (example is shown later). In that case, we can use Myhill–Nerode Theorem, which will always work.

## Myhill–Nerode Theorem

Full theorem: $L(M)$ is regular **iff** $M$ has a finite number of equivalence classes.

Here we will not prove the whole theorem, instead we prove two lemmas that are enough for us to argue a certain language is regular or not.

First look at some definitions used in the theorem.

Given state $p \in Q$ and input string $x \in \Sigma^\ast$, let $\delta^∗(p,x)$ denote the state that we reach, if we start in state $p$ and read string $x$. Suppose we have two states $p, q \in Q$. A **distinguishing string** (for $p$ and $q$) is some $z \in \Sigma^\ast$ such that exactly one of the two states $\delta^\ast (p, z)$ and $δ^\ast(q, z)$ is in $F$. If there is no such $z$, then we say the states $p$ and $q$ are **M-equivalent**, and we denote this relation by $p \; R_M \; q$; if there is such $z$, then we say $p$ and $q$ are **distinguishable**.

Similarly, if there is some string $z \in \Sigma^\ast$ such that exactly one of the two strings $xz$ and $yz$ is in $L$, we say $x$ and $y$ are distinguishable while $z$ being the distinguishing string; if there is not such $z$, then we say that strings $x$ and $y$ “L-equivalent”, and we denote this relation by $x \; R_L \; y$.

For convenience, denote $q_M (x) = \delta^\ast (q_0, x)$, meaning $q_M (x)$ is the state $M$ reaches if it starts in its initial state $q_0$, and reads the input string $x$.

**Lemma A**: suppose $M$ is a DFA, $L = L(M)$, and $x$ and $y$ are strings in $\Sigma^\ast$. Argue that if $q_M (x) \,R_M\, q_M (y)$, then $x \,R_L\, y$.

Contrapositive: if strings $x$ and $y$ are distinguishable, then $q_M (x)$ and $q_M (y)$ are distinguishable. Proof as following:

$x$ and $y$ are distinguishable
$\implies$ there exists a string $z$ such that exactly one of $xz$ and $yz$ is in $L$
$\implies$ exactly one of $q_M (xz)$ and $q_M (yz)$ is in $F$ (because $xz, yz \in L$)
$\implies$ exactly one of $\delta^\ast(q_M (x), z)$ and $\delta^\ast(q_M (y), z)$ is in $F$
$\implies$ $q_M (x)$ and $q_M (y)$ are distinguishable

Contrapositive is true; therefore, if $q_M (x) \,R_M\, q_M (y)$, then $x \,R_L\, y$.

**Lemma B**: suppose $L$, $S \subseteq \Sigma^\ast$. Suppose that for every pair of distinct strings $x$ and $y$ in $S$, there is a distinguishing $z$ (in other words, $x$ and $y$ are not $L$-equivalent). Argue that any DFA for $L$ has at least $\vert S \vert$ states.

First, for any distinguishable state $p$ and $q$ of DFA, $p \neq q$. Prove by contrapositive: if $p = q$, then for any string $z$, $\delta^\ast (p, z) = \delta^\ast (q, z)$, then $p$ and $q$ are either both in $F$ or both not in $F$, therefore $p$ and $q$ are not distinguishable.

for every pair of distinct strings $x$ and $y$ in $S$, $x$ and $y$ are distinguishable
$\implies$ for every pair of distinct strings $x$ and $y$ in $S$, $\delta^\ast (q_0, x)$ and $\delta^\ast (q_0, y)$ are distinguishable (by lemma a)
$\implies$ for every pair of distinct strings $x$ and $y$ in $S$, $\delta^\ast (q_0, x) \neq \delta^\ast (q_0, y)$ (by above proof)
$\implies$ each string $x_i$ in $S$ has its own distinct state $\delta^\ast (q_0, x_i)$ in $L$
$\implies$ there are at least $|S|$ states in $L$

## Example to Prove Irregular Language

Here we go through an example when the pumping lemma fails and how to use the lemma B to prove irregularity.

We will use a game version of the pumping lemma, which is equivalent to the pumping lemma but more verbose.

![pumping-lemma](/assets/img/legacy/pump3.png)

Consider the language $F = \{ a^i b^j c^k \vert i, j, k \geq 0 \text{ and if } i = 1 \text{ then } j = k \}$. Prove $F$ is irregular.

Let's first look at if we can prove by using the pumping lemma; if the pumping length $p$ doesn't exist, then $F$ will not be regular. However, such $p$ actually exists, as shown below:

1. R can choose any integer $p \geq 2$.
2. N chooses a string $s \in F$ such that $\vert s \vert \geq p$.
3. R chooses $x, y, z$ such that $s = xyz$. Let $x = \epsilon$; $y$ be $aa$ if $s$ has the form $\{ aa b^m c^n \}$, otherwise be the first letter of $s$; $z$ be the remaining string. Therefore, $\vert xy \vert \leq 2 \leq p$; $\vert y\vert  \geq 1 > 0$.
4. For whatever $i$ that N chooses, $x y^i z$ will always $\in F$; therefore, R can always win. Proof:
   * If $y = aa$, $s$ has the following form: $\{ aa b^m c^n \}$. Therefore, the pumping string $s' = x y^i z$ has the following forms: $\{ a^{2i} b^m c^n \vert  i \geq 0 \}$. According to $F$, because the length of $a$ is not $1$, $s' \in F$.
   * If $y = a$, $s$ can be one of the following forms: $\{ a b^m c^m \}$ or $\{ a^k b^m c^n \vert  k > 2 \}$. Therefore, the pumping string $s' = x y^i z$ has one of the following forms: $\{ a^i b^m c^m \vert  i \geq 0 \}$, or $\{ a^{k+i-1} b^m c^n \vert  i \geq 0, k > 2 \} $ which is equivalent to $\{ a^k b^m c^n \vert  k \geq 2 \}$. According to $F$, for either case, $s' \in F$.
   * If $y = b$, $s$ has the following form: $\{ b^m c^n \vert  m > 1 \}$. Therefore, the pumping string $s' = x y^i z$ has the form: $\{ b^{m+i-1} c^n \vert  i \geq 0, m > 1 \}$. According to $F$, $s' \in F$.
   * If $y = c$, $s$ has the following form: $\{ c^n \vert  n > 1 \}$. Therefore, the pumping string $s' = x y^i z$ has the form: $\{ c^{n+i-1} \vert  i \geq 0, n > 1 \}$. According to $F$, $s' \in F$.

Notice that even though $p$ exists, it doesn't mean $F$ is regular language. Since the pumping lemma doesn't work in this case, let's use the above lemma B to prove the irregularity. The key is to construct $S$.

1. Let $S =\{ a b^i \vert  i \geq 0 \}$. For every pair of strings $x, y \in S$, there exists a string $z$ that distinguishes $x, y$.
   * Proof: for any pair of distinct strings $x = ab^i, y = ab^j$ ($i \neq j$), let $z = c^k$ where $k = \max (i, j).$ If $k = i$, then $xz \in F$ and $yz \notin F$; if $k = j$, then $xz \notin F$ and $yz \in F$. Therefore, for any pairs of distinct strings $x, y$, there exists a distinguishing string $z$ such that exactly one of $x, y$ is in $F$.
2. $\vert S\vert  = \infty$ because the choice of $i$ in $S$ is infinite. According to the lemma, any DFA for $F$ has at least $\vert S\vert $ states, which means any DFA for $F$ needs to have infinite states, therefore no such DFA for $F$ exsits; therefore, $F$ is not regular.
