---
layout: post
title: "You Don't Need a Fancy Optimizer"
date: 2026-06-24
categories: [Deep Learning, Optimization]
tags: [SGD, Adam, Optimization, Training, PyTorch, Loss Landscape]
description: >
  A rigorous look at why SGD with momentum often beats Adam, when adaptive
  methods hurt generalization, and what the math actually tells us about
  choosing an optimizer.
---

There is a strange ritual in modern deep learning: you start a new project, copy a training script, and reach for Adam without thinking. It converges fast, it's forgiving of bad learning rates, and everyone uses it. What more do you need?

Quite a lot, it turns out. The optimizer you choose doesn't just control *how fast* you converge, it shapes *where* you converge. And the flat, wide minima that generalize best are often found not by the most sophisticated adaptive method, but by the simplest: SGD with momentum and a well-tuned learning rate.

This post is about why.

---

## The Setup: What We're Actually Optimizing

We want to minimize the empirical risk over a dataset $$\mathcal{D} = \{(\mathbf{x}_i, y_i)\}_{i=1}^N$$:

$$\mathcal{L}(\theta) = \frac{1}{N} \sum_{i=1}^N \ell(f_\theta(\mathbf{x}_i), y_i)$$

This is intractable to minimize exactly for large $N$, so we use **stochastic gradient descent**: at each step, we sample a minibatch $\mathcal{B} \subset \mathcal{D}$ and compute a noisy gradient estimate:

$$\mathbf{g}_t = \nabla_\theta \frac{1}{|\mathcal{B}|} \sum_{i \in \mathcal{B}} \ell(f_\theta(\mathbf{x}_i), y_i)$$

This estimate is **unbiased**: $$\mathbb{E}[\mathbf{g}_t] = \nabla_\theta \mathcal{L}(\theta)$$, but it has variance $$\sigma^2 / \vert\mathcal{B}\vert$$. Everything that follows, every optimizer, is a different strategy for using $$\mathbf{g}_t$$ to update $$\theta$$.

---

## SGD: The Baseline

Vanilla SGD updates parameters in the direction of the negative gradient:

$$\theta_{t+1} = \theta_t - \eta \mathbf{g}_t$$

where $\eta$ is the learning rate. This is the steepest descent step on the stochastic loss surface. It's geometrically clear: move opposite the gradient by a fixed step size.

The convergence rate for a $\mu$-strongly convex, $L$-smooth loss is:

$$\mathcal{L}(\theta_t) - \mathcal{L}(\theta^*) \leq \left(1 - \frac{\mu}{L}\right)^t [\mathcal{L}(\theta_0) - \mathcal{L}(\theta^*)]$$

The factor $\kappa = L / \mu$ is the **condition number** of the loss landscape. When $\kappa$ is large, elongated, poorly-scaled, loss bowls convergence is slow. This is SGD's core weakness.

### SGD with Momentum

Momentum accelerates SGD by accumulating a velocity vector:

$$\mathbf{v}_{t+1} = \beta \mathbf{v}_t + \mathbf{g}_t$$
$$\theta_{t+1} = \theta_t - \eta \mathbf{v}_{t+1}$$

Expanding the recursion, the effective update is a geometric series over past gradients:

$$\mathbf{v}_t = \sum_{k=0}^{t} \beta^k \mathbf{g}_{t-k}$$

In directions where gradients are consistently aligned (shallow but consistent slopes), momentum accumulates, effectively scaling up the step. In directions where gradients oscillate (steep ravines), consecutive terms cancel, damping the oscillation. The result is faster traversal of elongated loss basins.

The steady-state speed is amplified by a factor of $\frac{1}{1-\beta}$. At $\beta = 0.9$, that's a 10× effective learning rate in consistent gradient directions.

```python
class SGDMomentum:
    """Vanilla SGD with momentum, written explicitly for clarity."""

    def __init__(self, params, lr: float, momentum: float = 0.9):
        self.params   = list(params)
        self.lr       = lr
        self.beta     = momentum
        self.velocity = [torch.zeros_like(p) for p in self.params]

    @torch.no_grad()
    def step(self):
        for p, v in zip(self.params, self.velocity):
            if p.grad is None:
                continue
            v.mul_(self.beta).add_(p.grad)       # v = β·v + g
            p.sub_(v, alpha=self.lr)             # θ = θ - η·v
```

---

## Adam: What It Actually Does

Adam (Adaptive Moment Estimation, Kingma & Ba, 2015) maintains per-parameter running estimates of the first and second moments of the gradient:

$$\mathbf{m}_t = \beta_1 \mathbf{m}_{t-1} + (1 - \beta_1) \mathbf{g}_t \quad \text{(first moment — mean)}$$
$$\mathbf{v}_t = \beta_2 \mathbf{v}_{t-1} + (1 - \beta_2) \mathbf{g}_t^2 \quad \text{(second moment — uncentered variance)}$$

These are biased toward zero at initialisation (since $\mathbf{m}_0 = \mathbf{v}_0 = \mathbf{0}$), so we correct:

$$\hat{\mathbf{m}}_t = \frac{\mathbf{m}_t}{1 - \beta_1^t}, \qquad \hat{\mathbf{v}}_t = \frac{\mathbf{v}_t}{1 - \beta_2^t}$$

The parameter update is:

$$\theta_{t+1} = \theta_t - \eta \cdot \frac{\hat{\mathbf{m}}_t}{\sqrt{\hat{\mathbf{v}}_t} + \epsilon}$$

The division by $\sqrt{\hat{\mathbf{v}}_t}$ is the key: it normalises each parameter's update by the root mean square of its recent gradients. Parameters that receive large, consistent gradients get small steps; parameters that receive small or rare gradients get large steps. This is **implicit per-parameter learning rate scaling**.

```python
class AdamExplicit:
    """Adam written out step-by-step to make the math visible."""

    def __init__(self, params, lr=1e-3, betas=(0.9, 0.999), eps=1e-8):
        self.params = list(params)
        self.lr     = lr
        self.b1, self.b2 = betas
        self.eps    = eps
        self.m = [torch.zeros_like(p) for p in self.params]
        self.v = [torch.zeros_like(p) for p in self.params]
        self.t = 0

    @torch.no_grad()
    def step(self):
        self.t += 1
        bc1 = 1 - self.b1 ** self.t   # bias correction factors
        bc2 = 1 - self.b2 ** self.t

        for p, m, v in zip(self.params, self.m, self.v):
            if p.grad is None:
                continue
            g = p.grad

            m.mul_(self.b1).add_(g, alpha=1 - self.b1)   # first moment
            v.mul_(self.b2).addcmul_(g, g, value=1 - self.b2)  # second moment

            m_hat = m / bc1
            v_hat = v / bc2

            p.sub_(m_hat / (v_hat.sqrt() + self.eps), alpha=self.lr)
```

Standard defaults: $\beta_1 = 0.9$, $\beta_2 = 0.999$, $\epsilon = 10^{-8}$, $\eta = 10^{-3}$.

---

## Where Adam Wins

The adaptive scaling genuinely helps in three situations:

**1. Sparse gradients.** In NLP and recommendation systems, most embedding rows receive zero gradient on any given batch. Adam's $\mathbf{v}_t$ accumulates only when gradients are non-zero, so infrequently updated parameters can take large steps when they finally receive signal. SGD would require a global learning rate low enough to handle the common parameters, starving the rare ones.

**2. Ill-conditioned loss landscapes.** When $\kappa = L/\mu$ is large, the loss bowl is elongated. Adam's per-parameter scaling effectively preconditions the gradient, approximating a diagonal of the inverse Hessian. This dramatically accelerates convergence on badly-scaled problems.

**3. Early training.** Adam is forgiving of learning rate choice. A learning rate that's 10× too large will still converge (slowly, noisily), whereas SGD will diverge. This makes prototyping faster.

---

## Where Adam Loses - and Why

### The Generalisation Gap

The most reproduced empirical finding: **models trained with SGD generalize better than those trained with Adam**, even when Adam reaches a lower training loss.

Keskar et al. (2017) showed that sharp minima, with large Hessian eigenvalues correlate strongly with poor generalization. The intuition: a sharp minimum has high curvature, meaning small perturbations to $\theta$ cause large changes in loss. A flat minimum is robust to perturbations, which acts as implicit regularisation.

Adam's adaptive scaling tends to find sharper minima. Here's why: the normalisation $\hat{\mathbf{m}}_t / \sqrt{\hat{\mathbf{v}}_t}$ makes the effective step size roughly uniform across parameters regardless of gradient magnitude. This means Adam can escape the large-gradient directions (which often correspond to high-curvature directions) while still making progress in low-gradient directions. The result is convergence into tighter basins.

SGD with momentum, by contrast, takes larger steps in high-gradient directions and is naturally pushed away from sharp minima by its own momentum. The noise in stochastic gradients further acts as a regulariser, preventing the optimizer from settling into narrow basins.

### The Learning Rate Scaling Problem

For a batch size $$\vert\mathcal{B}\vert$$, the variance of the stochastic gradient scales as $$\sigma^2 / \vert\mathcal{B}\vert$$. For SGD, the **linear scaling rule** (Goyal et al., 2017) says: multiply the learning rate by $$k$$ when multiplying the batch size by $$k$$. This keeps the noise-to-signal ratio constant and transfers well across batch sizes.

Adam has no clean scaling rule. Its second moment estimate $\hat{\mathbf{v}}_t$ depends on gradient variance, which itself changes with batch size. Scaling Adam across batch sizes is empirically messy.

### Weight Decay is Broken in Adam

L2 regularisation adds $\frac{\lambda}{2} \|\theta\|^2$ to the loss, producing a gradient penalty $\lambda \theta$. In SGD, this is equivalent to **weight decay**: $\theta \leftarrow (1 - \lambda\eta)\theta - \eta \mathbf{g}_t$.

In Adam, the L2 gradient $\lambda\theta$ is absorbed into $\mathbf{g}_t$, then divided by $\sqrt{\hat{\mathbf{v}}_t}$. Parameters with large gradient variance get smaller regularisation than parameters with small gradient variance i.e L2 regularisation becomes non-uniform and loses its geometric meaning.

**AdamW** (Loshchilov & Hutter, 2019) fixes this by decoupling weight decay from the gradient update:

$$\theta_{t+1} = \theta_t - \eta \left(\frac{\hat{\mathbf{m}}_t}{\sqrt{\hat{\mathbf{v}}_t} + \epsilon} + \lambda \theta_t\right)$$

The weight decay $\lambda \theta_t$ is applied directly to the parameters, not through the gradient normalisation. This restores the correct geometric effect and is almost always preferable to Adam with L2 regularisation.

```python
class AdamW_Explicit:
    """AdamW: Adam with decoupled weight decay."""

    def __init__(self, params, lr=1e-3, betas=(0.9, 0.999),
                 eps=1e-8, weight_decay=1e-2):
        self.params       = list(params)
        self.lr           = lr
        self.b1, self.b2  = betas
        self.eps          = eps
        self.wd           = weight_decay
        self.m = [torch.zeros_like(p) for p in self.params]
        self.v = [torch.zeros_like(p) for p in self.params]
        self.t = 0

    @torch.no_grad()
    def step(self):
        self.t += 1
        bc1 = 1 - self.b1 ** self.t
        bc2 = 1 - self.b2 ** self.t

        for p, m, v in zip(self.params, self.m, self.v):
            if p.grad is None:
                continue
            g = p.grad

            m.mul_(self.b1).add_(g, alpha=1 - self.b1)
            v.mul_(self.b2).addcmul_(g, g, value=1 - self.b2)

            m_hat = m / bc1
            v_hat = v / bc2

            # Adaptive gradient step
            p.sub_(m_hat / (v_hat.sqrt() + self.eps), alpha=self.lr)
            # Decoupled weight decay — applied to parameters, not gradients
            p.mul_(1 - self.lr * self.wd)
```

---

## The Learning Rate Schedule Is Doing More Work Than You Think

A fixed learning rate is almost never optimal. Two schedules dominate modern practice:

### Cosine Annealing

$$\eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max} - \eta_{\min})\left(1 + \cos\frac{\pi t}{T}\right)$$

The learning rate starts at $\eta_{\max}$, decays smoothly to $\eta_{\min}$, and (optionally) restarts. Cosine annealing with warm restarts (SGDR, Loshchilov & Hutter, 2017) allows the optimizer to escape shallow local minima at restart points.

### Linear Warmup

Large learning rates early in training cause instability meaning the loss landscape is poorly explored and gradients are large and noisy. A linear warmup from $0$ to $\eta_{\max}$ over the first few hundred steps stabilises early training:

$$\eta_t = \eta_{\max} \cdot \frac{t}{t_{\text{warmup}}}, \quad t < t_{\text{warmup}}$$

```python
def get_scheduler(optimizer, warmup_steps: int, total_steps: int,
                  eta_min: float = 1e-6):
    """
    Linear warmup followed by cosine decay.
    This single schedule accounts for much of the performance gap
    attributed to 'better optimizers'.
    """
    def lr_lambda(step):
        if step < warmup_steps:
            return step / max(1, warmup_steps)
        progress = (step - warmup_steps) / max(1, total_steps - warmup_steps)
        return eta_min + 0.5 * (1 - eta_min) * (1 + math.cos(math.pi * progress))

    return torch.optim.lr_scheduler.LambdaLR(optimizer, lr_lambda)
```

The practical implication: much of the credit given to Adam's fast convergence actually belongs to the schedule. SGD with warmup + cosine annealing matches or beats Adam on image classification, and equals it on language model fine-tuning with careful tuning.

---

## When to Use What

| Setting | Recommended | Why |
|---|---|---|
| Image classification (ResNet, ViT) | SGD + momentum + cosine | Wider minima, better top-1 |
| Language model pretraining | AdamW + warmup + cosine | Sparse gradients, large $\kappa$ |
| Fine-tuning pretrained models | AdamW | Small LR regime; Adam's stability helps |
| Sparse embeddings (RecSys, NLP) | Adam / AdamW | Sparse gradient updates |
| Differentiable physics / science ML | L-BFGS or SGD | Smooth, low-dim losses |
| Quick prototyping | Adam | Forgiving of hyperparameters |

---

## The Practical Lesson

The optimizer is rarely the bottleneck. In a controlled comparison on ResNet-50 / ImageNet, SGD with momentum, a properly tuned learning rate, cosine annealing, and weight decay matches the best Adam variants and generalises better.

The variables that matter more than optimizer choice:

- **Learning rate** - the single highest-leverage hyperparameter
- **Batch size** - directly controls gradient variance and effective step size
- **Weight decay** - the most important regulariser in many regimes
- **Schedule shape** - warmup + decay accounts for most of Adam's perceived advantage

Before reaching for Lion, Adan, Sophia, or the optimizer-of-the-month, ask whether SGD with momentum and a cosine schedule has been properly tuned. It usually hasn't. And it usually works.

The most sophisticated algorithm is not always the best one. Sometimes the oldest tool, applied carefully, is exactly enough.