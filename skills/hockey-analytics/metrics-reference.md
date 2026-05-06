# Hockey Analytics Metrics Reference

> Level 3 reference file. Zero context cost until loaded. Contains exact formulas, sample calculations, stabilization rates, and data source mapping.

---

## Exact Formulas

### Corsi

```
CF  = goals_for + shots_on_goal_for + missed_shots_for + blocked_shots_for (at 5v5)
CA  = goals_against + shots_on_goal_against + missed_shots_against + blocked_shots_against (at 5v5)
CF% = CF / (CF + CA)
CF/60 = (CF / TOI_5v5) * 60
```

**Sample calculation (Sabres, 40 games):**
- CF: 1,842 | CA: 1,960
- CF% = 1842 / (1842 + 1960) = 48.45%
- Below-average possession team

### Fenwick

```
FF  = CF - blocked_shots_for
FA  = CA - blocked_shots_against
FF% = FF / (FF + FA)
```

**Why Fenwick over Corsi:** Blocked shots are partially skill (shot location) and partially system (opponent blocks more in defensive deployment). Fenwick removes one source of noise.

### PDO

```
PDO = (goals_for_5v5 / shots_on_goal_for_5v5) + (1 - goals_against_5v5 / shots_on_goal_against_5v5)
    = team_shooting_pct_5v5 + team_save_pct_5v5
```

League PDO sums to exactly 1.000 because every shot for one team is a shot against another.

**Sample:**
- Shooting%: 8.2% = 0.082
- Save%: 92.1% = 0.921
- PDO = 0.082 + 0.921 = 1.003 (slightly above average)

### Expected Goals (xG)

xG is model-derived. The general form:

```
xG_shot = P(goal | shot_type, shot_distance, shot_angle, traffic, rebound, rush)
```

Typical logistic regression form:
```
log(p / 1-p) = b0 + b1*distance + b2*angle + b3*shot_type + b4*rebound + b5*rush + ...
xG = 1 / (1 + e^(-linear_combination))
```

Team xGF = sum of xG for all shots taken
Team xGA = sum of xG for all shots allowed
xGF% = xGF / (xGF + xGA)

**Provider differences:**
| Provider | Key differentiators |
|----------|-------------------|
| Moneypuck | Includes pre-shot movement, pass origin |
| Natural Stat Trick | Zone-entry adjusted, widely used |
| Evolving Hockey | Feeds into their RAPM/WAR system |
| Sportlogiq | Tracking-data based, most accurate, paid |

### High-Danger Zone Definition

Most used definition (Natural Stat Trick standard):
```
High Danger: shots from inside the "royal road" (line from one post to the other)
             and within the inner slot, roughly:
             - Within 20 feet of the net center
             - Between the faceoff circles (x: -22 to +22, y: 0 to 54 in standard coords)
```

HDC% = high_danger_chances_for / (high_danger_chances_for + high_danger_chances_against)

Typical conversion rates (league average):
- High-danger shot: ~18-20% goal rate
- Medium-danger shot: ~6-8% goal rate
- Low-danger shot: ~2-4% goal rate

### RAPM Formula

```
Ridge regression minimizing:
  sum((actual_goals_per60 - predicted_goals_per60)^2) + lambda * sum(player_coefficients^2)

Where lambda is chosen via cross-validation (typically 2000-5000 for hockey samples).
```

Adjustments applied before fitting:
1. Zone start adjustment: remove shifts starting in offensive/defensive zone
2. Score state adjustment: weight by game state (tied vs. trailing vs. leading)
3. Power play removal: 5v5 only (separate PP/PK RAPM models exist)

Output: Goals added per 60 minutes of even-strength ice time, relative to a replacement player.

### WAR (Evolving Hockey methodology)

```
WAR = (offensive_contribution + defensive_contribution) / goals_per_win

Where:
  offensive_contribution  = RAPM_offense * TOI_5v5 + PP_contribution
  defensive_contribution  = RAPM_defense * TOI_5v5 + PK_contribution
  goals_per_win           = ~6 goals (varies slightly by season)
```

Separate goalie WAR: based on Goals Saved Above Average (GSAA):
```
GSAA = (expected_goals_against - actual_goals_against)
       where expected GA uses league-average save% on shots faced
```

---

## Stabilization Rates

Stabilization = sample size where the stat is 50% signal, 50% noise (r = 0.50 in split-half reliability).

| Metric | Games to Stabilize | Notes |
|--------|-------------------|-------|
| CF% (Corsi For %) | ~35 games | One of the fastest-stabilizing team stats |
| FF% (Fenwick For %) | ~35 games | Similar to Corsi |
| PDO | ~400 shots faced | Roughly half a season |
| Shooting % (individual) | 300-400 shots | Years of data for most players |
| Save % (goalie) | 500-600 shots faced | Near-full season |
| xGF% | ~25 games | Stabilizes faster than Corsi |
| HDCF% | ~40 games | Slightly slower than CF% |
| Individual points/60 | ~200 minutes | Relatively fast for forwards |
| RAPM (individual) | 1,000+ minutes | Requires multiple seasons for defenders |

**Practical rule:** Before 20 games, PDO is noise. Before 35 games, CF% is directional but uncertain. Trust xGF% earlier than Corsi.

---

## Regressing PDO

Expected PDO (regressed):
```
regressed_PDO = (actual_shots * actual_PDO + prior_shots * 1.000) / (actual_shots + prior_shots)

Where prior_shots represents your confidence in the prior (typically 400-600 shots).
```

After 200 shots: ~33% actual, ~67% regressed toward 1.000
After 400 shots: ~50% actual, ~50% regressed
After 800 shots: ~67% actual, ~33% regressed

---

## Which Tools Provide Each Metric

| Metric | PuckAPI | Natural Stat Trick | Evolving Hockey | Moneypuck |
|--------|---------------|-------------------|-----------------|-----------|
| CF% | via team_stats | Yes | Yes | Yes |
| FF% | via team_stats (compute) | Yes | Yes | Yes |
| PDO | via team_stats (compute) | Yes | Yes | Yes |
| xGF% | No (model-derived) | Yes | Yes | Yes |
| HDCF% | No | Yes | Yes | Yes |
| Zone entries | No | Yes (limited) | No | No |
| RAPM | No | No | Yes | No |
| WAR | No | No | Yes | No |
| GSAA | via goalie_stats (compute) | Yes | Yes | Yes |

**Implication:** PuckAPI MCP provides raw counting stats. Corsi, Fenwick, and PDO are computed from those. xG, RAPM, and WAR require either external data sources or building your own model.

---

## Common Misinterpretations (Reference)

| Misinterpretation | Reality |
|------------------|---------|
| "PDO above 1.010 means the team is lucky" | High-quality teams can sustain above-average PDO. Elite goalies matter. |
| "Corsi is the best possession metric" | xGF% predicts future goals better. Corsi ignores shot quality. |
| "A player's CF% reflects their individual skill" | CF% includes linemates and zone starts. Adjust before comparing players. |
| "RAPM is objective player value" | RAPM estimates are model-dependent and noisy for small samples. |
| "WAR from different sites is comparable" | Each provider uses different inputs and formulas. Source matters. |
| "High xG means high goals" | Goalies can save high-xG shots. Short-term, actual goals can diverge significantly from xG. |

---

## Glossary of Abbreviations

| Abbreviation | Full Term |
|-------------|-----------|
| CF / CA | Corsi For / Corsi Against |
| FF / FA | Fenwick For / Fenwick Against |
| GF / GA | Goals For / Goals Against |
| xGF / xGA | Expected Goals For / Against |
| HDC / HDCA | High-Danger Chances For / Against |
| SA | Shots Against |
| SV% | Save Percentage |
| Sh% | Shooting Percentage |
| TOI | Time on Ice |
| 5v5 | Even Strength (5 skaters per side) |
| PP | Power Play |
| PK | Penalty Kill |
| GSAA | Goals Saved Above Average |
| GSAx | Goals Saved Above Expected |
| RAPM | Regularized Adjusted Plus-Minus |
| WAR | Wins Above Replacement |
| GAR | Goals Above Replacement |
