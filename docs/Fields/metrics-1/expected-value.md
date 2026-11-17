---
title: Expected Value
deprecated: false
hidden: false
metadata:
  robots: index
---
---
title: Expected Values
description: EV metrics — the edge between a bookmaker's price and the model's fair price on the main line of Money Line, Asian Handicap, goals, corners and cards, with the fair price in meta.
---

# Expected values (EV)

## What they are

An EV metric is the edge between a bookmaker's price and the model's fair price for one
selection on the main line:

```
EV = round((bookmaker price - fair_odd) / fair_odd * 100)
```

Positive means the bookmaker pays more than the model's fair price; negative means less.
EV metrics are fixture-level (`team_id` 0) and pre-match only. They are part of the `metrics`
include (`GET /football/fixtures?include=metrics`) and can be used in
`GET /football/fixtures/search` with `where[...]` and `order[...]` (see [search.md](search.md)).

The region in the name is the bookmaker: `EUROPE` = bookmaker 2, `ASIA` = bookmaker 3.

## Metrics

| type\_id | developer\_name | market (`market_id`) | fair price from |
| -------- | --------------- | -------------------- | --------------- |
| 281-283 | `EV_EUROPE_ML_HOME`, `EV_EUROPE_ML_AWAY`, `EV_EUROPE_ML_DRAW` | Money Line (1) | `FT_EXPECTED_GOALS_V2` |
| 284-286 | `EV_ASIA_ML_HOME`, `EV_ASIA_ML_DRAW`, `EV_ASIA_ML_AWAY` | Money Line (1) | `FT_EXPECTED_GOALS_V2` |
| 287-290 | `EV_{EUROPE\|ASIA}_AH_{HOME\|AWAY}` | Asian Handicap (2) | `FT_EXPECTED_GOALS_V2` |
| 406-409 | `EV_{EUROPE\|ASIA}_GL_{OVER\|UNDER}_V2` | Goal Line (3) | `FT_TOTAL_GOALS_V2` |
| 410-413 | `EV_{EUROPE\|ASIA}_COR_{OVER\|UNDER}_V2` | Corners (6) | `FT_TOTAL_CORNERS_V2` |
| 414-417 | `EV_{EUROPE\|ASIA}_CARDS_{OVER\|UNDER}_V2` | Cards (203) | `FT_TOTAL_CARDS_V2` |

Money Line and Asian Handicap are priced from the two teams' expected goals. Over/under markets
are priced from the published distribution of the match total, which is narrower and better
centred than adding the two teams' means together.

Market 203 is settled as yellow cards + 2 x red cards, so its EV is called `CARDS`, not
yellow cards.

## `meta`

| field | meaning |
| ----- | ------- |
| `market_id` | The market, as in the odds. |
| `label_id` | The outcome, as in the odds (Money Line 1 = home, 0 = draw, 2 = away; AH 1 = home, 2 = away; over/under 1 = under, 2 = over). |
| `line` | The bookmaker's main line the EV was priced on (home handicap for AH; 0 for Money Line). |
| `bookmaker_id` | 2 or 3, same as the region in the name. |
| `fair_odd` | The model's fair decimal price the EV was measured against. |
| `source` | The metric the fair price came from. |

`fair_odd` lets you check an EV by hand: `price / fair_odd - 1`.

## When there is no EV

A missing EV metric means no price, never zero. That happens when:

* the bookmaker has no pre-match price for the market;
* the fixture has no V2 expected goals (Money Line, AH), or no total distribution for that
  market (over/under). Cups and friendlies are not covered, and corners and cards cover fewer
  competitions than goals;
* the main line sits past the end of the total distribution.

## Retired metrics

`EV_{EUROPE|ASIA}_{GL|COR|YC}_{OVER|UNDER}` without `_V2` (type\_id 291-302) are no longer
published. They added the two teams' means together, which prices a total too wide, and `YC`
counted yellow cards only. Older fixtures may still carry them. Use the `_V2` metrics above.

## Example

```json
{
    "team_id": 0,
    "type_id": 411,
    "developer_name": "EV_EUROPE_COR_UNDER_V2",
    "value": 16,
    "meta": {
        "market_id": 6,
        "label_id": 1,
        "line": 11,
        "bookmaker_id": 2,
        "fair_odd": 1.704,
        "source": "FT_TOTAL_CORNERS_V2"
    }
}
```

Bookmaker 2 pays 1.975 for under 11 corners; the model's fair price is 1.704;
1.975 / 1.704 - 1 = 16%.

EV is a comparison with one bookmaker's price, not a betting recommendation. The closing
price is usually closer to the result than the model.
