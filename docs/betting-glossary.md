# Betting & Odds Glossary

Quick reference for betting terms used across Sports Data HQ Skills.

## Odds Formats

| Term | Definition |
|------|-----------|
| **American Odds** | +150 means $150 profit on $100 bet. -180 means bet $180 to profit $100. Standard in US sportsbooks. |
| **Decimal Odds** | 2.50 means $2.50 total return per $1 bet ($1.50 profit). Standard internationally. |
| **Fractional Odds** | 3/2 means $3 profit per $2 bet. Standard in UK. |
| **Implied Probability** | The probability embedded in the odds (includes vig). -180 implies ~64.3% (with vig). |

## Vig & Devigging

| Term | Definition |
|------|-----------|
| **Vig (Vigorish)** | The sportsbook's margin built into odds. Also called juice, overround, or margin. A "fair" -110/-110 line has ~4.5% vig. |
| **Devig** | Removing the vig from odds to estimate true probability. Four methods: power, multiplicative, Shin, worst-case. |
| **No-vig Line** | The theoretical odds without sportsbook margin. What a "fair" market would show. |
| **Shin Method** | Devigging method that accounts for insider information. Gold standard for accuracy. |
| **Power Method** | Devigging that assumes equal overround per outcome. Simple but less accurate on lopsided lines. |
| **Overround** | Total implied probability across all outcomes. A fair market = 100%. Typical NHL ML market = 104-106%. |

## Edge & Value

| Term | Definition |
|------|-----------|
| **EV (Expected Value)** | (Model probability x payout) - 1. Positive EV = profitable long-term. |
| **Edge** | The difference between your estimated probability and the market's implied probability. |
| **+EV** | Positive expected value. A bet where your model says you have an advantage. |
| **CLV (Closing Line Value)** | Did the line move toward your number by game time? The gold standard for measuring bettor skill. Positive CLV = market agrees your bet had value. |
| **Sharp** | Professional bettors whose action moves lines. "Sharp money" = bets from winning players. |
| **Public / Square** | Recreational bettors. "Public money" = bets from losing players. |
| **Steam Move** | Rapid line movement from coordinated sharp action across multiple books simultaneously. |

## Bankroll Management

| Term | Definition |
|------|-----------|
| **Kelly Criterion** | f* = (bp - q) / b. Optimal bet fraction to maximize long-term growth. b = decimal odds - 1, p = win probability, q = 1-p. |
| **Fractional Kelly** | Betting a fraction (typically 1/4) of full Kelly to reduce variance. Recommended over full Kelly. |
| **Unit** | Standard bet size, typically 1-2% of bankroll. |
| **Bankroll** | Total capital allocated to betting. Separate from living expenses. |
| **Risk of Ruin** | Probability of losing entire bankroll. Lower with fractional Kelly, higher with full Kelly. |
| **Drawdown** | Peak-to-trough decline in bankroll. Maximum drawdown measures worst historical decline. |
| **ROI** | Return on investment. Total profit / total amount wagered. |

## Bet Types

| Term | Definition |
|------|-----------|
| **Moneyline (ML)** | Bet on which team wins outright. |
| **Puck Line / Spread** | Team must win by N goals (NHL standard: -1.5 / +1.5). |
| **Total (O/U)** | Bet on combined score being over or under a number. |
| **Prop** | Bet on individual player performance (goals, assists, SOG, saves). |
| **Same-Game Parlay (SGP)** | Multiple bets from the same game combined. Legs are correlated -- don't multiply probabilities independently. |
| **Futures** | Long-term bets (division winner, Stanley Cup, MVP). |

## Statistical Terms

| Term | Definition |
|------|-----------|
| **Brier Score** | Measures calibration quality. Lower = better. Decomposes into reliability + resolution + uncertainty. |
| **Log Loss** | Penalizes confident wrong predictions heavily. Used for model training. |
| **Calibration** | Whether predicted probabilities match actual outcomes. "60% predictions should win 60% of the time." |
| **Statistical Significance** | Whether a result is unlikely due to chance alone. p < 0.05 is conventional threshold. 200+ bets needed at typical edges. |
| **Regression to the Mean** | Extreme results tend to move toward the average over time. High PDO regresses. Hot streaks cool. |
