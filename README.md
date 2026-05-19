[README.md](https://github.com/user-attachments/files/28030045/README.md)
# NFL Spread Analyzer & Betting Market Research
### Built by Shaan Jagtiani | Python, Pandas, The Odds API, SciPy

---

## About This Project

This is my first Sports Trading project, built over the summer of 2026 as part of my goal of breaking into sports trading. I recently graduated from the University of Michigan with a degree in Sports Management and have been self-teaching Python and data analysis to build the technical skills needed for a career in sports betting markets.

I chose this project specifically because it sits at the intersection of what I already understand: how betting markets work and how books balance their books, and what I needed to prove I could do: pull real data, clean it, analyze it, and draw statistically tested conclusions from it.

Everything in this notebook was built from scratch using live API data from The Odds API and a historical NFL dataset covering 35 years of games. The analysis covers live book comparison, juice and hold percentage calculation, and three independently tested market inefficiency findings backed by binomial significance testing.
---

## Project Overview

**Part 1: Live Market Analysis:** Pulls current NFL spread lines from multiple sportsbooks via API and compares juice and hold percentages across books to identify pricing inefficiencies.

**Part 2: Historical Market Analysis:** Uses 9,455 NFL games (1990–2024) to test whether structural patterns exist in how the market prices favorites, rest advantages, and scheduling disadvantages.

**Core question:** Is the NFL betting market efficient, and if not, where does it break down?

---

## Setup

### Requirements
```bash
pip install requests pandas matplotlib seaborn scipy openpyxl jinja2
```

### API Key
Sign up for a free key at [the-odds-api.com](https://the-odds-api.com) and replace `YOUR_API_KEY_HERE` in Cell 1.

### Historical Data
Download `spreadspoke_scores.csv` from [Kaggle](https://www.kaggle.com/datasets/tobycrabtree/nfl-scores-and-betting-data) and place it in the same folder as the notebook.

---

## Part 1: Live Market Analysis

### Cell 1 — API Connection
Connects to The Odds API and pulls current NFL spread lines from all major US sportsbooks. The API returned **75 NFL games** with live spread lines across DraftKings, FanDuel, BetMGM, Bovada, LowVig, BetRivers, BetOnline, and BetUS.

---

### Cell 2 — Raw Data Exploration
Prints the raw JSON structure of a single game. The API returns nested JSON. Before writing any parsing code I needed to see exactly what fields existed and how they were nested.

Each game contains top-level info (teams, date, ID) and a nested list of bookmakers. Inside each bookmaker is a list of markets, and inside each market is a list of two outcomes: one per team. The key fields are buried inside: `bookmakers → markets → outcomes → point` (the spread) and `price` (the juice).

---

### Cell 3 — Parse Into Clean DataFrame
Flattens the nested JSON into a clean pandas DataFrame with one row per game per bookmaker.

**Immediate finding:** BetUS had the Seahawks at -4.5 while every other book had them at -3.5, a full point difference on the same game. That difference can be the difference between winning and losing a bet that lands exactly on the number.

---

### Cell 4 — Best Line Finder
Groups by game and finds the best available price on each side across all books. For most games DraftKings showed up as the best price; they're aggressive with pricing to acquire customers. The most interesting result was a game where LowVig.ag offered a home side at +100, meaningfully better than every other book.

---

### Cell 5 — Book Comparison Table with Juice & Hold %
For the Chiefs vs Broncos game (selected for having the most bookmakers), this table compares every book's spread, price, juice vs standard, total juice, and hold percentage.

**Hold % formula:**
- Negative odds: `abs(odds) / (abs(odds) + 100)` → implied probability
- Positive odds: `100 / (odds + 100)` → implied probability
- Sum both sides × 100 = hold %

| Bookmaker | Hold % | Notable |
|---|---|---|
| LowVig.ag | 102.48% | Cheapest — near zero vig |
| DraftKings | 104.55% | Charging -120 on Chiefs side |
| BetRivers | 105.61% | Most expensive book in this game |

---

## Part 2: Historical Market Analysis

### Cell 6 — Load Historical Data
Loaded `spreadspoke_scores.csv` — 14,371 NFL games from 1967–2024.

**Key limitation:** This dataset contains one spread line per game, not separate opening and closing lines. True CLV analysis requires both. I pivoted to cover rate and situational analysis which is still directly relevant to trading and better suited to this dataset.

---

### Cell 7 — Data Cleaning & Engineering
Filtered to 1990+ and dropped games with missing spread or score data → **9,455 clean games**.

Key engineered columns:
- **home_is_favorite**: mapped full team names to abbreviations (e.g. "Atlanta Falcons" → "ATL") to correctly identify the favorite across 35 seasons including historical franchise name changes
- **home_margin**: score_home minus score_away
- **favorite_covered**: True/False based on whether margin exceeded spread, logic verified manually on multiple rows

---

### Cell 8 — Cover Rate by Spread Size

Overall favorite cover rate: **46.0%**, already below the 50% break-even threshold.

| Spread Range | Cover Rate | Games |
|---|---|---|
| Pick em (0–1) | 47.8% | 454 |
| Small (1–3) | 46.4% | 2,873 |
| Medium (3–6) | 47.0% | 2,623 |
| Large (6–9) | 45.3% | 2,055 |
| Big (9–13) | 45.0% | 974 |
| **Huge (13+)** | **40.3%** | 395 |

This is the **favorite-longshot bias**: public bettors systematically overvalue heavy favorites, so books shade the line knowing they'll profit long-term.

---

### Cell 9 — Favorite-Longshot Bias Visualization
Bar chart showing declining cover rate as spread size increases, with secondary axis for sample size. Every bar sits below the 50% break-even line. The steepest drop is at Huge (13+) at 40.3% on 395 games.

---

### Cell 10 — Significance Testing: Spread Size

| Spread Range | Cover Rate | P-Value | Significant? |
|---|---|---|---|
| Pick em (0–1) | 47.8% | 0.1863 | No, market efficient |
| Small (1–3) | 46.4% | 0.0001 | **Yes** |
| Medium (3–6) | 47.0% | 0.0010 | **Yes** |
| Large (6–9) | 45.3% | ~0.0000 | **Yes** |
| Big (9–13) | 45.0% | 0.0009 | **Yes** |
| Huge (13+) | 40.3% | 0.0001 | **Yes** |

Pick em games are efficiently priced. Every other bucket is highly significant showing the favorite-longshot bias confirmed across 9,001 games of evidence.

**Practical implication:** To profit at standard -110 juice you need to win 52.4% of the time. Huge favorites cover at 40.3%, nearly 12 percentage points below that threshold.

---

### Cell 11 — Short Week Analysis

Filtered to true short week Thursday games only: Week 2+, regular season only leaving 242 games out of 4,363 total.

| Situation | Cover Rate | Games |
|---|---|---|
| All Games (baseline) | 46.3% | 4,121 |
| Short Week Thursday | 46.7% | 242 |
| Short Week — Home Favorite | 50.7% | 138 |
| **Short Week — Away Favorite** | **41.3%** | 104 |

The overall short week effect is negligible (0.4%). But split by home/away, away favorites on short weeks cover only 41.3%, facing compounding disadvantages of reduced prep time, travel fatigue, and public money inflating their line regardless.

---

### Cell 12 — Significance Testing: Short Week Away Favorite

| Metric | Value |
|---|---|
| Games | 104 |
| Covers | 43 |
| Cover Rate | 41.3% |
| P-Value | **0.0475** |
| Significant? | **Yes (p < 0.05)** |
| 95% CI | 0.0% — 49.9% |

Statistically significant at the 95% confidence level. I want to be intellectually honest, 0.0475 is right at the edge of the threshold. This is a real signal worth tracking, not an overwhelming edge. The sample grows by ~8 games per season so this finding will strengthen or weaken with time.

---

### Cell 13 — Market Efficiency Over Time

| Era | Cover Rate | P-Value | Significant? |
|---|---|---|---|
| 2010–2017 (Early) | 42.3% | 0.1659 | No |
| 2018–2024 (Modern) | 40.4% | 0.1058 | No |
| 2010–2024 (Full) | **41.3%** | **0.0475** | **Yes** |

Neither era is significant alone: splitting 104 games into 52 reduces statistical power. But the modern era (40.4%) is slightly worse than the early era (42.3%), meaning the market has **not** priced out this edge. The year-by-year chart shows high volatility (6–8 games per season) but the average consistently sits below 50%.

---

### Cell 14 — Bye Week Contrast: The Most Important Finding

| Situation | Cover Rate | P-Value | Market Efficient? |
|---|---|---|---|
| Baseline | 46.6% | — | — |
| Bye Week Favorite | 45.2% | 0.9533 | ✅ Yes — fully priced in |
| **Short Week Away Favorite** | **41.3%** | **0.0475** | ❌ No — edge persists |

**Bye week favorites show no edge (p=0.9533).** The market has fully priced in rest advantages as books shade the line accordingly and by the time the public bets, the advantage is already extracted from the spread.

**The asymmetry is the core insight of this project.** Books efficiently price in advantages (predictable, easy to model) but struggle to price in disadvantages when public money is too strong to fade. Bettors keep hammering away favorites regardless of rest situation, preventing books from moving the line far enough to reflect the true disadvantage.

This is what market inefficiency actually looks like, not a glaring mispricing, but a subtle structural bias driven by public betting behavior that persists even after years of data exists to identify it.

---

## Summary of Key Findings

| Finding | Result | Significant? |
|---|---|---|
| Overall favorite cover rate (1990–2024) | 46.0% | Yes |
| Huge favorites (13+) cover rate | 40.3% | Yes (p=0.0001) |
| Pick em games — market efficiency | 47.8% | No, efficiently priced |
| Short week away favorite cover rate | 41.3% | Yes (p=0.0475) |
| Bye week favorite cover rate | 45.2% | No, efficiently priced |
| Short week away edge — modern era | 40.4% | Edge persists |

---

## Key Concepts Demonstrated

- **Favorite-Longshot Bias**: systematic tendency for favorites to be overpriced relative to true cover probability
- **Market Efficiency**: bye week advantages are priced in; short week disadvantages are not
- **Hold Percentage**: calculating a bookmaker's theoretical profit margin from implied probabilities
- **Statistical Significance**: binomial testing to distinguish real patterns from random variance
- **Confidence Intervals**: quantifying uncertainty given sample size

---

## Limitations & Next Steps

- Dataset contains one line per game, true CLV requires separate opening and closing lines
- Short week sample (104 games) grows ~8 games per season; should be updated annually
- Project 2: regression-based NFL power ratings model to generate independent spread predictions and compare to market lines

---

## Data Sources

- [The Odds API](https://the-odds-api.com) — live NFL odds from major US sportsbooks
- [Kaggle: NFL Scores and Betting Data](https://www.kaggle.com/datasets/tobycrabtree/nfl-scores-and-betting-data) — historical scores and spread lines 1967–2024
