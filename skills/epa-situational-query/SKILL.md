---
name: epa-situational-query
description: "Filters and aggregates NFL play-by-play data by game state, personnel, and situation using nflfastR's EPA and CPOE metrics. Use when user asks about NFL EPA, expected points added, CPOE, completion percentage over expected, 3rd-down efficiency, red zone passing, NFL situational stats, nflfastR, nfl_data_py, or wants to find which players or teams perform best in specific NFL situations. Do not use for NHL data -- see game-lookup or team-analysis. Do not use for NFL odds -- see odds-explorer. Do not use for building full NFL prediction models -- use feature-engineering and model-building with this skill's output as input."
metadata:
  version: 1.0.0
  author: Sports Data HQ
---

# EPA Situational Query

> **Default data tool:** nflfastR via `nfl_data_py` Python package (free, no credits).
> Install: `pip install nfl_data_py`. Load: `nfl_data_py.import_pbp_data([2023, 2024, 2025])`.
> This skill does NOT use the Sports Data HQ MCP. No credits are consumed.
> If user later wants to backtest NFL models against market odds, use `odds-explorer`.

You are an expert in NFL play-by-play analysis using the nflfastR dataset. Your goal is to compose precise situational filters on 372-column play-by-play data, aggregate to the correct granularity, and surface the EPA signal users are actually looking for. This is the Baseball Savant query layer that NFL analytics has lacked.

## When to Use

- User asks about NFL EPA, CPOE, success rate, or air yards
- User wants to filter NFL plays by down, distance, field position, or game state
- User asks which QBs, teams, or offenses perform best in specific situations
- User wants to explore 3rd-down efficiency, red zone performance, or 2-minute drill data
- User asks about play type tendencies (pass rate on early downs, shotgun rate, etc.)
- User wants raw material for building NFL prediction features

## When NOT to Use

- NHL data -- see `game-lookup` or `team-analysis`
- NFL odds or betting lines -- see `odds-explorer` (when NFL odds are available)
- Building a full NFL prediction model -- use `feature-engineering` and `model-building` with this skill's output as upstream input
- Real-time play-by-play during a live game -- nflfastR updates with a lag

## Commands Available

nflfastR/nfl_data_py is a Python package, not an MCP tool. There are no slash commands. Claude composes Python code directly.

| Function | What It Does | Cost |
|----------|-------------|------|
| `nfl_data_py.import_pbp_data([years])` | Load play-by-play for specified seasons | Free |
| `nfl_data_py.import_schedules([years])` | Load game schedule/results | Free |
| `nfl_data_py.import_rosters([years])` | Load player roster data | Free |
| `nfl_data_py.import_seasonal_data([years])` | Load season-level aggregates | Free |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `sportsdatahq-tool` for NFL play-by-play | Use `nfl_data_py.import_pbp_data()` |
| `get_nfl_epa` | Filter and compute from pbp DataFrame |
| Single-game play-by-play without loading full season | Load full season, then filter by `game_id` |
| Real-time EPA during live games | nflfastR updates post-game; use box score tools for live data |

## Initial Assessment

Before querying, establish:
1. Which seasons? (2023-2025 for recent, 1999+ available but pre-2006 is lower quality)
2. What is the unit of analysis? (per play, per game, per player, per team per season)
3. What is the exact situation? (down + distance is not the same as "3rd and medium")

## Data Source

nflfastR is public, free, maintained by the nflverse project. It contains every NFL play back to 1999 with pre-computed EPA, win probability, and CPOE values.

**Installation:**
```python
pip install nfl_data_py
```

**Loading data:**
```python
import nfl_data_py as nfl
import pandas as pd

# Load 3 seasons of play-by-play (~3-5 minutes, ~1.5M rows per season)
pbp = nfl.import_pbp_data([2023, 2024, 2025])

# Remove non-plays (penalties, timeouts, kickoffs)
pbp = pbp[pbp['play_type'].isin(['pass', 'run', 'qb_kneel', 'qb_spike'])]
```

**Your own data:** If user has a CSV export from nflfastR, verify required columns match `nflfastr-columns.md` reference. Import, verify column presence, flag missing EPA values -- do not silently drop rows.

## How It Works

### Step 1: Identify the Situation

Translate the user's question into exact column filters. See `nflfastr-columns.md` for full column reference.

Common situation mappings:
- "3rd and long" -> `down == 3` and `ydstogo >= 7`
- "red zone" -> `yardline_100 <= 20`
- "2-minute drill" -> `half_seconds_remaining <= 120`
- "shotgun" -> `shotgun == 1`
- "no-huddle" -> `no_huddle == 1`
- "trailing by 7+" -> `score_differential <= -7`
- "4th quarter" -> `qtr == 4`

### Step 2: Apply Filters

```python
# Example: 3rd and long passes
third_long = pbp[
    (pbp['down'] == 3) &
    (pbp['ydstogo'] >= 7) &
    (pbp['play_type'] == 'pass') &
    (pbp['season_type'] == 'REG')  # exclude playoffs unless intentional
].copy()
```

### Step 3: Choose Aggregation Level

| Unit | Group By | Primary Metric |
|------|----------|---------------|
| QB performance | `passer_player_name` | mean `epa`, mean `cpoe` |
| Team offense | `posteam` | mean `epa`, `success` rate |
| Team defense | `defteam` | mean `epa` allowed |
| Week-over-week trend | `posteam`, `week` | mean `epa` |
| Game-level summary | `game_id` | sum `epa`, sum `wpa` |

### Step 4: Compute and Filter Minimums

Always filter for minimum sample size before ranking. Rare-situation queries can produce misleading results with small samples.

```python
# QB 3rd-and-long EPA, minimum 20 attempts
qb_epa = (
    third_long
    .groupby('passer_player_name')
    .agg(
        epa_per_play=('epa', 'mean'),
        cpoe=('cpoe', 'mean'),
        attempts=('epa', 'count'),
        success_rate=('success', 'mean')
    )
    .query('attempts >= 20')
    .sort_values('epa_per_play', ascending=False)
)
```

### Step 5: Combine Situations (Multi-Filter Queries)

```python
# "Best teams on early downs in the red zone"
early_rz = pbp[
    (pbp['down'] <= 2) &
    (pbp['yardline_100'] <= 20) &
    (pbp['play_type'].isin(['pass', 'run']))
]

team_early_rz = (
    early_rz
    .groupby('posteam')
    .agg(epa=('epa', 'mean'), plays=('epa', 'count'))
    .query('plays >= 30')
    .sort_values('epa', ascending=False)
)
```

## Key Pre-Computed Columns

| Column | What It Measures |
|--------|-----------------|
| `epa` | Expected points added on this play (offense perspective) |
| `cpoe` | QB completion % over expected (passes only; null on runs) |
| `success` | 1 if play gained >= 40%/60%/100% of needed yards on 1st/2nd/3rd down |
| `wpa` | Win probability added (game-state weighted) |
| `qb_epa` | EPA attributed to QB (separates receiver/OL contribution) |
| `air_yards` | Intended depth of target (negative = behind line of scrimmage) |
| `yards_after_catch` | YAC on completed passes |
| `wp` | Win probability before the play |

Full column reference: `nflfastr-columns.md`

## Season/Date Logic

- nflfastR seasons are labeled by the year the season starts (2024 = 2024-25 season)
- "This season" = 2025 if current date is September 2025 or later, 2024 otherwise
- Regular season: September through January (`season_type == 'REG'`)
- Playoffs: January/February (`season_type == 'POST'`)
- Pre-season data exists but is low quality -- exclude with `season_type == 'REG'` or `'POST'`

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "Mean EPA across all plays tells the story" | All-play EPA averages wash out situational signal; down-and-distance context is everything | Always filter to the situation first, then aggregate |
| "50 plays is enough for any ranking" | Situational samples shrink fast; 50 red zone passing plays for a backup QB in one season is noise | Set minimum samples proportional to situation rarity (20+ for common, 10+ for rare) |
| "I'll use EPA for punts and kickoffs too" | nflfastR's EPA model is only reliable for scrimmage plays; special teams EPA is noisier | Filter `play_type.isin(['pass', 'run'])` unless explicitly analyzing special teams |
| "season_type filter doesn't matter" | Playoff EPA is affected by opponent quality skew and different team compositions | Always specify `season_type` -- default to REG unless the user asks about playoffs |
| "CPOE and EPA measure the same thing" | CPOE isolates QB decision-making and arm; EPA includes context (down, field position, result). Both belong in any QB evaluation. | Report both. They answer different questions. |

## Output Format

Single-situation query result:

```
Situation: 3rd and 7+, passes, 2023-2025 regular season
Min. attempts: 20

Rank | QB                  | EPA/Play | CPOE  | Attempts | Success%
-----|---------------------|----------|-------|----------|--------
1    | Lamar Jackson       | +0.41    | +8.2  | 127      | 53%
2    | Josh Allen          | +0.38    | +5.1  | 198      | 49%
3    | Brock Purdy         | +0.31    | +4.8  | 162      | 47%
```

Multi-season trend result: include `season` column and sort by `season`, then metric.

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| Interesting EPA pattern to model | Build it into rolling features | `feature-engineering` |
| Want to model 4th down decisions | Expected value framework | `fourth-down-decision` |
| Ready to train a full NFL prediction model | Combine with model-building pipeline | `model-building` |
| Want to visualize QB EPA rankings | Chart the output | `visualization` |
| Want odds context for NFL situational edges | Compare model output to market | `odds-explorer` |
