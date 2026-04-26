---
title: Markets
excerpt: >-
  This reference guide explains the mapping and logic behind Market IDs and
  Label IDs within the SSTrader API.
deprecated: false
hidden: false
metadata:
  robots: index
---
Use the table below to map `market_id` to the correct human-readable market name and interpret the `label_id` for specific outcomes.

## Market and Label ID Reference

| Market ID | Market Name | Label ID Mapping |
|---|---|---|
| 1 | 3Way Result (1x2) | 1: Home, 2: Away, 0: Draw |
| 2 | Asian Handicap | 1: Home, 2: Away |
| 3 | Goal Line (Asian) | 1: Under, 2: Over |
| 4 | Match Goals (European) | 1: Under, 2: Over |
| 5 | Corners — Asian Handicap | 1: Home, 2: Away |
| 6 | Asian Total Corners | 1: Under, 2: Over |
| 7 | Correct Score | 1 + Home + Away Score (e.g., 100: 0:0, 142: 4:2) |
| 11 | 1st Half 3Way Result | 1: Home, 2: Away, 0: Draw |
| 12 | 1st Half Asian Handicap | 1: Home, 2: Away |
| 13 | 1st Half Goal Line | 1: Under, 2: Over |
| 16 | 1st Half Asian Total Corners | 1: Under, 2: Over |
| 17 | 1st Half Correct Score | 1 + Home + Away Score (e.g., 111: 1:1) |
| 22 | n-Goal (Next Goal) | 1: Home, 2: Away, 0: No Goal |
| 50 | Draw No Bet | 1: Home, 2: Away |
| 51 | HT/FT | 111: 1/1, 100: X/X, 122: 2/2 (Standard codes) |
| 52 | Odd/Even | 1: Odd, 2: Even |
| 53 | BTTS | 1: Yes, 2: No |
| 54 | 1st Half BTTS | 1: Yes, 2: No |
| 55 | 2nd Half BTTS | 1: Yes, 2: No |
| 56 | Team Corners | 11: H-Under, 12: H-Over, 21: A-Under, 22: A-Over |
| 57 | Team Total Goals | 11: H-Under, 12: H-Over, 21: A-Under, 22: A-Over |
| 58 | Double Chance | 10: Home/Draw, 2: Draw/Away, 12: Home/Away |
| 59 | Clean Sheet | 11: H-Yes, 12: H-No, 21: A-Yes, 22: A-No |
| 203 | Asian Total Cards | 1: Under, 2: Over |

------------------------------
## Implementation Notes

* Next Goal (ID 22): The specific goal number (1st, 2nd, etc.) is found in the line property of the Odd object.
* Correct Score (ID 7/17): The logic is always 1 followed by the score. To parse, remove the leading 1.
* Compound IDs (ID 56/57/59): The first digit represents the team (1=Home, 2=Away) and the second represents the outcome.
