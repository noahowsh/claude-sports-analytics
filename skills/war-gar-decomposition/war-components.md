# WAR/GAR Component Reference

> Level 3 reference. Read this when SKILL.md directs you here for mathematical formulations, regularization parameter selection, component definitions, and market rate derivation.

---

## RAPM Mathematical Formulation

### Problem Statement

Given `n` shifts and `p` players, we observe for each shift:
- Which players were on ice (home = +1, away = -1 indicator)
- The outcome of that shift (goals differential, xG differential, Corsi differential per 60 minutes)
- The duration of the shift in seconds

We want to estimate each player's individual contribution to the outcome, controlling for all other players present.

### Matrix Form

Let:
- `X`: n x p design matrix (sparse). `X[i,j] = 1` if player `j` was home for shift `i`, `-1` if away, `0` if not on ice.
- `y`: n-vector of shift outcomes (goals per 60 for that shift, weighted by duration)
- `w`: n-vector of shift durations (weights)
- `β`: p-vector of player coefficients (the RAPM values we want to estimate)

### Ridge Regression Objective

```
minimize: ||sqrt(W)(Xβ - y)||² + λ||β||²
```

Where `W = diag(w)` (diagonal weight matrix of shift durations) and `λ` is the regularization parameter.

**Closed-form solution:**
```
β = (X'WX + λI)^(-1) X'Wy
```

In practice, solve with `sklearn.linear_model.Ridge` with `sample_weight=shift_durations`. Never invert the matrix directly -- use the solver.

### Why Ridge (Not Lasso, Not OLS)

- **OLS fails:** The design matrix `X'WX` is nearly singular. Players share shifts; columns are correlated. OLS produces extreme, unstable coefficients.
- **Lasso fails:** Lasso shrinks coefficients to exactly zero, which is wrong for players who have real (if small) contributions. Ridge shrinks toward zero but doesn't zero out.
- **Ridge is correct:** Shrinks all coefficients proportionally toward zero. Players with few shifts get heavily shrunk toward the league mean. Players with thousands of shifts are shrunk minimally.

---

## Regularization Parameter Selection

The ridge penalty `λ` controls how aggressively low-sample players are shrunk.

### Published Values (Evolving Hockey / public research)

| Strength State | Target | Recommended λ | Notes |
|---------------|--------|---------------|-------|
| 5v5 (EV) | xGF per 60 | 2,000 - 3,000 | Most stable; large sample |
| PP (5v4, 5v3) | xGF per 60 | 4,000 - 6,000 | Fewer shifts; higher regularization |
| SH (4v5, 3v5) | xGA per 60 | 4,000 - 6,000 | Same as PP |
| All states | PD per 60 | 1,500 - 2,500 | Penalty rates are stable |
| All states | PT per 60 | 1,500 - 2,500 | Same |

### Selecting λ via Cross-Validation

If not using published values, select `λ` using group k-fold where groups = `game_id`:

```python
from sklearn.linear_model import RidgeCV
from sklearn.model_selection import GroupKFold

gkf = GroupKFold(n_splits=5)
alphas = [500, 1000, 2000, 3000, 5000, 8000]

model = RidgeCV(alphas=alphas, cv=gkf)
model.fit(X, y, sample_weight=w, groups=game_ids)
best_alpha = model.alpha_
```

**Never use random k-fold.** Shifts from the same game must be in the same fold to prevent leakage.

---

## GAR Component Definitions

### EV Offense (EVO)

- **What it measures:** How many more expected goals per 60 does the team generate at 5v5 when this player is on ice, relative to a replacement-level player?
- **RAPM input:** 5v5 shifts, xGF per 60 as target
- **Sign convention:** Positive = good (more offense generated)
- **Typical range for full-season NHL starters:** -4 to +8 EVO GAR
- **Evolving Hockey label:** EVO

### EV Defense (EVD)

- **What it measures:** How many fewer expected goals per 60 does the team allow at 5v5 when this player is on ice?
- **RAPM input:** 5v5 shifts, xGA per 60 as target, **inverted** (multiply coefficient by -1 so positive = fewer goals against)
- **Sign convention:** Positive = good (less offense allowed)
- **Typical range:** -4 to +6 EVD GAR
- **Evolving Hockey label:** EVD

### PP Contribution (PPO)

- **What it measures:** Expected goals per 60 added during power play shifts
- **RAPM input:** PP shifts (5v4 + 5v3 + 4v3), xGF per 60 as target
- **Sign convention:** Positive = good
- **Typical range for PP-heavy forwards:** 0 to +5 PPO GAR. Non-PP players: ~0.
- **Evolving Hockey label:** PPO

### PK Defense (PKD)

- **What it measures:** Expected goals per 60 prevented during penalty kill shifts
- **RAPM input:** SH shifts (4v5 + 3v5 + 3v4), xGA per 60 as target, inverted
- **Sign convention:** Positive = good (fewer goals allowed on PK)
- **Typical range for PK specialists:** 0 to +3 PKD GAR
- **Evolving Hockey label:** PKD

### Penalties Drawn (PD)

- **What it measures:** Penalties drawn per 60 above replacement (extra power plays generated)
- **RAPM input:** All states, penalty-drawn events per 60 as target
- **Conversion:** Penalties drawn → PP opportunities → xG value of those opportunities (~0.2 xG per PP)
- **Sign convention:** Positive = good
- **Evolving Hockey label:** PD

### Penalties Taken (PT)

- **What it measures:** Penalties taken per 60 above replacement (power plays given away)
- **RAPM input:** All states, penalty-taken events per 60 as target
- **Conversion:** Same as PD but sign is negative (giving away PP = hurts team)
- **Sign convention:** Negative = more penalties taken = bad
- **Evolving Hockey label:** PT

---

## WAR Conversion: Goals to Wins

### Pythagorean Expectation

The relationship between goals scored, goals allowed, and wins follows:

```
Win% ≈ GF^2.15 / (GF^2.15 + GA^2.15)
```

The exponent 2.15 is empirically derived for NHL (Schuckers and Curro 2013; recalibrated ~2020).

### Marginal Wins per Goal

At the league average (GF = GA = 3.0 goals per game, 82 games per season):

```python
import numpy as np

gf = 3.0  # goals for per game (league average)
ga = 3.0  # goals against per game
k = 2.15   # Pythagorean exponent
games = 82

# Win% partial derivative with respect to GF (at GF=GA):
# dW%/dGF = 2.15 * GF^(1.15) * GA^2.15 / (GF^2.15 + GA^2.15)^2
dW_dGF = k * (gf**(k-1)) * (ga**k) / ((gf**k + ga**k)**2)

# Goals to wins: (dW%/dGF) * games_per_season
goals_per_win = 1 / (dW_dGF * games)
# Result: approximately 5.5 - 6.5 goals per win depending on calibration year
```

**Published values:**
- Evolving Hockey: ~6.0 goals per WAR
- Dom Luszczyszyn (The Athletic): ~5.5 goals per WAR
- This skill uses 6.0 as the baseline; note which assumption is in use.

### Practical WAR Calculation

```python
GOALS_PER_WAR = 6.0

# player_gar = total GAR in goals above replacement
player_war = player_gar / GOALS_PER_WAR
```

---

## Market Rate: Dollars per WAR

The market rate is derived from the relationship between AAV and WAR for players who signed as unrestricted free agents (UFAs). Restricted free agents (RFAs) sign below market -- exclude from the derivation.

### Methodology

1. Collect all UFA signings in the past 2-3 seasons
2. Pull each player's trailing 3-season WAR (or best available)
3. Regress AAV on WAR: `AAV = rate * WAR + baseline`
4. The slope `rate` is the market dollars per WAR

**Published estimates:**
- 2023-24 market: ~$1.2M per WAR
- 2024-25 market: ~$1.4M per WAR (cap growth)
- 2025-26 market: ~$1.4-1.5M per WAR (projected)

Use the most recent available estimate. The rate grows with the salary cap; update annually.

### Surplus Value Formula

```python
years_remaining = 3  # from contract data
player_age = 28

# Aging curve adjustment (applies for players 30+)
AGE_CURVE = {28: 1.0, 29: 0.98, 30: 0.95, 31: 0.90, 32: 0.83, 33: 0.75, 34: 0.65, 35: 0.55}
age_factor = AGE_CURVE.get(player_age, 1.0 if player_age < 28 else 0.50)

# Project WAR over contract
projected_war = player_war * age_factor * years_remaining

# Market value vs actual cost
market_value = projected_war * MARKET_RATE_PER_WAR  # ~$1.4M per WAR
actual_cost = player_aav * years_remaining
surplus_value = market_value - actual_cost

# Positive surplus: player is underpaid (good contract)
# Negative surplus: player is overpaid (bad contract)
```

---

## Comparison to Evolving Hockey Public Values

Evolving Hockey publishes GAR components for all NHL skaters each season. Use these as ground truth for validation.

**How to validate your RAPM output:**
1. Compute correlation between your EVO values and Evolving Hockey's EVO for all qualifying players. Target r > 0.85.
2. Compare top-10 and bottom-10 players by total WAR. Major outliers signal a data or implementation error.
3. Check that your team-level WAR totals correlate with actual team standings points (r > 0.70).

**Expected discrepancies:**
- Your single-season RAPM will be noisier than EH's multi-season estimates
- EH may use different ridge penalty values (not fully public)
- Different xG models produce different RAPM values; document which xG source was used

---

## Minimum Sample Thresholds

| Strength State | Minimum Minutes | Minimum Shifts |
|---------------|----------------|----------------|
| 5v5 | 200 minutes | ~300 shifts |
| PP | 50 minutes | ~80 shifts |
| PK | 50 minutes | ~80 shifts |

Players below these thresholds have RAPM estimates dominated by the ridge penalty (they're essentially assigned replacement level). Flag these players explicitly -- do not report their component values as meaningful.

For a full season:
- Top-6 forwards: ~800-1,000 minutes of 5v5 TOI. Well above threshold.
- Bottom-6 forwards: ~300-500 minutes. Marginal -- single-season estimates are noisy.
- AHL callup who played 5 games: likely below threshold for most components.
