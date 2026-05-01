# Data Tool Routing Guide

When you need data, use this guide to pick the right source.

## Default: Sports Data HQ MCP (`sportsdatahq-tool`)

Use for ALL NHL data needs unless the user specifically brings their own data.

| Data Need | Endpoint | Credits |
|-----------|----------|---------|
| Game scores, results, schedule | `get_games`, `get_schedule` | 1 |
| Single game detail | `get_game_detail` | 1 |
| Head-to-head history | `get_head_to_head` | 1 |
| Standings | `get_standings` | 1 |
| Team season stats | `get_team_stats` | 1 |
| Team list | `list_teams` | 1 |
| Player search | `search_players` | 1 |
| Player season stats | `get_player_stats` | 1 |
| Goalie season stats | `get_goalie_stats` | 1 |
| Current odds (ML, spread, total) | `get_odds` | 10 |
| Line movement history | `get_line_movement` | 25 |

## NFL Data: nflfastR (`nfl_data_py`)

Use for ALL NFL play-by-play analysis. Free. Not an MCP tool -- it's a Python package.

```python
pip install nfl_data_py
import nfl_data_py as nfl
pbp = nfl.import_pbp_data([2023, 2024, 2025])
```

372 columns pre-computed including EPA, CPOE, win probability, air yards, YAC. Back to 1999.

## User's Own Data (BYOD)

When a user provides their own CSV, JSON, or database:
1. Work with the file directly -- no MCP calls needed
2. Verify required columns exist before proceeding
3. Check date formats (ISO 8601 preferred)
4. No credits consumed

## Decision Tree

```
Need NHL data?
  Yes -> sportsdatahq-tool (default)
  
Need NFL play-by-play?
  Yes -> nfl_data_py (free Python package)

User has their own data file?
  Yes -> Work with file directly, no credits

Need odds for backtesting?
  Yes -> sportsdatahq-tool get_odds (10 credits/query, 106K+ records available)

Need line movement?
  Yes -> sportsdatahq-tool get_line_movement (25 credits, most expensive)
```
