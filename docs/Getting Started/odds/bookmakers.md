---
title: Bookmakers
excerpt: This section details the available data sources within the SSTrader API.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Bookmaker Providers

SSTrader categorizes odds by provider type to help you distinguish between market averages and professional-grade pricing.

| Bookmaker ID | Classification  | Description                                                  |
| ------------ | --------------- | ------------------------------------------------------------ |
| 2            | European (Soft) | Aggregated averages from major European consumer bookmakers. |
| 3            | Asian (Sharp)   | High-liquidity pricing from professional Asian bookmakers.   |

> Reference Note: All odds provided are for informational and reference purposes only.

***

## ⚖️ Comparing Goal Lines: Asian vs. European

Understanding the difference between these two systems is critical for accurate settlement and data display.

## Asian Goal Lines

- Increments: Uses 0.25 steps (e.g., 1.0, 1.25, 1.75).
- Settlement: Supports Half Win and Half Loss outcomes.
- Market Position: Lines stay focused on the Main Line (the predicted outcome).
- Pricing: Odds typically stay near even (approx. 2.00).

## European Goal Lines

- Increments: Uses 0.5 steps only (e.g., 0.5, 1.5, 2.5).
- Settlement: Binary results only—Win or Loss (no push or half-results).
- Market Position: Offers lines far from the main line (e.g., Over 5.5).
- Pricing: Allows for extreme odds and higher volatility.

***

## Key Example

In a match currently at 0–0:

- Asian Market: You will see lines like 2.25 or 2.75 with odds close to 1.90.
- European Market: You can find lines like "Over 5.5" with very high odds (e.g., 20.00+).

> Note: Even on the same line (e.g., Over 2.5), the odds will often differ between Bookmaker 2 and Bookmaker 3 due to different margin structures and market liquidity.