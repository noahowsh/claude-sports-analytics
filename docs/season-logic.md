# Season & Date Resolution Rules

Deterministic rules for resolving time-sensitive references across all skills.

## NHL Season Resolution

| Current Month | "This season" | "Last season" |
|---------------|--------------|---------------|
| October - December | Current year start (e.g., Oct 2026 = 2026-27) | Previous year (2025-26) |
| January - September | Previous year start (e.g., Mar 2027 = 2026-27) | Two years back (2025-26) |

**Key dates:**
- Regular season: early October to mid-April (~82 games)
- Playoffs: mid-April to mid-June (4 rounds, best-of-7)
- Draft: late June / early July
- Free agency: July 1
- Preseason: mid-September to early October

**"Tonight" / "today":** Use the current date. If before 4 PM ET, also check if there are games from last night with no results yet.

**"This week":** Monday through Sunday of the current week.

## NFL Season Resolution

| Current Month | "This season" | "Last season" |
|---------------|--------------|---------------|
| September - December | Current year (e.g., Sep 2026 = 2026) | Previous year (2025) |
| January - February | Previous year (e.g., Jan 2027 = 2026 season) | Two years back (2025) |
| March - August | Most recently completed (e.g., Jul 2027 = 2026 season) | Two years back (2025) |

**Key dates:**
- Regular season: early September to early January (18 weeks, 17 games)
- Playoffs: mid-January to mid-February (Wild Card through Super Bowl)
- Draft: late April
- Free agency: mid-March
- Training camp: late July

## Game ID Format (Sports Data HQ)

NHL game IDs follow the format: `SSSSTTNNNN`
- `SSSS` = season start year (e.g., 2024 for the 2024-25 season)
- `TT` = game type (01 = preseason, 02 = regular, 03 = playoffs)
- `NNNN` = game number (0001-1312 regular season, 0111-0417 playoffs)

Example: `2024020001` = first regular season game of 2024-25 season.

## Data Coverage (Sports Data HQ)

| Data Type | Coverage |
|-----------|---------|
| Games | 2008-09 to present (16+ seasons, 22,037+ games) |
| Odds | 2019-20 to present (106,958+ records) |
| Players | Current + historical (3,021+ players) |
| Goalies | Current + historical (1,509+ goalie-seasons) |
| Standings | Current + historical (494+ records) |
| Teams | All 32 current NHL teams |

**Note:** Odds data before 2019-20 is not available. Backtests requiring odds are limited to ~5 seasons.
