---
layout: post
title: "Uncertainty Estimation"
date: 2022-03-09
categories: [Basics]
tags: [uncertainty, bayes]
math: true
---

## Basics

See the updated post "Bayes".

### MAP

Let $D$ be the observed data. See previous post.

$$
\begin{align}
\theta_{MAP} &= \text{argmax}_{\theta} \; \log P(\theta|D) \\
&= \text{argmax}_{\theta} \; \log P(D|\theta) P(\theta)\\
&= \text{argmax}_{\theta} \; \underbrace{\sum_i \log P(d_i|\theta)}_{MLE} + \underbrace{\log P(\theta)}_{\text{Priori}}
\end{align}
$$

### BNN

Instead of a single model, regard the model parameter $\theta$ as a posterior distribution given the observed data $D$.
Therefore, the predicted output is also marginalized over the model posteriori, which should gives more accurate prediction and uncertainty estimation.

$$p(y|x, D) = \int_{\theta} p \big(y|f^{\theta}(x) \big) p(\theta|D) d\theta$$

Analytically, we usually parameterize the model posteriori by a simple distribution such as Gaussian to perform tractable variational inference.

Note that **above BNN mainly addresses the model uncertainty**.
Be sure to distinguish model uncertainty and predictive/output uncertainty. The latter is the final output uncertainty which is partially determined by the model uncertainty.

### Bayesian Linear Regression

Let's directly address the predictive/output uncertainty from Bayesian point of view.
We can get analytical solution with simple linear regression.
Let the predictive uncertainty follows Gaussian parameterized by $\theta$.

$$
\begin{align}
W_{MLE} &= \mathrm{argmax}_{\theta} \log \hat{y}\\
&= \mathrm{argmax}_{\theta} \log \frac{1}{\sqrt{2\pi}\sigma} + \log \bigg( \exp \big( -\frac{(\hat{y} - W^T x)^2}{2 \sigma^2} \big) \bigg)\\
&= \mathrm{argmin}_{\theta} \frac{1}{2 \sigma^2}(\hat{y} - W^T x)^2 \;+\; \log \sigma
\end{align}
$$

## Uncertainty for Regression

We can loosely regard uncertainty as variance. Our target is to determine the final output uncertainty, which is to identify the predictive variance on the output.

[Law of total variance](https://en.wikipedia.org/wiki/Law_of_total_variance): with random variable $x$ and $y$, the variance of $y$ can be decomposed as:

$$\mathrm{Var}(y) = \underbrace{\mathrm{E}[\mathrm{Var}(y|x)]}_{\text{Model Uncertainty}} + \underbrace{\mathrm{Var}(\mathrm{E}[y|x])}_{\text{Data Uncertainty}}$$

Therefore, output uncertainty can be explained by:
1. variance from model uncertainty
2. variance from inherent data uncertainty

Previous work has categorized uncertainty ($\approx$ total variance on output):
* Epistemic uncertainty (model uncertainty)
* Aleatoric uncertainty (data uncertainty)
  * Heteroscedastic uncertainty (data-dependent uncertainty)
  * Homoscedastic uncertainty (data-independent or task-dependent uncertainty)

### Model Uncertainty

We can obtain model uncertainty by BNN, approximated by dropout variational inference.
The calculation can be done as:

$$\begin{align}
\mathrm{Var}^{\text{model}} (y) &= \mathrm{E}[(y - \mu)^2]\\
&= \mathrm{E} [(y - \mathrm{E}[y])^2]\\
&= \mathrm{E}[y^2] - E[y]^2\\
&= \frac{1}{m}\sum_{\theta} y^2 - (\frac{1}{m}\sum_{\theta} y)^2 \quad \text{Monte-Carlo Dropout on }\theta
\end{align}
$$

### Data Uncertainty

Kendall'17/18 propose to let the model predict the data variance (assuming Gaussian distribution on prediction) along with the value. See details in the papers.

Some other work also proposes different estimation formulation such as evidential uncertainty. Details in papers.

### Together

Let $\sigma^{\theta}_y$ be the predicted data variance for output $y$ by the model with parameter $\theta$.

$$\begin{align}
\mathrm{Var}(y) &= \mathrm{Var}^{\text{model}} (y) + \mathrm{Var}^{\text{data}} (y) \\
&= \underbrace{\frac{1}{m}\sum_{\theta} y^2 - (\frac{1}{m}\sum_{\theta} y)^2}_{\text{Model Variance by Dropout}} \quad + \underbrace{\frac{1}{m}\sum^{\theta} \sigma^{\theta}_y}_{\text{Predicted Data Variance}}
\end{align}
$$

Depending on interests, we don't have to address both model and data variance.
* Only address model variance: obtain final uncertainty by simply performing Monte-Carlo Dropout
* Only address data variance: obtain final uncertainty by using a single model to only predict $\sigma$ (discard BNN)

## Uncertainty for Classification

Usually assume Gaussian on logit space. Details in papers.

# Related Work

**On Mixup Training: Improved Calibration and Predictive Uncertainty for Deep Neural Networks**. Thulasidasan et al. NIPS'19\
Smooth one-hot label.\
<https://arxiv.org/abs/1905.11001v5>

**What Uncertainties Do We Need in Bayesian Deep Learning for Computer Vision?**. Kendall and Gal. NIPS'17\
<http://arxiv.org/abs/1703.04977>

**Multi-Task Learning Using Uncertainty to Weigh Losses for Scene Geometry and Semantics**. Kendall et al. CVPR'18\
<http://arxiv.org/abs/1705.07115>

**Quantifying Uncertainties in Natural Language Processing Tasks**. Xiao and Wang. AAAI'19\
<http://arxiv.org/abs/1811.07253>

**Types of Out-of-Distribution Texts and How to Detect Them**. Arora et al. EMNLP'21\
<https://aclanthology.org/2021.emnlp-main.835>
