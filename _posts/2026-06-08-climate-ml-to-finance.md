---
layout: post
title: "From Cyclones to Quant: How Climate ML Skills Transfer Directly to Finance"
date: 2026-06-07
author: "Deena Lad"
categories: [Career, Quantitative Finance, ML]
tags: [Quantitative Finance, Time Series, Signal Processing, AlphaML, Climate ML, Career]
excerpt: "The skills built processing terabytes of ERA5 reanalysis and INSAT-3D satellite data, spatiotemporal modelling, signal extraction, noise filtering, uncertainty quantification, are not niche. They map precisely onto the core problems of quantitative finance. Here's the proof, with code."
---

I spent two years building ML systems that predict the future state of physical systems from noisy, high-dimensional, time-ordered data. One system forecasts temperature fields 24 hours out from atmospheric reanalysis. Another tracks the evolution of a tropical cyclone across satellite frames.

When I started looking at quantitative finance problems, something immediately clicked: **these are the same problems, with different units**.

This post maps the methodological parallels explicitly and shows the code side-by-side.

---

## The Core Analogy

| Climate ML | Quantitative Finance |
|---|---|
| Atmospheric state at time $t$ | Market microstructure at time $t$ |
| ERA5 pressure / wind / humidity fields | Order book depth / bid-ask / volume |
| WRF simulation (physics baseline) | Factor model / Black-Scholes (theory baseline) |
| Neural weather forecast $\hat{x}_{t+\Delta t}$ | Return / volatility forecast $\hat{r}_{t+h}$ |
| Tropical cyclone track prediction | Asset price trajectory prediction |
| Ensemble uncertainty bounds | Prediction intervals / VaR |
| Data assimilation (Kalman update) | Bayesian updating of prior beliefs |
| Downscaling (27 km → 3 km) | Signal decomposition (market → alpha) |

The mapping is not superficial. Let me show you the code.

---

## 1. Loading Temporal Data: ERA5 vs Market Prices

Both domains involve loading large structured time-series from disk efficiently.

**Climate — ERA5 with xarray:**

```python
import xarray as xr
import numpy as np

# Load ERA5 2-metre temperature over India, 2020
ds = xr.open_dataset("era5_t2m_2020.nc")

# Select region, resample to 6-hourly, extract numpy
t2m = (
    ds["t2m"]
    .sel(latitude=slice(35, 8), longitude=slice(68, 97))
    .resample(time="6H")
    .mean()
    .values   # shape: (T, lat, lon)
)

# Normalise per grid point (mean/std over training period)
mu  = t2m[:8760].mean(axis=0, keepdims=True)
std = t2m[:8760].std(axis=0, keepdims=True) + 1e-6
t2m_norm = (t2m - mu) / std
```

**Finance — OHLCV price data with pandas:**

```python
import pandas as pd
import numpy as np

# Load daily OHLCV for NIFTY 50 constituents
df = pd.read_parquet("nifty50_ohlcv_2020.parquet")

# Compute log-returns (stationary, analogous to anomaly fields)
close = df.pivot(index="date", columns="ticker", values="close")
log_ret = np.log(close / close.shift(1)).dropna()

# Rolling z-score normalisation (analogous to ERA5 climatology removal)
mu  = log_ret.rolling(252).mean()
std = log_ret.rolling(252).std() + 1e-8
z_ret = (log_ret - mu) / std
```

**The pattern is identical**: load structured temporal data, remove a baseline (climatology / rolling mean), normalise by variance, produce a stationary anomaly signal.

---

## 2. Feature Engineering: Atmospheric Indices vs Technical Factors

In climate ML, we engineer physical indices from raw fields. In finance, we engineer alpha factors from raw prices. The code structure is the same.

```python
# ── CLIMATE: compute vorticity index from wind fields ──────────────────────
def relative_vorticity(u, v, dx, dy):
    """
    u, v: (T, lat, lon) wind components
    Returns: (T, lat, lon) relative vorticity ζ = ∂v/∂x - ∂u/∂y
    """
    dvdx = np.gradient(v, dx, axis=2)
    dudy = np.gradient(u, dy, axis=1)
    return dvdx - dudy

# ── FINANCE: compute momentum factor from returns ──────────────────────────
def momentum_factor(returns, lookback=252, skip=21):
    """
    Standard Jegadeesh-Titman momentum:
    12-month return excluding most recent month.
    returns: (T, N_assets)
    """
    cum = (1 + returns).cumprod()
    mom = cum.shift(skip) / cum.shift(lookback) - 1
    return mom.dropna()

# Both produce a (T, spatial_dim) matrix of "signals" that feed into the model.
```

---

## 3. Sequence Modelling: ConvLSTM vs LSTM for Returns

The ConvLSTM I trained on INSAT-3D satellite sequences has a near-identical counterpart in finance — an LSTM trained on cross-sectional return sequences.

```python
import torch
import torch.nn as nn

# ── CLIMATE: ConvLSTM predicts next satellite frame ────────────────────────
class CyclonePredictor(nn.Module):
    """Input: (B, T, 4, H, W) — T frames of 4-channel satellite imagery"""
    def __init__(self):
        super().__init__()
        self.encoder = ConvLSTMStack(in_channels=4,
                                     hidden_dims=[32, 64, 32])
        self.head    = nn.Conv2d(32, 4, kernel_size=1)

    def forward(self, x):
        return self.head(self.encoder(x))  # (B, 4, H, W)

# ── FINANCE: LSTM predicts next-period cross-sectional returns ─────────────
class ReturnPredictor(nn.Module):
    """Input: (B, T, N_factors) — T periods of N alpha factors per asset"""
    def __init__(self, n_factors=12, hidden=64, n_assets=50):
        super().__init__()
        self.lstm = nn.LSTM(n_factors, hidden, num_layers=2,
                            batch_first=True, dropout=0.2)
        self.head = nn.Linear(hidden, n_assets)

    def forward(self, x):
        out, _ = self.lstm(x)
        return self.head(out[:, -1, :])  # (B, N_assets)
```

The architectural decisions are analogous: sequence encoder → compressed state → prediction head. The only differences are the spatial structure (2D grid vs 1D asset universe) and the output type (field vs returns vector).

---

## 4. Uncertainty Quantification: Ensemble Forecasts vs Prediction Intervals

Good climate forecasting quantifies uncertainty via ensemble methods. Good quant modelling does the same via conformal prediction or Monte Carlo dropout.

```python
# ── CLIMATE: ensemble weather forecast uncertainty ─────────────────────────
def ensemble_uncertainty(model, x, n_members=50):
    """
    Monte Carlo dropout ensemble for weather model.
    Returns mean prediction and std (uncertainty) per grid point.
    """
    model.train()   # keep dropout active at inference
    preds = torch.stack([model(x) for _ in range(n_members)], dim=0)
    return preds.mean(0), preds.std(0)   # (B, C, H, W) each

# ── FINANCE: conformal prediction interval for returns ────────────────────
def conformal_intervals(cal_residuals, test_pred, alpha=0.1):
    """
    Split conformal prediction intervals at (1-alpha) coverage.
    cal_residuals: |y_cal - ŷ_cal|  (calibration set absolute errors)
    test_pred    : ŷ_test
    Returns: (lower, upper) bounds
    """
    q = np.quantile(cal_residuals, 1 - alpha)
    return test_pred - q, test_pred + q
```

In both cases, a point prediction is not enough. We need to know how confident the model is — whether that is a cyclone track uncertainty cone or a 90% prediction interval on portfolio returns.

---

## 5. Evaluation: Skill Scores vs Sharpe Ratio

Climate models are evaluated against climatological baselines. Quant models are evaluated against market baselines.

```python
# ── CLIMATE: Anomaly Correlation Coefficient (ACC) ─────────────────────────
def acc(forecast, truth, climatology):
    """
    ACC measures skill relative to climatology.
    ACC = 1 → perfect. ACC = 0 → no skill beyond climatology.
    """
    f_anom = forecast    - climatology
    t_anom = truth       - climatology
    num  = (f_anom * t_anom).sum()
    den  = np.sqrt((f_anom**2).sum() * (t_anom**2).sum()) + 1e-10
    return num / den

# ── FINANCE: Information Ratio (IR) ───────────────────────────────────────
def information_ratio(alpha_returns, benchmark_returns, freq=252):
    """
    IR measures alpha skill relative to benchmark.
    IR = 1.5+ → strong signal. IR = 0 → no edge.
    """
    active = alpha_returns - benchmark_returns
    return (active.mean() / (active.std() + 1e-10)) * np.sqrt(freq)

# ACC and IR are the same concept: signal power normalised by noise.
```

ACC in climate and Information Ratio in finance are **mathematically equivalent in structure**: they both measure how much better your prediction is compared to a naive baseline, normalised by variance.

---

## 6. The Full Pipeline Comparison

```
CLIMATE PIPELINE                    FINANCE PIPELINE
──────────────────────────────      ──────────────────────────────
ERA5 / INSAT-3D data ingestion  →   Market data / alt-data ingestion
Regridding & interpolation      →   Resampling & alignment
Climatology removal (anomalies) →   Rolling z-score (normalisation)
Physical feature engineering    →   Alpha factor construction
ConvLSTM / U-Net training       →   LSTM / Transformer training
Ensemble uncertainty            →   Conformal prediction intervals
ACC / RMSE evaluation           →   IC / IR / Sharpe evaluation
Operational forecast deployment →   Live signal generation
```

Every stage in my atmospheric ML pipeline has a direct counterpart in quant research.

---

## Why This Matters

The skills that make a good climate ML engineer — handling terabyte-scale structured datasets, building spatiotemporal models, quantifying uncertainty, evaluating against baselines — are exactly the skills quant funds need for **alternative data signal research**, **regime detection**, and **cross-asset return prediction**.

The physics changes. The methodology does not.

If you are a climate ML researcher exploring finance, you are not starting from scratch. You are translating.