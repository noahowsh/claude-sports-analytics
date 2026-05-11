# PuckAPI -- Full Endpoint Reference

> Level 3 reference. Load this file when you need exact parameter names, accepted values, or return field details. Do not load for every request -- SKILL.md covers the common cases.

---

## Games Tools

### `get_games`

Get NHL games filtered by date range, team, season, or game state.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `date_from` | string | No | Start date, ISO 8601 (`YYYY-MM-DD`) |
| `date_to` | string | No | End date, ISO 8601 (`YYYY-MM-DD`) |
| `team` | string | No | Team abbreviation -- matches home OR away (case-insensitive) |
| `season` | string | No | Season ID, 8 digits (e.g. `20252026`) |
| `game_state` | enum | No | `FUT` `LIVE` `FINAL` `OFF` |
| `game_type` | enum | No | `regular` `playoff` `preseason` |
| `limit` | integer | No | 1-500. Default: 100 |

**game_state values:**
- `FUT` -- scheduled, not started
- `LIVE` -- in progress
- `FINAL` -- completed (official result)
- `OFF` -- official (post-game stats confirmed)

**Return fields:**

| Field | Type | Notes |
|-------|------|-------|
| `id` | string | NHL game ID (e.g. `2025020887`) |
| `season` | string | Season ID |
| `gameDate` | string | `YYYY-MM-DD` |
| `startTime` | string | ISO timestamp, UTC |
| `homeTeam` | string | Team abbreviation |
| `awayTeam` | string | Team abbreviation |
| `homeTeamName` | string | Full team name |
| `awayTeamName` | string | Full team name |
| `homeScore` | integer | null if game not played |
| `awayScore` | integer | null if game not played |
| `gameState` | string | See enum above |
| `gameType` | string | `regular` `playoff` `preseason` |
| `venue` | string | Arena name |

**Example:**
```
get_games(
  team="BUF",
  date_from="2026-01-01",
  date_to="2026-01-31",
  game_state="FINAL"
)
```

---

### `get_schedule`

Upcoming NHL games that haven't been played yet (game_state=FUT). Always ordered ascending by date.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `team` | string | No | Team abbreviation filter |
| `days` | integer | No | Days ahead to look, 1-30. Default: 7 |

**Return fields:** Same as `get_games` minus score fields. Returns `upcoming_games` array.

**Notes:**
- Date window starts at today's date in Eastern Time (America/New_York)
- Only returns future games; completed games never appear even within the window

---

### `get_game_detail`

Full details for one game: team info, all available odds, and goalie starts.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `game_id` | string | Yes | NHL game ID (e.g. `2025020887`) |

**Return structure:**

```
{
  game: { ...all game fields },
  home_team: { ...team record },
  away_team: { ...team record },
  odds: [ ...odds records ],
  goalie_starts: { ...goalie record } | null
}
```

**Odds fields (within game_detail):**

| Field | Description |
|-------|-------------|
| `bookmaker` | Bookmaker key (e.g. `draftkings`, `fanduel`, `betmgm`) |
| `snapshotType` | `opening` `current` `closing` |
| `source` | Data source identifier |
| `mlHome` | Home moneyline (American odds, e.g. -130) |
| `mlAway` | Away moneyline |
| `mlHomeProb` | Implied probability, home (decimal, 0-1) |
| `mlAwayProb` | Implied probability, away (decimal, 0-1) |
| `spreadHome` | Home puck line (usually -1.5 or +1.5) |
| `spreadHomeOdds` | Odds on home spread |
| `spreadAway` | Away puck line |
| `spreadAwayOdds` | Odds on away spread |
| `total` | Over/under total |
| `totalOverOdds` | Odds on the over |
| `totalUnderOdds` | Odds on the under |
| `capturedAt` | ISO timestamp when odds were captured |

**Goalie starts fields:**

| Field | Description |
|-------|-------------|
| `gameId` | Game ID |
| `homeGoalieName` | Home starting goalie name |
| `awayGoalieName` | Away starting goalie name |
| `homeGoalieId` | Home starting goalie NHL player ID |
| `awayGoalieId` | Away starting goalie NHL player ID |
| `homeGoalieToi` | Home goalie time on ice |
| `awayGoalieToi` | Away goalie time on ice |

**Credit note:** Calling `get_game_detail` costs 10 credits (flat, regardless of whether odds data is present). This is the same cost as `get_odds` -- the difference is `get_game_detail` also returns team info and goalie starts in a single call.

---

### `get_head_to_head`

Matchup history between two specific teams.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `team1` | string | Yes | First team abbreviation (e.g. `BUF`) |
| `team2` | string | Yes | Second team abbreviation (e.g. `TOR`) |
| `season` | string | No | Filter to one season. Omit for all-time history. |
| `limit` | integer | No | 1-100. Default: 20 |

**Return structure:**

```
{
  team1: "BUF",
  team2: "TOR",
  record: { "BUF": 12, "TOR": 8 },
  total_games: 20,
  games: [ ...game records ]
}
```

**Notes:**
- Only returns completed games (`gameState` = `FINAL` or `OFF`)
- `team1` and `team2` must be different -- passing the same abbreviation returns an error
- Record counts wins for each team across the returned game sample (not all-time unless no limit is set)

---

## Team Tools

### `get_standings`

Current NHL standings with advanced stats. Returns the latest snapshot date automatically.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `conference` | enum | No | `Eastern` `Western` |
| `division` | string | No | `Atlantic` `Metropolitan` `Central` `Pacific` |
| `season` | string | No | Season ID. Default: current season (auto-resolved) |

**Return fields:**

| Field | Description |
|-------|-------------|
| `teamAbbrev` | 2-3 letter abbreviation |
| `teamName` | Full team name |
| `conference` | `Eastern` or `Western` |
| `division` | Division name |
| `rank` | Overall league rank |
| `points` | Points accumulated |
| `gamesPlayed` | Games played |
| `wins` | Regulation wins |
| `losses` | Regulation losses |
| `otLosses` | OT/SO losses |
| `pointPct` | Points percentage (decimal) |
| `goalDiff` | Goals for minus goals against |
| `goalsForPerGame` | Offensive rate |
| `goalsAgainstPerGame` | Defensive rate |
| `corsiPct` | Corsi for % (shot attempt share) |
| `fenwickPct` | Fenwick for % (unblocked shot attempt share) |
| `expectedGoalsFor` | Total xGF |
| `expectedGoalsAgainst` | Total xGA |
| `snapshotDate` | Date this standings snapshot was captured |

**Notes:**
- Returns the most recent snapshot date when multiple snapshots exist for a season
- `conference` filter: exact match, capital first letter (`Eastern` not `eastern`)
- `division` filter: exact match (`Atlantic` `Metropolitan` `Central` `Pacific`)

---

### `get_team_stats`

Detailed stats for a single team.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `team` | string | Yes | Team abbreviation (e.g. `BUF`) |
| `season` | string | No | Season ID. Default: current season |

**Return structure:**

```
{
  team: { abbrev, fullName, city, conference, division, arena, active },
  current_stats: { ...same fields as get_standings row } | null,
  season: "20252026"
}
```

**Error case:** Unknown abbreviation returns `{ error: "Team 'XYZ' not found..." }`.

---

### `list_teams`

All 32 active NHL teams. Excludes historical/relocated franchises (Atlanta Thrashers `ATL`, original Arizona Coyotes `ARI`).

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `conference` | enum | No | `Eastern` `Western` |
| `division` | string | No | `Atlantic` `Metropolitan` `Central` `Pacific` |

**Return fields per team:** `abbrev` `name` `city` `conference` `division` `arena`

**Notes:**
- Results ordered by division then team name
- 34 teams in the database total; `list_teams` filters to `active=true` (32 teams)

---

## Player Tools

### `search_players`

Find players by name across the full 3,021-player database.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Name to search. Partial match supported (contains, case-insensitive). Min 1 char. |
| `team` | string | No | Filter by current team abbreviation |
| `position` | enum | No | `G` `D` `C` `LW` `RW` |
| `active` | boolean | No | Default: `true`. Set `false` to include retired/historical players. |
| `limit` | integer | No | 1-50. Default: 10 |

**Return fields:**

| Field | Description |
|-------|-------------|
| `id` | NHL player ID (integer -- use this for `get_player_stats`) |
| `name` | Full name |
| `firstName` | First name |
| `lastName` | Last name |
| `team` | Current team abbreviation |
| `teamName` | Current team full name |
| `position` | Position code |
| `positionType` | `Forward` `Defenseman` `Goalie` |
| `jerseyNumber` | Jersey number |
| `birthCountry` | Country code |
| `active` | Boolean |

**Notes:**
- Partial match works on full name field (`name` column). "Connor" matches "Connor McDavid."
- For retired players, set `active=false` explicitly -- default excludes them
- Position `G` returns goalies; use `get_goalie_stats` for performance leaderboards

---

### `get_player_stats`

Bio and stats for one player by NHL player ID.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `player_id` | integer | Yes | NHL player ID from `search_players` |

**Return structure:**

```
{
  player: {
    id, name, firstName, lastName,
    team, teamName, position, positionType,
    jerseyNumber, birthDate, birthCountry,
    heightInches, weightLbs, shootsCatches,
    headshotUrl
  },
  goalie_stats: { ...most recent goalie season } | null
}
```

**Notes:**
- `goalie_stats` is populated only when `player.position === "G"` and goalie stats exist
- Returns the most recent goalie season snapshot if multiple exist
- `shootsCatches`: `L` or `R` for skaters; `L` or `R` for goalies (catching hand)

---

### `get_goalie_stats`

Goalie performance leaderboard. Filters and sorts across 1,509 goalie-season records.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `team` | string | No | Filter by team abbreviation |
| `season` | string | No | Season ID. Default: current season |
| `min_games` | integer | No | Minimum games played filter. Default: 10 |
| `sort_by` | enum | No | `save_pct` `gaa` `gsax` `wins`. Default: `save_pct` |
| `limit` | integer | No | 1-50. Default: 20 |

**Return fields:**

| Field | Description |
|-------|-------------|
| `playerId` | NHL player ID |
| `playerName` | Goalie name |
| `team` | Team abbreviation |
| `teamName` | Full team name |
| `gamesPlayed` | Games played |
| `wins` | Wins |
| `losses` | Regulation losses |
| `otLosses` | OT/SO losses |
| `savePct` | Save percentage (decimal, e.g. 0.918) |
| `gaa` | Goals against average |
| `gsax` | Goals saved above expected |
| `shutouts` | Shutout count |
| `highDangerSavePct` | Save % on high-danger shots |
| `rollingSavePct` | Rolling save % (window varies by implementation) |
| `trend` | `up` `down` `stable` or null |
| `restDays` | Days since last game |
| `snapshotDate` | Date this record was captured |

**Sort behavior:**
- `save_pct`: descending (higher is better)
- `gaa`: ascending (lower is better)
- `gsax`: descending (higher is better)
- `wins`: descending

---

## Odds Tools

### `get_odds`

Betting odds for a specific game across available bookmakers.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `game_id` | string | Yes | NHL game ID |
| `bookmaker` | string | No | Bookmaker key, lowercase (e.g. `draftkings` `fanduel` `betmgm` `espnbet`) |
| `snapshot_type` | enum | No | `opening` `current` `closing`. Default: all types returned |

**Return structure:**

```
{
  game_id: "2025020887",
  home_team: "BUF",
  away_team: "TOR",
  game_date: "2026-01-15",
  odds: [
    {
      bookmaker, snapshotType, source,
      mlHome, mlAway, mlHomeProb, mlAwayProb,
      spreadHome, spreadHomeOdds, spreadAway, spreadAwayOdds,
      total, totalOverOdds, totalUnderOdds,
      capturedAt
    }
  ],
  bookmaker_count: 4
}
```

**Coverage:**
- Data: 2019-20 season through 2025-26 season (106,958 records total)
- Pre-2019 games return `odds: []` -- not an error
- Bookmaker keys vary by season; not every game, book, or market has odds coverage

**Documented bookmaker keys:** `draftkings` `fanduel` `betmgm` `espnbet`

---

### `get_line_movement`

Odds snapshots for a game ordered chronologically by bookmaker, showing how lines moved from open to close.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `game_id` | string | Yes | NHL game ID |
| `bookmaker` | string | No | Filter to one bookmaker key (lowercase). Omit for all. |

**Return structure:**

```
{
  game_id: "2025020887",
  home_team: "BUF",
  away_team: "TOR",
  movement: {
    "draftkings": [
      { bookmaker, snapshotType, mlHome, mlAway, spreadHome, total, capturedAt },
      ...ordered chronologically
    ],
    "fanduel": [ ... ],
    ...
  }
}
```

**Notes:**
- Snapshots ordered ascending by `capturedAt` within each bookmaker
- Fields per snapshot: `bookmaker` `snapshotType` `mlHome` `mlAway` `spreadHome` `total` `capturedAt`
- Does not include probability fields (use `get_odds` for implied probabilities)
- Same coverage restriction as `get_odds` (2020-2026 NHL seasons)

---

## Game ID Format

NHL game IDs follow an 8-10 digit format:
- First 4 digits: season start year (e.g. `2025` for 2025-26 season)
- Next 2 digits: game type (`01` preseason, `02` regular, `03` playoffs)
- Last 4 digits: sequential game number

**Examples:**
- `2025020887` -- 2025-26 regular season, game #887
- `2025030211` -- 2025-26 playoffs, series/game encoded in last 4 digits

---

## Team Abbreviation Reference

All 32 active teams with standard abbreviations:

**Eastern Conference -- Atlantic Division**
| Abbrev | Team |
|--------|------|
| BOS | Boston Bruins |
| BUF | Buffalo Sabres |
| DET | Detroit Red Wings |
| FLA | Florida Panthers |
| MTL | Montreal Canadiens |
| OTT | Ottawa Senators |
| TBL | Tampa Bay Lightning |
| TOR | Toronto Maple Leafs |

**Eastern Conference -- Metropolitan Division**
| Abbrev | Team |
|--------|------|
| CAR | Carolina Hurricanes |
| CBJ | Columbus Blue Jackets |
| NJD | New Jersey Devils |
| NYI | New York Islanders |
| NYR | New York Rangers |
| PHI | Philadelphia Flyers |
| PIT | Pittsburgh Penguins |
| WSH | Washington Capitals |

**Western Conference -- Central Division**
| Abbrev | Team |
|--------|------|
| ARI | Utah Hockey Club (note: legacy ARI, now UTA in some systems) |
| CHI | Chicago Blackhawks |
| COL | Colorado Avalanche |
| DAL | Dallas Stars |
| MIN | Minnesota Wild |
| NSH | Nashville Predators |
| STL | St. Louis Blues |
| WPG | Winnipeg Jets |

**Western Conference -- Pacific Division**
| Abbrev | Team |
|--------|------|
| ANA | Anaheim Ducks |
| CGY | Calgary Flames |
| EDM | Edmonton Oilers |
| LAK | Los Angeles Kings |
| SEA | Seattle Kraken |
| SJS | San Jose Sharks |
| UTA | Utah Hockey Club (active abbreviation) |
| VAN | Vancouver Canucks |
| VGK | Vegas Golden Knights |

**Note:** Utah Hockey Club appears as both `ARI` (historical) and `UTA` (current) depending on season. Use `list_teams` to confirm abbreviations for a given season.

---

## Data Coverage Summary

| Dataset | Coverage | Records |
|---------|----------|---------|
| Games | 2010-11 through 2025-26 (16 seasons) | 22,037 |
| Odds | 2019-20 through 2025-26 | 106,958 |
| Players | Active + historical (16 seasons) | 3,021 |
| Goalie stats | Season snapshots | 1,509 |
| Standings | Season snapshots | 494 |
| Teams | 34 total (32 active) | 34 |
