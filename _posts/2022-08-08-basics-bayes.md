---
layout: post
title: "Bayes Basics"
date: 2022-08-08
categories: [Basics]
tags: [bayes]
math: true
---

# Basic Distributions

### Binomial, Multinomial & Poisson Distribution

**Binomial**: likelihood of events for certain attempts.

$$
\begin{align}
f(x, n, p) &= P(x;n,p) = C_n^x \cdot p^x (1-p)^{n-x}\\
E(x) &= np
\end{align}
$$

$C_n^x$ is the combinations of $x$ occurrences among $n$ total.

**Multinomial**: generalization of binomial; likelihood of $k$ different events for certain attempts.

$$
\begin{align}
P(x_1, ..., x_k; p_1, ..., p_k, n) &= C_n^{x_1, ..., x_k} \cdot p_1^{x_1} \dots p_k^{x_k}\\
E(x_i) &= np_i
\end{align}
$$

$C_n^{x_1, ..., x_k} = \frac{n!}{x_1! \dots x_k!}$ is the combinations of according occurrences:
total combinations divided by each group's view of combinations.

**Poisson**: number of events for a certain time frame.

$$
\begin{align}
f(x, \lambda) &= P(x;\lambda) = \frac{\lambda^x e^{-\lambda}}{x!}\\
E(x) &= Var(x) = \lambda
\end{align}
$$

#### Possion vs. Binomial

* Similarity: both for modeling the number of certain random events within a certain time frame.
* Difference: binomial has n attempts (limited attempts), with a fixed independent probability for each attempt; poisson has
"unlimited" attempts, with a small fixed independent probability for each attempt.
That is, if let $n \rightarrow \infty$ and $p \rightarrow 0$ such that $np \rightarrow \lambda$, then binomial $\approx$ poisson.
* Therefore, poisson distributions are used to model occurences of events that could happen a very large number of times, but happen rarely.
That is, they are used in situations that would be more properly represented by a Binomial distribution with a very large $n$ and small $p$, especially when the exact values of $n$ and $p$ are unknown. (<https://math.stackexchange.com/questions/1050184/difference-between-poisson-and-binomial-distributions>)

### Other Distributions

**Geometric Distribution**: the probability of the first success at the $x$'th attempt.

$$
\begin{align}
f(x, p) &= P(x; p) = (1-p)^{x-1}p\\
E(x) &= \frac{1}{p}
\end{align}
$$

**T-Distribution**: similar to Normal Distribution, parameterized by observed mean and variance. More uniform (uncertain) if observation sample size is small.

---

# Bayesian Inference

### Predictive Posteriori

Instead of obtaining event likelihood distribution estimated from a single point of parameter, Bayesian performs inference by marginalizing/integral over the parameter distribution,
to get a more certain estimation (hence the concept of BNN to incorporate model parameter uncertainty), called the predictive posterior distribution.

The hard part is the parameter distribution, which is a parameter posterior distribution after some observations.

Above can be formulated as:

$$\underbrace{p(x' | X)}_{\text{predictive posteriori}} = \int p(x' | \theta) \underbrace{p(\theta | X)}_{\text{parameter posteriori}} d\theta $$

### Parameter Posteriori

To compute the parameter posteriori, we use the Bayes theorem:\
parameter posterior distribution $\propto$ event likelihood distribution under parameter $\cdot$ parameter prior distribution\
or more exactly:

$$
\begin{align}
\overbrace{p(\theta | X)}^{\text{parameter posteriori}} &= \frac{\overbrace{p(X | \theta)}^{\text{event likelihood}} \cdot \overbrace{p(\theta)}^{\text{parameter priori}}}{p(X)}\\
&= \frac{p(X, \theta)}{p(X)}\\
&= \frac{p(X | \theta) p(\theta)}{\int p(X | \theta') p(\theta') d\theta'}
\end{align}
$$

Note that both the numerator and denominator have the same ingredient: the joint distribution $p(X, \theta)$.

To obtain the posterior distribution, we usually have three choices in Bayes inference, as follows.

#### Closed-form solution: analytically based

If the joint probability (priori $\cdot$ likelihood) has a nice analytic form ((a common distribution family such as Gaussian)) that we know how to perform integral for the denominator,
we can directly get the closed-form solution for the posterior distribution.

For an event, we can try the following steps to get an analytic posteriori:
1. Decide a distribution family for the event likelihood (based on whatever fits this specific event).
2. Pick a parameter prior distribution that works nicely with this likelihood distribution (that generates joint distribution in a nice form), e.g. the conjugate prior for this likelihood distribution.
3. Obtain the joint distribution and scale it (divide it) by its integral.

Note that $p(X)$ (the denominator) is independent of the parameter. Why do we compute it here (why not do $\propto$ so to ignore the denominator)?
* Because we are obtaining a closed-form PDF: for any observation $X$, we can obtain the posteriori directly.
* Otherwise, if without scaling, for any $X$, we would have to manually scale the resulting joint probability, which requires the joint integral (denominator) anyway.
* The joint integral (denominator) can be obtained either analytically or empirically (sampling-based). However, performing sampling defeats the purpose of analytic solution.

Common likelihood-priori pair ([conjugate prior](https://en.wikipedia.org/wiki/Conjugate_prior)):
* likelihood: Binomial; priori: Beta
* likelihood: Multinomial; priori: Dirichlet
* likelihood: Poisson; priori: Gamma

Pros: we can directly get the closed-form posteriori.\
Cons: for a real event, the event likelihood is likely to be complex, so it is hard to find a prior distribution that works nicely with it to generate a joint distribution whose integral is easy to compute.

#### Markov chain Monte Carlo (MCMC): sampling-based

If the joint probability does not have a nice form to compute the integral $p(X)$, we can perform sampling-based method:
* Metropolis-Hastings
* Gibbs Sampling (when joint probability is hard for integral but conditional probability has nice form)

Here, we only focus on [Gibbs Sampling](https://en.wikipedia.org/wiki/Gibbs_sampling):
* For generality, assume the parameter is multivariate ($m$ variables), and we sample $n$ iterations.
* At each iteration, sample each parameter variable as if other variables are constant (sampling from conditional distribution), which requires the conditional distribution having a good form.
* The total sampling time would be $nm$ (a walk-through [video](https://www.youtube.com/watch?v=2ff_Dm41Tx4)).
* All samples obtained based on the conditional distribution will approximate the joint distribution.

Note that in MCMC, we can safely ignore the denominator, and only sample from the joint distribution (or conditional distribution) directly. Why?
* The samples we obtained are more like "density"; to obtain the PDF, we still need to scale it by the denominator.
* However, since we already approximate the posterior density, we can empirically compute the integral $p(X)$ from the samples to obtain the denominator easily.

Pros: a general method for any likelihood and priori; **low bias**.\
Cons: computationally heavy; **high variance** because of sampling.

#### Variational Inference (VI): approximation/optimization based

We can also try to approximate the true posterior distribution by fitting the parameters of a distribution family that we choose.

The general framework for VI:
1. Decide the way to compute final **event likelihood** given the parameter (can be simple distribution or complex neural network)
2. Decide the parameter **priori family** (usually a fixed Gaussian) and the **approximated posteriori** family (usually another Gaussian)
    * The parameter for the approximated posteriori will be computed directly by an **inference network** (usually another neural network)
3. For training, perform sampling from the approximated posteriori, and optimize by ELBO.

Note that:
* The major thing to learn in the VI optimization is the parameter estimation of the approximated posteriori.
  * For common cases, the likelihood are also learned jointly.
* ELBO has the form: optimizing observation log-likelihood (MLE) + pushing posteriori closer to priori (regularization)
* Optimizing ELBO is equivalent to optimizing KL-divergence between the true posteriori and the approximated posteriori.
  * See the derivation of ELBO at this [tutorial](https://www.cs.princeton.edu/courses/archive/fall11/cos597C/lectures/variational-inference-i.pdf)
* The sampling from the approximated posteriori in training employs the reparameterization trick to allow for back-propagation.

Scenarios for VI: Variational Autoencoders (VAEs)
* See an introduction [post](https://towardsdatascience.com/bayesian-inference-problem-mcmc-and-variational-inference-25a8aa9bce29)
* For representation learning in a compressed latent space with smooth properties
* **Posteriori to learn**: the latent representation of encoded input
* **Event**: the likelihood to reconstruct a given latent representation to its original input

Scenarios for VI: Neural Topic Modeling
* see another dedicated post
* **Posteriori to learn**: the latent topic distribution of an input document
* **Event**: the likelihood to generate/reconstruct the input document given topic distribution
