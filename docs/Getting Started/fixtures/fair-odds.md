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
| value | number | The "Fair Odds" (decimal format). Calculated as 100 / probability. |
| handicap | number | The specific advantage or disadvantage assigned to a team (used in Asian Handicap or Spread markets). |
| line | number | The threshold value for the market (common in Over/Under or Goal Line markets). |

------------------------------
## 🔗 Mapping Logic

To map Fair Odds to Real Odds, you align them using the unique combination of market_id, label_id, and if applicable, the handicap or line.
Mapping these allows you to calculate the Value Edge (the difference between the market price and the true statistical probability).

To ensure you are comparing the same outcome, match the objects using these keys:

   1. market_id: Ensures both objects refer to the same market (e.g., Asian Total Cards).
   2. label_id: Ensures you are looking at the same outcome (e.g., Over 3.5).
   3. line : Ensures the threshold is identical (e.g., 3.5).

------------------------------
## Comparison Table

| Data Point | Fair Odds (True Value) | Real Odds (Bookmaker) | Difference (Edge) |
|---|---|---|---|
| Market | Asian Total Cards (203) | Asian Total Cards (203) | Match ✅ |
| Label | Over 3.5 (2) | Over 3.5 (2) | Match ✅ |
| Line | 3.5 | 3.5 | Match ✅ |
| Value (Odds) | 4.984 | 2.10 | -57.8% (No Value) |
| Probability | 20.07% | 47.62% | +27.55% Market Bias |

------------------------------

## Documentation: Identifying a Value Bet

A Value Bet exists when the real_value (bookmaker odds) is higher than the fair_value.

* Formula: (Real Odds / Fair Odds) - 1
* In this example: (2.1 / 4.984) - 1 = -0.57.
* Result: This is not a value bet. The bookmaker is offering a much lower price than the statistical probability suggests.