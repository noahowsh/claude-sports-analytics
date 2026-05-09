# PuckAPI Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-10b981.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/skills-28-10b981)](skills/)
[![MCP Tools](https://img.shields.io/badge/MCP_tools-12-10b981)](https://github.com/PuckAPI/mcp)

**Turn Claude into a hockey analyst.** 28 skills that give Claude deep knowledge of NHL analytics, betting methodology, and predictive modeling. Ask a question in plain English, get analysis backed by real methodology.

```
You:    "Build me an expected goals model from shot data"
Claude: [loads xg-model-building skill, walks you through feature selection,
         spatial coordinates, shot type encoding, logistic regression baseline,
         gradient boosting upgrade, calibration, and evaluation]
```

```
You:    "What's the edge on tonight's Sabres game?"
Claude: [loads odds-explorer + edge-detection, pulls live odds from 15+ books,
         compares to your model's probabilities, calculates EV, recommends sizing]
```

No code to install. No dependencies. Clone the repo, and Claude gains the expertise.

## Quick Start

**1. Clone the skills:**

```bash
git clone https://github.com/PuckAPI/claude-sports-analytics.git
```

**2. Connect live NHL data** (optional, 500 free credits):

```bash
claude mcp add puckapi --transport streamable-http "https://mcp.puckapi.com/mcp?key=YOUR_API_KEY"
```

Get your free key at [puckapi.com](https://puckapi.com). Skills also work with your own CSV/JSON files.

**3. Ask Claude anything:**

- "Analyze the Maple Leafs' season -- record, Corsi, xG, special teams"
- "Help me build a game prediction model using the model-building skill"
- "Show me tonight's odds and flag any line movement"
- "Compare McDavid and MacKinnon across every stat"
- "Backtest my betting strategy over the last 3 seasons"

The `dispatch` skill routes your request to the right 2-3 skills automatically. You don't need to memorize skill names.

## Skills

### Data Exploration

| Skill | What It Does |
|-------|-------------|
| `game-lookup` | Find games by date, team, season. Scores, schedule, results. |
| `team-analysis` | Standings, advanced stats, strength of schedule, power rankings. |
| `player-scouting` | Player search, comparison, NHLe translation, career trajectory. |
| `goalie-analysis` | GSAA, xSV%, high-danger save rate, workload tracking. |
| `odds-explorer` | Multi-book odds comparison, best price, line movement detection. |
| `nl-to-query` | Translates natural language questions into structured data queries. |
| `game-preview` | Full pre-game breakdown: matchup, goalies, trends, odds, prediction. |

### Hockey Analytics

| Skill | What It Does |
|-------|-------------|
| `hockey-analytics` | Corsi, Fenwick, xG, PDO, RAPM, WAR -- definitions, context, and proper usage. |
| `xg-model-building` | Build an expected goals model from scratch. Feature engineering through calibration. |

### Modeling Methodology

| Skill | What It Does |
|-------|-------------|
| `feature-engineering` | Rolling windows, lag features, leakage detection, feature catalogs. |
| `walk-forward-validation` | Temporal cross-validation. Refuses k-fold on time-series data. |
| `model-building` | LR to RF to XGBoost ladder. Hard-vote ensemble. Knows when to stop. |
| `elo-engineering` | 5 Elo variants (standard, margin, venue, recency, surface) with tuning. |
| `probability-calibration` | Platt scaling, isotonic regression, reliability diagrams, Brier decomposition. |
| `odds-analysis` | 4 devigging methods: power, multiplicative, Shin, worst-case. |
| `data-pipeline` | GitHub Actions automation, SQLite schema design, credit budgeting. |

### Betting

| Skill | What It Does |
|-------|-------------|
| `edge-detection` | Expected value, Kelly criterion sizing, closing line value tracking. |
| `backtesting` | Walk-forward historical strategy simulation with realistic constraints. |
| `daily-card` | Full slate analysis with edge rankings. Requires your model's probabilities. |
| `bet-tracker` | Log predictions, track P&L, CLV, and test for statistical significance. |
| `visualization` | Calibration plots, edge distributions, player cards, correlation matrices. |

### Advanced Models

| Skill | What It Does |
|-------|-------------|
| `totals-modeling` | Over/under prediction. Pace metrics, Poisson distribution, under bias. |
| `prop-modeling` | Player prop projections. TOI-first architecture, same-game parlay correlation. |
| `war-gar-decomposition` | RAPM ridge regression, component GAR, contract surplus valuation. |
| `playoff-simulation` | Monte Carlo bracket simulation. Series pricing, path probabilities. |

### Workflow

| Skill | What It Does |
|-------|-------------|
| `dispatch` | Routes your request to the right skills automatically. Start here. |
| `ai-hockey-workflow` | 4 patterns for hypothesis testing: explore, model, validate, deploy. |
| `puckapi-tool` | MCP tool router. 12 endpoints, credit tracking, BYOD data support. |

## How It Works

Skills are markdown files, not code. Each one teaches Claude a specific domain -- methodology, anti-patterns, code templates, and decision trees. When you mention a topic, Claude reads the relevant skill and gains that expertise for the conversation.

Every skill includes:

- **When to use / when NOT to use** -- prevents wrong-tool activation
- **Step-by-step methodology** -- the actual analytics workflow, not a summary
- **Code templates** -- Python/pandas, ready to run
- **Anti-patterns** -- mistakes the skill actively prevents (e.g., k-fold on time series)
- **Chaining** -- each skill knows which skill to hand off to next

## Data

| Source | Coverage |
|--------|----------|
| **PuckAPI MCP** | 22,000+ games, 107,000+ odds records, 3,000+ players, 16 seasons (2008-present) |
| **Your own files** | Every skill accepts CSV/JSON. Bring your own data, no credits needed. |

## Reference Docs

| Doc | Contents |
|-----|----------|
| [`hockey-glossary.md`](docs/hockey-glossary.md) | Corsi, Fenwick, xG, PDO, RAPM, WAR |
| [`betting-glossary.md`](docs/betting-glossary.md) | Odds formats, vig, devigging, Kelly, CLV |
| [`tool-routing.md`](docs/tool-routing.md) | Decision tree for picking the right data source |
| [`season-logic.md`](docs/season-logic.md) | NHL season IDs, game ID format, lockout handling |

## Related

- **[PuckAPI MCP Server](https://github.com/PuckAPI/mcp)** -- connect Claude to live NHL data
- **[puckapi.com](https://puckapi.com)** -- API keys, docs, dashboard
- **[puckapi.com/docs](https://puckapi.com/docs)** -- REST API documentation

## License

[MIT](LICENSE)
