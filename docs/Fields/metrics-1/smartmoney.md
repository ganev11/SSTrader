---
title: SmartMoney
deprecated: false
hidden: false
metadata:
  robots: index
---
## SmartMoney metrics

- Purpose: Quantify notable market support for a specific outcome by measuring a bookmaker's (or aggregated) price deviation versus the market. Values are expressed as a percentage and represent relative deviation used to surface potential market-driven value.
- Where to use: Attach to `fixture.metrics` for frontend display, alerts, filtering rules, signal generation, or manual review in pre-match analytics. Use the `developer_name` to identify the metric, `value` as the magnitude (percentage), and `meta` for contextual routing (contains `market_id`, `label_id`, `line`).
- Format notes: Example metric shows the expected JSON shape. Keep metrics read-only in consumers; treat them as signals, not definitive bets.

| type\_id | developer\_name        | description                                                                                       |
| -------- | ---------------------- | ------------------------------------------------------------------------------------------------- |
| 224      | SMARTMONEY\_GL\_OVER   | Indicates market movement toward higher goal expectation (support for the Over on the Goal Line). |
| 225      | SMARTMONEY\_GL\_UNDER  | Indicates market movement toward lower goal expectation (support for the Under on the Goal Line). |
| 226      | SMARTMONEY\_COR\_OVER  | Indicates increased market support for more corners (Over on Corners market).                     |
| 227      | SMARTMONEY\_COR\_UNDER | Indicates increased market support for fewer corners (Under on Corners market).                   |
| 229      | SMARTMONEY\_YC\_UNDER  | Indicates increased market support for fewer yellow cards (Under on Yellow Cards market).         |
| 228      | SMARTMONEY\_YC\_OVER   | Indicates increased market support for more yellow cards (Over on Yellow Cards market).           |
| 230      | SMARTMONEY\_AH\_HOME   | Indicates increased market support for the home side on the Asian Handicap market.                |
| 231      | SMARTMONEY\_AH\_AWAY   | Indicates increased market support for the away side on the Asian Handicap market.                |
| 232      | SMARTMONEY\_ML\_HOME   | Indicates increased market support for the home outcome on the Money Line market.                 |
| 233      | SMARTMONEY\_ML\_DRAW   | Indicates increased market support for the draw outcome on the Money Line market.                 |
| 234      | SMARTMONEY\_ML\_AWAY   | Indicates increased market support for the away outcome on the Money Line market.                 |

Example smartmoney metric with `meta` property:

```json
{
    "fixture_id": 12345,
    "team_id": 0,
    "developer_name": "SMARTMONEY_GL_OVER",
    "value": 2.3,
    "meta": {
        "market_id": 3,
        "label_id": 1,
        "line": 2.5
    }
}
```

<br />
