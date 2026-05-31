---
title: Expected Value
deprecated: false
hidden: true
metadata:
  robots: index
---
## Overview — Expected Value (EV) metrics

EV metrics quantify the edge between a bookmaker's offered odds and the model's fair odds
across five betting markets. Each metric is a fixture-level signal; `team_id` is always 0.

EV is calculated as: `EV = round(((realOdd - fairOdd) / fairOdd) * 100)`
A positive value means the bookmaker's price is above the model's fair value (value bet);
negative means the price is below fair value (no edge).
Fair odds are derived from xG or equivalent xMetrics via `BettingCalculator`.

Region encoding in `developer_name`:
  - `EUROPE` — bookmaker_id = 2
  - `ASIA`   — bookmaker_id = 3

| type_id | developer_name          | market         | market_id | label               | description |
| ------- | ----------------------- | -------------- | --------- | ------------------- | ----------- |
| 281     | EV_EUROPE_ML_HOME       | Money Line     | 1         | Home win            | EV% for European bookmaker home-win price vs xG-derived fair odd. |
| 282     | EV_EUROPE_ML_AWAY       | Money Line     | 1         | Away win            | EV% for European bookmaker away-win price vs xG-derived fair odd. |
| 283     | EV_EUROPE_ML_DRAW       | Money Line     | 1         | Draw                | EV% for European bookmaker draw price vs xG-derived fair odd. |
| 284     | EV_ASIA_ML_HOME         | Money Line     | 1         | Home win            | EV% for Asian bookmaker home-win price vs xG-derived fair odd. |
| 285     | EV_ASIA_ML_DRAW         | Money Line     | 1         | Draw                | EV% for Asian bookmaker draw price vs xG-derived fair odd. |
| 286     | EV_ASIA_ML_AWAY         | Money Line     | 1         | Away win            | EV% for Asian bookmaker away-win price vs xG-derived fair odd. |
| 287     | EV_EUROPE_AH_HOME       | Asian Handicap | 2         | Home side           | EV% for European bookmaker AH home price on the main handicap line. |
| 288     | EV_EUROPE_AH_AWAY       | Asian Handicap | 2         | Away side           | EV% for European bookmaker AH away price on the main handicap line. |
| 289     | EV_ASIA_AH_HOME         | Asian Handicap | 2         | Home side           | EV% for Asian bookmaker AH home price on the main handicap line. |
| 290     | EV_ASIA_AH_AWAY         | Asian Handicap | 2         | Away side           | EV% for Asian bookmaker AH away price on the main handicap line. |
| 291     | EV_EUROPE_GL_OVER       | Goal Line      | 3         | Over                | EV% for European bookmaker goal-line over price on the main total line. |
| 292     | EV_EUROPE_GL_UNDER      | Goal Line      | 3         | Under               | EV% for European bookmaker goal-line under price on the main total line. |
| 301     | EV_ASIA_GL_UNDER        | Goal Line      | 3         | Under               | EV% for Asian bookmaker goal-line under price on the main total line. |
| 302     | EV_ASIA_GL_OVER         | Goal Line      | 3         | Over                | EV% for Asian bookmaker goal-line over price on the main total line. |
| 299     | EV_EUROPE_COR_UNDER     | Corner Line    | 6         | Under               | EV% for European bookmaker corner-line under price on the main total line. |
| 300     | EV_EUROPE_COR_OVER      | Corner Line    | 6         | Over                | EV% for European bookmaker corner-line over price on the main total line. |
| 297     | EV_ASIA_COR_UNDER       | Corner Line    | 6         | Under               | EV% for Asian bookmaker corner-line under price on the main total line. |
| 298     | EV_ASIA_COR_OVER        | Corner Line    | 6         | Over                | EV% for Asian bookmaker corner-line over price on the main total line. |
| 295     | EV_EUROPE_YC_UNDER      | Yellow Cards   | 203       | Under               | EV% for European bookmaker yellow-card line under price on the main total line. |
| 296     | EV_EUROPE_YC_OVER       | Yellow Cards   | 203       | Over                | EV% for European bookmaker yellow-card line over price on the main total line. |
| 293     | EV_ASIA_YC_UNDER        | Yellow Cards   | 203       | Under               | EV% for Asian bookmaker yellow-card line under price on the main total line. |
| 294     | EV_ASIA_YC_OVER         | Yellow Cards   | 203       | Over                | EV% for Asian bookmaker yellow-card line over price on the main total line. |

## Practical Application & Use Cases

    Value Bet Identification: Scan all EV metrics for a fixture to instantly surface markets where
    the model has a positive edge over a specific bookmaker. A positive EV (e.g. +8 on EV_EUROPE_ML_HOME)
    signals the European bookmaker is overpricing the home win relative to the xG-implied probability —
    a directly actionable value bet opportunity.

    Cross-Market Edge Comparison: Compare EV across markets (ML, AH, GL, COR, YC) for the same fixture
    to identify where the sharpest mispricing exists. A fixture may show -2 EV on the Money Line but
    +11 on the Goal Line, pointing analysts toward the most profitable market for that game.

    Regional Bookmaker Benchmarking: EV metrics are split by region (EUROPE / ASIA), enabling direct
    comparison of how differently two major bookmaker pools price the same outcome. Consistent positive
    EV on one region and negative on another highlights structural pricing differences exploitable over
    large sample sizes.

    Market Efficiency Monitoring: Tracking how EV values shift as kick-off approaches reveals how quickly
    bookmakers adjust to model signals. A large positive EV that collapses in the final hours before
    a match indicates the sharp money has already moved the line, confirming the model's edge was real.

    Portfolio & Staking Decisions: EV values feed directly into Kelly-criterion or flat-staking frameworks.
    Filtering for fixtures where multiple related markets (e.g. AH and GL) show simultaneous positive EV
    provides higher-confidence opportunities for larger stake allocation.