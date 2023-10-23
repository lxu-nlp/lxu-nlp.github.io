---
layout: post
title: "Theory of Computing (7): More on Undecidable Languages; Linear Bounded Automaton"
date: 2019-05-06
categories: [Theory of Computing]
tags: [decidability]
math: true
---

This article is part of my review notes of “Theory of Computation” course. Last post we introduced the concept of Turing Machine, recognizable and decidable languages. This post continues the topic of decidability, and introduces several important **undecidable languages** by reduction. In addition, Linear Bounded Automaton (**LBA**) is also introduced.

## $A_{TM}$ Is Undecidable

We mentioned $A_{TM}$ is a basic undecidable language in the last post. Here is the proof by contradiction from the textbook.

$$A_{TM} = \{ <M,w> | \;M \text{ is a TM and } M \text{ accepts } w \}$$

Suppose $H$ is a decider for $A_{TM}$. We can construct a new Turing Machine $D$ using $H$:

$D$ = On input <$M$>, where $M$ is a TM:
1. Run $H$ on input <$M$, <$M$>>.
2. If $H$ accepts, reject; if $H$ rejects, accept. (always output the opposite)

Therefore, we have:

$$
D(<D>) =
\begin{cases}
accept & \text{if D does not accept <D>}\\
reject & \text{if D accepts <D>}
\end{cases}
$$

Therefore, such $H$ or $D$ cannot exist because of the contradiction; $A_{TM}$ is not decidable.

## More Undecidable Languages by Reduction

Since we know $A_{TM}$ is undecidable, we can derive more undecidable languages by **reduction**: if a language $A$ is decidable $\Rightarrow$ $A_{TM}$ is decidable, then $A$ is not decidable since $A_{TM}$ is not decidable.

### $HALT_{TM}$ Is Undecidable

$$HALT_{TM} = \{ <M, w> | \; M \text{ is a TM and } M \text{ halts on input } w\}$$

Suppose $R$ is the decider of $HALT_{TM}$. The objective is to construct a decider $S$ of $A_{TM}$ using $R$.

$S$ = On input $<M, w>$, where $M$ is a TM and $w$ is the input string,
1. Run $R$ on $<M,w>$.
2. If $R$ rejects ($M$ doesn't halt on $w$), reject.
3. Else, $M$ will halt on $w$. Run $M$ on $w$ until it halts.
4. If $M$ accepts, accept; if $M$ rejects, reject.

Therefore, $HALT_{TM}$ is decidable $\to$ $A_{TM}$ is decidable. Because we know $A_{TM}$ is not decidable, then $HALT_{TM}$ is not decidable.

### $E_{TM}$ Is Undecidable

$$E_{TM} = \{<M> | \; M \text{ is a TM and } L(M) =\emptyset \}$$

Same as $HALT_{TM}$, we want to construct a decider $S$ for $A_{TM}$ using the decider $R$ of $E_{TM}$.

To fully use the decider of $E_{TM}$, the trick here is to use a intermediate TM $M_1$ to confine the input to always be $w$. Therefore, if $R(< M_1 >)$ rejects, $L(M_1)$ can only be $w$ (so $M$ must have accepted $w$).

$M_1$ = On input $w$,
1. If $x \neq w$, reject.
2. If $x = w$, run $M$ on input $w$ and accept if $M$ accepts.

$S$ = On input $<M, w>$,
1. Use $M$ and $w$ to construct the corresponding $M_1$.
2. Run $R$ on $< M_1 >$.
3. If $R$ accepts, reject; if $R$ rejects, accept.

If $R$ accepts, then $L(M_1) = \emptyset$, so $M$ must have rejected $w$; if $R$ rejects, then $L(M_1) = w$, where $M$ must have accepted $w$; therefore $S$ is a decider for $A_{TM}$; therefore, $E_{TM}$ is not decidable.

### $EQ_{TM}$ Is Undecidable

$$EQ_{TM} = \{ <M_1, M_2> | \; M_1, M_2 \text{ are TMs and } L(M_1) = L(M_2) \}$$

Here we can use the reduction: $EQ_{TM}$ is decidable $\to$ $E_{TM}$ is decidable, since we just know that $E_{TM}$ is not decidable.

Let $R$ be the decider for $EQ_{TM}$, we can construct the decider $S$ for $E_{TM}$ as follows.

$S$ = On input $< M >$ where $M$ is TM,

1. Run $R$ on $<M, M_1>$ where $M_1$ is a TM that rejects all strings.
2. If $R$ accept, reject; else, accept.

This reduction is relatively straightforward. 

## Linear Bounded Automaton

A Linear Bounded Automaton (LBA) is a kind of TM that has limited tape length. The tape head cannot move outside from the available tape. This brings one direct benefit, that the total number of possible configurations are limited. If a LBA has $q$ states and $g$ symbols in the tape alphabet with tape length $n$, there are $qng^n$ distinct configurations for this LBA.

$A_{LBA}$ Is Decidable

Unlike $A_{TM}$, $A_{LBA}$ is decidable because of the limited number of possible configurations. We can construct a decider $L$ as follows.

$L$ = On input <M, w> where $M$ is a LBA and $w$ is input string,
1. Run $M$ on $w$ for at most $qng^n$ steps.
2. If $M$ has halted, accept or reject according to $M$; if $M$ hasn't halted, reject.

If after $qng^n$ steps, $M$ still hasn't halted, then there must exist a repeated configuration, which will lead to infinite loop; we can just reject in this case.

## Reduction Using Computation History

Here we introduce two more undecidable languages: $E_{LBA}$ and $ALL_{CFG}$.

$$E_{LBA} = \{ <M> | \; M \text{ is a LBA and } L(M) = \emptyset \}$$

$$ALL_{CFG} = \{ <G> | \; G \text{ is a CFG and } L(G) = \Sigma^\ast \}$$

The technique is similar: we reduce them to $A_{TM}$ which is not decidable. However, the key difference from the previous undecidable languages is that, $E_{LBA}$ or $ALL_{CFG}$ doesn't involve the TM $M$ in $A_{TM}$, but their own entities: LBA or CFG. We need a bridge to connect between their own entities and the TM $M$, and this bridge is the computation history of $M$.

### $E_{LBA}$ Is Undecidable

Similar to $E_{TM}$, to fully use the decider of $E_{LBA}$, we want to construct a LBA that only accepts some strings when $M$ accepts $w$, and these strings are computation history of $w$, serving as a bridge between the LBA and $M$.

Let $R$ be the decider of $E_{LBA}$. The decider $S$ of $A_{TM}$ is as follows.

$S$ = On input $<M, w>$, where $M$ is a TM and $w$ is the input string,

1. Construct a LBA $B$ using $M$ and $w$ that only accepts the accepting computation history of $w$ (if $M$ accepts $w$).
2. Run $R$ on $< B >$.
3. If $R$ rejects, accept; if $R$ accepts, reject.

Specifically, the LBA $B$ accepts the following strings:

1. Parse input string $w$ as configuration history $C_1, C_2, ..., C_l$. If not parsable, reject directly.
2. $C_1$ is the start configuration of $M$ on $w$: $q_0 w_1 w_2 ... w_n$, which is easy to check.
3. $C_l$ is the configuration for $M$ that contains $q_{accept}$, which is easy to check.
4. Each $C_{i+1}$ legally follows from $C_i$. Because there are finite states in $M$, the number of legal moves is finite. Therefore, it is feasible to check.

Therefore, if $S$ accepts, then $L(B) = \emptyset$, where $M$ must have rejected $w$; if $S$ rejects, then $L(B) = $ "accepting computation history of $w$", where $M$ must have accepted $w$. Therefore, $S$ is a decider for $A_{TM}$. Therefore, $E_{LBA}$ is not decidable.

Similar technique involving computation history can also be used for $ALL_{CFG}$.
