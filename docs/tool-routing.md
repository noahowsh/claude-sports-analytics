# Data Tool Routing Guide

When you need data, use this guide to pick the right source.

## Default: PuckAPI MCP (`puckapi-tool`)

Use for ALL NHL data needs unless the user specifically brings their own data.

| Data Need | Endpoint | Credits |
|-----------|----------|---------|
| Team list | `list_teams` | 1 |
| Upcoming schedule | `get_schedule` | 2 |
| Player search | `search_players` | 2 |
| Standings | `get_standings` | 2 |
| Game scores, results | `get_games` | 5 |
| Player bio + goalie stats | `get_player_stats` | 5 |
| Team season stats | `get_team_stats` | 5 |
| Goalie season stats | `get_goalie_stats` | 5 |
| Single game detail (with odds + goalies) | `get_game_detail` | 10 |
| Head-to-head history | `get_head_to_head` | 10 |
| Current odds (ML, spread, total) | `get_odds` | 10 |
| Line movement history | `get_line_movement` | 25 |

## User's Own Data (BYOD)

When a user provides their own CSV, JSON, or database:
1. Work with the file directly -- no MCP calls needed
2. Verify required columns exist before proceeding
3. Check date formats (ISO 8601 preferred)
4. No credits consumed

## Decision Tree

```
Need NHL data?
  Yes -> puckapi-tool (default)
  
User has their own data file?
  Yes -> Work with file directly, no credits

Need odds for backtesting?
  Yes -> puckapi-tool get_odds (10 credits/query, 106K+ records available)

Need line movement?
  Yes -> puckapi-tool get_line_movement (25 credits, most expensive)
```
