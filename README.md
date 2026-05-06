# PuckAPI Skills

Hockey analytics skills for Claude Code. 28 skills covering NHL analytics, betting methodology, and model building.

Free skills. Paid data via [PuckAPI](https://puckapi.com) MCP server.

## Install

Clone this repo:

```bash
git clone https://github.com/sports-data-hq/hockey-skills.git
```

Connect the PuckAPI MCP server for live data:

```bash
claude mcp add puckapi https://mcp.puckapi.com/mcp --header "Authorization: Bearer YOUR_API_KEY"
```

Get your free API key at [puckapi.com/signup](https://puckapi.com/signup).

Skills work without the MCP server using your own CSV/JSON files.

## How Skills Work

Skills are markdown files that Claude reads as context -- they are not executable code. There is nothing to install, compile, or run. The skills teach Claude how to use the 12 PuckAPI tools effectively, providing domain knowledge, methodology, anti-patterns, and code templates.

1. **Clone the repo** (above). That gives you the skill files on disk.
2. **Add the MCP server** with `claude mcp add` (above). This connects Claude to the PuckAPI data endpoints.
3. **Reference a skill by name.** Ask Claude to "load the dispatch skill" or mention any skill (e.g., "use the team-analysis skill"). Claude reads the skill file and gains the domain expertise it contains.
4. **The dispatch skill routes automatically.** If you describe what you want without naming a skill, dispatch picks the right 2-3 skills for your request.

## Quick Start

Once installed, try these first prompts:

- "Load the dispatch skill and show me tonight's NHL games with odds"
- "Help me build an expected goals model using the xg-model-building skill"
- "Analyze the Maple Leafs' season using the team-analysis skill"

## What's In Here

### Data Exploration (7 skills)
| Skill | What It Does |
|-------|-------------|
| `game-lookup` | Find games by date, team, season. Scores and schedule. |
| `team-analysis` | Standings, stats, strength of schedule, rankings. |
| `player-scouting` | Player search, stats, comparison, NHLe translation. |
| `goalie-analysis` | GSAA, xSV%, high-danger save%, workload tracking. |
| `odds-explorer` | Multi-book odds comparison, line movement detection. |
| `nl-to-query` | Natural language to structured data queries. |
| `game-preview` | Full game preview: matchup, goalie, trends, odds. ~15 credits. |

### Hockey Analytics (2 skills)
| Skill | What It Does |
|-------|-------------|
| `hockey-analytics` | Corsi, Fenwick, xG, PDO, RAPM, WAR definitions and context. |
| `xg-model-building` | Build an expected goals model from play-by-play shot data. |

### Methodology (7 skills)
| Skill | What It Does |
|-------|-------------|
| `feature-engineering` | Rolling windows, shift(1) leakage detection, feature catalogs. |
| `walk-forward-validation` | Temporal cross-validation. Refuses k-fold on time series. |
| `model-building` | LR to RF to XGBoost ladder. Hard-vote ensemble. |
| `elo-engineering` | 5 Elo variants with per-sport tuning configs. |
| `probability-calibration` | Platt scaling, isotonic regression, Brier score decomposition. |
| `odds-analysis` | 4 devigging methods (power, multiplicative, Shin, worst-case). |
| `data-pipeline` | GitHub Actions automation, SQLite schema, credit budgeting. |

### Betting (5 skills)
| Skill | What It Does |
|-------|-------------|
| `edge-detection` | EV calculation, Kelly criterion sizing, CLV tracking. |
| `backtesting` | Walk-forward historical strategy simulation. |
| `daily-card` | Full slate analysis with edge rankings. Requires your model. |
| `bet-tracker` | Log predictions, track CLV, test statistical significance. |
| `visualization` | Calibration plots, edge charts, player cards, correlation matrices. |

### Sport-Specific Models (4 skills)
| Skill | What It Does |
|-------|-------------|
| `totals-modeling` | Over/under prediction. Pace metrics, Poisson, under bias. |
| `prop-modeling` | Player prop projections. TOI-first architecture, SGP correlation. |
| `war-gar-decomposition` | RAPM ridge regression, component GAR, contract surplus. |
| `playoff-simulation` | Monte Carlo season/bracket simulation. |

### Workflow (2 skills)
| Skill | What It Does |
|-------|-------------|
| `ai-sports-workflow` | 4 workflow patterns for hypothesis testing with Claude. |
| `dispatch` | Routes requests to the right 2-3 skills automatically. |

### Infrastructure (1 skill)
| Skill | What It Does |
|-------|-------------|
| `puckapi-tool` | MCP tool router. 12 endpoints, credit tracking, BYOD support. |

## Data Sources

**PuckAPI MCP** (default): 22,000+ NHL games (2008-present), 107,000+ odds records (2020-2026 NHL seasons), 3,000+ players, 34 franchises (32 active + 2 historical). Pay-as-you-go credits.

**Your own data**: Every skill accepts CSV/JSON. No credits consumed.

## Credit Costs

| Endpoint | Credits |
|----------|---------|
| List teams | 1 |
| Schedule, search players, standings | 2 |
| Games, player stats, goalie stats, team stats | 5 |
| Game detail, head-to-head, odds snapshot | 10 |
| Line movement | 25 |

A full game preview costs ~34 credits. A full season backtest with odds can cost 12,000+. Skills warn you before expensive operations.

## Docs

- `docs/hockey-glossary.md` -- Corsi, Fenwick, xG, PDO, RAPM definitions
- `docs/betting-glossary.md` -- Odds formats, vig, devigging, Kelly, CLV
- `docs/tool-routing.md` -- Decision tree for picking data sources
- `docs/season-logic.md` -- NHL season resolution rules, game ID format

## How Skills Work

Skills are markdown files that teach Claude domain knowledge. When you ask a question, the `dispatch` skill routes to the right 2-3 skills based on your request. Each skill includes:

- **When to Use / When NOT to Use** -- prevents wrong-skill activation
- **Step-by-step methodology** -- the actual analytics workflow
- **Code templates** -- Python/pandas ready to run
- **Anti-patterns** -- common mistakes and how to avoid them
- **What to Do Next** -- chains to the logical next skill

Skills never make bets for you. They help you build, validate, and track your own models.

## Requirements

- Claude Code with plugin support
- Python 3.10+ (for code templates)
- PuckAPI API key (for MCP data access)

## License

MIT
