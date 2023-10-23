---
layout: post
title: "Theory of Computing (10): The Cook-Levin Theorem, More NP-Complete Languages"
date: 2019-05-09
categories: [Theory of Computing]
tags: [np]
math: true
---

This article is part of my review notes of “Theory of Computation” course. In the previous post, we introduced the classes of **P, NP, and NP-complete**. Here we examine one important NP-complete language - **SAT**, and we will see more NP-complete languages by using reduction from SAT.

## The Cook-Levin Theorem

### Satisfiability Problem

A boolean formula $\phi$ is an expression involving Boolean variables and operations (and $\land$, or $\lor$, not $\overline{x}$). It is satisfiable if some assignment of 0s and 1s to the variables makes the formula evaluate to 1. For example: $\phi = (x_1 \lor x_2) \land (x_1 \lor \overline{x_2})$.

$$SAT = \{ <\phi> | \; \phi \text{ is a satisfiable Boolean formula}\}$$

**Theorem**: SAT is NP-complete.

To prove SAT $\in$ NP-complete, we need to prove the following two conditions:
1. SAT $\in$ NP. This is straightforward since SAT is polynomially verifiable.
2. For any language $A \in NP$, we have $A \leq_p SAT$. The rest of this section will focus on this part.

How do we connect any NP language to SAT? We need a bridge which should be a universal property of any NP languages. And this bridge is computation history. This is not the first time we use computation history, as we have already seen it in the previous post, where we use it as bridge to do reduction from $A_{TM}$.

The idea is conceptually straightforward. Suppose any NP language $A$ and its NTM $N$, we want to construct a boolean formula $\phi$ using $N$ and $w$, which is true iff $w$ is accepted by $N$. Therefore, the focus of our task is to encode the accepting criteria of the computation history into a boolean formula $\phi$.

Since $A \in NP$, the NTM $N$ decides $w$ in polynomial steps $O(n^k)$. We obtain the computation history on $w$ and arrange each configuration on a $n^k \times n^k$table, where each row is the configuration for one step.

![sat](/assets/img/legacy/sat.png)

For any cell [$i, j$] at row $i$ and column $j$, the possible symbols are: the configuration separator $\#$, blank symbol, tape symbol, and state symbol. Denote the set of possible symbols as $C$. For each cell [$i, j$] and each symbol $s \in C$, we create a boolean variable $x_{i,j,s}$ for $\phi$. If $x_{i,j,s} = 1$, then the cell [$i, j$] has symbol $s$. Therefore, there are $\vert C\vert(n^k)^2$ boolean variables in total.

For an accepting computation history, four requirements need to be fulfilled:
1. $\phi_{cell}$: each cell takes one and only one symbol out of all possible symbols $C$.
2. $\phi_{start}$: the first configuration is the start configuration w.r.t. $w$ and $N$.
3. $\phi_{accept}$: there needs to be a configuration with "accept" state.
4. $\phi_{move}$: each configuration is a valid transition from the preceding configuration.

If we have the Boolean formula for each part, then we can obtain the final $\phi =\phi_{cell} \land \phi_{start} \land \phi_{accept} \land \phi_{move} $. Let's see how to encode each requirements into $\phi$.

$$\phi_{cell} = \bigwedge_{1 \leq i,j \leq n^k } \Big[\big(\bigvee_{s \in C} x_{i,j,s}\big) \land \big(\bigwedge_{s,t \in C, s \neq t} (\overline{x_{i,j,s}} \lor \overline{x_{i,j,t}})\big)\Big]$$

Explanation: everything inside $[...]$ guarantees there is exactly one symbol for each cell. The first part $\bigvee_{s \in C} x_{i,j,s}$ means this cell has at least one symbol (not empty); the second part $\bigwedge_{s,t \in C, s \neq t} (\overline{x_{i,j,s}} \lor \overline{x_{i,j,t}})$ means this cell cannot have more than one symbol. Together, only when each cell has exactly one symbol can satisfy this Boolean formula $\phi_{cell} $.

$$\phi_{start} = x_{1,1,\#} \land x_{1,2,q_0} \land x_{1,3,w_1} \land ... \land x_{1,n^k, \#}$$

Explanation: the first configuration needs to be the start configuration, which is #$ q_0 w \#$. We just need the symbols on the first row to be the exact start configuration. Note that there could be many blank symbols in the end before the last #.

$$\phi_{accept} =\bigvee_{1 \leq i,j \leq n^k} x_{i,j,q_{accept}}$$

Explanation: we just need to find $q_{accept}$ in any cells. As long as a configuration contains $q_{accept}$, then it means this computation path is accepted by $N$.

$$\phi_{move} = \bigwedge_{1 \leq i < n^k, 1 < j < n^k} \big( (i,j) \text{-window is legal} \big)$$

where each window represents a local $2 \times 3$ area of the table, and each legal window can be represented as:

$$\bigvee_{a_1, ..., a_6 \text{ is a legal window}} \big( x_{i, j-1, a_1} \land x_{i, j, a_2} \land x_{i, j+1, a_3} \land x_{i+1, j-1, a_4} \land x_{i+1, j, a_5} \land x_{i+1, j+1, a_6} \big)$$

Explanation: at each state, the tape head only writes one symbol and moves left or right. The effect on the configuration is therefore confined to a $2 \times 3$ window. Since there are finite states in any valid Turing machine model, we can represent every legal transition into the window Boolean formula. Then for the computation history, we just need to check whether every window is valid.

With each part figured out, we get the final Boolean formula:

$$\phi =\phi_{cell} \land \phi_{start} \land \phi_{accept} \land \phi_{move} $$

which is evaluated as True if and only if the configuration table is an valid accepting computation history, or equivalently, if and only if $w$ is accepted by $N$. Therefore, we proved that for any language $A \in NP$, we have $A \leq_p SAT$. Therefore, SAT is complete.

## More NP-Complete Languages by Reduction

From the previous post, we mentioned that we can get more NP-complete languages if we have a "base" NP-complete language. Specifically, if $B$ is NP-complete and $B \leq_p C$ for $C$ in NP, then $C$ is NP-complete. Let's see some examples here.

**Theorem**: 3SAT is NP-Complete.

We want to show the reduction SAT $\leq_p$ 3SAT, that is, $\phi \in$ SAT $\iff$ $\phi' \in $ 3SAT. Assuming SAT is in CNF (Conjunctive Normal Form). For each clause in SAT that has more than 3 literals, we can replace $(l_1 \lor l_2 \lor l_3 \lor l_4 \lor ...)$ with two clauses $(l_1 \lor l_2 \lor z) \land (\overline{z} \lor l_3 \lor l_4 ...)$, where $z$ is a new Boolean variable. Repeat till no clause has more than 3 literals. The Boolean formula $\phi$ is satisfiable iff the new Boolean formula is satisfiable. Therefore, since SAT is NP-complete, 3SAT is also NP-complete.

Some other reductions on the textbook:
* 3SAT $\leq_p$ CLIQUE $\leq_p$ VC (Vector Cover)
* 3SAT $\leq_p$ HAMPATH
* 3SAT $\leq_p$ SUBSET-SUM
