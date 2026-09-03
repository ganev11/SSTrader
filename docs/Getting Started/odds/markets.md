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

## SSTrader Markets Reference

This guide explains `market_id` and `label_id` mappings used in the SSTrader API.
Every bet selection is identified by a **market\_id** (what type of market) and a **label\_id** (which outcome within that market).

***

## How Label IDs Work

**Label IDs are NOT sequential outcome indices.** Each market has its own fixed label\_id mapping.
Common mistakes to avoid:

* In Under/Over markets: `label_id 1 = Under`, `label_id 2 = Over` (not the reverse)
* In 3-Way markets: `label_id 0 = Draw` (not label\_id 3), `label_id 1 = Home`, `label_id 2 = Away`
* In Correct Score markets: label\_id encodes the score itself (see below)

***

## Market Definitions

### Market 1 — 3Way Result (1x2)

Standard home/draw/away market for full-time result.

| label\_id | Outcome         |
| --------- | --------------- |
| 1         | Home Win        |
| 2         | Away Win        |
| 0         | Draw ⚠️ (not 3) |

***

### Market 2 — Asian Handicap

| label\_id | Outcome |
| --------- | ------- |
| 1         | Home    |
| 2         | Away    |

***

### Market 3 — Goal Line (Asian Total Goals)

| label\_id | Outcome          |
| --------- | ---------------- |
| 1         | Under ⚠️ (not 2) |
| 2         | Over             |

***

### Market 4 — Match Goals (European Total Goals)

| label\_id | Outcome |
| --------- | ------- |
| 1         | Under   |
| 2         | Over    |

***

### Market 5 — Corners Asian Handicap

| label\_id | Outcome |
| --------- | ------- |
| 1         | Home    |
| 2         | Away    |

***

### Market 6 — Asian Total Corners

| label\_id | Outcome |
| --------- | ------- |
| 1         | Under   |
| 2         | Over    |

***

### Market 7 — Correct Score

Label IDs encode the scoreline directly using the pattern: `1` + home goals digit + away goals digit.

| label\_id | Score               |
| --------- | ------------------- |
| 100       | 0:0                 |
| 101       | 0:1                 |
| 110       | 1:0                 |
| 111       | 1:1                 |
| 120       | 2:0                 |
| 121       | 2:1                 |
| 122       | 2:2                 |
| 130       | 3:0                 |
| 142       | 4:2                 |
| ...       | (pattern continues) |

**Parsing rule:** Remove the leading `1`. The remaining digits are `home_goals` + `away_goals`.
Example: `label_id 142` → remove `1` → `42` → Home 4, Away 2 → score **4:2**

***

### Market 11 — 1st Half 3Way Result

Same label mapping as Market 1, but for the first half only.

| label\_id | Outcome  |
| --------- | -------- |
| 1         | Home Win |
| 2         | Away Win |
| 0         | Draw     |

***

### Market 12 — 1st Half Asian Handicap

| label\_id | Outcome |
| --------- | ------- |
| 1         | Home    |
| 2         | Away    |

***

### Market 13 — 1st Half Goal Line (Asian Total Goals)

| label\_id | Outcome |
| --------- | ------- |
| 1         | Under   |
| 2         | Over    |

***

### Market 14 — 1st Half Match Goals (European Total Goals)

| label\_id | Outcome |
| --------- | ------- |
| 1         | Under   |
| 2         | Over    |

***

### Market 16 — 1st Half Asian Total Corners

| label\_id | Outcome |
| --------- | ------- |
| 1         | Under   |
| 2         | Over    |

***

### Market 17 — 1st Half Correct Score

Same encoding as Market 7, but for first-half scores only.

| label\_id | Score                            |
| --------- | -------------------------------- |
| 100       | 0:0                              |
| 111       | 1:1                              |
| 110       | 1:0                              |
| ...       | (same pattern: remove leading 1) |

***

### Market 22 — Next Goal (n-Goal)

Which team scores the Nth goal. The specific goal number (1st, 2nd, etc.) is in the `line` property of the Odd object.

| label\_id | Outcome                      |
| --------- | ---------------------------- |
| 1         | Home scores next             |
| 2         | Away scores next             |
| 0         | No more goals (neither team) |

***

### Market 50 — Draw No Bet

| label\_id | Outcome |
| --------- | ------- |
| 1         | Home    |
| 2         | Away    |

***

### Market 51 — Half-Time / Full-Time (HT/FT)

Label IDs encode the HT result and FT result combined: `HT_result` + `FT_result`, where `1` = Home, `2` = Away, `0` = Draw (represented as `0`/`X`).

| label\_id | HT Result            | FT Result |
| --------- | -------------------- | --------- |
| 111       | Home                 | Home      |
| 100       | Draw                 | Draw      |
| 122       | Away                 | Away      |
| 112       | Home                 | Away      |
| 121       | Away                 | Home      |
| 110       | Home                 | Draw      |
| ...       | (all 9 combinations) |           |

***

### Market 52 — Odd/Even (Total Goals)

| label\_id | Outcome |
| --------- | ------- |
| 1         | Odd     |
| 2         | Even    |

***

### Market 53 — Both Teams To Score (BTTS)

| label\_id | Outcome |
| --------- | ------- |
| 1         | Yes     |
| 2         | No      |

***

### Market 54 — 1st Half BTTS

| label\_id | Outcome |
| --------- | ------- |
| 1         | Yes     |
| 2         | No      |

***

### Market 55 — 2nd Half BTTS

| label\_id | Outcome |
| --------- | ------- |
| 1         | Yes     |
| 2         | No      |

***

### Market 56 — Team Corners

Compound label\_id: first digit = team (`1` = Home, `2` = Away), second digit = outcome (`1` = Under, `2` = Over).

| label\_id | Team | Outcome |
| --------- | ---- | ------- |
| 11        | Home | Under   |
| 12        | Home | Over    |
| 21        | Away | Under   |
| 22        | Away | Over    |

***

### Market 57 — Team Total Goals

Same compound structure as Market 56.

| label\_id | Team | Outcome |
| --------- | ---- | ------- |
| 11        | Home | Under   |
| 12        | Home | Over    |
| 21        | Away | Under   |
| 22        | Away | Over    |

***

### Market 58 — Double Chance

| label\_id | Outcome      |
| --------- | ------------ |
| 10        | Home or Draw |
| 2         | Draw or Away |
| 12        | Home or Away |

***

### Market 59 — Clean Sheet

Compound label\_id: first digit = team (`1` = Home, `2` = Away), second digit = outcome (`1` = Yes, `2` = No).

| label\_id | Team | Clean Sheet? |
| --------- | ---- | ------------ |
| 11        | Home | Yes          |
| 12        | Home | No           |
| 21        | Away | Yes          |
| 22        | Away | No           |

***

### Market 60 - To Qualify

| label\_id | Outcome | 
| --------- | ------- |
| 1         | Home    |
| 2         | Away    |

***

### Market 203 — Asian Total Cards

| label\_id | Outcome |
| --------- | ------- |
| 1         | Under   |
| 2         | Over    |

***

### Market 300 - Player to be booked
Which named player receives a booking in the match. The specific player is identified elsewhere in the Odd object; label\_id encodes when they are booked.

| label\_id | Outcome |
| --------- | ------- |
| 1         | Yes     |

***

### Market 301 - 1st Player Booked
Which named player is the first to receive a booking in the match. The specific player is identified elsewhere in the Odd object; label\_id encodes when they are booked.

| label\_id | Outcome |
| --------- | ------- |
| 1         | Yes     |

***

### Market 302 - Player to be Sent Off
Which named player is sent off in the match. The specific player is identified elsewhere in the Odd object; label\_id encodes when they are sent off.

| label\_id | Outcome |
| --------- | ------- |
| 1         | Yes     |

***

### Market 303 — Match Goalscorers

Which named player scores in the match. The specific player is identified elsewhere in the Odd
object; label\_id encodes when they score.

| label\_id | Outcome |
| --------- | ------- |
| 0         | Anytime |
| 1         | First   |
| 2         | Last    |

***

### Market 304 - Team Goalscorers

Which named player scores for a specific team. The specific player is identified elsewhere in the Odd object; label\_id encodes when they score.

| label\_id | Outcome |
| --------- | ------- |
| 1         | First   |
| 2         | Last    |

***

### Market 305 - Multi Scorers

Which named player scores multiple goals in the match. The specific player is identified elsewhere in the Odd object; label\_id encodes when they score. Odd line property indicates the number of goals.

| label\_id | Outcome |
| --------- | ------- |
| 2         | Over   |

***

### Market 306 - Player to Assist

Which named player provides an assist in the match. The specific player is identified elsewhere in the Odd object; label\_id encodes when they assist.

| label\_id | Outcome |
| --------- | ------- |
| 2         | Yes     |

***

### Market 307 - Player Shots On Target

Which named player has a shot on target in the match. The specific player is identified elsewhere in the Odd object; label\_id encodes when they have a shot on target. Odd line property indicates the number of shots on target.

| label\_id | Outcome |
| --------- | ------- |
| 2         | Over     |

***

### Market 308 - Player Shots

Which named player has a shot in the match. The specific player is identified elsewhere in the Odd object; label\_id encodes when they have a shot. Odd line property indicates the number of shots.

| label\_id | Outcome |
| --------- | ------- |
| 2         | Over     |

***

### Market 309 - Player Fouls Committed

Which named player commits fouls in the match. The specific player is identified elsewhere in the Odd object; label\_id encodes when they commit fouls. Odd line property indicates the number of fouls.

| label\_id | Outcome |
| --------- | ------- |
| 2         | Over     |

***

### Market 313 - Score or Assist

Which named player scores or provides an assist in the match. The specific player is identified elsewhere in the Odd object; label\_id encodes when they score or assist.

| label\_id | Outcome |
| --------- | ------- |
| 1         | Yes     |

***

## Quick Lookup Table

| market\_id | Market Name                     | label\_id → Outcome                                                    |
| ---------- | ------------------------------- | ---------------------------------------------------------------------- |
| 1          | 3Way Result (1x2)               | **0**=Draw, 1=Home, 2=Away                                             |
| 2          | Asian Handicap                  | 1=Home, 2=Away                                                         |
| 3          | Goal Line (Asian)               | **1=Under**, 2=Over                                                    |
| 4          | Match Goals (European)          | **1=Under**, 2=Over                                                    |
| 5          | Corners — Asian Handicap        | 1=Home, 2=Away                                                         |
| 6          | Asian Total Corners             | **1=Under**, 2=Over                                                    |
| 7          | Correct Score                   | Encoded score (remove leading 1: remaining = HomeGoals+AwayGoals)      |
| 11         | 1st Half 3Way Result            | **0**=Draw, 1=Home, 2=Away                                             |
| 12         | 1st Half Asian Handicap         | 1=Home, 2=Away                                                         |
| 13         | 1st Half Goal Line (Asian)      | **1=Under**, 2=Over                                                    |
| 14         | 1st Half Match Goals (European) | **1=Under**, 2=Over                                                    |
| 16         | 1st Half Asian Total Corners    | **1=Under**, 2=Over                                                    |
| 17         | 1st Half Correct Score          | Encoded score (same as Market 7)                                       |
| 22         | Next Goal                       | **0**=No Goal, 1=Home, 2=Away                                          |
| 50         | Draw No Bet                     | 1=Home, 2=Away                                                         |
| 51         | HT/FT                           | Compound: digits = HT result + FT result                               |
| 52         | Odd/Even                        | 1=Odd, 2=Even                                                          |
| 53         | BTTS                            | 1=Yes, 2=No                                                            |
| 54         | 1st Half BTTS                   | 1=Yes, 2=No                                                            |
| 55         | 2nd Half BTTS                   | 1=Yes, 2=No                                                            |
| 56         | Team Corners                    | Compound: 1st digit=team (1=Home,2=Away), 2nd=outcome (1=Under,2=Over) |
| 57         | Team Total Goals                | Compound: 1st digit=team (1=Home,2=Away), 2nd=outcome (1=Under,2=Over) |
| 58         | Double Chance                   | 10=Home/Draw, 2=Draw/Away, 12=Home/Away                                |
| 59         | Clean Sheet                     | Compound: 1st digit=team (1=Home,2=Away), 2nd=outcome (1=Yes,2=No)     |
| 60         | To Qualify                      | 1=Home, 2=Away                                                         |
| 203        | Asian Total Cards               | **1=Under**, 2=Over                                                    |
| 300        | Player to be booked              | 1=Yes                                                                 |
| 301        | 1st Player Booked               | 1=Yes                                                                 |
| 302        | Player to be Sent Off             | 1=Yes                                                                 |
| 303        | Match Goalscorers               | 0=Anytime, 1=First, 2=Last                                             |
| 304        | Team Goalscorers                | 1=First, 2=Last                                                       |
| 305        | Player Shots On Target          | 2=Over                                                                |
| 306        | Player to Assist               | 1=Yes                                                                 |
| 307        | Player Shots On Target          | 2=Over                                                                |
| 308        | Player Shots                    | 2=Over                                                                |
| 309        | Player Fouls Committed          | 2=Over                                                                |
| 313        | Score or Assist                 | 1=Yes                                                                 |

***

## Key Rules for LLMs

1. **Under = label\_id 1, Over = label\_id 2** — in all total/line markets (3, 4, 6, 13, 16, 203).
2. **Draw = label\_id 0** — in all 3-way markets (1, 11). It is NOT label\_id 3.
3. **Correct Score encoding** — label\_id always starts with `1`. Strip it, then read remaining digits as home+away goals (e.g., `142` → `4:2`).
4. **Compound label\_ids** — in markets 56, 57, 59: first digit is team (1=Home, 2=Away), second digit is outcome.
5. **Next Goal line property** — for market 22, the goal number (1st, 2nd, etc.) is in the `line` field of the Odd object, not the label\_id.