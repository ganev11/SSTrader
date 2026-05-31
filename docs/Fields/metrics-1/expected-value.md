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

- `EUROPE` — bookmaker\_id = 2
- `ASIA`   — bookmaker\_id = 3

| type\_id | developer\_name        | market         | market\_id | label     | description                                                                     |
| -------- | ---------------------- | -------------- | ---------- | --------- | ------------------------------------------------------------------------------- |
| 281      | EV\_EUROPE\_ML\_HOME   | Money Line     | 1          | Home win  | EV% for European bookmaker home-win price vs xG-derived fair odd.               |
| 282      | EV\_EUROPE\_ML\_AWAY   | Money Line     | 1          | Away win  | EV% for European bookmaker away-win price vs xG-derived fair odd.               |
| 283      | EV\_EUROPE\_ML\_DRAW   | Money Line     | 1          | Draw      | EV% for European bookmaker draw price vs xG-derived fair odd.                   |
| 284      | EV\_ASIA\_ML\_HOME     | Money Line     | 1          | Home win  | EV% for Asian bookmaker home-win price vs xG-derived fair odd.                  |
| 285      | EV\_ASIA\_ML\_DRAW     | Money Line     | 1          | Draw      | EV% for Asian bookmaker draw price vs xG-derived fair odd.                      |
| 286      | EV\_ASIA\_ML\_AWAY     | Money Line     | 1          | Away win  | EV% for Asian bookmaker away-win price vs xG-derived fair odd.                  |
| 287      | EV\_EUROPE\_AH\_HOME   | Asian Handicap | 2          | Home side | EV% for European bookmaker AH home price on the main handicap line.             |
| 288      | EV\_EUROPE\_AH\_AWAY   | Asian Handicap | 2          | Away side | EV% for European bookmaker AH away price on the main handicap line.             |
| 289      | EV\_ASIA\_AH\_HOME     | Asian Handicap | 2          | Home side | EV% for Asian bookmaker AH home price on the main handicap line.                |
| 290      | EV\_ASIA\_AH\_AWAY     | Asian Handicap | 2          | Away side | EV% for Asian bookmaker AH away price on the main handicap line.                |
| 291      | EV\_EUROPE\_GL\_OVER   | Goal Line      | 3          | Over      | EV% for European bookmaker goal-line over price on the main total line.         |
| 292      | EV\_EUROPE\_GL\_UNDER  | Goal Line      | 3          | Under     | EV% for European bookmaker goal-line under price on the main total line.        |
| 301      | EV\_ASIA\_GL\_UNDER    | Goal Line      | 3          | Under     | EV% for Asian bookmaker goal-line under price on the main total line.           |
| 302      | EV\_ASIA\_GL\_OVER     | Goal Line      | 3          | Over      | EV% for Asian bookmaker goal-line over price on the main total line.            |
| 299      | EV\_EUROPE\_COR\_UNDER | Corner Line    | 6          | Under     | EV% for European bookmaker corner-line under price on the main total line.      |
| 300      | EV\_EUROPE\_COR\_OVER  | Corner Line    | 6          | Over      | EV% for European bookmaker corner-line over price on the main total line.       |
| 297      | EV\_ASIA\_COR\_UNDER   | Corner Line    | 6          | Under     | EV% for Asian bookmaker corner-line under price on the main total line.         |
| 298      | EV\_ASIA\_COR\_OVER    | Corner Line    | 6          | Over      | EV% for Asian bookmaker corner-line over price on the main total line.          |
| 295      | EV\_EUROPE\_YC\_UNDER  | Yellow Cards   | 203        | Under     | EV% for European bookmaker yellow-card line under price on the main total line. |
| 296      | EV\_EUROPE\_YC\_OVER   | Yellow Cards   | 203        | Over      | EV% for European bookmaker yellow-card line over price on the main total line.  |
| 293      | EV\_ASIA\_YC\_UNDER    | Yellow Cards   | 203        | Under     | EV% for Asian bookmaker yellow-card line under price on the main total line.    |
| 294      | EV\_ASIA\_YC\_OVER     | Yellow Cards   | 203        | Over      | EV% for Asian bookmaker yellow-card line over price on the main total line.     |

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

## Examples for EV metrics

EV\_EUROPE\_ML\_HOME

```json
{
    "fixture_id": 12345,
    "type_id": 281,
    "team_id": 0,
    "developer_name": "EV_EUROPE_ML_HOME",
    "value": 8
}
```

Interpretation: European bookmaker's home-win price is 8% above the model's fair odd — value bet.

EV\_ASIA\_ML\_DRAW

```json
{
    "fixture_id": 12345,
    "type_id": 285,
    "team_id": 0,
    "developer_name": "EV_ASIA_ML_DRAW",
    "value": -5
}
```

Interpretation: Asian bookmaker's draw price is 5% below fair value — no edge.

EV\_EUROPE\_AH\_HOME

```json
{
    "fixture_id": 12345,
    "type_id": 287,
    "team_id": 0,
    "developer_name": "EV_EUROPE_AH_HOME",
    "value": 12
}
```

Interpretation: European bookmaker's Asian Handicap home price on the main line is 12% above fair value.

EV\_ASIA\_AH\_AWAY

```json
{
    "fixture_id": 12345,
    "type_id": 290,
    "team_id": 0,
    "developer_name": "EV_ASIA_AH_AWAY",
    "value": -3
}
```

Interpretation: Asian bookmaker's Asian Handicap away price on the main line is 3% below fair value.

EV\_EUROPE\_GL\_OVER

```json
{
    "fixture_id": 12345,
    "type_id": 291,
    "team_id": 0,
    "developer_name": "EV_EUROPE_GL_OVER",
    "value": 6
}
```

Interpretation: European bookmaker's goal-line over price on the main total line is 6% above fair value.

EV\_ASIA\_GL\_UNDER

```json
{
    "fixture_id": 12345,
    "type_id": 301,
    "team_id": 0,
    "developer_name": "EV_ASIA_GL_UNDER",
    "value": -10
}
```

Interpretation: Asian bookmaker's goal-line under price is 10% below fair value.

EV\_EUROPE\_COR\_OVER

```json
{
    "fixture_id": 12345,
    "type_id": 300,
    "team_id": 0,
    "developer_name": "EV_EUROPE_COR_OVER",
    "value": 9
}
```

Interpretation: European bookmaker's corner-line over price on the main line is 9% above fair value.

EV\_ASIA\_COR\_UNDER

```json
{
    "fixture_id": 12345,
    "type_id": 297,
    "team_id": 0,
    "developer_name": "EV_ASIA_COR_UNDER",
    "value": 4
}
```

Interpretation: Asian bookmaker's corner-line under price is 4% above fair value.

EV\_EUROPE\_YC\_OVER

```json
{
    "fixture_id": 12345,
    "type_id": 296,
    "team_id": 0,
    "developer_name": "EV_EUROPE_YC_OVER",
    "value": 7
}
```

Interpretation: European bookmaker's yellow-card line over price is 7% above fair value.

EV\_ASIA\_YC\_UNDER

```json
{
    "fixture_id": 12345,
    "type_id": 293,
    "team_id": 0,
    "developer_name": "EV_ASIA_YC_UNDER",
    "value": -2
}
```

Interpretation: Asian bookmaker's yellow-card line under price is 2% below fair value.
