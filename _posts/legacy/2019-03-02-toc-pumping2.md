---
layout: post
title: "Theory of Computing (5): The Pumping Lemma and Ogden's Lemma"
date: 2019-03-02
categories: [Theory of Computing]
tags: [regular language]
math: true
---

This article is part of my review notes of "Theory of Computation" course. It discusses the **Pumping Lemma** for context-free language; **Ogden's Lemma** is also introduced as a more generalized way to prove non-context-free language.

Similar to the pumping lemma for regular language (see last post), there is also another pumping lemma that can be used to prove non-context-free language. However, same limitation also applies here. When the pumping lemma doesn't work, we can try Ogden's lemma, which is a generalized version of the original pumping lemma.

## The Pumping Lemma for Context-Free Language

![pumping-lemma](/assets/img/legacy/pl1.png)

Similar to the pumping lemma for regular language, here we also have a pumping length $p$, which is closely bound with the grammar.

![pumping-lemma](/assets/img/legacy/pl2.png)
![pumping-lemma](/assets/img/legacy/pl3.png)

Notice that we set the pumping length $p = b^{\vert V\vert +1}$.

Based on the pumping lemma, we have the following two theorems:
* context-free language $\to$ $p$ exists
* $p$ doesn't exist $\to$ non-context-free language

Same as the case for regular language, we can claim a language is not context-free language if such $p$ doesn't exist. However, if $p$ exists, the language may or may not be context-free language. In other words, the pumping lemma is only significant if such $p$ doesn't exist.

When the pumping lemma doesn't work, we can try to use Ogden's lemma.

## Ogden's lemma

Ogden's lemma is a generalized form of the above pumping lemma, and can be used to show that certain languages are not context-free in cases where the pumping lemma is not sufficient.

![pumping-lemma](/assets/img/legacy/pl4.png)

When every position is marked, it is equivalent to the original pumping lemma.

##### Example to Prove Non-Context-Free Language

Here we go through an example when the pumping lemma fails and how to use the Ogden's lemma to prove non-context-free language.

We will use a game version of the pumping lemma, which is equivalent to the pumping lemma but more verbose.

![pumping-lemma](/assets/img/legacy/pl5.png)

Consider the language $L = \{a^i b^j c^k d^l: i=0 \text{ or } j=k=l\}$. Prove that $L$ is non-context-free language.

Let's first see if we can use the pumping lemma; if the pumping length $p$ doesn't exist, then we can claim $L$ is not context-free. However, such $p$ does exist as shown below.

1. C can choose any integer $p \geq 1$.
2. N chooses a string $s \in L$ such that $\vert s\vert  \geq p$.
3. C chooses strings $u, v, x, y, z$ such that $s = uvxyz$. Let $u = x = y = \epsilon$, $v$ be the first letter of $s$, $z$ be the remaining string. Therefore, $\vert vxy\vert  = 1 \leq p$, $\vert vy\vert  = 1 > 0$.
4. Whatever integer $m$ that N chooses, $u v^i x y^i z$ is always $\in L$. Therefore, C always wins. Proof:
   * If $v = a$, s has the following form: $\{ a^i b^j c^j d^j \vert  i > 0 \}$. Therefore, the pumping string $s' = u v^i x y^i z$ has the following form: $\{ a^{m+i-1} b^j c^j d^j \vert  i > 0, m \geq 0 \}$ which is equivalent to $\{ a^i b^j c^j d^j \vert  i \geq 0\}$. According to $L$, $s' \in L$.
   * If $v = b$, s has the following form: $\{ b^j c^k d^l \vert  j > 0 \}$. Therefore, the pumping string $s' = u v^i x y^i z$ has the following form: $\{ b^{m+j-1} c^k d^l \vert  j > 0, m \geq 0 \}$ which is equivalent to $\{ b^j c^k d^l \vert  j \geq 0 \}$. According to $L$, $s' \in L$.
   * If $v = c$, s has the following form: $\{ c^k d^l \vert  k > 0 \}$. Therefore, the pumping string $s' = u v^i x y^i z$ has the following form: $\{ c^{m+k-1} d^l \vert  k > 0, m \geq 0 \}$ which is equivalent to $\{ c^k d^l \vert  k \geq 0 \}$. According to $L$, $s' \in L$.
   * If $v = d$, s has the following form: $\{ d^l \vert  l > 0 \}$. Therefore, the pumping string $s' = u v^i x y^i z$ has the following form: $\{ d^{m+l-1} \vert  l > 0, m \geq 0 \}$ which is equivalent to $\{ d^l \vert  l \geq 0 \}$. According to $L$, $s' \in L$.

Since $p$ exists, we have to try the Ogden's lemma. We also use a game version here:

1. C chooses an integer $p \geq 0$.
2. N chooses a string $s \in L$ such that $\vert s\vert  \geq p$, and mark any $q$ positions in $s$ where $q \geq p$.
3. C chooses strings $u, v, x, y, z$ where $s = uvxyz$, such that:
   * $vy$ has at least one marked position.
   * $vxy$ has at most $p$ marked positions.
   * N chooses an integer $i \geq 0$ such that $u v^i x y^i z \notin L$.

Now we can see such $p$ doesn't exist.

1. C chooses an integer $p \geq 0$.
2. N chooses the string $s \in L$: $a b^p c^p d^p$. $\vert s\vert  = 3p+1 > p$. Mark the last $p$ positions in $s$, so that the first letter $a$ is always not marked.
3. C chooses strings $u, v, x, y, z$ where $s = uvxyz$, such that:
   * $vy$ has at least one marked position.
   * $vxy$ has at most $p$ marked positions.
4. N can choose the integer $i = 2$, and $u v^i x y^i z \notin L$. Proof:
   * C cannot choose $v$ or $y$ that involves more than one type of letter. Otherwise, the letter arrangement of the pumped string will be out of order, so the new string will not be $\in L$.
   * C cannot choose to pump letter $b$ or $c$ or $d$. Because C can only choose two types of letter, the resulting pumped string will not have equal length of $b, c, d$, and will not be $\in L$.
   * Therefore, C can only choose to pump the first letter $a$ to keep the resulting string valid. Therefore, C has two potential choices: $v = a$, $y = \epsilon$; or $v = \epsilon$, $y = a$. However, because $a$ is not marked, neither choice satisfies the condition: $vy$ has at least one marked position. Therefore, there is no choice that keeps the pumped string in $L$.

Now we can claim $L$ is not context-free since such $p$ doesn't exist in the Ogden's lemma.
