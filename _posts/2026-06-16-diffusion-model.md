---
layout: post
title: "Diffusion Models: The Math, the Architecture, and the Code"
date: 2026-06-23
categories: [Deep Learning, Generative Models]
tags: [Diffusion Models, DDPM, Score Matching, PyTorch, Generative AI, U-Net]
description: >
  A rigorous walkthrough of denoising diffusion probabilistic models: the forward and
  reverse processes, the variational lower bound, score matching, and a complete
  PyTorch implementation of DDPM with a time-conditioned U-Net.
---

Generative adversarial networks dominated image synthesis for nearly a decade. Then diffusion models arrived and outperformed them on almost every benchmark while being more stable to train and easier to reason about theoretically.

This post explains *why* that happened, starting from first principles: the forward noising process, the reverse denoising process, the variational lower bound, and how all of it reduces to a surprisingly simple training objective. Then we build the full stack in PyTorch.

---

## The Core Intuition

Diffusion models operate on a simple idea: **if you can learn to reverse a noise process, you can generate data from noise**.

The forward process takes a clean data sample $\mathbf{x}_0$ and gradually corrupts it with Gaussian noise over $T$ timesteps until it becomes indistinguishable from pure Gaussian noise $\mathbf{x}_T \sim \mathcal{N}(\mathbf{0}, \mathbf{I})$.

The reverse process starts from that noise and learns to iteratively recover the original sample. Training a neural network to do this reversal is the core task.

---

## The Forward Process

Define a fixed Markov chain of increasingly noisy versions of $\mathbf{x}_0$:

$$q(\mathbf{x}_t \mid \mathbf{x}_{t-1}) = \mathcal{N}\!\left(\mathbf{x}_t;\, \sqrt{1-\beta_t}\,\mathbf{x}_{t-1},\, \beta_t \mathbf{I}\right)$$

where $\beta_1 < \beta_2 < \cdots < \beta_T$ is a **noise schedule** (typically linear or cosine). The variance $\beta_t$ grows with $t$, so early steps add tiny perturbations and later steps add large noise.

### The Closed-Form Shortcut

The crucial insight that makes training tractable: we can sample $$\mathbf{x}_t$$ directly from $$\mathbf{x}_0$$ *without* running all $$t$$ steps. Define $$\alpha_t = 1 - \beta_t$$ and $$\bar{\alpha}_t = \prod_{s=1}^{t} \alpha_s$$. Then:

$$q(\mathbf{x}_t \mid \mathbf{x}_0) = \mathcal{N}\!\left(\mathbf{x}_t;\, \sqrt{\bar{\alpha}_t}\,\mathbf{x}_0,\, (1-\bar{\alpha}_t)\mathbf{I}\right)$$

This means we can write any $\mathbf{x}_t$ as:

$$\mathbf{x}_t = \sqrt{\bar{\alpha}_t}\,\mathbf{x}_0 + \sqrt{1 - \bar{\alpha}_t}\,\boldsymbol{\epsilon}, \quad \boldsymbol{\epsilon} \sim \mathcal{N}(\mathbf{0}, \mathbf{I})$$

This is the reparameterisation that makes DDPM fast to train.

---

## The Reverse Process

The reverse process is what we model with a neural network:

$$p_\theta(\mathbf{x}_{t-1} \mid \mathbf{x}_t) = \mathcal{N}\!\left(\mathbf{x}_{t-1};\, \boldsymbol{\mu}_\theta(\mathbf{x}_t, t),\, \sigma_t^2 \mathbf{I}\right)$$

The posterior mean $\boldsymbol{\mu}_\theta$ can be parameterised in several equivalent ways. Ho et al. (DDPM, 2020) showed that **predicting the noise** $\boldsymbol{\epsilon}$ added at each step leads to the simplest and most stable training:

$$\boldsymbol{\mu}_\theta(\mathbf{x}_t, t) = \frac{1}{\sqrt{\alpha_t}}\left(\mathbf{x}_t - \frac{1-\alpha_t}{\sqrt{1-\bar{\alpha}_t}}\,\boldsymbol{\epsilon}_\theta(\mathbf{x}_t, t)\right)$$

---

## The Training Objective

The full objective is the variational lower bound on the log-likelihood $\log p_\theta(\mathbf{x}_0)$. After careful derivation — which involves decomposing the ELBO into per-step KL divergences between Gaussians — it simplifies to:

$$\mathcal{L}_{\text{simple}} = \mathbb{E}_{t, \mathbf{x}_0, \boldsymbol{\epsilon}}\!\left[\left\|\boldsymbol{\epsilon} - \boldsymbol{\epsilon}_\theta\!\left(\sqrt{\bar{\alpha}_t}\,\mathbf{x}_0 + \sqrt{1-\bar{\alpha}_t}\,\boldsymbol{\epsilon},\; t\right)\right\|^2\right]$$

This is just **mean squared error between the true noise and the predicted noise**. The network $\boldsymbol{\epsilon}_\theta$ takes a noisy image and a timestep, and predicts what noise was added. Nothing more.

---

## Noise Schedules

The schedule $\{\beta_t\}$ controls how quickly signal is destroyed. Two standard choices:

```python
import torch
import numpy as np


def linear_schedule(T: int, beta_start: float = 1e-4,
                    beta_end: float = 0.02) -> torch.Tensor:
    """
    Linear noise schedule from Ho et al. (DDPM, 2020).
    Small beta values early, larger values late.
    """
    return torch.linspace(beta_start, beta_end, T)


def cosine_schedule(T: int, s: float = 0.008) -> torch.Tensor:
    """
    Cosine noise schedule from Nichol & Dhariwal (Improved DDPM, 2021).
    Avoids over-noising at the end of the schedule, which can waste capacity
    on near-pure-noise timesteps where the network can't learn anything useful.

    Returns betas derived from cosine-shaped alpha_bar curve.
    """
    steps = T + 1
    t     = torch.linspace(0, T, steps)
    f     = torch.cos(((t / T) + s) / (1 + s) * np.pi * 0.5) ** 2
    alpha_bar = f / f[0]
    betas = 1 - (alpha_bar[1:] / alpha_bar[:-1])
    return betas.clamp(max=0.999)
```

The cosine schedule is generally preferred for high-resolution images because the linear schedule destroys too much signal in the final steps.

---

## DDPM Diffusion Engine

```python
class DiffusionEngine:
    """
    Manages the forward (noising) and reverse (denoising) processes.
    Acts as a stateless utility class — the model itself is external.
    """

    def __init__(self, T: int = 1000, schedule: str = 'cosine',
                 device: str = 'cuda'):
        self.T      = T
        self.device = device

        betas = (cosine_schedule(T) if schedule == 'cosine'
                 else linear_schedule(T))

        # Pre-compute all derived quantities and move to device
        alphas      = 1.0 - betas
        alpha_bar   = torch.cumprod(alphas, dim=0)
        alpha_bar_prev = torch.cat([torch.tensor([1.0]), alpha_bar[:-1]])

        def reg(t): return t.float().to(device)

        self.betas          = reg(betas)
        self.alphas         = reg(alphas)
        self.alpha_bar      = reg(alpha_bar)
        self.alpha_bar_prev = reg(alpha_bar_prev)
        self.sqrt_alpha_bar         = reg(torch.sqrt(alpha_bar))
        self.sqrt_one_minus_ab      = reg(torch.sqrt(1.0 - alpha_bar))
        self.sqrt_recip_alpha       = reg(torch.sqrt(1.0 / alphas))
        # Posterior variance
        self.posterior_variance = reg(
            betas * (1.0 - alpha_bar_prev) / (1.0 - alpha_bar)
        )

    def _gather(self, coeff, t, shape):
        """Index coefficient array at timesteps t, then broadcast to shape."""
        out = coeff[t]
        return out.view(t.shape[0], *([1] * (len(shape) - 1)))

    def q_sample(self, x0: torch.Tensor, t: torch.Tensor,
                 noise: torch.Tensor = None) -> torch.Tensor:
        """
        Forward process: sample x_t from x_0.
        x_t = sqrt(alpha_bar_t) * x_0 + sqrt(1 - alpha_bar_t) * eps
        """
        if noise is None:
            noise = torch.randn_like(x0)
        sqrt_ab  = self._gather(self.sqrt_alpha_bar, t, x0.shape)
        sqrt_1ab = self._gather(self.sqrt_one_minus_ab, t, x0.shape)
        return sqrt_ab * x0 + sqrt_1ab * noise, noise

    @torch.no_grad()
    def p_sample(self, model: torch.nn.Module,
                 x_t: torch.Tensor, t: int) -> torch.Tensor:
        """
        Single reverse step: sample x_{t-1} from x_t using the model.
        """
        t_tensor = torch.full((x_t.shape[0],), t,
                              device=self.device, dtype=torch.long)
        eps_pred  = model(x_t, t_tensor)

        # Compute predicted mean
        betas_t    = self._gather(self.betas, t_tensor, x_t.shape)
        sqrt_1ab_t = self._gather(self.sqrt_one_minus_ab, t_tensor, x_t.shape)
        sqrt_ra_t  = self._gather(self.sqrt_recip_alpha, t_tensor, x_t.shape)
        mu         = sqrt_ra_t * (x_t - betas_t / sqrt_1ab_t * eps_pred)

        if t == 0:
            return mu

        var   = self._gather(self.posterior_variance, t_tensor, x_t.shape)
        noise = torch.randn_like(x_t)
        return mu + torch.sqrt(var) * noise

    @torch.no_grad()
    def sample(self, model: torch.nn.Module,
               shape: tuple, show_progress: bool = True) -> torch.Tensor:
        """
        Full reverse chain: generate samples from pure Gaussian noise.
        """
        from tqdm import tqdm
        x = torch.randn(shape, device=self.device)
        timesteps = reversed(range(self.T))
        if show_progress:
            timesteps = tqdm(list(timesteps), desc='Sampling')
        for t in timesteps:
            x = self.p_sample(model, x, t)
        return x   # in [-1, 1] if model was trained on normalised images
```

---

## The Time-Conditioned U-Net

The backbone of the noise predictor $\boldsymbol{\epsilon}_\theta$ is a U-Net augmented with **sinusoidal timestep embeddings** and **cross-attention** at bottleneck layers.

### Sinusoidal Timestep Embedding

We encode $t \in \{1, \ldots, T\}$ as a fixed-frequency sinusoidal vector, then project it to the model's hidden dimension:

```python
import torch
import torch.nn as nn
import math


class SinusoidalEmbedding(nn.Module):
    """
    Fixed sinusoidal positional encoding for timesteps, following
    the Transformer convention. Projected to d_model via a small MLP.
    """

    def __init__(self, d_model: int):
        super().__init__()
        self.d_model = d_model
        self.mlp = nn.Sequential(
            nn.Linear(d_model, d_model * 4),
            nn.SiLU(),
            nn.Linear(d_model * 4, d_model),
        )

    def forward(self, t: torch.Tensor) -> torch.Tensor:
        # t: (B,) integer timesteps
        half = self.d_model // 2
        freqs = torch.exp(
            -math.log(10000) *
            torch.arange(half, device=t.device).float() / half
        )
        args  = t[:, None].float() * freqs[None]       # (B, half)
        emb   = torch.cat([torch.sin(args), torch.cos(args)], dim=-1)  # (B, d)
        return self.mlp(emb)
```

### Residual Block with Time Conditioning

```python
class ResBlockTime(nn.Module):
    """
    Residual block that conditions on the timestep embedding via
    scale-shift normalisation (also called AdaGN or FiLM conditioning).

    The timestep embedding gamma-shifts the GroupNorm statistics,
    allowing the network to adapt its behaviour to each noise level.
    """

    def __init__(self, in_ch: int, out_ch: int, t_dim: int,
                 n_groups: int = 8, dropout: float = 0.1):
        super().__init__()
        self.norm1 = nn.GroupNorm(n_groups, in_ch)
        self.conv1 = nn.Conv2d(in_ch, out_ch, 3, padding=1)

        self.t_proj = nn.Sequential(
            nn.SiLU(),
            nn.Linear(t_dim, out_ch * 2)   # scale + shift
        )

        self.norm2   = nn.GroupNorm(n_groups, out_ch)
        self.drop    = nn.Dropout2d(dropout)
        self.conv2   = nn.Conv2d(out_ch, out_ch, 3, padding=1)
        self.skip    = (nn.Conv2d(in_ch, out_ch, 1)
                        if in_ch != out_ch else nn.Identity())

    def forward(self, x: torch.Tensor, t_emb: torch.Tensor) -> torch.Tensor:
        h = self.conv1(torch.nn.functional.silu(self.norm1(x)))

        # Scale-shift conditioning
        scale, shift = self.t_proj(t_emb).chunk(2, dim=-1)
        h = self.norm2(h) * (1 + scale[:, :, None, None]) \
                          + shift[:, :, None, None]

        h = self.conv2(self.drop(torch.nn.functional.silu(h)))
        return h + self.skip(x)
```

### Self-Attention at Bottleneck

```python
class SpatialAttention(nn.Module):
    """
    Multi-head self-attention over spatial positions.
    Applied at the bottleneck (coarsest resolution) where global context
    is needed — modelling long-range dependencies across the image.
    """

    def __init__(self, channels: int, n_heads: int = 4):
        super().__init__()
        self.norm  = nn.GroupNorm(8, channels)
        self.attn  = nn.MultiheadAttention(channels, n_heads, batch_first=True)
        self.proj  = nn.Conv2d(channels, channels, 1)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        B, C, H, W = x.shape
        h = self.norm(x).view(B, C, H * W).permute(0, 2, 1)  # (B, HW, C)
        h, _ = self.attn(h, h, h)
        h = h.permute(0, 2, 1).view(B, C, H, W)
        return x + self.proj(h)
```

### Full Noise-Prediction U-Net

```python
class DiffusionUNet(nn.Module):
    """
    Time-conditioned U-Net for noise prediction (epsilon parameterisation).

    Input : x_t (B, C, H, W) — noisy image at timestep t
            t   (B,)          — integer timesteps
    Output: eps (B, C, H, W) — predicted noise
    """

    def __init__(self, img_channels: int = 3,
                 features: tuple = (64, 128, 256, 512),
                 t_dim: int = 256, n_res_blocks: int = 2):
        super().__init__()
        self.t_embed = SinusoidalEmbedding(t_dim)
        self.head    = nn.Conv2d(img_channels, features[0], 3, padding=1)

        self.encoder_blocks = nn.ModuleList()
        self.downsamples    = nn.ModuleList()
        ch = features[0]
        for f in features:
            blocks = nn.ModuleList([
                ResBlockTime(ch, f, t_dim) for _ in range(n_res_blocks)
            ])
            self.encoder_blocks.append(blocks)
            self.downsamples.append(
                nn.Conv2d(f, f, 3, stride=2, padding=1) if f != features[-1]
                else nn.Identity()
            )
            ch = f

        # Bottleneck: ResBlock → Attention → ResBlock
        self.mid_res1 = ResBlockTime(ch, ch, t_dim)
        self.mid_attn = SpatialAttention(ch)
        self.mid_res2 = ResBlockTime(ch, ch, t_dim)

        self.decoder_blocks = nn.ModuleList()
        self.upsamples      = nn.ModuleList()
        for f in reversed(features):
            self.upsamples.append(
                nn.ConvTranspose2d(ch, f, 2, stride=2) if ch != features[0]
                else nn.Identity()
            )
            self.decoder_blocks.append(nn.ModuleList([
                ResBlockTime(f * 2, f, t_dim) for _ in range(n_res_blocks)
            ]))
            ch = f

        self.tail = nn.Sequential(
            nn.GroupNorm(8, ch),
            nn.SiLU(),
            nn.Conv2d(ch, img_channels, 3, padding=1)
        )

    def forward(self, x: torch.Tensor, t: torch.Tensor) -> torch.Tensor:
        t_emb  = self.t_embed(t)
        h      = self.head(x)
        skips  = []

        for blocks, down in zip(self.encoder_blocks, self.downsamples):
            for block in blocks:
                h = block(h, t_emb)
            skips.append(h)
            h = down(h)

        h = self.mid_res1(h, t_emb)
        h = self.mid_attn(h)
        h = self.mid_res2(h, t_emb)

        for up, blocks, skip in zip(
            self.upsamples, self.decoder_blocks, reversed(skips)
        ):
            h = up(h)
            h = torch.cat([h, skip], dim=1)
            for block in blocks:
                h = block(h, t_emb)

        return self.tail(h)
```

---

## Training Loop

```python
import torch
from torch.optim import AdamW
from torch.optim.lr_scheduler import CosineAnnealingLR


def train_ddpm(model: torch.nn.Module,
               dataloader,
               diffusion: DiffusionEngine,
               n_epochs: int = 100,
               lr: float = 2e-4,
               device: str = 'cuda') -> list:
    """
    Standard DDPM training loop.
    Loss = MSE(true_noise, predicted_noise), averaged over random (x0, t) pairs.
    """
    model     = model.to(device)
    optimiser = AdamW(model.parameters(), lr=lr, weight_decay=1e-4)
    scheduler = CosineAnnealingLR(optimiser, T_max=n_epochs)
    losses    = []

    for epoch in range(n_epochs):
        model.train()
        epoch_loss = 0.0
        for batch in dataloader:
            x0 = batch[0].to(device)  # (B, C, H, W), normalised to [-1, 1]
            B  = x0.shape[0]

            # Sample random timesteps uniformly for each item in batch
            t = torch.randint(0, diffusion.T, (B,), device=device)

            # Sample noise and corrupt x0
            x_t, noise = diffusion.q_sample(x0, t)

            # Predict the noise
            noise_pred = model(x_t, t)

            loss = torch.nn.functional.mse_loss(noise_pred, noise)

            optimiser.zero_grad(set_to_none=True)
            loss.backward()
            torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
            optimiser.step()
            epoch_loss += loss.item()

        scheduler.step()
        avg = epoch_loss / len(dataloader)
        losses.append(avg)
        if epoch % 10 == 0:
            print(f'Epoch {epoch:4d} | Loss {avg:.5f}')

    return losses
```

---

## Faster Sampling: DDIM

The original DDPM requires $T = 1000$ reverse steps to generate one sample — slow at inference. **DDIM** (Song et al., 2020) reformulates the reverse process as a non-Markovian deterministic ODE, enabling high-quality samples in as few as 50 steps:

```python
@torch.no_grad()
def ddim_sample(model: torch.nn.Module,
                diffusion: DiffusionEngine,
                shape: tuple,
                n_steps: int = 50,
                eta: float = 0.0,
                device: str = 'cuda') -> torch.Tensor:
    """
    DDIM sampling. eta=0 is fully deterministic (ODE);
    eta=1 recovers stochastic DDPM sampling.

    Selects n_steps evenly spaced timesteps from the full schedule.
    """
    timesteps = torch.linspace(diffusion.T - 1, 0, n_steps).long()
    x = torch.randn(shape, device=device)

    for i, t in enumerate(timesteps):
        t_batch = torch.full((shape[0],), t, device=device, dtype=torch.long)
        eps     = model(x, t_batch)

        ab_t    = diffusion.alpha_bar[t]
        ab_prev = diffusion.alpha_bar[timesteps[i + 1]] if i + 1 < n_steps \
                  else torch.tensor(1.0, device=device)

        # Predicted x_0 from current x_t and eps
        x0_pred = (x - torch.sqrt(1 - ab_t) * eps) / torch.sqrt(ab_t)
        x0_pred = x0_pred.clamp(-1, 1)

        sigma = eta * torch.sqrt(
            (1 - ab_prev) / (1 - ab_t) * (1 - ab_t / ab_prev)
        )
        noise  = torch.randn_like(x) if eta > 0 else torch.zeros_like(x)

        x = (torch.sqrt(ab_prev) * x0_pred
             + torch.sqrt(1 - ab_prev - sigma ** 2) * eps
             + sigma * noise)

    return x
```

---

## Conditional Generation

So far, the model generates unconditional samples. For class-conditional or text-conditional generation, the standard approach is **classifier-free guidance (CFG)**.

During training, the conditioning signal $\mathbf{c}$ (a class label, text embedding, etc.) is dropped with probability $p_{\text{uncond}} \approx 0.1$, replaced by a null token. This teaches the model both the conditional and unconditional score.

At inference, the noise prediction is steered:

$$\hat{\boldsymbol{\epsilon}} = \boldsymbol{\epsilon}_\theta(\mathbf{x}_t, \emptyset) + w \cdot \left[\boldsymbol{\epsilon}_\theta(\mathbf{x}_t, \mathbf{c}) - \boldsymbol{\epsilon}_\theta(\mathbf{x}_t, \emptyset)\right]$$

where $w > 1$ is the **guidance scale** — higher values increase fidelity to the condition but reduce diversity.

```python
@torch.no_grad()
def cfg_sample(model, diffusion, shape, condition,
               guidance_scale: float = 7.5,
               n_steps: int = 50, device: str = 'cuda'):
    """
    Classifier-free guidance sampling.
    Assumes model accepts (x_t, t, c) where c=None means unconditional.
    """
    null = torch.zeros_like(condition)   # null conditioning token
    x    = torch.randn(shape, device=device)
    timesteps = torch.linspace(diffusion.T - 1, 0, n_steps).long()

    for t in timesteps:
        t_b = torch.full((shape[0],), t, device=device, dtype=torch.long)

        eps_uncond = model(x, t_b, null)
        eps_cond   = model(x, t_b, condition)

        # Linear extrapolation in noise-prediction space
        eps = eps_uncond + guidance_scale * (eps_cond - eps_uncond)

        # DDIM update with guided eps (simplified, eta=0)
        ab_t  = diffusion.alpha_bar[t]
        x0_p  = (x - torch.sqrt(1 - ab_t) * eps) / torch.sqrt(ab_t)
        x0_p  = x0_p.clamp(-1, 1)
        ab_p  = diffusion.alpha_bar[max(t - diffusion.T // n_steps, 0)]
        x     = torch.sqrt(ab_p) * x0_p + torch.sqrt(1 - ab_p) * eps

    return x
```

---

## The Score Matching Connection

There is a deep theoretical connection between diffusion models and **score matching**. The score function is the gradient of the log-probability density:

$$\mathbf{s}(\mathbf{x}) = \nabla_{\mathbf{x}} \log p(\mathbf{x})$$

Song et al. showed that the noise-predicting network $\boldsymbol{\epsilon}_\theta(\mathbf{x}_t, t)$ is equivalent (up to a scaling factor) to learning the score function of the noisy distribution at noise level $t$:

$$\boldsymbol{\epsilon}_\theta(\mathbf{x}_t, t) \approx -\sqrt{1 - \bar{\alpha}_t} \cdot \nabla_{\mathbf{x}_t} \log q(\mathbf{x}_t)$$

The reverse process follows the gradient of the log-density — walking uphill on the probability landscape, from low-probability noise toward high-probability data. This unifies DDPM with **Langevin dynamics** and **stochastic differential equations (SDEs)**, connecting it to a rich body of statistical physics literature.

---

## Applications

| Domain | Application | Key Modification |
|--------|-------------|-----------------|
| Image synthesis | Photo generation | Standard DDPM / DDIM |
| Text-to-image | Stable Diffusion | Latent space + cross-attention |
| Audio | WaveGrad, DiffWave | 1D U-Net over waveform |
| Molecular design | DiffSBDD | Graph diffusion over 3D atom positions |
| Weather forecasting | GenCast | Spherical U-Net + diffusion on ERA5 |
| SAR image translation | SAR-to-optical | Conditional diffusion, paired data |

The architectural core (U-Net + timestep conditioning) remains consistent across domains; what changes is the data representation and the conditioning mechanism.

---

## Key Takeaways

Diffusion models achieve state-of-the-art generation quality by decomposing an intractable generative task into a sequence of small, tractable denoising problems. The full derivation reduces to training a network to predict Gaussian noise which is an MSE objective that is globally stable and free of the mode collapse and training instability that plagued GANs.

The components built here: the diffusion engine, the time-conditioned U-Net, DDIM sampling, and classifier-free guidance are the exact primitives that underlie production systems like Stable Diffusion, DALL·E 3, and GenCast. Mastering them at this level means the architecture of any modern diffusion system becomes immediately legible.

Diffusion Models DDPM DDIM Score Matching PyTorch Generative AI U-Net