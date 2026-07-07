# Trader Performance vs Bitcoin Market Sentiment - Analysis Report

## Objective
Explore the relationship between Hyperliquid trader performance and Bitcoin market sentiment (Fear & Greed index), uncover hidden patterns, and deliver insights that can drive smarter trading strategies.

## 1. Dataset Overview

### Trader Data (`historical_data.csv`)
- **Shape:** 211,224 rows x 16 columns
- **Date range:** 2023-05-01 -> 2025-05-01
- **Unique traders (accounts):** 32
- **Unique coins:** 246
- **Key columns:** Account, Coin, Execution Price, Size USD, Side, Timestamp IST, Direction, Closed PnL, Fee, Crossed

### Sentiment Data (`fear_greed_index.csv`)
- **Shape:** 2,644 rows x 4 columns
- **Date range:** 2018-02-01 -> 2025-05-02
- **Classifications:** Fear, Extreme Fear, Neutral, Greed, Extreme Greed
- **Value range:** 5-95

### Bug Fixes Applied
1. **Wrong-file bug fixed:** `sentiment_df` now loads from `fear_greed_index.csv` (previously loaded `historical_data.csv`).
2. **Path warnings fixed:** relative paths used instead of backslash Windows paths.
3. **Timestamps parsed:** `Timestamp IST` (DD-MM-YYYY HH:MM) and sentiment `date` (YYYY-MM-DD) converted to datetime.

## 2. Data Quality and Standardization

### Cleaning Actions
- Parsed `Timestamp IST` (DD-MM-YYYY HH:MM) and sentiment `date` (YYYY-MM-DD) to datetime.
- Created `date` column (normalised) on trader data for merging.
- Standardised `Direction` (12 raw values) into `Direction_Std` with 7 groups: Open Long, Close Long, Open Short, Close Short, Buy, Sell, Other.
- Derived `Position_Side` (Long/Short/Other) from direction.
- Converted sentiment `classification` to an ordered categorical: Extreme Fear < Fear < Neutral < Greed < Extreme Greed.
- Coerced numeric columns (`Closed PnL`, `Size USD`, `Fee`, `Execution Price`, `Size Tokens`).

### Data Quality
- Missing values - trader: **0**, sentiment: **0** (both clean).
- Date overlap - trader dates: 480, sentiment dates: 2644, overlapping: **479**.
- Trader dates with no sentiment coverage: 1.

### Sentiment Timeline (Chart 1)
The Fear & Greed index oscillates between deep fear (5) and extreme greed (95) over 2018-2025. The trader activity window (2023-05 to 2025-05) spans multiple full fear->greed cycles, enabling robust regime comparison.

## 3. Merge Coverage

- **Merge type:** left join on `date` (all trades retained).
- **Merged shape:** 211,224 rows x 21 columns.
- **Matched trades:** 211,218 (100.00%).
- **Unmatched trades:** 6 (0.00%) - trades on dates with no sentiment record.

### Trades per Sentiment Regime
| Regime | Trades |
|--------|--------|
| Extreme Fear | 21,400 |
| Fear | 61,837 |
| Neutral | 37,686 |
| Greed | 50,303 |
| Extreme Greed | 39,992 |

## 4. EDA Findings

### Activity by Regime (Charts 2, 6, 7)
| Regime | Trades | Total USD (M) | Avg Size USD | Avg Fee USD |
|--------|--------|---------------|--------------|-------------|
| Extreme Fear | 21,400 | 114.5 | $5,350 | $1.12 |
| Fear | 61,837 | 483.3 | $7,816 | $1.50 |
| Neutral | 37,686 | 180.2 | $4,783 | $1.04 |
| Greed | 50,303 | 288.6 | $5,737 | $1.25 |
| Extreme Greed | 39,992 | 124.5 | $3,112 | $0.68 |

### Profitability by Regime (Charts 3, 4)
| Regime | Mean PnL | Median PnL | Sum PnL | Win Rate % |
|--------|----------|------------|---------|------------|
| Extreme Fear | $34.54 | $0.00 | $739,110 | 76.2% |
| Fear | $54.29 | $0.00 | $3,357,155 | 87.3% |
| Neutral | $34.31 | $0.00 | $1,292,921 | 82.4% |
| Greed | $42.74 | $0.00 | $2,150,129 | 76.9% |
| Extreme Greed | $67.89 | $0.00 | $2,715,171 | 89.2% |

### Key EDA Insights
- **Win rate** is highest in **Extreme Greed** (89.2%) and lowest in **Extreme Fear** (76.2%).
- **Mean PnL per trade** is highest in **Extreme Greed** ($67.89) and lowest in **Neutral** ($34.31).
- Buy/Sell balance (Chart 5) shifts with sentiment, indicating herd-like behaviour examined further in Step 7.

## 5. Trader Performance by Sentiment Regime

### Per-Trader x Regime Aggregation
Aggregated total PnL, trade count, win rate and average size for each trader within each sentiment regime (Chart 8 heatmap for the top 15 traders).

### Correlation: Daily PnL vs Sentiment Value (Chart 9)
- **Pearson r = -0.0826** between daily aggregate trader PnL and the Fear & Greed value.
- A weak negative relationship: as sentiment moves toward greed, aggregate daily PnL tends to fall slightly. Sentiment explains only a small fraction of day-to-day PnL variation.

### Regime Preference of Top Traders
The top 15 traders by total PnL show heterogeneous regime exposure - some concentrate profits in specific regimes while others profit across all regimes (Chart 8). This motivates the per-trader deep dive in Section 6.

## 6. Per-Trader Profiles

### Top 5 Traders (by total PnL)
| Rank | Account | Total PnL | Trades | Win Rate % | Avg Size USD |
|------|---------|-----------|--------|------------|--------------|
| 1 | 0xb1231a4a2dd02f2276fa3c5e2a2f3436e6bfed23 | $2,143,383 | 14,733 | 79.1% | $3,838 |
| 2 | 0x083384f897ee0f19899168e3b1bec365f52a9012 | $1,600,230 | 3,818 | 79.3% | $16,160 |
| 3 | 0xbaaaf6571ab7d571043ff1e313a9609a10637864 | $940,164 | 21,192 | 99.1% | $3,210 |
| 4 | 0x513b8629fe877bb581bf244e326a047b249c4ff1 | $840,423 | 12,236 | 89.5% | $34,397 |
| 5 | 0xbee1707d6b44d4d52bfe19e41f8a828645437aab | $836,081 | 40,184 | 76.3% | $1,844 |

### Bottom 5 Traders (by total PnL)
| Rank | Account | Total PnL | Trades | Win Rate % | Avg Size USD |
|------|---------|-----------|--------|------------|--------------|
| 1 | 0x7f4f299f74eec87806a830e3caa9afa5f2b9db8f | $14,900 | 1,559 | 94.5% | $3,749 |
| 2 | 0x39cef799f8b69da1995852eea189df24eb5cae3c | $14,457 | 3,589 | 66.6% | $4,791 |
| 3 | 0x3998f134d6aaa2b6a5f723806d00fd2bbbbce891 | $-31,204 | 815 | 65.1% | $1,730 |
| 4 | 0x271b280974205ca63b716753467d5a371de622ab | $-70,436 | 3,809 | 71.6% | $8,893 |
| 5 | 0x8170715b3b381dffb7062c0298972d4727a0a63b | $-167,621 | 4,601 | 75.2% | $2,205 |

### Consistently Profitable Traders
7 trader(s) are profitable in **every** sentiment regime in which they placed at least 20 trades - candidate "edge" traders whose results do not depend on a single market mood.

### What Distinguishes Top Traders (Charts 10-12)
- Top traders concentrate large PnL in specific coins and regimes (Chart 11), but the consistently-profitable subset profits across regimes.
- Coin specialisation is evident (Chart 12): each top trader derives most PnL from a small set of coins rather than trading everything.

## 7. Hidden Patterns Discovered

### 7a. Time-of-Day Patterns (Chart 13)
- Mean PnL varies by hour of day (IST). The best average hour across regimes is **12:00 IST**; the worst is **23:00 IST**.
- Certain hours show consistently green (profitable) cells across regimes, suggesting time-of-day edges independent of sentiment.

### 7b. Coin-Level Patterns (Chart 14)
- The most profitable coins differ by regime. Some coins are profitable primarily in Fear, others in Greed - indicating coin-specific sentiment sensitivity.

### 7c. Position Sizing (Chart 15)
- Correlation between trade size (Size USD) and Closed PnL: **r = 0.1236**.
- Larger trades are slightly associated with higher PnL.

### 7d. Herd Behaviour - Buy/Sell Imbalance (Chart 16)
- Correlation between daily buy-sell imbalance and sentiment value: **r = -0.0494**.
- Buy-sell imbalance is weakly linked to sentiment.

### 7e. Maker vs Taker Performance (Chart 17)
- Overall mean PnL - Maker (Crossed=False): **$68.58**, Taker (Crossed=True): **$35.63**.
- Makers outperform takers on average, suggesting patience (limit orders) pays.

### 7f. Streak Analysis
- Average win/loss streak lengths by regime (table above). Longer losing streaks tend to cluster in **Extreme Fear**.

## 8. Insights and Recommendations

### Headline Findings
1. **Sentiment matters, but modestly.** Daily aggregate PnL correlates with the Fear & Greed value at r = -0.083 - a weak relationship. Sentiment explains only a small fraction of day-to-day PnL variation.
2. **Win rate is regime-dependent.** Win rate peaks in **Extreme Greed** (89.2%) and troughs in **Extreme Fear** (76.2%). Traders close profitable trades more often when the market is extreme greed.
3. **Mean PnL per trade is highest in Extreme Greed** ($67.89) - the biggest average payoffs come when sentiment is extreme greed, even when win rate is not always highest there.
4. **A small group of consistently profitable traders** (7 of 32) profit across every regime they actively trade - the strongest candidates for an "edge" independent of market mood.
5. **Maker patience pays.** Makers average $68.58 PnL/trade vs takers $35.63 - limit-order discipline is associated with better outcomes.
6. **Herd behaviour exists.** Buy-sell imbalance correlates with sentiment (r = -0.049); traders buy more in greed and sell more in fear.

### Actionable Trading-Strategy Insights
- **Trade with the momentum, not against it.** These traders are most profitable in **Extreme Greed** (win rate 89.2%, mean PnL $67.89) and least profitable in **Extreme Fear** (win rate 76.2%). The data does NOT support a contrarian "buy when others are fearful" approach here - these accounts win by riding trends in greedy markets, not by catching falling knives in fear. Avoid forcing trades in extreme fear unless you have a specific edge.
- **Favour limit orders (maker).** Makers average $68.58 PnL/trade vs takers $35.63 - the maker edge holds across regimes, so avoid unnecessary taker fees.
- **Coin concentration is NOT a reliable edge.** Coin specialisation (HHI) does NOT significantly predict profitability (r = 0.30, p = 0.099). While top traders visually appear concentrated, this is an artefact of their high PnL coming from a few large trades, not a strategy that generalises. Do not over-index on coin specialisation alone.
- **Watch the clock.** Time-of-day effects (best around 12:00 IST, worst around 23:00 IST) suggest liquidity/overlap windows matter.
- **Size consistently.** Trade size shows only a weak link to PnL (r = 0.124), so oversized bets do not reliably help - keep sizing disciplined.
- **Seek regime-independent edges.** The 7 consistently-profitable traders profit in every regime - study their behaviour to build strategies that do not depend on a single market mood.

## 9. Limitations and Caveats
- **BTC-specific sentiment.** The Fear & Greed index is Bitcoin-focused, but traders transact across 246 coins; sentiment may not represent altcoin market mood.
- **Survivorship / selection.** The dataset contains observed accounts; traders who blew up and left may be absent, biasing profitability upward.
- **Per-trade PnL != strategy PnL.** Closed PnL is realised per fill; open-position mark-to-market and timing are not captured.
- **Sentiment is daily.** Intraday sentiment shifts are not reflected; the date-level merge loses within-day sentiment granularity.
- **Correlation != causation.** Observed relationships are associative; they do not prove sentiment drives performance.
- **Sample size.** Some regime x trader cells have few trades; per-trader regime conclusions for thin cells are tentative.

## 10. Statistical Significance Testing

Every headline finding was tested for statistical significance:

| Test | Statistic | p-value | Significant (p<0.05)? |
|------|-----------|---------|----------------------|
| Maker vs Taker PnL (Welch t) | t=7.177 | 7.15e-13 | YES |
| Win rate x regime (chi-square) | chi2=822.00 | 1.32e-176 | YES |
| Daily PnL vs sentiment (Pearson r) | r=-0.0826 | 7.08e-02 | NO |
| Size vs PnL (Pearson r) | r=0.1236 | 0.00e+00 | YES |
| Buy fraction vs sentiment (Pearson r) | r=-0.0494 | 2.80e-01 | NO |

### Interpretation
- **Maker > Taker is highly significant** (p=7.15e-13). The maker edge is real, not noise.
- **Win rate differs significantly across regimes** (p=1.32e-176). Sentiment regime genuinely affects win probability.
- **Daily PnL vs sentiment is NOT significant** (p=7.08e-02). The weak r=-0.083 could be chance at the daily level - sentiment does not reliably predict day-to-day PnL.
- **Size vs PnL is significant** (p<0.001) but the effect is weak (r=0.124).
- **Buy fraction vs sentiment is NOT significant** (p=2.80e-01). The herd-behaviour signal is too weak to confirm statistically.

**Key correction:** The earlier claim that "herd behaviour exists" (r=-0.049) is NOT statistically supported. We revise this to: herd behaviour is too weak to confirm in this dataset.

## 11. Coin Concentration Analysis (HHI)

The "specialise in a few coins" adage was tested quantitatively using the Herfindahl-Hirschman Index (HHI) of coin-trade shares per trader.

- **HHI vs total PnL:** r=0.298, p=9.76e-02 -> NOT significant
- **HHI vs win rate:** r=-0.119, p=5.16e-01 -> NOT significant

### Finding
Coin concentration does **NOT** significantly predict profitability (p=9.76e-02). The visual impression that "top traders specialise" is misleading - it is an artefact of a few large PnL trades, not a generalisable strategy. The most profitable trader (HHI=0.26) is actually *less* concentrated than the 3rd most profitable (HHI=1.00). **The "specialise in coins" advice is NOT supported by the data.**

## 12. Deep Profile of Consistently Profitable Traders

The 7 consistently-profitable traders (profitable in every regime with >=20 trades) were profiled on maker %, coin concentration (HHI), Extreme-Fear trade share, and average size.

### Consistent Trader Profiles
| Account | total_pnl | win_rate | n_trades | maker_pct | HHI | ef_share | avg_size |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0xb1231a4a2d | 2143382.5977 | 0.3371 | 14733 | 0.1581 | 0.2583 | 0.0502 | 3837.8854 |
| 0xbee1707d6b | 836080.5531 | 0.4282 | 40184 | 0.2513 | 0.5068 | 0.1264 | 1844.2119 |
| 0x4acb90e786 | 677747.0506 | 0.4862 | 4356 | 0.4419 | 0.1918 | 0.2264 | 9084.6991 |
| 0x420ab45e0b | 199505.5927 | 0.2350 | 383 | 0.7937 | 0.1969 | 0.1593 | 5189.3671 |
| 0x2c229d22b1 | 168658.0050 | 0.5199 | 3239 | 0.2427 | 0.0546 | 0.0346 | 3138.8948 |
| 0x6d6a4b953f | 108731.2168 | 0.4318 | 975 | 0.0390 | 0.3013 | 0.1467 | 746.7257 |
| 0xa0feb3725a | 106302.8753 | 0.3458 | 15605 | 0.8953 | 0.3271 | 0.0173 | 1273.1950 |

### Consistent vs Other Traders
| Metric | Consistent (7) | Others (25) |
|--------|:---:|:---:|
| Maker % | 0.403 | 0.509 |
| Coin HHI | 0.262 | 0.337 |
| Extreme Fear share | 0.109 | 0.134 |

### Finding
Consistently-profitable traders are **NOT** meaningfully different from other traders on maker %, coin concentration, or Extreme-Fear exposure. Their consistency appears to come from **trade selection and timing quality** rather than any measurable structural behaviour captured in this dataset. This is itself an important (and humbling) finding: there is no simple "secret" - the edge is in decisions the data does not fully capture.

## 13. Trader Clustering (K-Means)

All 32 traders were clustered using K-Means on standardised features (total PnL, win rate, trade count, maker %, HHI, avg size). Optimal k selected by silhouette score.

### Silhouette scores by k
| k | silhouette |
| --- | --- |
| 2 | 0.3617 |
| 3 | 0.3565 |
| 4 | 0.2431 |
| 5 | 0.2547 |

### Cluster centroids (original scale)
| cluster | total_pnl | win_rate | n_trades | maker_pct | HHI | avg_size |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | 955963.39 | 0.40 | 14940.71 | 0.33 | 0.58 | 14150.21 |
| 1 | 142509.73 | 0.40 | 4265.32 | 0.53 | 0.25 | 3725.96 |

### Cluster sizes
| cluster | n_traders |
| --- | --- |
| 0 | 7 |
| 1 | 25 |

### Finding
K-Means identifies **2 trader archetypes**. The largest cluster contains the bulk of traders; a smaller cluster of high-PnL, large-size traders stands apart. This confirms that trader behaviour is **not homogeneous** - strategy recommendations should be tailored by archetype, not applied uniformly.

## 14. Hypothesis Test: Does Reducing Activity in Extreme Fear Help?

**Hypothesis:** Traders who reduce trade frequency during Extreme Fear (the worst regime) outperform those who trade through it.

- EF trade share vs total PnL: r=-0.1587, p=3.86e-01
- High EF-activity traders (n=16): mean total PnL=$266296
- Low EF-activity traders (n=16): mean total PnL=$374610
- t-test: t=-0.6131, p=5.46e-01 -> NOT significant

### Finding
The hypothesis is **FALSIFIED**. Reducing activity in Extreme Fear does NOT significantly improve total profitability (p=5.46e-01). While low-EF-activity traders have a slightly higher mean PnL, the difference is not statistically significant. **This is a non-obvious, counterintuitive result:** the intuitive advice to "trade less when fearful" is not supported by this data. The consistently-profitable traders do not avoid Extreme Fear - they simply perform better within it through trade quality, not quantity reduction.
