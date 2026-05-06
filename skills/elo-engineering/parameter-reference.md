# Elo Parameter Reference

> Level 3 reference. Loaded on demand. Zero context cost until Claude reads this file.

## NHL Default Parameters

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

## Cross-Sport Reference (for comparison)

These parameters are useful benchmarks when tuning NHL Elo, even though this plugin focuses on hockey.

| Sport | K | HFA (pts) | Carryover | Pythagorean Exp | Source |
|-------|---|-----------|-----------|-----------------|--------|
| NHL | 10 | 35 | 0.88 | 2.15 | PuckCast tuning |
| NBA | 20 | 100 | 0.75 | 13.91 | FiveThirtyEight NBA Elo |
| MLB | 4 | 24 | 0.50 | 1.83 | FiveThirtyEight MLB Elo |

**Note:** FiveThirtyEight did not publish an NHL Elo model. The NHL defaults above are from independent tuning.

## Grid Search Ranges

```
K_values = [5, 8, 10, 12, 15]
HFA_values = [20, 30, 35, 45, 55]
MOV_cap_values = [1.5, 1.8, 2.0]
decay_values = [0.002, 0.004, 0.006]  # Fading Elo only
form_windows = [5, 7, 10]              # Form Elo only
```

## Pythagorean Win Expectation

Alternative to Elo for estimating team strength from cumulative season stats.

```
expected_win_pct = GF^exp / (GF^exp + GA^exp)
```

Where `exp` is the Pythagorean exponent (NHL: 2.15). Use this as an independent feature alongside Elo -- they capture different information.

## Elo to Win Probability Conversion

```
win_prob_home = 1 / (1 + 10^((away_elo - home_elo - HFA) / 400))
```

This is the standard logistic conversion. For NHL (which has OT/SO), you need a three-outcome model -- either use a multinomial extension or treat OT loss as 0.5 and tune separately.
