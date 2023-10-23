---
layout: post
title: "Algorithms (5): Greedy Approximation for Maximum Weight Matching and Set Cover"
date: 2018-10-19
categories: [Algorithms]
tags: [algorithm, greedy]
math: true
---

This article is part of my review of Algorithms course. It introduces greedy approximation algorithms on two problems: **Maximum Weight Matching** and **Set Cover**.

## Greedy Approximation Algorithm

Apart from reaching the optimal solution, greedy algorithm is also used to find an approximated solution as well. If a greedy approximation gives a certain ratio of optimal solution, then it is useful because greedy approximation often has lower time complexity.

We will see two examples in this article: Maximum Weight Matching and Set Cover; especially, Set Cover is a NP-Complete problem.

## Maximum Weight Matching

Let's first see the [Maximum Matching](https://en.wikipedia.org/wiki/Matching_(graph_theory)) problem: find the largest edge set in a graph such that no two edges are adjacent to each other (no two edges share a vertex).

![max-matching](/assets/img/legacy/max-matching.png)

We can generalize it to Maximum Weight Matching: each edge has a weight, and the target is to find the maximum weight edge set.

Maximum Weight Matching is solvable in polynomial time; however, greedy approximation can give us a solution in linear time, with an approximation ratio of 2. We can call the approximated solution as "Maximal Weight Matching".

Here is the greedy algorithm:

1. Arrange the edges $E = \{e_1, ..., e_n\}$ by weight in decreasing order.
2. $S =\emptyset$
3. for $i \leftarrow 1$ to $n$ do:
4. \- if $e_i$ doesn't share vertex with $S$ then:
5. \- \- $S \leftarrow S + e_i$
6. return $S$

### Analysis

It is easy to see the greedy algorithm runs in linear time, if edges are sorted. Next we want to know why it would give us an approximation ratio of 2, that is, $2 *$ \vert Maximal Matching\vert  $\geq$ \vert Maximum Matching\vert . Let's denote the maximal set as $S$, and maximum set as $S^\ast$. Here is the proof:

1. If an edge $e \in S^\ast$ is not in $S$, then there must be an edge $e' \in S$ such that $e'$ and $e$ shares a same vertex; because the greedy algorithm goes by weight-decreasing order, $w(e') \geq w(e)$. Therefore, for each $e \in S^\ast$, there exists an edge $e' \in S$ such that $w(e') \geq w(e)$.
2. Let $W^\ast = \sum w(e_i)$ where $e_i \in S^\ast$, and $W' = \sum w(e'_i)$ where $e'_i$ is the $e'$ for each $e_i$ selected by above step. We have $W' \geq W^\ast$.
3. The problem with $W'$ is, the edge set $\{e'_i\}$ can have duplicates. Let $W$ be the total weight of greedy edge set. Because each $e'$ can account for two edges in $S$, we have $W \leq W' \leq 2W$. Combined with $W' \geq W^\ast$, we have $2W \geq W^\ast$.

## Set Cover

The [Set Cover](https://en.wikipedia.org/wiki/Set_cover_problem) problem is NP-Complete. Here is a description of the problem:

- Input: $S_1, ..., S_m \subset B$, $\vert B\vert  = n$.
- Output: selection of $S_{i1}, ..., S_{ik}$ such that $\cup S_{ij} = B$, and $k$ being as small as possible.

Let $k^\ast$ be the number of optimal sets, $k$ be the number of sets selected by greedy algorithm. Greedy algorithm can give us an approximation ratio of $\ln n$, that is, $k \leq k^\ast \ln n$.

The greedy algorithm is very intuitive: in each iteration, select the set that has the maximum uncovered elements; keep iterating until we cover all elements.

### Analysis

1. In each iteration, the selected set $S_{ij}$ covers at least $1/k^\ast$ of remaining uncovered elements. This is because at least one of the unknown optimal set covers $1/k^\ast$ of remaining uncovered elements; since the selected $S_{ij}$ covers the maximum uncovered elements, $S_{ij}$ performs at least as well as that unknown optimal set.
2. Denote $m$ as the number of uncovered elements. From above step, we have $m \leq n(1-k^\ast)^t$ for iteration t. Since $m \leq n(1-k^\ast)^{k^\ast \ln n} < ne^{-(k^\ast/k^\ast) \ln n} = 1$, after iterating $k^\ast \ln n$ times, there must be 0 uncovered element.

Therefore, the size of sets selected by the greedy algorithm is at most $k^\ast \ln n$.
