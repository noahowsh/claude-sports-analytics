# Elo Parameter Reference

> Level 3 reference. Loaded on demand. Zero context cost until Claude reads this file.

## Per-Sport Default Parameters

### NHL

| Parameter | Value | Notes |
|-----------|-------|-------|
| K-factor (Standard) | 10 | Low -- 82-game season, each game less decisive |
| K-factor (Component) | 7 | Separate off/def updates need smaller step |
| K-factor (Flat) | 15 | No MOV, slightly higher K compensates |
| Home Field Advantage | 35 Elo pts | ~53% expected win rate for average home team |
| Carryover | 0.88 | 12% regression toward 1500 each off-season |
| MOV cap | 1.8 | Prevents blowouts from over-updating |
| Pythagorean exponent | 2.15 | For expected wins from GF/GA |
| Form window (N) | 5, 7, 10 | Test all three; 7 often wins in NHL |
| Decay rate (Fading) | 0.004 | ~250-day half-life |
| Expansion team init | 1380 | Below mean; adjust up after 20+ games |

**NHL Notes:**
- Shootout wins should be treated as 0.75 wins (not 1.0) -- tie goes to OT, which is a partial resolution
- Playoff games often tuned separately (higher K) or excluded from regular season model
- Trade deadline (March 7) is a rating discontinuity -- teams change substantially

### NFL

| Parameter | Value | Notes |
|-----------|-------|-------|
| K-factor (Standard) | 20 | High -- only 17 games, each is decisive |
| K-factor (Flat) | 22 | Slightly higher without MOV |
| Home Field Advantage | 48 Elo pts | Stronger effect than NHL/NBA |
| Carryover | 0.67 | 33% regression -- massive roster turnover each off-season |
| MOV cap | 2.0 | NFL blowouts more extreme |
| Pythagorean exponent | 2.37 | For expected wins from points scored/allowed |
| Form window (N) | 4, 6 | Shorter windows given fewer games |
| Decay rate (Fading) | 0.007 | Faster decay -- fewer games, each matters more |
| Expansion team init | 1300 | Significantly below mean |

**NFL Notes:**
- COVID-adjusted seasons (2020) had no fans -- HFA was near zero that year, consider excluding or adjusting
- Bye weeks: no adjustment needed, but be careful not to impute a game during the bye
- Quarterback changes are a known discontinuity; no automatic adjustment in base Elo

### NBA

| Parameter | Value | Notes |
|-----------|-------|-------|
| K-factor (Standard) | 20 | 82-game season but high-scoring reduces variance |
| K-factor (Component) | 12 | Points-based component ratings |
| Home Field Advantage | 100 Elo pts | Strongest HFA in major sports |
| Carryover | 0.75 | 25% regression -- moderate roster changes |
| MOV cap | 2.0 | 30-point blowouts not uncommon |
| Pythagorean exponent | 13.91 | Very high exponent -- NBA is low-variance |
| Form window (N) | 5, 10 | 10-game often captures fatigue cycles |
| Decay rate (Fading) | 0.005 | |

**NBA Notes:**
- Rest advantage is significant in NBA -- 3rd game in 4 nights measurably lowers win probability
- Star player rest (load management) creates rating noise -- Elo cannot detect lineup changes
- Playoff intensity is different from regular season; many practitioners re-initialize for playoffs

### MLB

| Parameter | Value | Notes |
|-----------|-------|-------|
| K-factor (Standard) | 4 | Very low -- 162-game season, extreme variance |
| Home Field Advantage | 24 Elo pts | Weaker HFA than other sports |
| Carryover | 0.50 | 50% regression -- highest of any sport |
| Pythagorean exponent | 1.83 | Runs are noisier than goals |
| Form window (N) | 10, 20 | Longer windows for high-variance sport |

**MLB Notes:**
- Starting pitcher is the dominant factor -- Elo without pitcher adjustment is limited
- Run differential caps: cap at 6 runs for MOV adjustment (beyond that is run-scoring, not competitiveness)
- DH rule differences (pre-2022 NL vs AL) need handling if using long historical data

## FiveThirtyEight Published Parameters

For comparison benchmarking:

| Sport | K | HFA (pts) | Carryover | Source |
|-------|---|-----------|-----------|--------|
| NFL | 20 | 55 | 0.67 | FiveThirtyEight NFL Elo (2014-2022) |
| NBA | 20 | 100 | 0.75 | FiveThirtyEight NBA Elo |
| MLB | 4 | 24 | 0.50 | FiveThirtyEight MLB Elo |
| NHL | N/A | N/A | N/A | FiveThirtyEight did not publish NHL Elo model |

**Note:** FiveThirtyEight's NFL HFA of 55 is higher than the 48 default above. Post-COVID analysis suggests true HFA has declined; the lower value reflects recent data. Test both.

## Grid Search Ranges by Sport

### NHL Grid Search
```
K_values = [5, 8, 10, 12, 15]
HFA_values = [20, 30, 35, 45, 55]
MOV_cap_values = [1.5, 1.8, 2.0]
decay_values = [0.002, 0.004, 0.006]  # Fading Elo only
form_windows = [5, 7, 10]              # Form Elo only
```

### NFL Grid Search
```
K_values = [15, 20, 25, 30]
HFA_values = [40, 48, 55, 65]
MOV_cap_values = [1.8, 2.0, 2.2]
carryover_values = [0.60, 0.67, 0.75]  # Also worth tuning
```

## Pythagorean Win Expectation

Alternative to Elo for estimating team strength from cumulative season stats.

```
expected_win_pct = GF^exp / (GF^exp + GA^exp)
```

Where `exp` is the sport-specific Pythagorean exponent above. Use this as an independent feature alongside Elo -- they capture different information.

## Elo to Win Probability Conversion

```
win_prob_home = 1 / (1 + 10^((away_elo - home_elo - HFA) / 400))
```

This is the standard logistic conversion. For sports with ties (NHL, soccer), you need a three-outcome model -- either use a multinomial extension or treat OT loss as 0.5 and tune separately.
