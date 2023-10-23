---
layout: post
title: "Theory of Computing (1): Finite Automaton and Regular Language"
date: 2019-02-02
categories: [Theory of Computing]
tags: [automaton]
math: true
---

This article is part of my review notes of "Theory of Computation" course. It introduces the concept of Deterministic Finite Automaton (**DFA**), Nondeterministic Finite Automaton (**NFA**), **Regular Language**, Regular Operations. Especially, the relations between DFA, NFA and Regular Languages are discussed.

Definitions and examples are from the textbook "Introduction to the Theory of Computation" by Michael Sipser.

**Notation**:

$A \times B$: the Cartesian product or cross product of $A$ and $B$, is the set of all ordered pairs wherein the first element is a member of $A$ and the second element is a member of $B$.

$f: D \to R$: a function $f$ maps input in domain $D$ to output in domain $R$.

## Deterministic Finite Automaton (DFA)

DFA is a computational model with finite memories. The formal definition is:

![dfa](/assets/img/legacy/dfa.png)

Here, the transition function $\delta$ always gives output of one certain state. It is deterministic because given the current state and the input, we know exactly what the next state will be.

## Regular Language

![dfa](/assets/img/legacy/reg.png)

Simply said, starting from the initial state $q_0$, given the sequence of symbols, follow the state transition; if the finally reached state is one of the final states, then this string is accepted by $M$.

A language $L$ is called a regular language if every string $w \in L$ is accepted by some finite automaton $M$: $L = \{ w \vert M \text{ accepts w} \}$.

## Regular Operations

![dfa](/assets/img/legacy/reg-op.png)

Examples:

![dfa](/assets/img/legacy/reg-ops-example.png)

### Closure under Regular Operations

Regular languages are closed under the above three operations, which means the output language from the above operations are still regular.

To prove this, we need to introduce Nondeterministic Finite Automaton (NFA).

## Nondeterministic Finite Automaton (NFA)

![dfa](/assets/img/legacy/nfa.png)

Some notations: $\Sigma_{\varepsilon} = \Sigma \cup \{ \varepsilon \}$; $P(Q)$ is the collection of all subsets of $Q$.

### The difference between NFA and DFA:

* Given a certain input symbol, DFA always maps to one state, but NFA can maps to multiple states, or,  a set of states (hence nondeterministic).
* NFA can have state transition based on $\varepsilon$, which means state transition can happen without any input. 

## Equivalence of NFAs and DFAs

NFA is a more flexible representation of languages than DFA. To understand NFA, we can think about it as a tree expansion of DFA: after reading a symbol, the machine splits into multiple copies of itself and follows all the possibilities in parallel; if any one of these copies of the machine is in an accept state at the end of the input, the NFA accepts the input string.

The claim is NFA and DFA are equivalent. From the definition, we can see that DFA is a special case of NFA; now we need to prove any NFA can be converted to DFA that accepts the same languages. I found the proof idea on the textbook is very helpful:

![dfa](/assets/img/legacy/nfa-proof.png)

To build an equivalent DFA, following the above idea, we make each state of the DFA as a subset of $Q$. Given current state $S$ of DFA and input symbol $a$, the next state of DFA is the set of all states that are reachable from any states in $S$ on input $a$ or $\varepsilon$. Then final states of DFA are any states that contain the original final states in NFA.

Since each state of the newly built DFA is the subset of all states in the NFA, the total number of possible states in the new DFA is $2^N$ where $N$ is the number of states in the NFA. This means any conversion from NFA to DFA will have $O(2^N)$ increased complexity in the worst case.

## Proof on Closure under Regular Operations

If we can build finite automaton for the languages produced by the regular operations, we can automatically prove the closure. Since NFA is equivalent to DFA, we can use either of them to represent regular languages. Here we will see how easy it is to have NFA representation.

Union: $A_1 \cup A_2$; corresponding FAs: $N_1$ and $N_2$.

![dfa](/assets/img/legacy/reg-proof1.png)

Concatenation: $A_1 \circ A_2$; corresponding FAs: $N_1$ and $N_2$.

![dfa](/assets/img/legacy/reg-proof2.png)

Star: $A^\ast$

![dfa](/assets/img/legacy/reg-proof3.png)

it can also be proved that intersection operation $A_1 \cap A_2$ is also close to regular languages. We can actually prove both intersection and union operations by building product machine, with the only difference of final states: intersection takes product states that contain both original final states as its final states; union takes product states that contain either original final states as its final states.




