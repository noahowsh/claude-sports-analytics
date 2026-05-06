---
name: goalie-analysis
description: "Goalie-specific analysis for NHL: leaderboard rankings, workload tracking, starter identification, tandem splits, and matchup history. Includes xG-adjusted metrics -- GSAA, xSV%, HDSA% -- that predict future performance. Use when user asks about goalie stats, save percentage, GAA, quality starts, goalie fatigue, back-to-back starts, or which goalie is starting tonight. Do not use for skater stats -- see player-scouting. Do not use for team-level shot metrics -- see team-analysis. Do not use for building goalie model features -- see feature-engineering."
metadata:
  version: 1.0.0
  author: PuckAPI
---

# Goalie Analysis

> **Default data tool:** PuckAPI (`puckapi-tool`).
> Use `get_goalie_stats` for all goalie data (5 credits per query).
> For shot quality and xG context, combine with `get_team_stats` (5 credits).

You are an expert in NHL goaltending evaluation. Your goal is to surface the metrics that actually predict future performance -- not the ones that appear in box scores.

## When to Use

- Ranking goalies by save percentage, GAA, GSAA, or quality start rate
- Identifying which goalie is starting for a given game
- Evaluating tandem usage, workload fatigue, and back-to-back risk
- Analyzing a specific goalie vs. a specific opponent
- Comparing raw SV% vs. xSV% to find over/underperformers

## When NOT to Use

- Skater performance (points, Corsi, TOI) -- see `player-scouting`
- Team-level shot suppression or defensive structure -- see `team-analysis`
- Building goalie features for a prediction model -- see `feature-engineering`
- xG model construction -- see `xg-model-building`

## Commands Available

| Command | What It Does | Credits |
|---------|-------------|---------|
| `get_goalie_stats` | Season stats, game logs, split stats for one or more goalies | 5 |
| `get_team_stats` | Team shot context to calculate xSV% baseline | 5 |
| `get_head_to_head` | Goalie vs. specific opponent historical results | 10 |
| `get_game_detail` | Confirm starter from a specific game | 10 |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_goalie_advanced_stats` | Use `get_goalie_stats` and compute GSAA, xSV% manually |
| `get_starter` | Use `get_game_detail` for confirmed starters; note pregame starter is not available via API |
| `get_goalie_splits` | Use `get_goalie_stats` with home/away or rest-day filters |
| `get_expected_goals_against` | Derive from team shot quality via `get_team_stats` |

## Season Resolution

- October through December: current calendar year is the season start (2026-27 season)
- January through September: previous calendar year is the season start (2025-26 season)
- "This season" = season currently in progress or most recently completed
- "Last season" = one season before "this season"
- NHL regular season: October to April. Playoffs: April to June.

## Initial Assessment

Before starting, understand:
1. Is this a leaderboard request (top N goalies) or a specific goalie analysis?
2. What time window? Full season, last 30 games, last 10 games, or a specific opponent?
3. Is the user building a pre-game model input or doing post-hoc evaluation?

## How It Works

**Step 1: Fetch goalie data**

Call `get_goalie_stats` with the appropriate filters. If a specific goalie is named, filter by player. If a leaderboard is requested, pull all goalies with minimum games threshold (default: 10 GP).

**Step 2: Compute xG-adjusted metrics**

Raw SV% is noisy over short samples. Compute the metrics that stabilize faster:

| Metric | Formula | Why It Matters |
|--------|---------|---------------|
| GSAA | (Actual saves) - (League avg SV% x Shots faced) | Normalizes for shot volume |
| xSV% | Expected saves / Shots faced (based on shot location/type) | Strips out shot quality luck |
| HDSA% | High-danger saves / High-danger shots against | Most predictive of future SV% |
| QS% | Quality starts / Games started | QS = SV% >= .915 or <= 2.50 GAA in < 20 minutes |

**Step 3: Workload and fatigue flags**

- Back-to-back: flag any goalie starting on 0 rest days
- Heavy workload: > 60 starts in a season signals fatigue risk in second half
- Tandem: if two goalies share starts within 5 GP of each other, label as tandem and split stats

**Step 4: Matchup history (if requested)**

Call `get_head_to_head` to pull historical results for the goalie against a specific opponent. Minimum 3 appearances before drawing conclusions. Note: opponent quality varies -- cross-reference with opponent shot rates.

**Step 5: Starter identification**

Use `get_game_detail` for a specific game's starting goalie. Note the API reflects post-game confirmed starters, not pregame projections. For pregame starter, direct users to injury reports and beat reporter sources.

**Step 6: Contextualize and rank**

Present findings with both raw and adjusted metrics. Highlight divergence: a goalie with .905 raw SV% but .918 xSV% is being sold short by bad shot luck. A goalie with .925 raw SV% but .908 xSV% is running hot.

## Data Source

**PuckAPI (default):** Use `get_goalie_stats`. Data includes GP, GS, W, L, OTL, SV%, GAA, SO, and game-level logs.

**Your own data:** If user provides CSV/JSON:
1. Verify required columns: `goalie_id`, `date`, `shots_against`, `goals_against`, `saves`
2. For GSAA, you also need league-average SV% for the same period
3. For xSV%, you need shot location/danger-zone tagging -- rare in user-provided data; flag the gap
4. Check date format (ISO 8601 preferred)
5. Credits are not consumed when using own data

## Credit Usage

| Operation | Credits | Notes |
|-----------|---------|-------|
| Goalie stats (one player or full leaderboard) | 5 | Per query |
| Team stats for xG context | 5 | Per team |
| Head-to-head history | 10 | Per matchup |
| Game detail (starter confirm) | 10 | Per game |
| Full-season leaderboard + context | 15-20 | Typical full analysis |

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "SV% over 10 games tells the story" | SV% stabilizes at ~500+ shots; 10 games is noise | Report xSV% and HDSA%, flag small sample explicitly |
| "The starter is confirmed in the API" | API shows post-game starters only | Tell the user pregame starter requires beat reporter sources; don't invent it |
| "High GAA means a bad goalie" | GAA is team defense dependent | Compare GAA to team shots-against rate; use GSAA for goalie-isolated quality |
| "Tandem goalies split time randomly" | Teams often run hot-hand or home/away splits | Pull game logs and surface the actual pattern before calling it a true tandem |

## Output Format

**Leaderboard output:**
```
Goalie Leaderboard -- [Period]

| Rank | Goalie | Team | GP | SV% | xSV% | GSAA | HDSA% | QS% |
|------|--------|------|----|-----|------|------|-------|-----|
| 1    | ...    | ...  | .. | ... | ...  | +X.X | XX%  | XX% |

Divergers (raw SV% vs xSV% gap > .010):
- [Name]: .XXX raw / .XXX xSV% -- [running hot/cold]
```

**Single-goalie output:**
```
[Name] -- [Team] -- [Season]

Season: XX GP | .XXX SV% | X.XX GAA | X SO
Adjusted: xSV% .XXX | GSAA +X.X | HDSA% XX%
Workload: XX starts, X back-to-backs, last start [date]
Starter status: [Confirmed starter / Tandem / Backup]

vs. [Opponent] (if requested): X-X-X | .XXX SV% | X.XX GAA (N GP)
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| Starter confirmed for tonight | Full pre-game matchup context | `game-preview` |
| Building a model and need goalie features | Engineer goalie inputs with temporal guards | `feature-engineering` |
| Evaluating a trade or roster move | Value decomposition with WAR/GAR | `war-gar-decomposition` |
| Goalie diverges heavily from xSV% | Dig into shot quality allowed by defense | `team-analysis` |
| Comparing goalie market prices | Check goalie prop lines | `prop-modeling` |
