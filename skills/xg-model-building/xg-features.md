# xG Feature Reference

> Level 3 reference. Read this when SKILL.md directs you here for feature formulas and data source details.

## Coordinate System

NHL coordinates use feet. Center ice = (0, 0). Net centers are at approximately x = ±89, y = 0. Shots toward the right net (positive x) are recorded as positive x values.

Always normalize to "attacking direction" before computing distance and angle -- flip coordinates for shots at the left net.

```python
# Normalize to attacking direction
df['x_norm'] = df['x_coord'].abs()  # Always positive (toward the net)
df['y_norm'] = df['y_coord'] * df['shot_toward_right_net'].map({True: 1, False: -1})
```

---

## Core Feature Table

| Feature | Formula | Data Source | Used By |
|---------|---------|-------------|---------|
| `shot_distance` | `sqrt((89 - x_norm)^2 + y_norm^2)` | coordinates | MoneyPuck, Evolving Hockey, all public models |
| `shot_angle` | `abs(atan2(y_norm, 89 - x_norm)) * (180/pi)` | coordinates | MoneyPuck, Evolving Hockey, all public models |
| `shot_type_wrist` | 1 if shot_type == "Wrist" else 0 | event type | All public models |
| `shot_type_slap` | 1 if shot_type == "Slap" else 0 | event type | All public models |
| `shot_type_snap` | 1 if shot_type == "Snap" else 0 | event type | All public models |
| `shot_type_backhand` | 1 if shot_type == "Backhand" else 0 | event type | All public models |
| `shot_type_tip` | 1 if shot_type == "Tip-In" else 0 | event type | All public models |
| `shot_type_deflection` | 1 if shot_type == "Deflected" else 0 | event type | MoneyPuck, Evolving Hockey |
| `shot_type_wrap` | 1 if shot_type == "Wrap-around" else 0 | event type | MoneyPuck |
| `strength_5v5` | 1 if strength_state == "5v5" else 0 | event state | All public models (or separate model) |
| `strength_pp` | 1 if strength_state in {"5v4","5v3","4v3"} else 0 | event state | All public models |
| `strength_sh` | 1 if strength_state in {"4v5","3v5","3v4"} else 0 | event state | All public models |
| `strength_en` | 1 if goalie pulled else 0 | event state | All public models |

---

## Time and Transition Features

| Feature | Formula | Data Source | Notes |
|---------|---------|-------------|-------|
| `seconds_since_last_event` | `game_seconds - prev_event_game_seconds` | event sequence | Clip at 30 seconds -- longer gaps break rebound/rush logic |
| `distance_from_last_event` | `sqrt((x - prev_x)^2 + (y - prev_y)^2)` | coordinates | Measures puck travel, not player travel |
| `is_rebound` | `seconds_since_last_event < 3 AND prev_event in {SHOT, GOAL, MISS}` | event sequence | MoneyPuck, Evolving Hockey |
| `is_rush` | `seconds_since_last_event < 4 AND distance_from_last_event > 30` | event sequence | MoneyPuck; proxy for zone entry transition |
| `angle_change` | `abs(shot_angle - prev_shot_angle)` | coordinates | Only defined when is_rebound == 1 |
| `rebound_angle_speed` | `is_rebound * angle_change / max(seconds_since_last_event, 0.1)` | derived | MoneyPuck "shotAnglePlusReboundSpeed" |
| `period` | raw period value (1, 2, 3, OT) | event metadata | OT shots in 3v3 have higher conversion |
| `game_seconds` | seconds elapsed since puck drop | event metadata | Capture fatigue/desperation effects |

---

## Features NOT in Public Models (Genuine Gaps)

| Feature | Why It Matters | Availability |
|---------|---------------|-------------|
| Screen / traffic in front of goalie | Screened shots convert at 2-3x the rate of unscreened shots at the same distance and angle | Not available in standard NHL event data. NHL EDGE tracking (sensor pucks, camera-based player tracking) may expose this in future datasets. |
| Goalie positioning at shot release | A goalie out of position increases shot xG independent of distance/angle | Not available in standard event data. EDGE tracking only. |
| Shot velocity / release speed | Faster shots give goalies less reaction time | Not available in standard event data. |
| Shot release location vs stick position | Deceptive release points | Not available in standard event data. |

These are genuine model improvement opportunities, not oversights in public methodology.

---

## Shot Type Goal Rates (Empirical Reference)

Approximate conversion rates from public NHL data (all strengths, all seasons):

| Shot Type | Goal Rate | Notes |
|-----------|-----------|-------|
| Tip-In | ~15-18% | Highest conversion; goalie rarely set |
| Deflection | ~14-17% | Similar to tip; puck trajectory changes |
| Wrap-around | ~5-7% | Angle extreme; typically low quality |
| Backhand | ~7-9% | Goalie can't read spin; underrated |
| Snap | ~6-8% | Quick release from wrist shot position |
| Wrist | ~5-7% | Most common; baseline reference |
| Slap | ~5-6% | High power but telegraphed; goalie sets |

These rates vary by distance and angle. The model learns these implicitly through the interaction of shot_type and coordinate features.

---

## Strength State Shot Distributions

| Strength State | Avg Distance | Avg Angle | Rebound Rate | Notes |
|---------------|-------------|----------|-------------|-------|
| 5v5 | ~32 ft | ~22 deg | ~8% | Baseline |
| PP | ~27 ft | ~18 deg | ~12% | Closer, more slot shots |
| SH | ~38 ft | ~28 deg | ~4% | Perimeter, desperation |
| EN | ~55 ft | ~10 deg | ~2% | Long-range, near-certain |

---

## Feature Engineering Notes

**Distance squared:** Some implementations use `shot_distance^2` instead of raw distance to better capture the nonlinear drop-off in goal probability beyond 40 feet. XGBoost learns this automatically via splits, so this matters more for logistic regression implementations.

**Angle squared:** Same logic as distance. Nonlinear at extreme angles (>50 degrees). XGBoost handles this natively.

**Interaction terms:** For non-tree models (logistic regression), manually construct `distance * angle` and `is_rebound * distance`. XGBoost discovers these interactions through splits.

**Missing coordinates:** A non-trivial fraction of NHL events (~3-5%) have missing or implausible coordinates. Options:
1. Drop the event (conservative; reduces sample slightly)
2. Impute with strength-state average (acceptable for minor missingness)
3. Flag as `coord_missing = 1` and impute (preferred; preserves sample, captures the pattern)

---

## Public Model Comparison

| Model | Features Included | Blocked Shots | Strength State Handling | Public Brier Score |
|-------|------------------|---------------|------------------------|-------------------|
| MoneyPuck | Distance, angle, shot type, rebound, rush, rebound angle speed | Excluded | Separate models | ~0.048 |
| Evolving Hockey | Distance, angle, shot type, rebound, rush, last event | Included (flagged) | Single model with state features | ~0.050 |
| Naive distance-only logistic | Distance | N/A | None | ~0.055 |

Use MoneyPuck and Evolving Hockey team-level xGF% for validation. A model within 2 percentage points of their team rankings is calibrated correctly.
