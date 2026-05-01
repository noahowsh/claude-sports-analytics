# Chart Templates -- Sports Data HQ Skills

> Level 3 reference. Zero context cost until loaded.
> These are copy-paste matplotlib/seaborn templates. Replace the DATA SECTION placeholders with actual values from the relevant skill output.

---

## 1. Calibration Curve

Source skill: `probability-calibration`
Input: bin midpoints, actual win rates, bin sample sizes

```python
# Calibration Curve -- Built with Sports Data HQ Skills
import matplotlib.pyplot as plt
import matplotlib.patches as patches
import numpy as np

# DATA SECTION -- paste from probability-calibration output
bin_midpoints = [0.05, 0.15, 0.25, 0.35, 0.45, 0.55, 0.65, 0.75, 0.85, 0.95]
actual_rates  = [0.04, 0.13, 0.22, 0.31, 0.47, 0.54, 0.66, 0.74, 0.87, 0.94]
bin_counts    = [12,   28,   41,   55,   67,   71,   58,   43,   29,   11 ]
# END DATA SECTION

fig, ax = plt.subplots(figsize=(8, 7))

# Perfect calibration diagonal
ax.plot([0, 1], [0, 1], 'k--', alpha=0.4, label='Perfect calibration', linewidth=1.5)

# Actual calibration line
ax.plot(bin_midpoints, actual_rates, 'o-', color='#2563EB', linewidth=2.5,
        markersize=7, label='Model calibration', zorder=3)

# Confidence intervals (Wilson interval approximation)
for x, y, n in zip(bin_midpoints, actual_rates, bin_counts):
    if n > 0:
        z = 1.96
        ci = z * np.sqrt((y * (1 - y)) / n)
        ax.plot([x, x], [max(0, y - ci), min(1, y + ci)], color='#2563EB',
                alpha=0.4, linewidth=1.5)

# Histogram of predictions (secondary axis)
ax2 = ax.twinx()
ax2.bar(bin_midpoints, bin_counts, width=0.08, alpha=0.15, color='gray', zorder=1)
ax2.set_ylabel('Predictions per bin', color='gray', fontsize=10)
ax2.tick_params(axis='y', labelcolor='gray')

ax.set_xlabel('Predicted Probability', fontsize=12)
ax.set_ylabel('Actual Win Rate', fontsize=12)
ax.set_title('Model Calibration Curve', fontsize=14, fontweight='bold')
ax.set_xlim(0, 1)
ax.set_ylim(0, 1)
ax.legend(loc='upper left', fontsize=10)
ax.grid(True, alpha=0.3)

fig.text(0.99, 0.01, 'Built with Sports Data HQ Skills',
         ha='right', va='bottom', fontsize=8, color='gray')

plt.tight_layout()
plt.savefig('calibration-curve.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## 2. Equity Curve with Drawdown Bands

Source skill: `backtesting` or `bet-tracker`
Input: sequential P&L in units (one value per bet)

```python
# Equity Curve -- Built with Sports Data HQ Skills
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
import numpy as np

# DATA SECTION -- paste pnl_units column from bet-tracker or backtesting
pnl_units = [0.91, -1.0, 0.91, 0.91, -1.0, -1.0, 0.91, 0.91, -1.0, 0.91]
# END DATA SECTION

cumulative = np.cumsum(pnl_units)
peak = np.maximum.accumulate(cumulative)
drawdown = cumulative - peak
bet_nums = np.arange(1, len(pnl_units) + 1)

fig = plt.figure(figsize=(12, 7))
gs = gridspec.GridSpec(2, 1, height_ratios=[3, 1], hspace=0.05)

# Equity curve (top panel)
ax1 = fig.add_subplot(gs[0])
ax1.plot(bet_nums, cumulative, color='#16A34A', linewidth=2.5, label='Equity', zorder=3)
ax1.axhline(y=0, color='black', linewidth=1, alpha=0.5, linestyle='--')
ax1.fill_between(bet_nums, cumulative, 0,
                 where=(cumulative >= 0), alpha=0.1, color='#16A34A')
ax1.fill_between(bet_nums, cumulative, 0,
                 where=(cumulative < 0), alpha=0.1, color='#DC2626')
ax1.set_ylabel('Cumulative P&L (units)', fontsize=11)
ax1.set_title('Equity Curve', fontsize=14, fontweight='bold')
ax1.grid(True, alpha=0.3)
ax1.legend(fontsize=10)
ax1.set_xticklabels([])

# Drawdown (bottom panel)
ax2 = fig.add_subplot(gs[1])
ax2.fill_between(bet_nums, drawdown, 0, alpha=0.6, color='#DC2626')
ax2.plot(bet_nums, drawdown, color='#DC2626', linewidth=1)
ax2.axhline(y=0, color='black', linewidth=1, alpha=0.5)
ax2.set_ylabel('Drawdown', fontsize=10)
ax2.set_xlabel('Bet Number', fontsize=11)
ax2.grid(True, alpha=0.3)

# Annotate key stats
max_dd = drawdown.min()
total_return = cumulative[-1]
ax1.text(0.02, 0.95, f'Total: {total_return:+.2f}u | Max DD: {max_dd:.2f}u',
         transform=ax1.transAxes, fontsize=10, va='top',
         bbox=dict(boxstyle='round', facecolor='white', alpha=0.8))

fig.text(0.99, 0.01, 'Built with Sports Data HQ Skills',
         ha='right', va='bottom', fontsize=8, color='gray')

plt.savefig('equity-curve.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## 3. Team Comparison Radar

Source skill: `team-analysis` or `game-preview`
Input: two teams, 6-8 metrics each, league averages

```python
# Team Comparison Radar -- Built with Sports Data HQ Skills
import matplotlib.pyplot as plt
import numpy as np

# DATA SECTION -- paste from team-analysis or game-preview
team_a_name = 'Buffalo Sabres'
team_b_name = 'Toronto Maple Leafs'
metrics = ['CF%', 'xGF%', 'PP%', 'PK%', 'GF/GP', 'GA/GP']
# Values normalized 0-100 (where 50 = league average)
team_a_values = [53, 55, 58, 62, 52, 48]  # CF%=53.2, xGF%=55.1, etc.
team_b_values = [48, 47, 62, 55, 61, 56]
league_avg    = [50, 50, 50, 50, 50, 50]
# END DATA SECTION

N = len(metrics)
angles = np.linspace(0, 2 * np.pi, N, endpoint=False).tolist()
angles += angles[:1]  # close the polygon

team_a_values += team_a_values[:1]
team_b_values += team_b_values[:1]
league_avg    += league_avg[:1]

fig, ax = plt.subplots(figsize=(8, 8), subplot_kw=dict(polar=True))

ax.plot(angles, team_a_values, 'o-', linewidth=2, color='#2563EB', label=team_a_name)
ax.fill(angles, team_a_values, alpha=0.1, color='#2563EB')

ax.plot(angles, team_b_values, 'o-', linewidth=2, color='#DC2626', label=team_b_name)
ax.fill(angles, team_b_values, alpha=0.1, color='#DC2626')

ax.plot(angles, league_avg, '--', linewidth=1, color='gray', alpha=0.5, label='League avg')

ax.set_xticks(angles[:-1])
ax.set_xticklabels(metrics, fontsize=12)
ax.set_ylim(0, 100)
ax.set_yticks([25, 50, 75])
ax.set_yticklabels(['25', 'Avg', '75'], fontsize=8, color='gray')
ax.grid(True, alpha=0.3)

ax.set_title(f'{team_a_name} vs {team_b_name}', fontsize=13,
             fontweight='bold', pad=20)
ax.legend(loc='upper right', bbox_to_anchor=(1.3, 1.1), fontsize=10)

fig.text(0.99, 0.01, 'Built with Sports Data HQ Skills',
         ha='right', va='bottom', fontsize=8, color='gray')

plt.tight_layout()
plt.savefig('team-radar.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## 4. Matchup Card

Source skill: `game-preview`
Input: team names, key stats, goalie names/stats, odds

```python
# Matchup Card -- Built with Sports Data HQ Skills
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
import numpy as np

# DATA SECTION -- paste from game-preview output
game_date = '2026-05-01'
away_team  = 'Buffalo Sabres'
home_team  = 'Toronto Maple Leafs'
away_abbr  = 'BUF'
home_abbr  = 'TOR'
game_time  = '7:00 PM ET'

metrics = ['CF% (5v5)', 'xGF% (5v5)', 'PP%', 'PK%', 'GF/GP', 'GA/GP']
away_vals = [53.2, 55.1, 22.1, 83.4, 3.42, 2.91]
home_vals = [47.8, 46.9, 24.3, 80.1, 3.61, 3.12]
# Higher is better for all except GA/GP
higher_is_better = [True, True, True, True, True, False]

away_goalie = 'Ukko-Pekka Luukkonen'
away_sv  = '.918'
away_gaa = '2.64'
home_goalie = 'Joseph Woll'
home_sv  = '.911'
home_gaa = '2.88'

away_ml = '+120'
home_ml = '-140'
total   = '6.0'
# END DATA SECTION

fig, axes = plt.subplots(1, 2, figsize=(14, 7))
fig.patch.set_facecolor('#F8FAFC')

def draw_team_panel(ax, team_name, abbr, goalie, sv_pct, gaa,
                    vals, other_vals, higher_better, ml_odds, side):
    ax.set_facecolor('#F8FAFC')
    ax.set_xlim(0, 1)
    ax.set_ylim(0, 1)
    ax.axis('off')

    color = '#2563EB' if side == 'away' else '#DC2626'

    ax.text(0.5, 0.95, abbr, ha='center', va='top',
            fontsize=32, fontweight='bold', color=color)
    ax.text(0.5, 0.88, team_name, ha='center', va='top',
            fontsize=11, color='#374151')
    ax.text(0.5, 0.82, f'ML: {ml_odds}', ha='center', va='top',
            fontsize=13, fontweight='bold', color='#111827')

    ax.axhline(0.79, color='#E5E7EB', linewidth=1)

    ax.text(0.5, 0.76, 'Starting Goalie', ha='center', va='top',
            fontsize=9, color='#6B7280')
    ax.text(0.5, 0.72, goalie, ha='center', va='top',
            fontsize=11, fontweight='bold', color='#111827')
    ax.text(0.5, 0.67, f'SV% {sv_pct} | GAA {gaa}', ha='center', va='top',
            fontsize=10, color='#374151')

    ax.axhline(0.64, color='#E5E7EB', linewidth=1)

    y = 0.60
    for i, (metric, val, other, hib) in enumerate(
            zip(metrics, vals, other_vals, higher_better)):
        better = (val > other) if hib else (val < other)
        val_color = '#16A34A' if better else '#DC2626'
        ax.text(0.1, y, metric, ha='left', va='center', fontsize=9, color='#6B7280')
        ax.text(0.9, y, f'{val}', ha='right', va='center',
                fontsize=11, fontweight='bold', color=val_color)
        y -= 0.08

draw_team_panel(axes[0], away_team, away_abbr, away_goalie, away_sv, away_gaa,
                away_vals, home_vals, higher_is_better, away_ml, 'away')
draw_team_panel(axes[1], home_team, home_abbr, home_goalie, home_sv, home_gaa,
                home_vals, away_vals, higher_is_better, home_ml, 'home')

fig.suptitle(f'{away_abbr} @ {home_abbr}  |  {game_date}  |  {game_time}  |  O/U {total}',
             fontsize=13, fontweight='bold', y=0.02, color='#374151')

fig.text(0.99, 0.01, 'Built with Sports Data HQ Skills',
         ha='right', va='bottom', fontsize=8, color='gray')

plt.tight_layout(rect=[0, 0.05, 1, 1])
plt.savefig('matchup-card.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## 5. Prediction Confidence Histogram

Source skill: `model-building`
Input: list of model probabilities (one per prediction)

```python
# Prediction Confidence Histogram -- Built with Sports Data HQ Skills
import matplotlib.pyplot as plt
import numpy as np

# DATA SECTION -- paste model probability output (home team win probability)
probabilities = [0.42, 0.61, 0.55, 0.38, 0.72, 0.49, 0.53, 0.67, 0.44, 0.58]
# END DATA SECTION

fig, ax = plt.subplots(figsize=(10, 6))

bins = np.linspace(0, 1, 21)
colors = ['#DC2626' if b < 0.5 else '#16A34A' for b in bins[:-1]]

n, bins_out, patches = ax.hist(probabilities, bins=bins, edgecolor='white',
                                linewidth=0.5)
for patch, color in zip(patches, colors):
    patch.set_facecolor(color)
    patch.set_alpha(0.7)

ax.axvline(x=0.5, color='black', linewidth=2, linestyle='--', alpha=0.6,
           label='50% threshold')
ax.axvline(x=np.mean(probabilities), color='#7C3AED', linewidth=2,
           linestyle='-', label=f'Mean: {np.mean(probabilities):.3f}')

ax.set_xlabel('Predicted Probability (Home Team Win)', fontsize=12)
ax.set_ylabel('Number of Predictions', fontsize=12)
ax.set_title('Model Prediction Distribution', fontsize=14, fontweight='bold')
ax.legend(fontsize=10)
ax.grid(True, alpha=0.3, axis='y')

# Annotation
below_50 = sum(1 for p in probabilities if p < 0.5)
above_50 = sum(1 for p in probabilities if p >= 0.5)
ax.text(0.02, 0.95, f'Favors Away: {below_50} ({below_50/len(probabilities):.0%})',
        transform=ax.transAxes, fontsize=10, va='top', color='#DC2626')
ax.text(0.98, 0.95, f'Favors Home: {above_50} ({above_50/len(probabilities):.0%})',
        transform=ax.transAxes, fontsize=10, va='top', ha='right', color='#16A34A')

fig.text(0.99, 0.01, 'Built with Sports Data HQ Skills',
         ha='right', va='bottom', fontsize=8, color='gray')

plt.tight_layout()
plt.savefig('prediction-histogram.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## 6. Season Performance Timeline

Source skill: `bet-tracker` or `backtesting`
Input: dates, rolling metric values (accuracy, ROI, CLV)

```python
# Season Performance Timeline -- Built with Sports Data HQ Skills
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd

# DATA SECTION -- paste from bet-tracker or backtesting
dates = pd.date_range('2025-10-01', periods=30, freq='3D')
roi_rolling = [0.01, 0.02, -0.01, 0.03, 0.05, 0.04, 0.06, 0.03, 0.08,
               0.07, 0.09, 0.06, 0.10, 0.08, 0.11, 0.09, 0.12, 0.10,
               0.08, 0.07, 0.09, 0.11, 0.10, 0.12, 0.14, 0.13, 0.15,
               0.14, 0.16, 0.15]
window_size = 20  # rolling window (number of bets)
breakeven_rate = 0.0  # 0% ROI = breakeven
# END DATA SECTION

roi = np.array(roi_rolling)
std = np.std(roi) * np.ones(len(roi))

fig, ax = plt.subplots(figsize=(12, 6))

ax.plot(dates, roi, color='#2563EB', linewidth=2.5, label=f'Rolling ROI ({window_size}-bet)')
ax.fill_between(dates, roi - std, roi + std, alpha=0.15, color='#2563EB')
ax.axhline(y=breakeven_rate, color='black', linewidth=1.5,
           linestyle='--', alpha=0.6, label='Breakeven')
ax.fill_between(dates, roi, breakeven_rate,
                where=(roi >= breakeven_rate), alpha=0.1, color='#16A34A')
ax.fill_between(dates, roi, breakeven_rate,
                where=(roi < breakeven_rate), alpha=0.1, color='#DC2626')

ax.set_xlabel('Date', fontsize=12)
ax.set_ylabel('Rolling ROI', fontsize=12)
ax.set_title('Season Performance Timeline', fontsize=14, fontweight='bold')
ax.yaxis.set_major_formatter(plt.FuncFormatter(lambda y, _: f'{y:.1%}'))
ax.legend(fontsize=10)
ax.grid(True, alpha=0.3)

fig.text(0.99, 0.01, 'Built with Sports Data HQ Skills',
         ha='right', va='bottom', fontsize=8, color='gray')

plt.tight_layout()
plt.savefig('performance-timeline.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## 7. Player Card (WAR/GAR Component Radar)

Source skill: `war-gar-decomposition`
Input: player name, team, component values, comparison baseline

```python
# Player Card -- Built with Sports Data HQ Skills
import matplotlib.pyplot as plt
import numpy as np

# DATA SECTION -- paste from war-gar-decomposition output
player_name = 'Tage Thompson'
player_team = 'Buffalo Sabres'
position    = 'C'
season      = '2025-26'

components  = ['Offense\n5v5', 'Defense\n5v5', 'PP\nOffense', 'PK\nDefense',
               'Faceoffs', 'Penalty\nDraw']
player_vals = [4.2, 0.8, 1.9, -0.1, 0.3, 0.4]   # runs above average
league_avg  = [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]     # zero = average by definition
top_player  = [3.0, 1.5, 2.0, 0.5, 0.8, 0.6]     # top-10 player baseline
total_war   = sum(player_vals)
# END DATA SECTION

# Normalize for radar: shift so average = 50, max reasonable = 100
max_val = 5.0
norm = lambda v: 50 + (v / max_val) * 50

player_norm = [norm(v) for v in player_vals]
top_norm    = [norm(v) for v in top_player]
avg_norm    = [50.0] * len(components)

N = len(components)
angles = np.linspace(0, 2 * np.pi, N, endpoint=False).tolist()
angles += angles[:1]

player_norm += player_norm[:1]
top_norm    += top_norm[:1]
avg_norm    += avg_norm[:1]

fig, ax = plt.subplots(figsize=(8, 8), subplot_kw=dict(polar=True))

ax.plot(angles, player_norm, 'o-', linewidth=2.5, color='#2563EB', label=player_name)
ax.fill(angles, player_norm, alpha=0.15, color='#2563EB')

ax.plot(angles, top_norm, '--', linewidth=1.5, color='#F59E0B', alpha=0.7, label='Top-10 baseline')
ax.plot(angles, avg_norm, ':', linewidth=1, color='gray', alpha=0.5, label='League average')

ax.set_xticks(angles[:-1])
ax.set_xticklabels(components, fontsize=10)
ax.set_ylim(0, 100)
ax.set_yticks([25, 50, 75])
ax.set_yticklabels(['', 'Avg', ''], fontsize=8, color='gray')
ax.grid(True, alpha=0.3)

ax.set_title(f'{player_name} — {player_team}\n{position} | {season} | WAR: {total_war:+.1f}',
             fontsize=12, fontweight='bold', pad=20)
ax.legend(loc='upper right', bbox_to_anchor=(1.3, 1.1), fontsize=9)

fig.text(0.99, 0.01, 'Built with Sports Data HQ Skills',
         ha='right', va='bottom', fontsize=8, color='gray')

plt.tight_layout()
plt.savefig('player-card.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## ASCII Chart Templates

For terminal-only output (no Python required).

### Horizontal Bar Comparison
```
[Away Team] vs [Home Team] -- Key Stats
========================================
CF% (5v5):  BUF [██████████] 53.2%
            TOR [████████  ] 47.8%

PP%:        BUF [████████  ] 22.1%
            TOR [█████████ ] 24.3%

xGF%:       BUF [██████████] 55.1%
            TOR [█████████ ] 46.9%
========================================
Green = better vs league avg (50.0%)
```

### Text Equity Curve
```
Equity (units)
+4 |                          *
+3 |                    *   *
+2 |             *    *
+1 |      *   *
 0 |*  *
-1 |
   +----------------------------> Bet #
   1    5    10   15   20   25
Running: +3.8u | Max DD: -1.2u
```

### Win Rate Significance Table
```
Current Record: 31W-22L (58.5% win rate)
Breakeven: 52.4% at avg -110 odds
Sample: 53 bets

Significance:
  p-value: 0.041  [SIGNIFICANT at 95%]
  Confidence interval: 45.1% -- 71.9%
  Min bets for 95% CI at 58.5%: ~160

Status: EDGE LIKELY -- continue tracking
```
