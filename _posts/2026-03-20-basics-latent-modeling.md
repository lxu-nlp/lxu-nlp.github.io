---
layout: post
title: "Notes on Latent Modeling, EM and VAE"
date: 2026-03-20
categories: [Basics]
tags: [bayes, latent]
math: true
---

Also see the [post](https://lxu-nlp.github.io/posts/basics-bayes/) on Bayes basics.


# ELBO, EM, and VAE: From Latent Variable Models to Trainable Deep Generative Models

## 1. Starting from the “log-sum wall”

In maximum likelihood estimation, we want to find parameters $\theta$ that make the observed data $x$ as probable as possible. The objective is the log-likelihood:

$$
\ell(\theta) = \log p(x \mid \theta).
$$

If the model contains no latent variables, this objective is often manageable. But many expressive probabilistic models assume that the observation $x$ is generated together with some hidden variable $z$. In that case, the marginal likelihood becomes

$$
\ell(\theta) = \log p(x \mid \theta) = \log \sum_z p(x, z \mid \theta),
$$

or an integral in the continuous case. We marginalize over all possible latent “stories.” To know how likely x is, we must consider every possible hidden explanation z that could have produced it.

The difficulty is immediate: in

$$
\log \sum_z p(x, z \mid \theta),
$$

the logarithm cannot simply be pushed inside the summation. This **log-sum structure** makes exact optimization, clean gradients, and analytic updates difficult or impossible. That is the first fundamental obstacle in latent variable modeling: the **log-sum wall**.

From this point, the story of ELBO, EM, and VAE unfolds naturally:

- **ELBO** gives us a tractable surrogate objective;
- **EM / GEM** give us the optimization logic for improving that objective;
- **VAE** turns this logic into a scalable neural implementation.

#### What does $p(x, z \mid \theta)$ actually represent?

In the VAE setting, $p(x, z \mid \theta)$ is the **joint probability distribution** of the observed data $x$ and the latent variable $z$. Intuitively, it is the model’s full story of how a datapoint comes into existence.

One useful way to think about it is this: the model first chooses a hidden code $z$, and then uses that code to generate the observation $x$. In that sense, $p(x, z \mid \theta)$ is the probability that the model picks a particular hidden “secret” $z$ and successfully turns it into the observed datapoint $x$.

By the chain rule of probability,

$$
p(x, z \mid \theta) = p(x \mid z, \theta)\, p(z \mid \theta).
$$

This decomposition separates the joint distribution into two manageable parts:

- **The prior** $p(z \mid \theta)$: the model’s assumption about where latent codes live before seeing any data. In a standard VAE, this is usually chosen to be a standard Gaussian, so we typically write simply $p(z) = \mathcal{N}(0, I)$.
- **The likelihood / decoder** $p(x \mid z, \theta)$: the probability of generating the observation $x$ from a given latent code $z$. In a neural VAE, this is implemented by the decoder network.

So the word **joint** matters: it describes the co-occurrence of the hidden cause $z$ and the visible effect $x$ under one coherent generative story.

---

## 2. ELBO: turning an intractable likelihood into an optimizable lower bound

### 2.1 Introducing a variational distribution $q(z \mid x)$

To deal with the log-sum wall, we introduce an auxiliary distribution $q(z \mid x)$, which we can interpret as our approximation to the posterior over the hidden variable after observing $x$. In informal derivations, people often shorten this to $q(z)$ once $x$ is fixed, but the conditional form $q(z \mid x)$ is the more precise notation. Formally, we multiply and divide by the same quantity:

$$
\log p(x \mid \theta)
:= \log \sum_z q(z \mid x) \frac{p(x, z \mid \theta)}{q(z \mid x)}.
$$

As long as $q(z \mid x)$ is nonzero wherever needed, this transformation changes nothing mathematically. But now the sum has the form of an expectation under $q(z \mid x)$.

### 2.2 Using Jensen’s inequality to move the log inward

Because the logarithm is a concave function, Jensen’s inequality implies

$$
\log \mathbb{E}_{q(z \mid x)}\left[\frac{p(x, z \mid \theta)}{q(z \mid x)}\right]
\ge
\mathbb{E}_{q(z \mid x)}\left[\log \frac{p(x, z \mid \theta)}{q(z \mid x)}\right].
$$

This gives the **Evidence Lower Bound (ELBO)**:

$$
\mathcal{L}(q, \theta; x)
:= \sum_z q(z \mid x) \log \frac{p(x, z \mid \theta)}{q(z \mid x)}
:= \mathbb{E}_{q(z \mid x)}[\log p(x, z \mid \theta)] - \mathbb{E}_{q(z \mid x)}[\log q(z \mid x)].
$$

Hence,

$$
\log p(x \mid \theta) \ge \mathcal{L}(q, \theta; x).
$$

So ELBO is a lower bound on the true log-likelihood. We may no longer optimize the exact marginal likelihood directly, but we obtain a tractable objective that is tightly connected to it.

One last notation polish is worth making explicit: the bound above is the **per-observation ELBO** for one fixed datapoint $x$. For a dataset $\mathcal{D} = \{x_i\}_{i=1}^N$, the training objective is the sum or average over datapoints,

$$
\mathcal{L}(q, \theta; \mathcal{D}) = \sum_{i=1}^N \mathcal{L}(q_i, \theta; x_i),
$$

or, in amortized form,

$$
\mathcal{L}(\phi, \theta; \mathcal{D}) = \sum_{i=1}^N \mathcal{L}(\phi, \theta; x_i).
$$

In practice, VAEs optimize a minibatch estimate of this dataset-level objective, which is why papers and code often move back and forth between the per-sample ELBO and the averaged training loss.

---

## 3. Three equivalent views of the ELBO

The ELBO is not just useful because it is computable. It is useful because several equivalent rearrangements reveal different aspects of the same underlying idea.

### 3.1 View 1: Likelihood = ELBO + gap

The most important identity is

$$
\log p(x \mid \theta)
:= \mathcal{L}(q, \theta; x) + D_{\mathrm{KL}}\bigl(q(z \mid x) \parallel p(z \mid x, \theta)\bigr).
$$

Since KL divergence is always nonnegative, the ELBO is indeed a lower bound. This form tells us three crucial things:

1. **A larger ELBO usually means we are closer to the true log-likelihood**;
2. The gap between ELBO and the true objective is exactly
   the KL discrepancy between our guessed posterior and the true posterior;
3. When $q(z \mid x) = p(z \mid x, \theta)$, the gap is zero, so ELBO matches the log-likelihood exactly.

This makes ELBO more than just a bound: it couples parameter learning and posterior approximation in one equation.

### 3.2 View 2: Energy plus entropy

We can rewrite ELBO as

$$
\mathcal{L}(q, \theta; x)
:= \mathbb{E}_{q(z \mid x)}[\log p(x, z \mid \theta)] + \mathcal{H}(q(\cdot \mid x)),
$$

where

$$
\mathcal{H}(q(\cdot \mid x)) = -\mathbb{E}_{q(z \mid x)}[\log q(z \mid x)]
$$

is the entropy of the variational posterior for that observation.

This is the **energy-entropy view**. Here, $\log p(x, z \mid \theta)$ can be read as the **complete-data log-likelihood**, or informally as the energy term:

- the first term encourages $q(z \mid x)$ to place mass on latent states that make $(x,z)$ likely under the model;
- the second term discourages $q(z \mid x)$ from becoming unnecessarily sharp or collapsed.

In plain language, the model should explain the data well, but its uncertainty about latent structure should remain meaningful rather than degenerately narrow.

### 3.3 View 3: Reconstruction minus KL in the VAE form

If we factor the joint distribution by the chain rule as

$$
p(x, z \mid \theta) = p(x \mid z, \theta) p(z \mid \theta),
$$

and in the standard VAE case take the prior to be fixed so that $p(z \mid \theta)=p(z)$, then ELBO becomes

$$
\mathcal{L}(q, \theta; x)
:= \mathbb{E}_{q(z \mid x)}[\log p(x \mid z, \theta)] - D_{\mathrm{KL}}\bigl(q(z \mid x) \parallel p(z \mid \theta)\bigr).
$$

In a standard VAE this is usually written more compactly as

$$
\mathcal{L}(q, \theta; x)
:= \mathbb{E}_{q(z \mid x)}[\log p_\theta(x \mid z)] - D_{\mathrm{KL}}\bigl(q(z \mid x) \parallel p(z)\bigr).
$$

This is the form most commonly seen in VAEs:

- the first term is the **reconstruction term**;
- the second term is a **regularization term** or **prior penalty**, which keeps the approximate posterior close to the prior.

What deep learning often calls “reconstruction loss plus KL regularization” is, mathematically, the same ELBO.

---

## 4. EM: the classical logic for optimizing latent variable models

Having a good objective is not enough; we also need a strategy for improving it. The classical answer is EM (Expectation-Maximization).

### 4.1 The basic idea of EM

EM alternates between two steps.

#### E-step

Fix the current parameter value $\theta^{(t)}$ and update the latent-variable distribution for the current observation so that

$$
q(z \mid x) \approx p(z \mid x, \theta^{(t)}).
$$

In classical EM, the ideal choice is to set

$$
q(z \mid x) = p(z \mid x, \theta^{(t)}),
$$

which drives the KL gap to zero under the current parameter setting.

#### M-step

Now fix $q(z \mid x)$ and update the model parameters by maximizing

$$
\theta^{(t+1)} = \arg\max_\theta \mathcal{L}(q, \theta; x).
$$

For a full dataset, one sums or averages this bound over all observations. Once $q$ is fixed, the optimization is only over the model parameters.

### 4.2 Why EM works

The effectiveness of EM comes from the following chain of reasoning:

1. the E-step tightens the bound by making $q(z \mid x)$ closer to the true posterior;
2. the M-step increases the bound with respect to $\theta$;
3. because ELBO is a lower bound on the log-likelihood, raising the bound guarantees that the true likelihood does not decrease.

This is the core monotonic-improvement logic behind EM.

One comparison point is worth postponing until the VAE setup is fully on the table: why classical EM is usually **not** described as suffering from VAE-style posterior collapse. The clearest answer only appears once amortization and prior matching enter the story, so we return to that contrast in Section 6.3.

---

## 5. GEM: a more flexible and more realistic version of EM

In many complex models, we cannot perform either step exactly. That leads to **Generalized EM (GEM)**.

GEM relaxes the requirements of classical EM:

- the E-step does not need to set $q(z \mid x)$ equal to the exact posterior, as long as it improves the ELBO;
- the M-step does not need to find the exact maximizer either, as long as it increases the ELBO.

So the essence of GEM is simple:

> **If each update to the posterior approximation and each update to the model parameters increases the ELBO, then the true likelihood will not decrease.**

This matters because modern deep learning almost never performs exact E-steps or exact M-steps. Instead, we use gradient-based updates, minibatches, and approximate optimization. In that sense, VAE training is much closer to GEM than to textbook exact EM.

---

## 6. VAE: variational inference implemented with neural networks

### 6.1 From general latent variable models to VAEs

A Variational Autoencoder (VAE) does not introduce a fundamentally new objective. It still optimizes the ELBO. What it changes is the implementation:

1. it uses a parameterized approximate posterior
   $q_\phi(z \mid x);$
2. it uses a parameterized generative model
   $p_\theta(x \mid z);$
3. it trains both with stochastic gradient descent and backpropagation.

Here,

- $q_\phi(z \mid x)$ is the **encoder**;
- $p_\theta(x \mid z)$ is the **decoder**.

The VAE objective is usually written as

$$
\mathcal{L}(\phi, \theta; x)
:= \mathbb{E}_{q_\phi(z \mid x)}[\log p_\theta(x \mid z)]
- D_{\mathrm{KL}}\bigl(q_\phi(z \mid x) \parallel p(z)\bigr).
$$

This is not a different theory. It is the ELBO placed inside a neural, differentiable, scalable architecture.

### 6.2 Amortized inference: why VAEs scale better than classical variational EM

In traditional EM or variational EM, we often solve for a separate distribution $q_i(z) = q_i(z \mid x_i)$ for each data point $x_i$. On large datasets, that becomes expensive.

The key shortcut in VAEs is this:

> Instead of optimizing a separate posterior approximation for every individual sample, we learn a single function $q_\phi(z \mid x)$ that can instantly predict a good approximate posterior for any input $x$.

This is called **amortized inference**.

So compared with classical EM:

- the traditional E-step says: “find the best posterior for this specific sample”;
- the VAE-style E-step says: “learn a reusable inference mechanism that works for all samples.”

That is the real meaning of the encoder. It is not a concept outside EM; it is an amortized, shared, parameterized version of the E-step.

This speedup comes with a price. Even if the variational family is expressive enough, the shared encoder $q_\phi(z \mid x)$ may still fail to match the per-sample optimum one would obtain by separately optimizing a variational distribution for each $x_i$. That extra approximation cost is often called the **amortization gap**. In plain terms, VAEs gain scalability by giving up some sample-specific inference accuracy.

### 6.3 Why classical EM is usually not discussed in terms of “collapse”, and what extra risks VAEs introduce

Before listing the extra risks of VAEs, it is useful to make one contrast explicit. Classical EM can certainly fail, but usually in **different** ways: it may get stuck in poor local optima, become sensitive to initialization, or in some models drift toward degenerate parameter settings. What is less typical is the specific VAE-style phenomenon called **posterior collapse**.

The reason is structural. In classical EM, the E-step is trying to compute or closely approximate the current model posterior,

$$
q(z \mid x) \approx p(z \mid x, \theta^{(t)}),
$$

and if exact EM is possible, it simply sets them equal. There is no separate learned inference network that is shared across all examples, and there is no extra pressure pushing the posterior toward a simple, input-independent prior. So EM is not rewarded for making the latent variable uninformative; if the posterior needs to depend strongly on $x$, EM simply uses that dependence.

A VAE changes that geometry. It still inherits the latent-variable logic of EM, but it adds several ingredients that make training more fragile.

First, it introduces **amortization**: instead of solving a separate inference problem for each datapoint, it asks one shared encoder to handle all datapoints. That creates the amortization gap just discussed.

Second, it introduces explicit **prior matching** through the KL term,

$$
D_{\mathrm{KL}}\bigl(q_\phi(z \mid x) \parallel p(z)\bigr),
$$

which is not merely an optimization convenience. It actively pushes the latent representation toward a simple, globally organized distribution. This is useful for sampling and for obtaining a smooth latent space, but it also creates pressure to reduce how much input-specific information the latent variable carries.

Third, VAEs are often paired with powerful neural decoders. If the decoder can reconstruct $x$ reasonably well on its own, then the model can improve the objective by letting the encoder transmit less information and by shrinking the KL term. That is the basic route to posterior collapse.

So compared with classical EM, a VAE is solving a harder combined problem: it is not only fitting a latent-variable model, but also learning a shared inference network, shaping the geometry of the latent space, and training a powerful decoder end-to-end by stochastic optimization. Those extra goals are exactly why VAEs gain scalability and flexibility — and also why they introduce additional failure modes beyond standard EM.

---

## 7. The E-step and M-step inside a VAE: which knob is being turned?

Let us write the VAE ELBO again:

$$
\mathcal{L}(\phi, \theta; x)
:= \underbrace{\mathbb{E}_{q_\phi(z \mid x)} [\log p_\theta(x \mid z)]}_{\text{Term A: Reconstruction}}
- \underbrace{D_{\mathrm{KL}}(q_\phi(z \mid x) \parallel p(z))}_{\text{Term B: Prior Penalty}}.
$$

### 7.1 The E-step: updating $\phi$

From the E-step perspective, the question is:

> Given the current generative model, how should the encoder represent this data point in latent space?

The parameters being updated are $\phi$, the encoder parameters. Optimizing them has two effects:

- it places latent samples where the decoder can reconstruct $x$ well;
- it keeps the approximate posterior from drifting too far from the prior.

At a deeper level, although the code visibly contains only the reconstruction term and the KL-to-prior term, maximizing the full ELBO also implicitly encourages

$$
q_\phi(z \mid x) \approx p_\theta(z \mid x),
$$

which means the encoder is learning a better approximation to the true posterior.

### 7.2 The M-step: updating $\theta$

From the M-step perspective, the question is:

> Given the current latent representation, how should the generative model change to make the data more probable?

The parameters being updated are $\theta$, the decoder parameters. Since the KL-to-prior term usually depends only on $q_\phi(z \mid x)$ and the fixed prior $p(z)$, the M-step mainly focuses on increasing

$$
\mathbb{E}_{q_\phi(z \mid x)} [\log p_\theta(x \mid z)],
$$

namely, making the decoder better at reconstructing or generating $x$ from $z$.

### 7.3 The “single-pass” synergy

Classical EM alternates between E-step and M-step explicitly. A VAE folds both roles into the same training loop:

- the encoder learns where to place the latent clouds so that they are both prior-compatible and reconstruction-friendly;
- the decoder learns how to interpret those latent clouds and map them back to the data space.

So implementation-wise, a VAE often looks like just one `loss.backward()` call. But conceptually, it still follows the GEM logic:

- one part of the parameters improves the **guess** of the latent variables;
- the other part improves the **model** itself.

More precisely, a VAE should be understood as **EM/GEM-like**, not as literal textbook coordinate-ascent EM. In textbook EM, one ideally performs an exact posterior update in the E-step and then an exact or monotonic-improvement update in the M-step while holding the other side fixed. In a VAE, $\phi$ and $\theta$ are usually updated jointly with minibatch SGD, so those exact monotonicity guarantees are softened. The connection to EM remains structural and conceptual rather than literal.

---

## 8. The reparameterization trick: why end-to-end training is possible

There is still one practical issue. The ELBO contains an expectation

$$
\mathbb{E}_{q_\phi(z \mid x)}[\cdot],
$$

which means we need to sample $z$ from $q_\phi(z \mid x)$. But sampling is not directly differentiable, so gradients cannot easily flow back through the random variable.

The VAE solution is the **reparameterization trick**. For a Gaussian approximate posterior, we write

$$
z = \mu_\phi(x) + \sigma_\phi(x) \odot \epsilon,
\qquad \epsilon \sim \mathcal{N}(0, I).
$$

This does two things:

- it moves the randomness into a noise variable $\epsilon$ that is independent of the parameters;
- it expresses $z$ as a differentiable function of $\phi$.

As a result, gradients can flow through $z$ back into both encoder and decoder. The mathematical target is still the ELBO, but the optimization becomes compatible with standard backpropagation.

### A brief practical note: posterior collapse

One important training issue in VAEs is **posterior collapse**. This happens when the decoder becomes so strong that it learns to reconstruct the data while using little or almost no information from the latent variable. In that case,

$$
q_\phi(z \mid x) \approx p(z),
$$

so the approximate posterior becomes nearly indistinguishable from the prior. The key idea is not that $z$ disappears, but that $z$ stops carrying much information about the specific input $x$. Equivalently, the dependence between $x$ and $z$ becomes weak: the latent code is present, but it is no longer very informative.

At first this sounds paradoxical: if reconstruction is based on $z$, how can the decoder still reconstruct well when $z$ has collapsed toward the prior? The answer is that a sufficiently expressive decoder may learn to rely mostly on its own modeling power rather than on the latent code. In other words, the decoder behaves as if $p_\theta(x \mid z)$ were only weakly sensitive to $z$, so reconstruction can remain decent even though the latent channel is barely being used.

This is especially easy to understand in settings where the decoder has another route for prediction:

- **Text VAEs with an autoregressive decoder and teacher forcing.** When predicting the next token, the decoder is already given the previous ground-truth tokens during training. A strong RNN or Transformer can often reconstruct the sentence mainly from local language-model structure, so $z$ contributes little and the KL term collapses.
- **Image VAEs with a very strong autoregressive decoder (for example, PixelCNN-style).** Such a decoder can model many pixel dependencies directly from previously generated pixels, so $z$ may be used only weakly or only for coarse global information.
- **Architectures with skip connections or deterministic bypass paths from encoder features to decoder features.** In that case, the decoder may recover much of the input through the bypass path, making the stochastic latent variable less necessary.

By contrast, in a plain VAE with a relatively weak decoder and no bypass path, severe collapse usually hurts reconstruction more visibly. In practice, one often sees **partial collapse** rather than an absolute all-or-nothing failure: $z$ may still encode a little global information, while the decoder handles most of the fine detail on its own.

A few common mitigation strategies are:

- **KL annealing**: gradually increase the weight of the KL term during training;
- **free bits**: prevent the KL term from being driven to zero too aggressively;
- **weakening the decoder**: make the model rely more on the latent code instead of solving everything autoregressively.

So in practice, it is useful to monitor not only the total ELBO, but also the reconstruction term and the KL term separately. A very small KL may look numerically good while actually indicating that the latent space is not being used meaningfully.

### A brief practical note: observation model and reconstruction loss

It is also useful to remember that the reconstruction term

$$
\mathbb{E}_{q_\phi(z \mid x)}[\log p_\theta(x \mid z)]
$$

is not an arbitrary loss. It is determined by the **observation model** you choose for $p_\theta(x \mid z)$. For example:

- a **Gaussian** observation model often leads to an **MSE / squared-error** style reconstruction loss;
- a **Bernoulli** observation model often leads to **binary cross-entropy**;
- a **categorical** observation model often leads to standard **cross-entropy**.

So the reconstruction loss in code is really the negative log-likelihood implied by the decoder’s probabilistic assumption about the data.

---

## 9. A unified map of ELBO, EM, GEM, and VAE

We can now summarize the whole picture in one table:

| Concept              | What it is                                                          | Mathematical role                                        | In a VAE                               |
| -------------------- | ------------------------------------------------------------------- | -------------------------------------------------------- | -------------------------------------- |
| log-sum problem      | The source of intractability in latent variable likelihoods         | Prevents direct MLE                                      | The reason we need variational methods |
| ELBO                 | A tractable lower bound on the log-likelihood                       | Optimization target                                      | The loss function                      |
| EM                   | Classical alternating optimization for latent variable models       | Optimization logic                                       | Prototype of the VAE training idea     |
| GEM                  | A relaxed EM that only requires improvement, not exact maximization | Practical optimization framework                         | Closer to real VAE training            |
| $q_\phi(z \mid x)$   | Approximate posterior / guesser                                     | E-step object                                            | Encoder                                |
| $p_\theta(x \mid z)$ | Generative model                                                    | M-step object                                            | Decoder                                |
| Reparameterization   | Differentiable sampling trick                                       | Enables gradient-based learning through stochastic nodes | Core training mechanism                |

This entire chain can be compressed into one sentence:

> **Because the marginal likelihood contains a difficult log-sum term, we use Jensen’s inequality to construct the ELBO; because the ELBO separates naturally into posterior-improvement and parameter-improvement roles, we obtain the logic of EM and GEM; because classical inference is too slow at scale, we amortize the E-step with an encoder network and use reparameterization to train everything by backpropagation, which gives us the VAE.**

---

## 10. A concrete image-VAE walkthrough: architecture, learning, and manifold

To make the mathematics operational, consider the standard image-VAE setup.

### 10.1 The usual architecture

For images, the encoder and decoder are typically spatial models rather than plain MLPs.

- The **encoder** $q_\phi(z \mid x)$ is usually a stack of convolutional layers that progressively downsamples the image, for example from $64 \times 64 \times 3$ into a compact feature vector.
- Instead of ending in a single prediction head, the encoder usually ends in **two parallel heads**:
  - one outputs the mean vector $\mu_\phi(x)$;
  - the other outputs the log-variance vector $\log \sigma_\phi^2(x)$.
- The **latent variable** $z$ is a relatively low-dimensional vector, often something like 128 or 256 dimensions.
- The **decoder** $p_\theta(x \mid z)$ is typically a stack of transposed convolutions or upsampling blocks that maps the latent vector back to image space.

So for one input image $x$, the encoder defines

$$
q_\phi(z \mid x) = \mathcal{N}(\mu_\phi(x), \operatorname{diag}(\sigma_\phi^2(x))),
$$

and we sample via

$$
z = \mu_\phi(x) + \sigma_\phi(x) \odot \epsilon,
\qquad \epsilon \sim \mathcal{N}(0,I).
$$

The decoder then uses $z$ to parameterize the image likelihood $p_\theta(x \mid z)$.

### 10.2 What is being learned?

An image VAE is learning two coupled things at once.

#### (a) The encoder is learning inference

The encoder is learning how to map visual structure into a probabilistic latent description. Concretely, it learns weights $\phi$ that transform pixels into the parameters of a Gaussian posterior approximation.

In intuitive terms, the encoder is learning which visual factors — such as shape, color, pose, texture, or global layout — should correspond to coordinates in latent space.

From the GEM viewpoint, this is the amortized E-step side: the encoder is trying to become a better guesser of the hidden explanation behind the image.

#### (b) The decoder is learning generation

The decoder is learning weights $\theta$ that transform a latent code back into a plausible image.

In intuitive terms, it is learning the visual rules of the data distribution: if one region of latent space corresponds to cats, another to sunsets, and another to handwritten digits, the decoder must learn how to turn those abstract codes into the right pixels.

From the GEM viewpoint, this is the M-step side: the decoder is trying to maximize reconstruction likelihood given the latent description.

### 10.3 The hidden object being learned: a latent manifold

Beyond the network weights themselves, the model is also learning a **latent manifold**. Because the posterior is regularized toward a simple prior,

$$
D_{\mathrm{KL}}\bigl(q_\phi(z \mid x) \parallel \mathcal{N}(0,I)\bigr),
$$

the model is pushed to organize images into a smooth, navigable latent space.

This is what gives VAEs their characteristic geometric behavior:

- **continuity**: if two nearby points in latent space correspond to similar images, then interpolation between them often produces gradual visual transitions;
- **factorization or partial disentanglement**: some latent dimensions may end up controlling partly separable attributes such as stroke thickness, rotation, color tone, or object pose.

This is why a VAE is not just compressing images. It is trying to compress them into a space that is also structured enough for sampling, interpolation, and controlled generation.

---

## 11. VAE in 2026 multimodal systems: from continuous latents to visual tokens

The most important modern extension is not merely that VAEs compress images. It is that VAE-like models solve the **tokenization problem** for multimodal large models.

### 11.1 The tokenization problem

LLMs are built to predict the next token. Pixels are not tokens. So if we want an LLM or a native multimodal model to process images in the same general autoregressive framework as text, we need to convert images into a sequence of visual symbols.

This is where the VAE philosophy returns in a modern form: **discrete VAEs**, especially **VQ-VAE** and **VQ-GAN**.

A typical pipeline is:

1. an image encoder compresses a high-resolution image, for example $512 \times 512$, into a lower-resolution latent grid, for example $32 \times 32$;
2. each latent vector is **quantized** to the nearest entry in a learned **codebook**;
3. the resulting codebook indices become a sequence of **visual tokens**;
4. those tokens can then be fed into a multimodal language model alongside ordinary text tokens.

This is the key shift: instead of representing an image as one global embedding, the model represents it as a **sequence of visual words**.

### 11.2 Why this is more VAE-like than CLIP-like

Many earlier vision-language systems relied on discriminative encoders such as CLIP. That design is excellent for alignment and retrieval, but it is not naturally generative.

The contrast is important:

| Feature         | CLIP-style discriminative setup | VAE / VQ-style generative setup                      |
| --------------- | ------------------------------- | ---------------------------------------------------- |
| Main question   | “What does this image mean?”    | “How is this image built?”                           |
| Typical output  | One semantic embedding          | A sequence or grid of latent codes                   |
| Generative role | Weak or indirect                | Native support for image generation / reconstruction |
| Core math       | Contrastive alignment           | ELBO-style latent modeling or VQ reconstruction      |

A CLIP-style embedding is good for saying what an image is about. A VAE-style tokenizer is better suited to saying how an image can be decomposed and regenerated. For any-to-any multimodal systems, that difference matters.

### 11.3 Where the VAE sits in 2026 systems

The role of VAE now splits into two major regimes.

#### (a) Continuous VAE for latent generative modeling

In latent-diffusion systems and diffusion transformers, a continuous VAE is often used to compress images into a lower-dimensional latent space before diffusion begins. This is the standard “do generation in latent space, not in pixel space” trick that makes large-scale visual generation computationally practical.

#### (b) Discrete VAE for native multimodal token streams

In native multimodal architectures — the kind often discussed in connection with systems such as Gemini-, GPT-4o-, or Chameleon-style models — the more relevant idea is often **discrete latent tokenization**. Vision, and sometimes audio or video, is converted into token sequences that can be modeled in a unified autoregressive or sequence-prediction framework.

This is why VAE has become relevant again: not only as a generator, but as the mechanism that makes non-text modalities legible to token-based foundation models.

### 11.4 A concise systems map: ViT, CLIP, JEPA, VAE, and generative priors

To get the full picture, we should separate **backbones**, **representation-learning objectives**, and **generative priors**. A **ViT** is mainly a visual backbone; **CLIP**, **JEPA**, and **VAE/VQ** describe what kind of representation is learned; **diffusion** or **autoregressive decoding** describes how generation is carried out once that representation exists.

| Component / paradigm                 | What it mainly learns                                      | Typical output                           | Native any-to-any generation?          | Best use in a VLM                                       |
| ------------------------------------ | ---------------------------------------------------------- | ---------------------------------------- | -------------------------------------- | ------------------------------------------------------- |
| **ViT**                              | Patch-level visual features through self-attention         | Patch tokens or pooled visual embeddings | **No, not by itself**                  | General-purpose visual front-end                        |
| **CLIP-style training**              | Cross-modal semantic alignment                             | One or a few semantic embeddings         | **Usually no**                         | Retrieval, grounding, matching, semantic understanding  |
| **JEPA-style training**              | Predictive, invariant latent structure                     | Abstract latent representations          | **Not directly**                       | Robust perception, world models, predictive abstraction |
| **VAE / VQ-style training**          | Reconstructible latent variables or discrete visual tokens | Continuous latents or codebook indices   | **Yes**                                | Native visual tokenization, reconstruction, generation  |
| **Diffusion / autoregressive prior** | Probability model over visual latents or tokens            | A generative process over learned units  | **Yes, if latent units already exist** | High-quality image/video synthesis                      |

The key distinction is simple:

- **ViT** tells us how the image is encoded;
- **CLIP / JEPA / VAE** tell us what representation is learned from that encoding;
- **diffusion or autoregressive generation** tells us how those learned units are turned into images again.

This is why “ViT versus VAE” is not quite the right question. A ViT can be the image encoder inside a CLIP system, a JEPA system, or a VQ/VAE tokenizer.

### 11.5 A compact picture of frontier any-to-any VLMs

The strongest **publicly described** any-to-any systems today do not rely on just one of the paradigms above. They typically combine them in a modular way, especially on the visual side.

#### Pattern A: discrete native-multimodal stack

This is the clearest route when the goal is to make images look like text to a next-token model.

1. a **visual encoder** — often ViT-like or otherwise transformer-friendly — compresses the image into a latent grid;
2. a **VQ-VAE / VQ-GAN-style tokenizer** quantizes that grid into discrete codebook entries;
3. the resulting **visual token IDs** are fed into a multimodal autoregressive transformer together with text tokens;
4. for generation, the model predicts visual tokens autoregressively and a visual decoder maps them back to pixels.

This is the most direct path to **native any-to-any generation**, because the model can process text, image, audio, or video as variations of one token stream. Public examples in this family include systems such as **Emu3** and **Chameleon-style** designs.

#### Pattern B: continuous-latent generation stack

This is the dominant route when image quality matters more than forcing everything into one discrete next-token stream.

1. an image is encoded into a **continuous latent space** by a VAE;
2. a transformer, diffusion model, or related generative prior operates in that latent space;
3. a decoder maps the generated latent back to pixels.

This is the logic behind **latent diffusion** and many diffusion-transformer pipelines: the VAE handles compression and reconstruction, while the generative model handles image synthesis in latent space.

#### What the visual side looks like in the strongest systems

A useful high-level summary is that frontier any-to-any systems tend to separate **visual understanding** from **visual generation**, even when the whole system is presented as one multimodal model.

- For **understanding**, they often use a strong visual front-end such as a ViT, sometimes trained with CLIP-like or JEPA-like objectives to produce semantically useful representations.
- For **generation**, they need a **reconstructible interface** — usually a VQ-style visual tokenizer or a continuous VAE latent space — because semantic embeddings alone cannot be turned back into detailed images.
- On top of that interface, they place the actual **generative controller**: either an autoregressive multimodal transformer over discrete visual tokens, or a diffusion / DiT-style model over continuous latents.

So the best current any-to-any picture is not “replace ViT with VAE.” It is closer to this:

> **ViT for seeing, VAE/VQ for making vision reconstructible, and autoregressive or diffusion models for actually generating.**

That is also why VAE remains central in 2026. It is the part that turns pixels into units a foundation model can both **reason over** and **decode back into visible content**.

A useful summary is this: in 2026, VAE is no longer only a chapter in generative modeling. It is part of the infrastructure that turns pixels into latent units that a foundation model can actually model, predict, and decode.

---

## 12. Conclusion

ELBO, EM, GEM, and VAE are not isolated concepts. They form a single coherent progression.

We begin with the log-sum difficulty in latent variable models. Jensen’s inequality then gives us the ELBO, replacing an intractable marginal likelihood with a manageable lower bound. EM and GEM show that we can improve this bound by alternating between better latent-variable inference and better model fitting. Finally, the VAE packages the same logic into a modern deep-learning framework using an encoder, a decoder, amortized inference, reparameterization, and stochastic gradient optimization.

The cleanest way to understand a VAE, then, is not as an isolated deep-learning trick, but as the modern continuation of a much older line of thought in latent variable modeling:

**VAE = ELBO as the objective + GEM as the optimization logic + neural parameterization and reparameterization as the implementation.**
