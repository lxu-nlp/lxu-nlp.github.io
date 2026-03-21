---
layout: post
title: "Notes on Bayes Basics"
date: 2022-08-08
categories: [Basics]
tags: [bayes]
math: true
---

# Basic Bayes Rules

Before discussing specific distributions and Bayesian inference, it is helpful to start from the **joint distribution**, because it is the central object from which many other probability expressions are derived.

## The Central Role of the Joint Distribution

Let:

- $H$ denote a hidden hypothesis, state, or event of interest;
- $E$ denote observed evidence.

Then the **joint probability** $P(H, E)$ means the probability that **both** the hidden event and the observed evidence occur together.

Intuitively, the joint distribution tells us how probability mass is assigned to **combinations** of hidden states and observations. Once we know the joint distribution, we can derive:

- marginals, such as $P(H)$ or $P(E)$,
- conditionals, such as $P(H \mid E)$ or $P(E \mid H)$,
- and therefore Bayes' rule itself.

This is why Bayes' rule is best viewed not as an isolated formula, but as one rearrangement of identities built around the joint distribution.

## Conditional Probability from the Joint

If $P(E) > 0$, then

$$
P(H \mid E) = \frac{P(H, E)}{P(E)}.
$$

Such that the **conditional probability is the normalization on the joint probability**.

$$
P(H, E) = P(H \mid E)P(E) = P(E \mid H)P(H).
$$

The same joint probability can be factorized in different ways depending on which quantity we want to isolate.

## Bayes' Rule

Rearranging the joint-factorization identity gives the standard Bayes' rule:

$$
P(H \mid E) = \frac{P(E \mid H)P(H)}{P(E)}.
$$

This formula says:

- start with a prior belief $P(H)$,
- weigh it by how compatible the evidence is with the hypothesis, $P(E \mid H)$,
- then normalize by the total probability of the evidence, $P(E)$.

A common shorthand is:

$$
P(H \mid E) \propto P(E \mid H)P(H),
$$

which emphasizes that the posterior is proportional to **likelihood × prior**.

## Law of Total Probability

To fully compute Bayes' rule, we often need the denominator $P(E)$, the total probability of observing the evidence.

If $H$ and $\neg H$ form a partition of possibilities, then:

$$
P(E) = P(E \mid H)P(H) + P(E \mid \neg H)P(\neg H).
$$

This is the **Law of Total Probability**. It says that the evidence can arise in multiple contexts, and we must account for all of them.

## Illustration: Event Detection

Imagine we have a system trying to detect a specific event. We observe a signal $E$ and want to know whether the hidden event $H$ actually happened.

The posterior probability is:

$$
P(H \mid E) = \frac{P(E \mid H)P(H)}{P(E)}.
$$

To compute it, we expand the denominator using the Law of Total Probability:

$$
P(E) = P(E \mid H)P(H) + P(E \mid \neg H)P(\neg H).
$$

Substituting this into Bayes' rule gives:

$$
P(H \mid E) = \frac{P(E \mid H)P(H)}{P(E \mid H)P(H) + P(E \mid \neg H)P(\neg H)}.
$$

This form makes two important Bayesian behaviors very clear.

### 1. The power of the prior

If $P(H)$ is very small, meaning the event is rare, then the denominator can easily be dominated by the **false-alarm** term:

$$
P(E \mid \neg H)P(\neg H).
$$

So even a seemingly strong signal may still correspond to a low posterior probability. This is why rare events require very reliable evidence before they become believable.

### 2. The strength of evidence comes from a likelihood ratio

What matters is not just whether $P(E \mid H)$ is large, but whether it is large **relative to** $P(E \mid \neg H)$.

- If $P(E \mid H) \gg P(E \mid \neg H)$, the evidence strongly supports $H$.
- If $P(E \mid H) \approx P(E \mid \neg H)$, the evidence tells us almost nothing.

In the second case, the posterior will remain close to the prior.

## From Event Probabilities to Bayesian Inference

In Bayesian inference, we use exactly the same logic, but replace the hidden event $H$ by a parameter value or latent variable $\theta$, and replace observed evidence by data $X$:

$$
p(\theta \mid X) = \frac{p(X \mid \theta)p(\theta)}{p(X)}.
$$

Again, the central quantity is the **joint distribution**:

$$
p(X, \theta) = p(X \mid \theta)p(\theta) = p(\theta \mid X)p(X).
$$

Once the joint is known, we can derive different marginals and conditionals depending on what question we want to answer. In that sense, Bayes' rule is a flexible way of navigating around the joint distribution rather than a single standalone trick.

---

# Distributions

## Binomial, Multinomial, and Poisson Distributions

### Binomial Distribution

The **binomial distribution** models the number of successes in **$n$ independent Bernoulli trials**, where each trial has the same success probability $p$.

$$
\begin{align}
P(X=x \mid n,p) &= \binom{n}{x} p^x (1-p)^{n-x}, \quad x=0,1,\dots,n \\
\mathbb{E}[X] &= np \\
\mathrm{Var}(X) &= np(1-p)
\end{align}
$$

Here,

$$
\binom{n}{x} = \frac{n!}{x!(n-x)!}
$$

is the number of ways to choose which $x$ of the $n$ trials are successes.

A useful intuition is:

- $p^x (1-p)^{n-x}$ is the probability of **one specific sequence** with $x$ successes and $n-x$ failures;
- $\binom{n}{x}$ counts how many such sequences exist.

So the binomial probability is simply:

$$
\text{probability of one valid sequence} \times \text{number of valid sequences}.
$$

### Multinomial Distribution

The **multinomial distribution** generalizes the binomial distribution to **$k$ possible categories** per trial.

If $X_1, \dots, X_k$ are the category counts after $n$ independent trials, and the category probabilities are $p_1, \dots, p_k$ with $\sum_{i=1}^k p_i = 1$, then:

$$
\begin{align}
P(X_1=x_1,\dots,X_k=x_k \mid p_1,\dots,p_k,n)
&= \frac{n!}{x_1!\cdots x_k!} p_1^{x_1}\cdots p_k^{x_k}, \\
\mathbb{E}[X_i] &= np_i, \\
\mathrm{Var}(X_i) &= np_i(1-p_i)
\end{align}
$$

subject to

$$
\sum_{i=1}^k x_i = n.
$$

Also, for $i \neq j$,

$$
\mathrm{Cov}(X_i, X_j) = -np_i p_j.
$$

The coefficient

$$
\binom{n}{x_1,\dots,x_k} = \frac{n!}{x_1!\cdots x_k!}
$$

counts the number of ways to arrange the observed category counts.

### Where does the multinomial coefficient come from?

Suppose we want exactly:

- $x_1$ outcomes of type 1,
- $x_2$ outcomes of type 2,
- $\dots$,
- $x_k$ outcomes of type $k$,

with $x_1 + \cdots + x_k = n$.

If all $n$ trial positions were distinct, there would be $n!$ ways to arrange them. But the $x_1$ outcomes of category 1 are indistinguishable among themselves, the $x_2$ outcomes of category 2 are indistinguishable among themselves, and so on. So we divide out the internal overcounting:

$$
\frac{n!}{x_1!x_2!\cdots x_k!}.
$$

That is exactly the number of distinct sequences that produce the same count vector $(x_1,\dots,x_k)$.

For any one such sequence, the probability is

$$
p_1^{x_1} \cdots p_k^{x_k},
$$

because multiplication over independent trials simply groups together identical factors. Therefore,

$$
P(X_1=x_1,\dots,X_k=x_k)
= \underbrace{\frac{n!}{x_1!\cdots x_k!}}_{\text{number of valid sequences}}
\underbrace{p_1^{x_1}\cdots p_k^{x_k}}_{\text{probability of each sequence}}.
$$

So the multinomial formula is the direct $k$-category extension of the binomial argument.

### Poisson Distribution

The **Poisson distribution** models the number of events occurring in a **fixed interval of time, space, or area**, when events happen independently at an average rate $\lambda$.

$$
\begin{align}
P(X=x \mid \lambda) &= \frac{\lambda^x e^{-\lambda}}{x!}, \quad x=0,1,2,\dots \\
\mathbb{E}[X] &= \lambda \\
\mathrm{Var}(X) &= \lambda
\end{align}
$$

Typical assumptions behind a Poisson model are:

- events occur independently,
- the average rate is constant over the interval,
- in a very small sub-interval, the chance of more than one event is negligible.

These assumptions make Poisson a natural model for counts such as arrivals, defects, or rare incidents over time or space.

## Poisson vs. Binomial

### Similarity

Both distributions model **counts of events**.

### Difference

- **Binomial**: the number of successes in a **fixed number of trials** $n$, with constant success probability $p$.
- **Poisson**: the number of events in a **fixed interval**, characterized by an average rate $\lambda$.

A Poisson distribution can be viewed as a limiting case of a binomial distribution when:

$$
n \to \infty, \quad p \to 0, \quad np \to \lambda.
$$

In that regime,

$$
\mathrm{Binomial}(n,p) \approx \mathrm{Poisson}(\lambda).
$$

So Poisson models are especially useful for **rare events with many opportunities to occur**, particularly when the exact values of $n$ and $p$ are unknown or not meaningful.

A practical way to think about the difference is:

- use **binomial** when you can clearly count attempts;
- use **Poisson** when you mainly observe a rate over time/space rather than explicit trials.

## Other Distributions

### Geometric Distribution

The **geometric distribution** models the probability that the **first success occurs on the $x$-th trial**.

$$
\begin{align}
P(X=x \mid p) &= (1-p)^{x-1}p, \quad x=1,2,\dots \\
\mathbb{E}[X] &= \frac{1}{p} \\
\mathrm{Var}(X) &= \frac{1-p}{p^2}
\end{align}
$$

A useful property is that the geometric distribution is **memoryless**:

$$
P(X>s+t \mid X>s) = P(X>t).
$$

Intuitively, after $s$ failures, the process “starts fresh” because each trial is still independent with the same success probability.

### Student's t-Distribution

The **Student's t-distribution** is similar to the normal distribution but has **heavier tails**, which reflects greater uncertainty.

It is commonly used when:

- the sample size is small, and/or
- the population variance is unknown and must be estimated from data.

As the sample size increases, the t-distribution approaches the normal distribution.

---

# Bayesian Inference

## Why Bayesian Inference?

The core Bayesian idea is simple: **treat unknown parameters as random variables and update uncertainty about them after observing data**.

This is useful because, in many real problems, a single point estimate is not enough. We often want to know:

- which parameter values are plausible,
- how uncertain we are,
- how that uncertainty affects future predictions.

Frequentist procedures often return a point estimate such as $\hat{\theta}_{\mathrm{MLE}}$. Bayesian inference instead returns a **distribution over parameters**, which lets us reason about uncertainty in a more direct way.

In short:

- **prior** = what we believed before seeing data,
- **likelihood** = how well each parameter explains the observed data,
- **posterior** = what we believe after combining both.

## Parameter Posterior

The posterior distribution over parameters is obtained by Bayes' rule:

$$
\begin{align}
\underbrace{p(\theta \mid X)}_{\text{posterior}}
&= \frac{\underbrace{p(X \mid \theta)}_{\text{likelihood}} \cdot \underbrace{p(\theta)}_{\text{prior}}}{\underbrace{p(X)}_{\text{evidence}}} \\
&= \frac{p(X,\theta)}{p(X)} \\
&= \frac{p(X \mid \theta)p(\theta)}{\int p(X \mid \theta')p(\theta') \, d\theta'}
\end{align}
$$

Here:

- $p(X \mid \theta)$ is the **likelihood**,
- $p(\theta)$ is the **prior**,
- $p(\theta \mid X)$ is the **posterior**,
- $p(X)$ is the **evidence** or **marginal likelihood**.

The posterior is proportional to:

$$
p(\theta \mid X) \propto p(X \mid \theta)p(\theta).
$$

This proportional form is often enough for derivations and algorithms, because the evidence does not depend on $\theta$.

### Intuition for each term

- The **likelihood** tells us which parameter values explain the data well.
- The **prior** tells us which parameter values were plausible before seeing the data.
- The **posterior** is the updated compromise between them.
- The **evidence** normalizes the posterior and also measures how well the model explains the data overall.

The evidence matters for at least two reasons:

1. it turns an unnormalized quantity into a proper probability distribution;
2. it is useful in **model comparison**, because models that explain the data well without spreading probability too thinly tend to get larger evidence.

## Predictive Posterior

Instead of predicting with a single parameter value, Bayesian inference averages over the whole posterior:

$$
\underbrace{p(x_{\mathrm{new}} \mid X)}_{\text{predictive posterior}} = \int p(x_{\mathrm{new}} \mid \theta) \, \underbrace{p(\theta \mid X)}_{\text{posterior}} \, d\theta.
$$

This is sometimes called **Bayesian model averaging** over parameter values.

Why is this important?

- A point estimate ignores uncertainty in $\theta$.
- The predictive posterior propagates parameter uncertainty into predictions.
- This often produces better-calibrated uncertainty estimates, especially when data are limited.

A good mental picture is:

- MLE/MAP prediction: “use the single best parameter”;
- Bayesian prediction: “average predictions from all plausible parameters, weighted by how plausible they are after seeing data.”

## A Concrete Example: Beta-Binomial Updating

Suppose we observe Bernoulli outcomes with unknown success probability $p$.

- Prior: $p \sim \mathrm{Beta}(\alpha, \beta)$
- Data: $s$ successes and $f$ failures, so $n=s+f$
- Likelihood: $p(X \mid p) \propto p^s(1-p)^f$

Then the posterior is:

$$
p \mid X \sim \mathrm{Beta}(\alpha + s, \beta + f).
$$

This example is useful because it shows the Bayesian update very cleanly:

- the prior contributes pseudo-counts $(\alpha-1, \beta-1)$ in one common interpretation,
- the data add real counts $(s,f)$,
- the posterior combines both.

The posterior mean is:

$$
\mathbb{E}[p \mid X] = \frac{\alpha+s}{\alpha+\beta+n}.
$$

Compare this with the MLE:

$$
\hat{p}_{\mathrm{MLE}} = \frac{s}{n}.
$$

The Bayesian estimate shrinks the raw empirical fraction toward the prior belief, especially when $n$ is small.

For a new Bernoulli observation,

$$
P(x_{\mathrm{new}}=1 \mid X) = \mathbb{E}[p \mid X] = \frac{\alpha+s}{\alpha+\beta+n}.
$$

So the predictive probability comes from averaging over uncertainty in $p$, not plugging in a single estimate.

## MAP vs. Full Bayesian Inference

A useful intermediate idea is **maximum a posteriori (MAP)** estimation:

$$
\theta_{\mathrm{MAP}} = \arg\max_\theta p(\theta \mid X) = \arg\max_\theta p(X \mid \theta)p(\theta).
$$

So:

- **MLE** maximizes likelihood only,
- **MAP** maximizes likelihood times prior,
- **full Bayesian inference** keeps the whole posterior instead of collapsing it to one point.

MAP can be viewed as a regularized version of MLE. It is often useful computationally, but it still loses the richer uncertainty information contained in the full posterior.

## Common Approaches to Bayesian Inference

### Closed-Form Solutions: Analytic Inference

If the prior and likelihood combine into a tractable analytic form, the posterior can be derived exactly.

A common strategy is:

1. choose a likelihood model appropriate for the data,
2. choose a mathematically convenient prior,
3. derive the posterior analytically.

When the prior is chosen so that the posterior belongs to the same family, it is called a **conjugate prior**.

Common conjugate pairs include:

- **Binomial likelihood** + **Beta prior**
- **Multinomial likelihood** + **Dirichlet prior**
- **Poisson likelihood** + **Gamma prior**

Why do we explicitly compute the denominator $p(X)$ in closed-form inference?

- Because we want the exact posterior distribution, not just something proportional to it.
- The denominator is the normalizing constant that makes the posterior integrate to 1.
- In analytic inference, obtaining that constant is part of the exact solution.

**Pros:** exact posterior; interpretable; often elegant.  
**Cons:** limited to simple models and convenient prior-likelihood pairs.

### Markov Chain Monte Carlo (MCMC): Sampling-Based Inference

When the posterior cannot be derived in closed form, we can approximate it using samples.

The idea is to generate a Markov chain whose stationary distribution is the posterior $p(\theta \mid X)$. After enough iterations, the samples behave approximately like draws from the posterior.

Common MCMC methods include:

- **Metropolis-Hastings**: propose a move, then accept or reject it with a probability designed so that the chain targets the posterior.
- **Gibbs Sampling**: sample each variable from its conditional distribution given the others.

#### Why can MCMC ignore the evidence $p(X)$?

Because many MCMC algorithms only need the posterior **up to proportionality**:

$$
p(\theta \mid X) \propto p(X \mid \theta)p(\theta).
$$

The evidence is a constant with respect to $\theta$, so:

- in **Metropolis-Hastings**, it cancels in the acceptance ratio;
- in **Gibbs sampling**, it disappears when forming conditional distributions.

This is one of the main practical strengths of MCMC.

#### Gibbs Sampling

Suppose the parameter vector is multivariate:

$$
\theta = (\theta_1, \dots, \theta_m).
$$

Gibbs sampling updates one component at a time:

$$
\theta_i^{(t+1)} \sim p(\theta_i \mid \theta_1^{(t+1)}, \dots, \theta_{i-1}^{(t+1)}, \theta_{i+1}^{(t)}, \dots, \theta_m^{(t)}, X).
$$

Over many iterations, the samples approximate the posterior distribution.

In practice, MCMC involves ideas such as:

- **burn-in**: discard early samples before the chain reaches a typical region;
- **mixing**: how well the chain explores the posterior;
- **autocorrelation**: nearby samples are not independent;
- **convergence diagnostics**: checks for whether the chain seems to have stabilized.

**Pros:** broadly applicable; often highly accurate; captures full posterior uncertainty.  
**Cons:** computationally expensive; can mix slowly in high-dimensional or highly correlated problems.

### Laplace Approximation

Another useful approximation is the **Laplace approximation**.

The idea is:

1. find the posterior mode (often the MAP estimate),
2. approximate the log-posterior locally by a quadratic function,
3. which implies a Gaussian approximation to the posterior.

This is attractive when the posterior is concentrated and reasonably unimodal. It is less reliable when the posterior is strongly skewed, multimodal, or far from Gaussian.

**Pros:** faster than MCMC; often simple to implement near a mode.  
**Cons:** local approximation only; may miss important global posterior structure.

## Summary of the Bayesian Workflow

A practical Bayesian workflow is usually:

1. choose a likelihood model for the data,
2. choose a prior over unknown parameters,
3. compute or approximate the posterior,
4. use the posterior to make predictions via the predictive posterior,
5. inspect uncertainty, not just point estimates.

The main conceptual difference from non-Bayesian point estimation is that Bayesian inference keeps uncertainty in the model all the way through the prediction stage.
