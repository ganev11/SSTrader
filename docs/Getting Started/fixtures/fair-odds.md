---
title: Fair Odds
deprecated: false
hidden: false
metadata:
  robots: index
---
## Fair Odds Schema Documentation

The `fair_odds` object represents the mathematically calculated "true" value of a betting market, removing the bookmaker's margin (overround). This data is used to identify value bets and understand the raw statistical probability of an outcome.

------------------------------

## Field Definitions

| Field | Type | Description |
|---|---|---|
| market_id | integer | Unique identifier for the betting market (e.g., 1 for Match Winner, 4 for Over/Under). |
| label_id | integer | Unique identifier for the specific outcome (e.g., Home Team, Away Team, or Draw). |
| probability | number | The calculated likelihood of the outcome occurring, expressed as a percentage. |
| value | number | The "Fair Odds" (decimal format). Calculated as $100 / probability$. |
| handicap | number | The specific advantage or disadvantage assigned to a team (used in Asian Handicap or Spread markets). |
| line | number | The threshold value for the market (common in Over/Under or Goal Line markets). |