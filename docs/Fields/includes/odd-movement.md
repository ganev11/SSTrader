---
title: Odds Movement
excerpt: >-
  A historical log of price fluctuations and market status updates used to
  analyze market trends and perform accurate backtesting.
deprecated: false
hidden: false
metadata:
  robots: index
---
---
title: Odds Movement
description: Full price history of a fixture's odds, filtered by bookmaker, market, pre-match or in-play, and time, for charts, backtests and incremental sync.
---

# Odds Movement

`include=movement` · `filter[movement]`

The `movement` array is the price history of a fixture's odds: one row each time a price or its suspended flag changed. Use it to chart a market, backtest a strategy, spot sharp moves ("steam") or study closing lines.

`movement` is an include of `GET /football/fixtures` (use `fixture_id=` for one or more fixtures). It is returned only when you ask for it with `include=movement`.

## Fields

| Field | Type | Description |
|---|---|---|
| `odd_id` | integer | The odd that changed. Matches `odd_id` in the `odds` include. |
| `value` | number | Decimal price after the change. |
| `suspend` | integer | `1` = suspended at this moment, `0` = open. |
| `last_update` | integer | Unix timestamp (seconds) when we received the change. |

A row does not repeat the odd's bookmaker, market, selection or line. Request `include=odds,movement` and match the two on `odd_id`.

Rows are ordered by `last_update`, then `odd_id`. Changes with the same `last_update` arrived in the same update.

## Filtering

`filter[movement]` narrows the history. Separate keys with `;` (all must match). Inside `bookmakers` and `markets`, separate values with `,` (any may match).

| Key | Values | Meaning |
|---|---|---|
| `bookmakers` | ids, e.g. `2,3` | Changes of these bookmakers' odds only. |
| `markets` | ids, e.g. `1,2,3` | Changes of odds in these markets only. |
| `is_live` | `0` or `1` | `0` = pre-match odds, `1` = in-play odds. |
| `since` | Unix timestamp (seconds) | Only changes with `last_update` at or after this time. |

- Without `filter[movement]`, every change of the fixture is returned: all bookmakers, all markets, the whole history. On an in-play fixture that can be tens of thousands of rows, so filter whenever you can.
- `bookmakers`, `markets` and `is_live` mean the same as in `filter[odds]`, with one difference: `filter[odds]` uses a default bookmaker list when you leave `bookmakers` out, `filter[movement]` does not (no `bookmakers` = every bookmaker). Send the same lists in both filters to get odds and history that match.
- `since` is for keeping a local copy up to date: store the largest `last_update` you hold and send it back as `since`. A change at exactly that second is sent again, so skip rows you already have by (`odd_id`, `last_update`).
- An unknown key, a value that is not a whole number, `is_live` other than `0`/`1`, or more than one value for `is_live` or `since` returns `400`.

## Example

Bookmaker 2's in-play 1X2, Asian handicap and goal line odds of one fixture, with their history since a given time:

```
GET /football/fixtures?fixture_id=1208520
    &include=odds,movement
    &filter[odds]=bookmakers:2;markets:1,2,3;is_live:1
    &filter[movement]=bookmakers:2;markets:1,2,3;is_live:1;since:1776776700
```

```json
{
  "fixtures": [
    {
      "id": 1208520,
      "odds": [
        { "odd_id": 88782664, "bookmaker_id": 2, "market_id": 3, "label_id": 2, "handicap": 2.5, "value": 1.5, "suspend": 1, "is_live": 1 }
      ],
      "movement": [
        { "odd_id": 88782664, "value": 1.403, "suspend": 0, "last_update": 1776776759 },
        { "odd_id": 88782664, "value": 1.45,  "suspend": 0, "last_update": 1776776812 },
        { "odd_id": 88782664, "value": 1.5,   "suspend": 1, "last_update": 1776776901 }
      ]
    }
  ]
}
```

(The `odds` row is shortened; see the odds include for every field.)

Reading the history of odd 88782664 (Over 2.5): it opened in this window at 1.403, drifted to 1.45 and then 1.50, and the last row shows the market suspended at 1.50, typically around a goal or another major incident.
