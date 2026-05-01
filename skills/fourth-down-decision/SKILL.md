---
name: fourth-down-decision
description: "Expected value framework for evaluating 4th down go/punt/FG decisions in NFL games. Use when user asks about 4th down analytics, whether a coach should go for it, punt vs field goal tradeoffs, expected value of NFL decisions, analytics-based play-calling, or 'was that the right call' analysis after a game. Do not use for general NFL situational stats -- see epa-situational-query. Do not use for NHL analysis. Do not use for full game prediction -- see model-building."
metadata:
  version: 1.0.0
  author: Sports Data HQ
---

# Fourth Down Decision

> **Default data tool:** nflfastR via `nfl_data_py` (free, no credits) for conversion rates and EP values.
> Historical kicking data also from nflfastR. No Sports Data HQ MCP credits consumed.
> For contextualizing decisions against betting lines, use `odds-explorer`.

You are an expert in NFL 4th down decision analysis using expected value frameworks. Your goal is to compute the EV of each option (go/punt/FG), surface the analytically correct call, and explain the context adjustments that change the answer. This is the single most impactful in-game analytics application in football -- coaches leave wins on the field every week by not going for it.

## When to Use

- User wants to know if a team made the right 4th down call
- User asks whether a coach should go for it on 4th down in a specific situation
- User wants to build a 4th-down EV model
- User asks about punt vs FG tradeoffs
- User asks for post-game "was that the right call?" analysis
- User wants to analyze a team's historical 4th down decision-making

## When NOT to Use

- General NFL situational stats (EPA by down, team tendencies) -- see `epa-situational-query`
- Full game prediction models -- see `model-building`
- NHL analysis of any kind
- Real-time 4th down decisions during a live game without pre-loaded data

## Commands Available

No MCP commands -- this skill composes Python analysis using nflfastR data.

| Function | What It Does | Cost |
|----------|-------------|------|
| `nfl_data_py.import_pbp_data([years])` | Historical play-by-play for conversion rates | Free |
| `nfl_data_py.import_schedules([years])` | Game context (score, venue) | Free |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_fourth_down_ev` | Compute from historical nflfastR conversion rates |
| `get_punt_distance_model` | Use average net punt yards from nflfastR `punt` plays |
| `get_fg_probability` | Compute from historical `field_goal_attempt` plays by distance |

## Initial Assessment

Before computing EV, establish:
1. Exact situation: yard line (yardline_100), distance (ydstogo), score differential, time remaining
2. Which team? (team-specific tendencies affect go-for-it conversion rates)
3. Is this historical analysis ("was that right?") or live decision support?

## The EV Framework

Three mutually exclusive options. Compare all three, recommend the highest EV.

### Option 1: GO FOR IT

```
EV(go) = P(convert) × EP(first down, current spot)
        + P(fail) × EP(opponent at current spot)
```

- `P(convert)` = historical conversion rate by distance (see table below)
- `EP(first down, current spot)` = expected points for offense after converting (from EP model)
- `EP(opponent at current spot)` = expected points for opponent if turnover on downs (negative for offense = opponent at that yard line)

### Option 2: PUNT

```
EV(punt) = EP(opponent at expected punt landing spot)
```

- Expected net punt distance varies by field position
- Average net punt: 41-44 yards, but field position-constrained (can't pin opponent inside 5 easily from midfield)
- `EP(opponent at landing spot)` is negative for the punting team (opponent gets possession)

```python
# Rough net punt distance by field position
def net_punt_distance(yardline_100):
    # yardline_100 = yards to opponent end zone
    gross = min(44, yardline_100 - 5)  # can't punt through end zone
    touchback_adjustment = 0.15 if yardline_100 > 60 else 0
    net = gross - 8 + (20 * touchback_adjustment)  # touchback = ball at 25
    return net
```

### Option 3: FIELD GOAL

```
EV(fg) = P(make) × 3 + P(miss) × EP(opponent at miss spot)
```

- FG attempt distance = yardline_100 + 17 (snap + hold distance)
- `P(make)` from historical kicking data (see table below)
- On miss: opponent takes over approximately 7 yards behind line of scrimmage (LOS - 7)
- The EP of "opponent gets ball back" is negative for the kicking team

## Conversion Rate Reference

Historical NFL 4th down conversion rates from nflfastR (2010-2024 regular season):

| Distance (ydstogo) | Conversion Rate |
|-------------------|----------------|
| 1 | 68% |
| 2 | 60% |
| 3 | 56% |
| 4 | 51% |
| 5 | 45% |
| 6 | 40% |
| 7 | 37% |
| 8 | 33% |
| 9 | 31% |
| 10 | 29% |
| 11-15 | 22% |
| 16+ | 14% |

Load from nflfastR for accuracy:
```python
fourth_downs = pbp[(pbp['down'] == 4) & (pbp['play_type'].isin(['pass', 'run']))]
conv_by_dist = (fourth_downs
    .groupby('ydstogo')['first_down']
    .agg(['mean', 'count'])
    .query('count >= 50'))
```

## FG Make Probability Reference

Historical NFL FG make rates from nflfastR (2010-2024):

| Distance (yards) | Make Rate |
|-----------------|-----------|
| 18-29 | 96% |
| 30-39 | 88% |
| 40-44 | 80% |
| 45-49 | 72% |
| 50-54 | 61% |
| 55-59 | 50% |
| 60+ | 32% |

## Expected Points (EP) by Field Position

nflfastR's pre-computed EP model. Use directly from `ep` column or approximate:

| Yard Line (yardline_100) | Approx EP for Offense |
|--------------------------|----------------------|
| 99 (own 1) | -1.4 |
| 80 (own 20) | 0.1 |
| 60 (own 40) | 1.2 |
| 50 | 2.0 |
| 40 (opp 40) | 2.8 |
| 20 (opp 20) | 4.0 |
| 10 (opp 10) | 5.0 |
| 5 (opp 5) | 5.8 |

Load actual values:
```python
ep_by_yardline = (pbp
    .groupby('yardline_100')['ep']
    .mean()
    .reset_index())
```

## Context Adjustments

The base EV framework is the floor. These adjustments change the answer:

### Score Differential
- **Trailing by 7+**: Go for it more. A team that needs multiple possessions must take risk to generate them. A punt extending a losing drive is often EV-neutral but strategically wrong.
- **Leading late**: Consider the risk of leaving the opponent with good field position. Punting to pin them deep has asymmetric upside late in games.
- Quantify: add the possession value of the outcome (trailing team's win probability benefit from gaining extra possession).

### Time Remaining
- **4th quarter, trailing**: WPA (win probability added) replaces EP as the correct metric. A 4th-and-1 from your own 30 with 2 minutes left trailing by 3 is completely different from the same situation in the 2nd quarter.
- Switch from EP framework to WPA framework when `game_seconds_remaining < 300` and `score_differential` matters.

```python
# Use wpa column instead of ep for late-game decisions
late_game = pbp[
    (pbp['down'] == 4) &
    (pbp['game_seconds_remaining'] < 300)
]
```

### Field Position Extremes
- **Own territory (yardline_100 > 60)**: Going for it on 4th-and-1 from your own 32 is analytically correct but gives opponent short field on failure. EP model captures this, but verify the EP(fail) is realistic.
- **Opponent territory (yardline_100 < 45)**: Going for it is almost always correct when FG is also an option. The difference in EP between FG and conversion is largest here.

### Team-Specific Conversion Rate
- Some offenses convert 4th downs at rates 5-10 points above league average. Account for this.
```python
team_conv = (fourth_downs
    .groupby(['posteam', 'ydstogo'])['first_down']
    .mean())
```

## How to Build the Full Decision

```python
def fourth_down_ev(yardline_100, ydstogo, score_diff, seconds_remaining,
                   team_conv_rate=None):

    # GO
    p_convert = team_conv_rate or league_conv_rate[ydstogo]
    ep_first_down = ep_model[yardline_100]  # opponent at this spot after convert
    ep_fail = -ep_model[yardline_100]  # opponent takes over at same spot
    ev_go = p_convert * ep_first_down + (1 - p_convert) * ep_fail

    # PUNT
    net_punt = net_punt_distance(yardline_100)
    opponent_yardline = min(99, 100 - (yardline_100 - net_punt))
    ev_punt = -ep_model[opponent_yardline]  # negative because opponent scores

    # FG
    fg_distance = yardline_100 + 17
    p_make = fg_make_rate[fg_distance]
    miss_yardline = min(99, yardline_100 + 7)  # opponent takes over ~7 behind LOS
    ev_fg = p_make * 3 + (1 - p_make) * (-ep_model[miss_yardline])

    return {'go': ev_go, 'punt': ev_punt, 'fg': ev_fg,
            'recommendation': max(['go', 'punt', 'fg'],
                                  key=lambda x: {'go': ev_go, 'punt': ev_punt, 'fg': ev_fg}[x])}
```

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "Punting is always safe" | Punting surrenders possession; EP of opponent at punt landing is -0.5 to -2.0 points, not zero | Quantify the EP of opponent at expected landing spot; it's often worse than going for it |
| "4th-and-3 from own 35 is too risky to go for it" | EP(fail) is opponent at their own 38; EP(convert) is your offense at 1st-and-10 at their 35 -- the EV of going for it is often positive even at 45% conversion | Run the numbers. Coaches' gut says "too risky"; math says go. |
| "FG attempt is always better than going for it" | A 50-yard FG has 61% make rate; the 39% miss gives opponent the ball at their 43 -- EP-wise this is often worse than a low-conversion go attempt | Compare FG EV to go EV explicitly; at distances > 48 yards, go is often higher EV |
| "Use the same conversion rate for all teams" | Kansas City's offense converts 4th-and-2 at ~65%; a bottom-5 offense converts at ~45% | Pull team-specific rates from nflfastR when analyzing a specific team |
| "Late-game decisions are the same as mid-game" | With 2 minutes left, win probability drives the decision, not expected points | Switch to WPA framework when time < 5 minutes and score differential matters |

## Output Format

For a single decision:

```
Situation: 4th-and-4, own 38-yard line, down 3, Q3 8:22 remaining

Option       | EV    | Key Assumption
-------------|-------|------------------------------------------
GO           | +1.82 | 51% conversion rate, EP(1st) = 2.8
PUNT         | +0.31 | 43-yard net, opponent at own 19
FIELD GOAL   | -0.14 | 55-yard attempt, 50% make rate

RECOMMENDATION: GO
Margin: +1.51 EP vs punt, +1.96 EP vs FG

Context: Down by a score. Punt produces ~0.3 EP gain. Going for it with league-average
conversion rate produces ~1.8 EP. The analytics strongly support going for it.
```

## Season/Date Logic

- nflfastR seasons labeled by season start year: 2024 = 2024-25 NFL season
- Regular season: September through January (`season_type == 'REG'`)
- Use 2010-2024 for conversion rates (pre-2010 is reliable but offense was lower-scoring)
- "This season" follows NFL schedule: 2025 if September 2025 or later, 2024 otherwise

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| Built 4th-down model, want to backtest | Test against historical decisions | `backtesting` |
| Want to analyze more NFL situations | EPA by down, distance, personnel | `epa-situational-query` |
| Want full game prediction model | Incorporate as a decision quality feature | `model-building` |
| Want to share as content | Chart decisions across the full game | `visualization` |
