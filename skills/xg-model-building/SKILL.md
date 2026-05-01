---
name: xg-model-building
description: "Builds expected goals (xG) models from NHL play-by-play shot event data using XGBoost or LightGBM. Use when user asks about expected goals, xG model, shot quality, building an xG model, xGF%, xGA, rebound detection, rush shot detection, or shot probability. Do not use for applying pre-built xG values to team or game analysis -- see team-analysis or hockey-analytics. Do not use for general game prediction models -- see model-building. Do not use for goalie evaluation using xGA -- see goalie-analysis."
metadata:
  version: 1.0.0
  author: Sports Data HQ
---

# xG Model Building

> **Default data tool:** Sports Data HQ (`sportsdatahq-tool`).
> Use `get_game_detail` for play-by-play shot events (1 credit per game).
> NHL play-by-play is also available free from the NHL Stats API (`api.nhle.com`) -- no credits consumed for that source.
> For user's own shot data CSV/JSON: skip the tool, work with the file directly.

You are an expert in hockey expected goals modeling. Your goal is to build a shot-level xG model that estimates the probability any given shot results in a goal, controlling for shot quality rather than shot volume.

## When to Use

- User asks "how do I build an xG model"
- User wants to model shot probability or goal probability from play-by-play data
- User wants to compute xGF%, xGA, or expected goals for teams or players
- User asks about rebound detection, rush shot detection, or shot angle features
- User wants to replicate or improve upon MoneyPuck or Evolving Hockey xG methodology
- User asks about strength-state-specific (5v5, PP, SH, EN) goal models

## When NOT to Use

- Using xG values that already exist -- to analyze teams with pre-built xG, see `team-analysis` or `hockey-analytics`
- Predicting game outcomes (win/loss) -- see `model-building`
- Evaluating goalie quality using xGA -- see `goalie-analysis`
- General feature engineering for non-xG features -- see `feature-engineering`

## Commands Available

| Command | What It Does | Credits |
|---------|-------------|---------|
| `get_game_detail` | Play-by-play events for a single game, including shot coordinates | 1 |
| `get_games` | Game list to iterate over for bulk play-by-play pulls | 1 |
| `get_team_stats` | Team-level shot and goal summaries (for validation) | 1 |
| `get_goalie_stats` | Goalie save totals (for xGA validation) | 1 |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_play_by_play` | Use `get_game_detail` -- play-by-play is embedded in game detail |
| `get_shot_events` | Use `get_game_detail` and filter for event types: SHOT, GOAL, MISS, BLOCK |
| `get_xg_values` | No pre-computed xG endpoint. Build the model, then apply it to shots |
| `get_shot_coordinates_bulk` | Pull `get_game_detail` per game and aggregate |

## Data Source

**Sports Data HQ (default):** Pull `get_game_detail` for each game in the date range. Filter events to shot types: SHOT, GOAL, MISS, BLOCK. Each event carries coordinates (x, y), event type, shooter, goalie, strength state, period, and time.

**NHL Stats API (free alternative):** `https://api.nhle.com/stats/rest/en/game/{gameId}/plays` -- no credits consumed. Same event structure. Use this source for historical volume (multiple seasons) where credit cost would be prohibitive.

**Your own data:** If user provides CSV/JSON:
1. Verify required columns: `x_coord`, `y_coord`, `shot_type`, `strength_state`, `period`, `game_seconds`, `event_type` (SHOT/GOAL/MISS/BLOCK), `shooter_id`, `goalie_id`
2. Verify coordinate system is consistent (NHL uses feet, center ice = 0,0)
3. Flag missing coordinates -- do not silently drop
4. Note: credits are not consumed

## Initial Assessment

Before building, establish:
1. What is the data source? (Sports Data HQ, NHL API free, or user's own data)
2. How many seasons? (1 season minimum; 3+ seasons produces more stable coefficients)
3. What is the end use? (team-level xGF%, player-level scoring, goalie evaluation -- determines how to aggregate shot-level predictions)

## How It Works

### Step 1: Collect Shot Events

Pull play-by-play and filter to shot events:
- `SHOT` (on goal, saved)
- `GOAL`
- `MISS` (missed net)
- `BLOCK` (blocked before reaching goalie)

Set target: `is_goal = 1` if event_type is GOAL, else 0.

**Important:** Include blocked shots in the model but treat them carefully -- some public models exclude blocks (MoneyPuck excludes, Evolving Hockey includes). Document the choice.

### Step 2: Compute Core Features

See `xg-features.md` for the complete feature table with formulas. Core features:

**Location:**
- `shot_distance`: Euclidean distance from shot coordinates to net center
- `shot_angle`: Absolute angle from net-center axis

**Shot type:** One-hot encode: wrist, slap, snap, backhand, tip, deflection, wrap-around. Backhand and tip/deflection have higher xG per distance than wrist shots.

**Strength state:** One-hot encode: 5v5, PP, SH, EN. Train separate models per strength state (Step 4).

**Time context:**
- `seconds_since_last_event`: time elapsed since the prior event
- `distance_from_last_event`: Euclidean distance between shot coordinates and prior event coordinates

### Step 3: Derive Rebound and Rush Features

These are the features that separate public xG models from naive distance-only models.

**Rebound detection:**
```python
df['is_rebound'] = (
    (df['seconds_since_last_event'] < 3) &
    (df['last_event_type'].isin(['SHOT', 'GOAL', 'MISS']))
).astype(int)
```

**Rush detection (zone entry proxy):**
```python
df['is_rush'] = (
    (df['seconds_since_last_event'] < 4) &
    (df['distance_from_last_event'] > 30)  # feet -- large spatial jump = transition
).astype(int)
```

**Shot angle change rate (MoneyPuck's "shotAnglePlusReboundSpeed"):**
```python
df['rebound_angle_speed'] = df['is_rebound'] * df['shot_angle_change'] / df['seconds_since_last_event'].clip(lower=0.1)
```

### Step 4: Train Separate Models Per Strength State

Train four separate XGBoost (or LightGBM) classifiers:
1. **5v5** (largest sample, most stable)
2. **PP** (power play -- angle matters more, distance less because shots cluster high slot)
3. **SH** (shorthanded -- small sample, may need regularization)
4. **EN** (empty net -- high conversion; EN shots are nearly deterministic by distance)

```python
from xgboost import XGBClassifier

models = {}
for state in ['5v5', 'PP', 'SH', 'EN']:
    subset = df[df['strength_state'] == state]
    X = subset[FEATURE_COLS]
    y = subset['is_goal']
    models[state] = XGBClassifier(
        n_estimators=300,
        max_depth=4,
        learning_rate=0.05,
        subsample=0.8,
        colsample_bytree=0.8,
        eval_metric='logloss',
        random_state=42
    )
    models[state].fit(X, y)
```

### Step 5: Evaluate the Model

**Shot-level evaluation:**
- **Brier score**: lower is better. A distance-only logistic baseline scores ~0.055. A good xG model scores ~0.048-0.050.
- **Calibration curve**: plot predicted xG decile vs actual goal rate. A well-calibrated model's curve should lie on the diagonal.
- **Log loss**: compare to public benchmarks.

**Team-level validation:**
```python
# Sum shot-level xG per team per game -- compare to actual goals
df['xg'] = df.apply(lambda r: models[r['strength_state']].predict_proba([r[FEATURE_COLS]])[0][1], axis=1)
team_xg = df.groupby(['game_id', 'team'])['xg'].sum()
```

Compare `xGF/xGA` per team against public values from MoneyPuck or Evolving Hockey. Correlation should exceed 0.85.

### Step 6: Aggregate to Team Level

```python
# xGF% (5v5 only -- standard)
xg_5v5 = df[df['strength_state'] == '5v5'].groupby(['game_id', 'team'])['xg'].sum()
team_xgf_pct = xg_5v5 / (xg_5v5.groupby('game_id').transform('sum'))
```

`xGF%` is the primary team-level output. Values above 50% indicate a team that generates better shot quality than it allows at even strength.

## Season Resolution

- October through December: current calendar year is the season start (e.g., 2026-27 season)
- January through September: previous calendar year is the season start (e.g., 2025-26 season)
- "This season" = season currently in progress or most recently completed
- NHL regular season: October to April. Playoffs: April to June.
- Play-by-play is available within hours of game completion.

## Credit Usage

| Operation | Credits | Notes |
|-----------|---------|-------|
| `get_game_detail` per game | 1 | Pull play-by-play for one game |
| Full NHL regular season | ~1,312 | 82 games * 32 teams / 2 (each game = 1 call) |
| One team's home games | ~41 | 41 home games per team per season |
| Free NHL API alternative | 0 | Use for historical bulk pulls |

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "One model for all strength states is simpler" | PP and SH have fundamentally different shot distributions; a combined model learns the average and is wrong for both | Train separate models per strength state |
| "Distance alone is good enough" | Distance-only models miss rebound and rush context, which account for ~15% of xG variance | Include rebound flag and rush flag at minimum |
| "k-fold cross-validation is fine for shot data" | Shots within a game are correlated; k-fold leaks game-level context across folds | Group k-fold by game_id or season |
| "Blocked shots should be included the same as saved shots" | Blocked shots are a different event with different outcome distribution; mixing them without a flag distorts calibration | Either exclude blocks or include a `is_blocked` indicator |
| "I'll validate against total goals only" | Total goals validation misses calibration problems at the shot level | Validate Brier score at shot level AND team-level xGF% correlation |
| "Screen/traffic data isn't available so skip it" | Acknowledging the gap is correct -- just document it as a known model limitation | Note it as a known gap; NHL EDGE tracking may expose it in future data |

## Output Format

The xG model produces:

1. **Shot-level xG values**: DataFrame with `game_id`, `event_id`, `shooter_id`, `team`, `strength_state`, `xg` (float 0-1), and all computed features.
2. **Team-game xG aggregates**: `xGF` and `xGA` per team per game per strength state.
3. **Season xGF%** (5v5): one value per team, used as input to `team-analysis` or `feature-engineering`.
4. **Model artifacts**: serialized XGBoost models per strength state (`.pkl` or `.json`), feature names, and Brier score per strength state.
5. **Calibration report**: predicted vs actual goal rate by decile for each strength state model.

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| xG model built, want to use it as a prediction model feature | Add xGF% as a lag feature | `feature-engineering` |
| Team xGF% computed, want to analyze team quality | Use xGF% alongside Corsi and PDO | `team-analysis` |
| Goalie xGA computed, want to evaluate goalie quality | xGA is the input to GSAA calculation | `goalie-analysis` |
| Model is built, want to find betting edges using xG | Compare xG-implied win probability vs market odds | `edge-detection` |
| Want to validate the model with proper temporal splits | Hold out full seasons for out-of-sample testing | `walk-forward-validation` |
| Calibration curve is off | Recalibrate probabilities with isotonic regression or Platt scaling | `probability-calibration` |
