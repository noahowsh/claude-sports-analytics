# Hockey Analytics Glossary

Quick reference for hockey analytics terms used across PuckAPI Skills.

## Possession & Shot Metrics

| Term | Definition |
|------|-----------|
| **Corsi (CF/CA/CF%)** | All shot attempts (shots on goal + missed + blocked). CF% = CF / (CF + CA). The baseline possession metric at 5v5. |
| **Fenwick (FF/FA/FF%)** | Unblocked shot attempts (shots on goal + missed). Excludes blocked shots. Slightly more predictive than Corsi in some contexts. |
| **Shots on Goal (SOG)** | Shots that reach the goalie or score. Subset of Corsi. |
| **High-Danger Chances (HDC)** | Shots from the inner slot / crease area. Zone-based definition, not model-derived. More predictive than raw shot volume. |
| **Scoring Chances (SC)** | Shots from areas with historically elevated goal rates. Broader than HDC. |
| **Shot Attempts** | Same as Corsi. All shots: on goal, missed, and blocked. |

## Expected Goals & Goal Metrics

| Term | Definition |
|------|-----------|
| **xG (Expected Goals)** | Model-estimated probability of a shot becoming a goal, based on distance, angle, type, game state, etc. Summed across shots. |
| **xGF / xGA** | Expected goals for / against. Sum of xG values for all shots by/against a team. |
| **xGF%** | xGF / (xGF + xGA). Expected goals share. The best single-number team quality metric at 5v5. |
| **GF / GA** | Actual goals for / against. |
| **GF%** | GF / (GF + GA). Actual goal share. |
| **HDGF%** | High-danger goal percentage. Goals from high-danger areas. |

## Goalie Metrics

| Term | Definition |
|------|-----------|
| **SV%** | Save percentage. Saves / shots on goal. Noisy in small samples. |
| **GAA** | Goals against average per 60 minutes. Affected by team defense. |
| **GSAA** | Goals saved above average. (League avg SV% x shots faced) - goals allowed. Positive = better than average. |
| **xSV%** | Expected save percentage given shot quality faced. Compares to actual SV% to isolate goalie skill. |
| **HDSA%** | High-danger save percentage. Most predictive single-season goalie stat. |
| **QS%** | Quality start percentage. Starts with SV% > .900 (approximately). |

## Player Evaluation

| Term | Definition |
|------|-----------|
| **RAPM** | Regularized Adjusted Plus-Minus. Ridge regression isolating player impact from teammates/opponents at shift level. |
| **WAR** | Wins Above Replacement. Total player value converted to wins using Pythagorean expectation. |
| **GAR** | Goals Above Replacement. Component-level player value (EV offense, EV defense, PP, PK, penalties). WAR is GAR converted to wins. |
| **NHLe** | NHL Equivalency. Translation factor converting stats from other leagues to NHL-equivalent production. |
| **TOI** | Time on ice. Minutes played. Foundation for all rate stats. |
| **P/60** | Points per 60 minutes of ice time. Rate-adjusted production. |

## Game State & Situation

| Term | Definition |
|------|-----------|
| **5v5 / Even Strength** | Both teams at full strength (5 skaters + goalie each). ~60-65% of game time. Most analytics focus here. |
| **PP / Power Play** | Team has a man advantage (5v4, 5v3). |
| **PK / Penalty Kill** | Team is shorthanded (4v5, 3v5). |
| **EN / Empty Net** | Goalie pulled. Excluded from most rate stats. |
| **Score State** | Leading, trailing, or tied. Affects shot rates significantly (trailing teams shoot more). |

## Standings & Records

| Term | Definition |
|------|-----------|
| **P%** | Points percentage. Points earned / (games played x 2). Better than raw points for comparing teams with different GP. |
| **ROW** | Regulation + Overtime Wins. Excludes shootout wins. NHL tiebreaker. |
| **RW** | Regulation Wins only. First NHL tiebreaker since 2019-20. |
| **PDO** | Shooting% + Save% at 5v5. Expected value is 100 (or 1.000). Teams far from 100 are experiencing luck that will regress. |

## Betting Terms

See `betting-glossary.md` for odds, vig, devigging, and bankroll terms.
