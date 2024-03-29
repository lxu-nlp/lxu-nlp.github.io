---
layout: post
title: "Algorithms (3): More on Tree Decomposition and Baker's Technique"
date: 2018-10-18
categories: [Algorithms]
tags: [algorithm, dp]
math: true
---

This article is part of my review of Algorithms course. It further discusses tree decomposition and introduces **Baker's technique**, which gives a Polynomial-Time Approximation Scheme ([PTAS](https://en.wikipedia.org/wiki/Polynomial-time_approximation_scheme)) on Maximum Independent Set ([MIS](https://en.wikipedia.org/wiki/Independent_set_(graph_theory))) problem. You can refer to my previous review for the introduction of tree decomposition and MIS problem.

## Baker's Technique

In my previous review, we knew that MIS problem is to find the maximum set of vertices that are not adjacent to each other, and it is NP-hard. And we saw a dynamic programming algorithm that solves MIS in linear time, if given the tree decomposition of graph. Baker's technique makes use of tree decomposition, and gives an MIS approximation in [planar graph](https://en.wikipedia.org/wiki/Planar_graph), in polynomial time.

The following describes Baker's technique on solving MIS problem in planar graph. Let $G=(V, E)$ be any planar graph, OPT be the optimal MIS, and let $\varepsilon$ be a real number we choose in $(0, 1)$. The output is $\vert  I \vert  \geq (1-\varepsilon)OPT$.

1. Let $k=\lceil 4/\varepsilon \rceil$.
2. Pick arbitrary root $r$ in $G$. For each vertex $v_i \in V$, mark $d_i = $ distance from $v_i$ to root $r$. This can be achieved by BFS.
3. Partition the graph $G$ into $k$ parts, such that $V_j = \{ v: d_v \equiv j\;(mod\;k) \}$. We have $V_0, V_1, ..., V_{k-1}$.
4. Pick $m$ such that $\vert V_m\vert $ is the minimum set.
5. Get $G' = G - V_m$, so that $G'$ is the result of removing $V_m$ and all their adjacent edges from $G$. It can be proved 1 that each component of $G'$ is a k-outerplanar graph. It can also be proved 2 that the treewidth of k-outerplanar graph is less than $3k$.
6. Use the DP algorithm to compute MIS on $G'$, given the tree width is $3k$ at most. Return the MIS.

Regarding proof 1 and 2, it is outside the scope of my study, but you can refer to the paper if you are interested: [Planar Graph with Bounded Treewidth](https://dspace.library.uu.nl/handle/1874/16567).

## Analysis on Approximation Ratio

How does Baker's technique give an approximation ratio of $(1-\varepsilon)$?

Simply said, in last step we compute the MIS on $G' = G - V_m$, and get the approximated MIS $I$. Let $I^\ast$ be the optimal MIS. We can show that the size of $V_m$ has an upper bound: $\vert V_m \vert  \leq \varepsilon \vert  I^\ast \vert $. Then we have $\vert I\vert  \geq \vert I^\ast - V_m \vert  \geq (1-\varepsilon) \vert I^\ast \vert $.

The key part is to show $\vert V_m\vert  \leq \varepsilon \vert I^\ast\vert $:

1. Planar graph is 4-colorable, so the largest color class (which is an independent set) must have size $\geq \vert V\vert /4$. Therefore, the optimal MIS $\vert I^\ast\vert  \geq \vert V\vert 4$.
2. Since we pick the $V_m$ that has the smallet size, $V_m$ must have size $\leq \vert V\vert /k$.
3. Therefore, $\vert V_m\vert  \leq \vert V\vert /k \leq \varepsilon n/4 \leq\varepsilon \vert I^\ast\vert $.

## Analysis on Time Complexity

Step 1-5 takes linear time $O(\vert V\vert )$, because planar graph has edge size $O(\vert V \vert)$.

In step 6, the DP algorithm takes $O(\vert V\vert 2^k) = O(\vert V\vert 2^{1/\varepsilon})$. If we consider $k$ or $\varepsilon$ as just a hyperparameter we choose, then it is also linear time.
