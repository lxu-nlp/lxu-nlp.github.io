---
layout: post
title: "Basics Recap"
date: 2022-09-02
categories: [Basics]
tags: [ML]
math: true
---

# Calculus Basics

Gradient: 斜率; therefore, going along with gradient will increase; thus, GD has form: $x - \eta \nabla f(x)$\
For non-smooth functions e.g. L1 reg, we can use proximal gradient descent.

Chain rule: let $y = f(g(x)) = f(u)$. The derivative of $x$ regarding $y$ is: $\frac{dy}{dx} = \frac{dy}{du} \cdot \frac{du}{dx}$

Multi-variate chain rule: if $y = f(g_1(x), g_2(x)) = f(u_1, u_2)$, The derivative of $x$ regarding $y$ is: $\frac{dy}{dx} = \frac{dy}{du_1}\frac{du_1}{dx} + \frac{dy}{du_2}\frac{du_2}{dx}$

Backpropagation: TODO

Talyor's Theorem (高阶导数逼近 near $a$): $f(x) \approx \sum_{k=0}^n \frac{f^{(k)}(a)}{k!} (x-a)^k $

### Convex Optimization

Lagrangian Duality:
* For a constrained optimization problem on $x$, convert to an unconstrained optimization problem that introduces $\lambda$.
* Solve a harder unconstrained optimization problem ($min_x$ conditioned on $max_{\lambda}$) to an easier unconstrained optimization problem ($max_{\lambda}$ conditioned on $min_x$).
  * See the formulation on $max_{\lambda}$: <https://www-cs.stanford.edu/people/davidknowles/lagrangian_duality.pdf>
  * Solution of the dual is a lower bound to the solution of the primal, and will be optimal under certain conditions.
    * Specifically, KKT conditions: complementary slackness (which says the extra added term should have no effect (= 0))
  * $max_{\lambda}$ conditioned on $min_x$ will always be convex/concave (regard $x$ as constant); therefore if $min_x$ is easy to get first (regard $\lambda$ as constant), then the problem is solved.
  * To get $min_x$ first, either:
    * Analytically: regarding $\lambda$ as constant and set derivative to 0 for each $x_i$, combined with KKT conditions.
    * Other numerical optimization method (iterative solution e.g. interior point method), combined with KKT conditions.
* <https://www.youtube.com/watch?v=uh1Dk68cfWs>
* <https://cs229.stanford.edu/section/cs229-cvxopt2.pdf>

Linear programming:
* Simplex method.

### SVM

SVM overlooks the ENTIRE feature space geometry and tries to find the best boundary,
different from logistic regression that only focuses on SINGLE point at a time.

Therefore, SVM leverages the point metric (distance/similarity/proximity);
linear SVM uses dot-product of feature vectors ($x_1 \cdot x_2$) as similarity metric.

To obtain more complex decision boundary, we need to map feature to higher dimensional space and obtain similarity: $f(x_1) \cdot f(x_2)$.\
However, we can circumvent "high-dim mapping then (simple) similarity" by "(low-dim) similarity then (simple) mapping", finding $g$ such that $f(x_1) \cdot f(x_2) = g(x_1, x_2)$ where $g$ is much simpler than $f$.\
$g$ is called the kernel that represents the similarity metric in higher-dim feature space.

* [Polynomial kernel](https://en.wikipedia.org/wiki/Polynomial_kernel)
* [RBF](https://en.wikipedia.org/wiki/Radial_basis_function_kernel): $\mathrm{exp}(-\gamma \Vert x_1 - x_2 \Vert^2)$ which can represent INFINITE feature space (as exponential function expands to infinite polynomial by Taylor series)

Solving: convert to dual problem then use numerical optimization

---

# Linear Algebra

A * Diag: elementwise multiplication per column\
Diag * A: elementwise multiplication per row

Eigenvalues/eigenvectors: matrix (linear transformation) will not change the direction of certain vectors (only changing the scale); these vectors and scalars can be the representatives of this transformation (matrix).

Eigendecomposition: $Av = \lambda v \implies AQ = Q\Lambda \implies A = Q\implies Q^{-1}$

definite matrix: if any vectors after a transformation (matrix) still has a positive projection to original, or $z^TAz > 0$ for any $z$.
* A matrix is positive definite if and only if all its eigenvalues are positive.
