# nflfastR Key Column Reference

> Level 3 reference. Claude reads this when composing complex queries or when the user asks about a specific column. Not loaded by default.
> Full 372-column reference: https://nflreadr.nflverse.com/articles/dictionary_pbp.html

---

## Outcome Columns

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `epa` | float | Expected points added on this play. Positive = offense gained ground vs expectation. | 0.82 (successful 3rd-down conversion) |
| `ep` | float | Expected points before the play (offensive perspective) | 2.4 |
| `wpa` | float | Win probability added. Positive = increased win probability for offense. | 0.03 |
| `wp` | float | Win probability for possession team before the snap (0-1) | 0.61 |
| `vegas_wp` | float | Vegas-adjusted win probability (accounts for spread) | 0.58 |
| `success` | int | 1 if play was "successful": gained 40%+ needed on 1st, 60%+ on 2nd, 100%+ on 3rd/4th | 1 |
| `yards_gained` | int | Yards gained on the play (can be negative) | 8 |

---

## QB-Specific Columns

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `cpoe` | float | Completion percentage over expected. Positive = completed at higher rate than model predicts given depth/coverage. Null on non-pass plays. | +6.3 |
| `qb_epa` | float | EPA attributed to the QB decision (strips out receiver/OL contribution using air yards adjustment) | 0.55 |
| `air_epa` | float | EPA attributable to the air portion of the throw (pre-catch) | 0.40 |
| `yac_epa` | float | EPA attributable to yards after catch (post-catch) | 0.42 |
| `comp_air_epa` | float | `air_epa` on completed passes only | 0.48 |
| `comp_yac_epa` | float | `yac_epa` on completed passes only | 0.44 |
| `passer_player_name` | str | QB name (format: "P.Mahomes"). Use for groupby. Null on non-pass plays. | "J.Allen" |
| `passer_player_id` | str | GSIS player ID. More stable than name for multi-season joins. | "00-0023459" |
| `complete_pass` | int | 1 = completed, 0 = incomplete/int | 1 |
| `incomplete_pass` | int | 1 = incomplete | 0 |
| `interception` | int | 1 = interception | 0 |
| `air_yards` | int | Intended depth of target in yards past line of scrimmage. Negative = behind LOS (screens). | 14 |
| `yards_after_catch` | float | YAC on completed passes. Null on incompletions. | 6 |
| `pass_location` | str | "left", "middle", "right" | "right" |
| `pass_length` | str | "short" (< 15 air yards), "deep" (>= 15 air yards) | "short" |

---

## Rusher Columns

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `rusher_player_name` | str | Ball carrier name. Null on pass plays. | "D.Henry" |
| `rusher_player_id` | str | GSIS player ID | "00-0036983" |
| `run_location` | str | "left", "middle", "right" | "left" |
| `run_gap` | str | "end", "guard", "tackle" | "guard" |

---

## Game State / Situation Columns

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `down` | int | Current down (1-4). Null on kickoffs/penalties. | 3 |
| `ydstogo` | int | Yards needed for first down (or TD on 4th) | 7 |
| `yardline_100` | int | Yards to opponent end zone (100 = own goal line, 1 = opponent 1-yard line) | 15 |
| `score_differential` | int | Possession team score minus opponent score | -7 |
| `half_seconds_remaining` | int | Seconds remaining in current half | 87 |
| `game_seconds_remaining` | int | Seconds remaining in game | 1287 |
| `qtr` | int | Quarter (1-5, where 5 = OT) | 4 |
| `goal_to_go` | int | 1 if ydstogo equals yardline_100 (true goal-line situation) | 0 |

---

## Formation / Personnel Columns

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `shotgun` | int | 1 if offense lined up in shotgun | 1 |
| `no_huddle` | int | 1 if offense did not huddle | 0 |
| `offense_personnel` | str | Offensive personnel grouping string | "1 RB, 1 TE, 3 WR" |
| `defense_personnel` | str | Defensive personnel grouping string | "4 DL, 2 LB, 5 DB" |
| `defenders_in_box` | int | Defenders in the box at snap | 6 |
| `number_of_pass_rushers` | int | Pass rushers on the play | 4 |

---

## Play Type / Result Columns

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `play_type` | str | "pass", "run", "punt", "field_goal", "kickoff", "qb_kneel", "qb_spike", "no_play" | "pass" |
| `pass` | int | 1 if play was a pass attempt (dropbacks including sacks, scrambles) | 1 |
| `rush` | int | 1 if play was a rush | 0 |
| `sack` | int | 1 if QB was sacked | 0 |
| `qb_hit` | int | 1 if QB was hit regardless of result | 1 |
| `qb_scramble` | int | 1 if QB scrambled for yards | 0 |
| `penalty` | int | 1 if play had a penalty | 0 |
| `first_down` | int | 1 if play resulted in a first down | 1 |
| `touchdown` | int | 1 if play resulted in a touchdown | 0 |
| `fumble` | int | 1 if play had a fumble | 0 |

---

## Game / Season Identifier Columns

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `game_id` | str | Unique game identifier | "2024_01_KC_BAL" |
| `home_team` | str | Home team abbreviation | "BAL" |
| `away_team` | str | Away team abbreviation | "KC" |
| `posteam` | str | Team with possession on this play | "KC" |
| `defteam` | str | Defending team on this play | "BAL" |
| `season` | int | NFL season year | 2024 |
| `week` | int | Week of the season (1-22 including playoffs) | 1 |
| `season_type` | str | "REG" (regular season) or "POST" (playoffs) | "REG" |
| `game_date` | str | Date of game in YYYY-MM-DD format | "2024-09-05" |

---

## Common Filter Recipes

```python
# Standard scrimmage plays only (exclude kickoffs, punts, penalties)
pbp = pbp[pbp['play_type'].isin(['pass', 'run'])]

# Dropbacks (passes + scrambles, for QB analysis)
dropbacks = pbp[(pbp['pass'] == 1) | (pbp['qb_scramble'] == 1)]

# Non-garbage time (within 2 scores, not last 2 min of 4th)
competitive = pbp[
    (abs(pbp['score_differential']) <= 16) &
    ~((pbp['qtr'] == 4) & (pbp['game_seconds_remaining'] < 120))
]

# 11 personnel (spread, 3-receiver) only
eleven_pers = pbp[pbp['offense_personnel'] == '1 RB, 1 TE, 3 WR']

# Red zone scoring opportunities
red_zone = pbp[pbp['yardline_100'] <= 20]

# 3rd and medium (4-6 yards to go)
third_medium = pbp[(pbp['down'] == 3) & (pbp['ydstogo'].between(4, 6))]
```

---

## Notes on Data Quality

- Pre-2006 data: missing some columns (CPOE, defenders_in_box, personnel). Use cautiously.
- 2006+: full column coverage for most metrics.
- `cpoe` requires the nflfastR completion probability model. Introduced 2006+.
- Personnel columns occasionally missing for 1-2% of plays.
- Sack yards are negative in `yards_gained`. Don't double-count sacks and pass plays -- use `pass == 1` not `play_type == 'pass'` to include sack dropbacks.
