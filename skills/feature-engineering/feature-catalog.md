# Feature Catalog

> Level 3 reference. Loaded on demand -- zero context cost until accessed.
> All features are computed as home-minus-away differences unless marked as matchup-invariant.
> All features require `.shift(1)` before rolling calculation. Leakage risk assumes shift was applied correctly.

---

## How to Read This Catalog

- **Window**: The rolling N-game window used. "Season" = cumulative season-to-date.
- **Raw endpoint**: The `sportsdatahq-tool` call that provides the source data.
- **Leakage risk**: LOW = safe with shift(1). MEDIUM = careful construction required. HIGH = do not use without explicit audit.
- **Predictive value**: Based on PuckCast feature importance and published literature.

---

## Scoring & Goals Features

| Feature Name | Formula | Window | Raw Endpoint | Leakage Risk | Predictive Value |
|-------------|---------|--------|-------------|-------------|-----------------|
| `goals_for_roll5` | mean(goals_for[shift(1):]) | 5 games | `get_games` | LOW | HIGH -- captures hot streaks |
| `goals_for_roll10` | mean(goals_for[shift(1):]) | 10 games | `get_games` | LOW | HIGH -- stable recent form |
| `goals_for_roll20` | mean(goals_for[shift(1):]) | 20 games | `get_games` | LOW | HIGH -- smoothed trend |
| `goals_for_roll40` | mean(goals_for[shift(1):]) | 40 games | `get_games` | LOW | MEDIUM -- regresses to mean |
| `goals_against_roll10` | mean(goals_against[shift(1):]) | 10 games | `get_games` | LOW | HIGH |
| `goal_diff_roll10` | goals_for_roll10 - goals_against_roll10 | 10 games | `get_games` | LOW | HIGH -- direct strength signal |
| `season_goals_for` | cumsum(goals_for).shift(1) / games_played | Season | `get_team_stats` | MEDIUM | MEDIUM -- slow to update |

---

## Shot Attempt Features (5v5)

| Feature Name | Formula | Window | Raw Endpoint | Leakage Risk | Predictive Value |
|-------------|---------|--------|-------------|-------------|-----------------|
| `corsi_pct_roll10` | CF / (CF + CA), rolling | 10 games | `get_team_stats` | LOW | HIGH -- best single possession metric |
| `corsi_pct_roll20` | CF / (CF + CA), rolling | 20 games | `get_team_stats` | LOW | HIGH |
| `fenwick_pct_roll10` | FF / (FF + FA), rolling | 10 games | `get_team_stats` | LOW | HIGH -- like Corsi, excludes blocked shots |
| `shots_for_roll10` | mean(shots_for[shift(1):]) | 10 games | `get_games` | LOW | MEDIUM -- volume proxy |
| `shots_against_roll10` | mean(shots_against[shift(1):]) | 10 games | `get_games` | LOW | MEDIUM |

---

## PDO and Luck Features

| Feature Name | Formula | Window | Raw Endpoint | Leakage Risk | Predictive Value |
|-------------|---------|--------|-------------|-------------|-----------------|
| `pdo_roll10` | (team_sv_pct + team_sh_pct) * 1000 | 10 games | `get_team_stats` | LOW | MEDIUM -- mean-reverts; high PDO = due for regression |
| `sh_pct_roll10` | goals / shots_for, rolling | 10 games | `get_games` | LOW | LOW -- noisy, mean-reverts fast |
| `sv_pct_roll10` | (shots_against - goals_against) / shots_against | 10 games | `get_games` | LOW | LOW -- mostly goalie signal |
| `pdo_deviation` | pdo_roll10 - 1000 | 10 games | derived | LOW | MEDIUM -- negative = underperforming luck |

---

## Power Play / Penalty Kill Features

| Feature Name | Formula | Window | Raw Endpoint | Leakage Risk | Predictive Value |
|-------------|---------|--------|-------------|-------------|-----------------|
| `pp_pct_roll10` | PP goals / PP opportunities, rolling | 10 games | `get_team_stats` | LOW | MEDIUM |
| `pk_pct_roll10` | 1 - (PK goals against / times shorthanded) | 10 games | `get_team_stats` | LOW | MEDIUM |
| `net_pp_roll10` | pp_pct_roll10 - opp_pk_pct_roll10 | 10 games | derived | LOW | HIGH -- matchup-adjusted special teams |

---

## Goalie Features

| Feature Name | Formula | Window | Raw Endpoint | Leakage Risk | Predictive Value |
|-------------|---------|--------|-------------|-------------|-----------------|
| `starter_sv_pct_ytd` | cumulative SV%, lagged | Season | `get_goalie_stats` | MEDIUM | HIGH -- best single goalie predictor |
| `starter_sv_pct_roll5` | mean(SV%[shift(1):]) over last 5 starts | 5 starts | `get_goalie_stats` | LOW | HIGH -- recent form |
| `starter_gsaa_ytd` | cumulative GSAA, lagged | Season | `get_goalie_stats` | MEDIUM | HIGH -- quality above average |
| `starter_confirmed` | 1 if announced starter, 0 if uncertain | Game-day | `get_game_detail` | LOW | MEDIUM -- uncertainty penalty |
| `goalie_starts_roll10` | starts in last 10 team games | 10 games | `get_goalie_stats` | LOW | LOW -- fatigue proxy |

**GSAA formula:**
```
GSAA = shots_against * (league_avg_sv_pct - goalie_sv_pct)
Positive GSAA = better than average. Negative = worse than average.
```

**Leakage note on `starter_sv_pct_ytd`:** Must be season-to-date through game N-1. If you join full-season SV% as a static feature, you are leaking the rest of the season. Rebuild per-game cumulatively.

---

## Rest and Schedule Features

| Feature Name | Formula | Window | Raw Endpoint | Leakage Risk | Predictive Value |
|-------------|---------|--------|-------------|-------------|-----------------|
| `rest_days` | (game_date - prev_game_date).days - 1 | — | `get_games` dates | LOW | HIGH -- back-to-backs matter |
| `back_to_back` | 1 if rest_days == 0 | — | derived | LOW | HIGH |
| `rest_bucket` | categorical: b2b / 1day / 2day / 3plus | — | derived | LOW | HIGH |
| `days_since_last_away` | days since last road game (home team) | — | `get_games` | LOW | LOW -- marginal |
| `opp_back_to_back` | opponent's back_to_back flag | — | derived | LOW | HIGH -- opponent fatigue advantage |

---

## Elo Features

| Feature Name | Formula | Window | Raw Endpoint | Leakage Risk | Predictive Value |
|-------------|---------|--------|-------------|-------------|-----------------|
| `pre_game_elo` | Elo rating before game (after last game update) | — | `elo-engineering` skill | LOW | HIGH |
| `elo_diff` | home_elo - away_elo | — | derived | LOW | HIGH -- best single Elo feature |
| `elo_win_prob` | logistic(elo_diff / 400) | — | derived | LOW | HIGH -- market comparison baseline |

See `elo-engineering` skill for calculation methodology and parameter choices.

---

## Strength of Schedule Features

| Feature Name | Formula | Window | Raw Endpoint | Leakage Risk | Predictive Value |
|-------------|---------|--------|-------------|-------------|-----------------|
| `sos_ytd` | Iterative SOS through game N-1 | Season | `get_standings` | MEDIUM | MEDIUM |
| `opp_sos_ytd` | Opponent's iterative SOS | Season | derived | MEDIUM | MEDIUM |
| `sos_adjusted_gf` | goals_for_roll10 / (1 + sos_ytd) | 10 games | derived | MEDIUM | HIGH |

**Iterative SOS construction (required):**
```python
# Never use simple opponent win% -- it underweights quality opponents
# Convergence typically reached in 10-20 iterations
ratings = {team: win_pct for team in all_teams}
for _ in range(20):
    new_ratings = {}
    for team in all_teams:
        opponents = schedule[team][:current_game]
        new_ratings[team] = np.mean([ratings[opp] for opp in opponents])
    ratings = new_ratings
```

---

## Opponent-Adjusted Features

| Feature Name | Formula | Window | Raw Endpoint | Leakage Risk | Predictive Value |
|-------------|---------|--------|-------------|-------------|-----------------|
| `adj_goals_for_roll10` | home_gf_roll10 - opp_ga_roll10 | 10 games | derived | LOW | HIGH |
| `adj_corsi_roll10` | home_cf_pct_roll10 - 0.5 (league neutral) | 10 games | derived | LOW | HIGH |
| `adj_shots_roll10` | home_shots_roll10 - opp_shots_against_roll10 | 10 games | derived | LOW | MEDIUM |

---

## Matchup-Invariant Features (Not Differenced)

These features describe the matchup context, not individual team strength. Use as-is.

| Feature Name | Formula | Window | Raw Endpoint | Leakage Risk | Predictive Value |
|-------------|---------|--------|-------------|-------------|-----------------|
| `is_playoff` | 1 if playoff game | — | `get_games` | LOW | HIGH -- model behavior differs |
| `month_of_season` | 1-7 (Oct=1, Apr=7) | — | game_date | LOW | LOW -- late-season urgency |
| `division_rival` | 1 if same division | — | `list_teams` | LOW | LOW -- marginal home effect |

---

## Features to Avoid

| Feature | Why Dangerous |
|---------|---------------|
| Current-game shots (as a feature for current game) | Leakage -- you don't know shots until the game ends |
| Full-season SV% as static join | Leaks future games in the season |
| Opponent standings position from final standings | Leaks end-of-season result |
| Win streak including current game | Current game hasn't happened |
| Any stat pulled for "today" and used for "today's prediction" | Always shift -- even today's morning skate data needs scrutiny |

---

## Recommended Minimal Feature Set (to Start)

If starting from scratch, begin with these 12 features (all home-minus-away):

1. `goals_for_roll10`
2. `goals_against_roll10`
3. `corsi_pct_roll10`
4. `pdo_roll10`
5. `pp_pct_roll10`
6. `pk_pct_roll10`
7. `starter_sv_pct_ytd`
8. `starter_gsaa_ytd`
9. `elo_diff`
10. `rest_days` (home)
11. `rest_days` (away)
12. `back_to_back` (both teams, separate flags)

Run walk-forward validation on this set. Add features only when walk-forward accuracy improves.
