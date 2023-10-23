---
layout: post
title: "Theory of Computing (6): Turing Machine and Decidability"
date: 2019-03-05
categories: [Theory of Computing]
tags: [decidability]
math: true
---

This article is part of my review notes of “Theory of Computation” course. It introduces the concept of **Turning Machine**, **recognizable** and **decidable** languages. On the textbook chapter 3, there are some interesting related topics that I found interesting but won't be covered here.

## Turing Machine

The Turning machine is similar to DFA but with an infinite tape serving as unlimited memory. There is a tape head that can read and write symbols and move around on the tape. One difference from DFA or CFG is that the input for the Turing machine is actually on the tape: initially the tape contains only the input string and is blank everywhere else. When the machine starts, it will move around its states, tape head, and write content on the tape according to the transition function. When it goes into any accept or reject states, the machine terminates immediately.

Formally, the definition of the Turing machine is as follows:

![turing](/assets/img/legacy/turing1.png)

It is worth noting that the Turing machine simulates the random-access machine (RAM), or the general computer. However, Turing machine has limitation; later we will see some problems are not solvable by this model.

## Recognizable/Decidable Language

A language $L(M)$ is called recognizable if some Turing machine $M$ accepts(recognizes) it.

Recognizable languages intuitively include all regular languages and context-free languages, since the Turing machine is able to simulate DFA and PDA.

When we give an input to the machine, there are three outcomes:
* Explicit accept (eventually goes into accept state)
* Explicit reject (eventually goes into reject state)
* Non-halting (Infinite looping without going into either accept or reject state)

Because there can be infinite loop, when the machine is running on some input, it is hard to tell whether the machine will eventually halt. So we would prefer a more practical machine that always halts. We call this kind of always-halting machine "decider".

A language is called **decidable** if some decider recognizes it.

### Variants of Turing Machine

There are different variants of Turing machine: multitape Turing machine, nondeterministic Turing machine... But what is important is that they can all simulate each other, therefore they are equivalent and all agree on recognizable and decidable. Besides, the conversion between them has polynomial complexity; except that nondeterminism requires exponential complexity.

There is also another kind of variant called enumerator. It is also equivalent to the Turing machine and details are skipped here.

## Decidable Language

Since practically we prefer decidable language, we want to examine several following problems.

First, some problem definitions.

$$A_{DFA} = \{ <B, w> | B \text{ is a DFA that accepts input string } w \}$$

If $A_{DFA}$ is decidable, it means that the testing of whether $B$ accepts $w$ is decidable.

$$E_{DFA} = \{ A \text{ is a DFA and } L(A) = \emptyset \}$$

Here we determine whether the DFA accepts any string at all.

$$EQ_{DFA} = \{ <A, B> | \text{A and B are DFAs and } L(A) = L(B) \}$$

Here we determine whether two DFAs accepts the same language.

Similarly, we can also define $A_{NFA}$, $A_{REX}$, $A_{TM}$, $E_{CFG}$, $EQ_{CFG}$, etc.

### Regular Language:

* $A_{DFA}$: decidable. (just run DFA and take the output)
* $A_{NFA}$: decidable. (NFA is equivalent to DFA)
* $A_{REX}$: decidable. (regular expression is equivalent to DFA)
* $E_{DFA}$: decidable. (graph traversal; check if any final states are reachable)
* $EQ_{DFA}$: decidable. (construct symmetric difference and convert to $E_{DFA}$)

### Context-Free Language:

* $A_{CFG}$: decidable. (convert to CNF and parse; "CYK" algorithm can do $O(n^3)$)
* $A_{PDA}$: decidable. (PDA is equivalent to CFG)
* $E_{CFG}$: decidable. (traverse on the grammar from terminals to start variables; check if any start variables are reachable)
* $EQ_{CFG}$: NOT decidable. (proof is shown in later chapter)

### Recognizable Language:

$A_{TM}$: NOT decidable. (proof see my next post)

## Unrecognizable Language

So far, all of the languages presented above are recognizable, even though some of them are not decidable. Here we show that there exists a language that is not even recognizable. But first let's look at a definition and a theorem.

A language $\bar{A}$ is **co-recognizable** if it is the complement of a recognizable language $A$.

Theorem: a language is decidable **iff** it is both recognizable and co-recognizable.

**decidable $\to$ recognizable and co-recognizable**: if $A$ is decidable, then $A$ is recognizable (any decidable language is recognizable by definition). Then $\bar{A}$ is recognizable (just swap accept and reject states).

**recognizable and co-recognizable $\to$ decidable**: let $M_1$ be the recognizer for $A$, $M_2$ be the recognizer for $\bar{A}$, let $M$ be the decider for $A$. On any input string $w$, let $M_1$ and $M_2$ run in parallel. Since $w$ either $\in A$ or $\in \bar{A}$, one of $M_1$ and $M_2$ will eventually halt and accept. If $M_1$ accepts, then $M$ accepts; if $M_2$ accepts, then $M$ rejects. Therefore, $M$ will always halt so it is a decider.

### Corollary

$\overline{A}_{TM}$ is not recognizable.

Proof: if $\overline{A}_{TM}$ is recognizable, and we know $A_{TM}$ is recognizable, then according to the theorem, $A$ is decidable, which contradicts the previous conclusion. Therefore, $\overline{A}_{TM}$ is not recognizable.
