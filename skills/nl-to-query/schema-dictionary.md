# Schema Dictionary

> Level 3 reference. Loaded on demand by `nl-to-query` when translating complex or ambiguous terms.

Full mapping of natural language sports terms to data fields, operators, and values.

---

## Hockey (NHL) Terms

### Game Conditions

| User Said | Field | Operator | Value | Notes |
|-----------|-------|----------|-------|-------|
| "outshot" | shots_for < shots_against | — | — | Team received more shots than they generated |
| "outshot 2:1" | shots_against / shots_for | > | 2.0 | Severe shot differential |
| "outshot badly" | shots_against - shots_for | > | 10 | Absolute differential |
| "dominated possession" | shots_for / (shots_for + shots_against) | > | 0.55 | ~Corsi% proxy |
| "back-to-back" | rest_days | = | 0 | Zero days between games |
| "fresh legs" | rest_days | >= | 3 | 3+ days rest |
| "overtime win" | result | in | ["OT", "OTW", "SOW"] | Win in extra time |
| "overtime loss" | result | in | ["OTL", "SOL"] | Regulation tie, OT/SO loss |
| "shootout" | result | in | ["SOW", "SOL"] | Game decided in shootout |
| "regulation win" | result | = | "W" AND periods_played | = | 3 | Not OT/SO |
| "shutout" | goals_against | = | 0 | No goals allowed |
| "blowout" | abs(goal_differential) | >= | 4 | 4+ goal margin |
| "close game" | abs(goal_differential) | <= | 1 | 1 goal or less |
| "comeback win" | was_trailing | = | true AND result | = | "W" | Trailed at some point |
| "home ice" OR "home team" | venue_type | = | "home" | — |
| "road game" OR "away team" | venue_type | = | "away" | — |
| "division rival" | same_division | = | true | Same division matchup |
| "conference game" | same_conference | = | true | Same conference |
| "first half of season" | game_number | <= | 41 | NHL 82-game season |
| "back half of season" | game_number | >= | 42 | — |
| "last 10 games" | (sort by date desc, limit 10) | — | — | Apply to filtered results |
| "power play game" | pp_opportunities | >= | 3 | 3+ PP chances |
| "penalty-filled" | total_penalties | >= | 10 | Combined |

### Team Metrics

| User Said | Field | Operator | Value | Notes |
|-----------|-------|----------|-------|-------|
| "good offense" | goals_for_avg | >= | 3.2 | Season average |
| "weak offense" | goals_for_avg | <= | 2.5 | — |
| "stingy defense" | goals_against_avg | <= | 2.5 | — |
| "porous defense" | goals_against_avg | >= | 3.2 | — |
| "good power play" | pp_pct | >= | 0.22 | 22%+ PP |
| "struggling PP" | pp_pct | <= | 0.15 | — |
| "good penalty kill" | pk_pct | >= | 0.82 | 82%+ PK |
| "hot team" | points_last_10 | >= | 14 | 7+ wins in 10 |
| "cold team" | points_last_10 | <= | 8 | — |
| "playoff contender" | playoff_odds | >= | 0.50 | If available |
| "bottom feeder" | standings_rank | >= | 25 | League rank |

### Goalie Conditions

| User Said | Field | Operator | Value | Notes |
|-----------|-------|----------|-------|-------|
| "strong goalie performance" | sv_pct | >= | 0.915 | Single-game |
| "bad goalie game" | sv_pct | <= | 0.875 | — |
| "quality start" | (sv_pct >= 0.915) OR (gaa <= 2.50 AND shots_faced < 20) | — | — | QS definition |
| "really bad start" | sv_pct | <= | 0.850 | Threshold for RBS |
| "high workload" | shots_faced | >= | 40 | Heavy game |
| "goalie pulled" | goalie_pulled | = | true | Mid-game change |
| "starter" | is_starter | = | true | Confirmed starter |
| "backup in" | starts | = | 0 AND gp | = | 1 | Emergency usage |

### Player Conditions

| User Said | Field | Operator | Value | Notes |
|-----------|-------|----------|-------|-------|
| "point game" | points | >= | 1 | Scored or assisted |
| "multi-point game" | points | >= | 2 | — |
| "hat trick" | goals | >= | 3 | — |
| "big night" | points | >= | 3 | — |
| "held scoreless" | points | = | 0 | — |
| "heavy minutes" | toi | >= | 22 | Time on ice (minutes) |
| "sheltered" | toi | <= | 12 | Limited ice time |
| "plus game" | plus_minus | > | 0 | — |

---

## Football (NFL) Terms

### Game Conditions

| User Said | Field | Operator | Value | Notes |
|-----------|-------|----------|-------|-------|
| "home team" | venue_type | = | "home" | — |
| "road game" | venue_type | = | "away" | — |
| "blowout" | abs(score_differential) | >= | 14 | 2-TD margin |
| "close game" | abs(score_differential) | <= | 7 | One-score game |
| "overtime" | result | contains | "OT" | — |
| "division game" | same_division | = | true | — |
| "primetime" | game_time_et | >= | 20:00 | SNF/MNF/TNF |
| "cold weather" | temperature_f | <= | 32 | Freezing or below |
| "dome game" | is_dome | = | true | Indoor stadium |
| "short week" | days_rest | <= | 5 | TNF situation |
| "bye week benefit" | days_rest | >= | 14 | Post-bye |
| "back half of season" | week | >= | 10 | Week 10+ |
| "early season" | week | <= | 4 | — |

### Team Metrics

| User Said | Field | Operator | Value | Notes |
|-----------|-------|----------|-------|-------|
| "high-scoring offense" | pts_per_game | >= | 27 | Season average |
| "run-heavy" | rush_att_pct | >= | 0.45 | Run/total play ratio |
| "pass-heavy" | pass_att_pct | >= | 0.60 | — |
| "strong defense" | pts_allowed_avg | <= | 18 | — |
| "porous defense" | pts_allowed_avg | >= | 27 | — |
| "turnover prone" | turnovers_per_game | >= | 2.0 | — |
| "good red zone" | red_zone_td_pct | >= | 0.60 | — |

### Play-Level Conditions (EPA)

| User Said | EPA Field | Notes |
|-----------|-----------|-------|
| "big play" | epa | > | 5.0 | High-value single play |
| "disaster play" | epa | < | -5.0 | — |
| "efficient offense" | epa_per_play | > | 0.1 | Season or game avg |
| "run on first down" | down = 1 AND play_type = "run" | — |
| "third and long" | down = 3 AND ydstogo >= 7 | — |
| "red zone" | yardline_100 <= 20 | — |
| "two-minute drill" | half_seconds_remaining <= 120 | — |
| "garbage time" | wp < 0.10 OR wp > 0.90 | Score blowout |

---

## Ambiguous Terms (Require Clarification)

These terms mean different things in different contexts. Ask before mapping:

| Term | Could Mean | Ask |
|------|-----------|-----|
| "won" | Regulation win only vs. any win including OT | "Regulation wins only, or including OT?" |
| "shot attempts" | Shots on goal vs. Corsi (all attempts) | "Shots on goal, or all shot attempts?" |
| "recent" | Last 5 / 10 / 30 games | "Last 10 games, or last 30?" |
| "struggling" | Record, goal differential, or advanced metrics | "By wins, or by shot metrics?" |
| "pressure situation" | NHL: late-game tie / NFL: inside 5 minutes | Clarify sport context |
| "big game" | Rivalry, playoff, primetime, or point differential | "Do you mean a high-stakes game or just a high-scoring one?" |

---

## Threshold Reference

Standard thresholds used when user says "good," "bad," "strong," etc. Adjust based on context.

### NHL Thresholds

| Metric | Poor | Average | Good | Elite |
|--------|------|---------|------|-------|
| Team SV% | < .900 | .905 | .910 | > .920 |
| Team GAA | > 3.2 | 2.9 | 2.6 | < 2.4 |
| Team SF% (shots) | < .470 | .490 | .510 | > .530 |
| PP% | < .150 | .185 | .220 | > .270 |
| PK% | < .770 | .800 | .825 | > .850 |

### NFL Thresholds

| Metric | Poor | Average | Good | Elite |
|--------|------|---------|------|-------|
| EPA/play (offense) | < -0.10 | 0.00 | 0.10 | > 0.20 |
| EPA/play (defense) | > 0.10 | 0.00 | -0.10 | < -0.20 |
| DVOA (offense) | < -15% | 0% | +15% | > +30% |
| Completion % | < 58% | 63% | 67% | > 72% |

---

## Common Multi-Condition Queries

Pre-built filter chains for recurring research questions:

**"Outshot but won" (NHL)**
- shots_against > shots_for AND result IN ["W", "OTW"]

**"Back-to-back struggles" (NHL)**
- rest_days = 0 AND result NOT IN ["W", "OTW", "SOW"]

**"Goalie stolen win"**
- sv_pct >= 0.930 AND goals_for <= 2 AND result IN ["W", "OTW"]

**"Trap game setup" (NHL)**
- previous_opponent_quality = "high" AND current_opponent_quality = "low" AND venue_type = "away"
- Note: opponent_quality must be derived -- use standings rank proxy

**"Home dog covers" (NFL)**
- venue_type = "home" AND opening_spread > 0 AND score_differential >= opening_spread

**"Total goes over in cold weather" (NFL)**
- temperature_f <= 32 AND (home_score + away_score) > total_line
