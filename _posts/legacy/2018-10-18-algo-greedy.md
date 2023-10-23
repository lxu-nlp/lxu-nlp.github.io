---
layout: post
title: "Algorithms (4): Greedy Algorithm and Matroid"
date: 2018-10-18
categories: [Algorithms]
tags: [algorithm, greedy]
math: true
---

This article is part of my review of Algorithms course. It introduces the concept of **Matroid**, and its relation with **Greedy** algorithm: greedy algorithm can get to the optimal solution if and only if the problem space is a Matroid.

## Matroid

[Matroid](https://en.wikipedia.org/wiki/Matroid) is a mathematical concept and is defined on abstract space, and there are multiple equivalent definitions. Here we only deal with the definition that is related with the greedy algorithm.

A finite Matroid is a pair $(E, I)$, where $E$ is a ground set (all elements), $I$ is a type of subsets (called Independent Sets) of $E$, such that it follows three properties:

1. $\emptyset \in I$, meaning empty set is independent.
2. Every subset of an independent set is independent.
3. If $A, B \in I$, and $|A| > |B|$, there exists $x \in A \backslash B$, such that $B \cup \{x\} \in I$. (Augmentation Property)

## Greedy Algorithm and Matroid

Applying greedy algorithm sometimes generates optimal solutions, but sometimes doesn't. Can we identify a common pattern, such that applying greedy algorithm on any problem that follows this pattern, will always lead to the optimal solution?

The answer is Matroid! It turns out that not only it is a sufficient condition, it is also a necessary condition: if a problem can be solved by greedy algorithm optimally, it has to be a Matroid.

Here is the statement: for any pair $(E, I)$, the below greedy algorithm returns the maximum-weight base **if and only if** $M = (E, I)$ is a matroid.

1. Arrange the elements in $E=\{1, 2, ..., n\}$ so that their weights are in decreasing order.
2. $S =\emptyset$
3. for $i \leftarrow 1$ to $n$ do:
4. \- if $S + i \in I$ then:
5. \- \- $S \leftarrow S + i$
6. return $S$

## Analysis

Here I am only going to show the proof for the sufficient condition, which is fairly straightforward; the necessary condition is a bit harder to prove. For a full proof of the statement, you can refer to this.

We show the sufficient condition by induction: after each iteration, $S_i$ must be a subset of a maximum-weight set $B_i$. Before the current iteration $i$, we have $S_{i-1} \subset B_{i-1}$. For the current iteration $i$, there are two possible outcomes:

1. $i$ is not added into $S_i$, then we can take $B_i = B_{i-1}$. Because if $i$ is not added into $S_i$, it means $S_{i-1} + i \notin I$. By the property 2 of Matroid, $B_{i-1} + i \notin I$, so $B_i$ must not contain $i$.
2. $i$ is added into $S_i$:
   * If $i \in B_{i-1}$, then we can just take $B_i = B_{i-1}$.
   * If$i \notin B_{i-1}$: by the property 3 of Matroid, we can add elements into $S_i$ till it has the same size as $B_{i-1}$. Denote the new set as $S'$. Since $|S'| = |B_{i-1}|$, and $i \in S'$ but $\notin B_{i-1}$, we have $S' = B_{i-1} + i - k$, where $k$ is an element that is larger than $i$. Because we arrange the element by weight-decreasing order, $w(i) > w(k)$. Therefore, $S'$ now has more weight than $B_{i-1}$. We take $B_i = S'$, and $S_i \subset B_i = S'$ which satisfies the induction.

Therefore, if a problem is a Matroid, greedy algorithm can always give the optimal solution.

## Example

Even though we know Matroid leads to optimal solution, we still need to "manually" find out the "independent set" of the problem, in order to claim it is a Matroid. Let's now see an example, that how we identify the independent set, and use Matroid to solve the problem.

A task scheduling problem with analysis: <https://drive.google.com/file/d/1FGweeSh5TPXXYfaESsGuvftU-lWQDIim>
