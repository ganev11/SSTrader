---
title: Predictions
excerpt: Prediction metrics
deprecated: false
hidden: false
metadata:
  robots: index
---
## Overview — Prediction metrics
These metrics are structured numeric signals attached to a fixture's `metrics` array that
describe predicted match behavior (for example: goals, corners, yellow cards, shot scores,
and pressure).
Use these metrics in the frontend, filters, analytics dashboards, or betting pipelines to
display prediction scores, highlight advantages, show confidence, map predictions to market
lines, and provide short explanatory context for users.
When present, the optional `meta`
object supplies human-friendly details such as reasons, the derived betting line, market and
label ids, or concise notes useful for UI tooltips and downstream workflows.

| type_id | developer_name | description |
| --- | --- | --- |
| 94 | PREDICTION_GL_OVER | Prediction that total match goals will be over the Asian goal line; value is combined expected goals (xG). |
| 95 | PREDICTION_GL_UNDER | Prediction that total match goals will be under the Asian goal line; value is combined expected goals (xG). |
| 97 | PREDICTION_GL_DIFF | Absolute difference between home and away expected goals (xG diff). |
| 98 | PREDICTION_GL_LEADER | Leader indicator for goals: 1 = home leading, 2 = away leading, 0 = even. |
| 96 | PREDICTION_GL_CONFIDENCE | Confidence score (30-95) for the goals prediction; meta contains base/bonus breakdown. |
| 99 | PREDICTION_COR_OVER | Prediction that total corners will be over the Asian corners line; value is combined expected corners. |
| 100 | PREDICTION_COR_UNDER | Prediction that total corners will be under the Asian corners line; value is combined expected corners. |
| 102 | PREDICTION_COR_DIFF | Difference between home and away expected corners. |
| 103 | PREDICTION_COR_LEADER | Leader indicator for corners: 1 = home, 2 = away, 0 = even. |
| 101 | PREDICTION_COR_CONFIDENCE | Confidence score (30-95) for the corners prediction; meta contains base/bonus breakdown. |
| 104 | PREDICTION_YC_OVER | Prediction that total yellow cards will be over the main line; value is combined expected yellow cards. |
| 105 | PREDICTION_YC_UNDER | Prediction that total yellow cards will be under the main line; value is combined expected yellow cards. |
| 107 | PREDICTION_YC_DIFF | Difference between home and away expected yellow cards. |
| 108 | PREDICTION_YC_LEADER | Leader indicator for yellow cards: 1 = home, 2 = away, 0 = even. |
| 106 | PREDICTION_YC_CONFIDENCE | Confidence score (30-95) for the yellow cards prediction; meta contains base/bonus breakdown. |
| 109 | PREDICTION_HOME_SHOTS_SCORE | Composite home shots-on-target score combining home attack with opponent shot-defense. |
| 110 | PREDICTION_AWAY_SHOTS_SCORE | Composite away shots-on-target score combining away attack with opponent shot-defense. |
| 111 | PREDICTION_SHOTS_ADVANTAGE | Advantage indicator for shots-on-target: 1 = home, 2 = away, 0 = even. |
| 112 | PREDICTION_HOME_DANGEROUS_SCORE | Composite home dangerous-attacks score combining home dangerous attack power and opponent defense. |
| 113 | PREDICTION_AWAY_DANGEROUS_SCORE | Composite away dangerous-attacks score combining away dangerous attack power and opponent defense. |
| 114 | PREDICTION_DANGEROUS_ADVANTAGE | Advantage indicator for dangerous attacks: 1 = home, 2 = away, 0 = even. |
| 115 | PREDICTION_TOTAL_ATTACK_PRESSURE | Aggregate attack pressure metric summing dangerous attacks and goal-attack powers; meta.classification = Low/Medium/High. |
| 116 | PREDICTION_CONFIDENCE_SCORE | Composite overall confidence (20-95) combining xMetric diffs and enhanced indicators. |

## Examples for prediction metrics that include a `meta` property

PREDICTION_GL_OVER
```json
{
    "type_id": 94,
    "team_id": 0,
    "developer_name": "PREDICTION_GL_OVER",
    "value": 3.20,
    "meta": {
        "reasons": ["Both teams have weak defenses (>1.1)", "High combined x-goals (>2.75)"],
        "line": 2.75,
        "market_id": 3,
        "label_id": 2
    }
}
```

PREDICTION_GL_UNDER
```json
{
    "type_id": 95,
    "team_id": 0,
    "developer_name": "PREDICTION_GL_UNDER",
    "value": 1.90,
    "meta": {
        "reasons": ["Both teams have strong defenses (<0.9)"],
        "line": 2.75,
        "market_id": 3,
        "label_id": 1
    }
}
```

PREDICTION_GL_CONFIDENCE
```json
{
    "type_id": 96,
    "team_id": 0,
    "developer_name": "PREDICTION_GL_CONFIDENCE",
    "value": 78,
    "meta": {
        "note": "Base Confidence=70 (Clear xMetric advantage), Bonus=8, Final Confidence=78"
    }
}
```

PREDICTION_COR_OVER
```json
{
    "type_id": 99,
    "team_id": 0,
    "developer_name": "PREDICTION_COR_OVER",
    "value": 12.0,
    "meta": {
        "reasons": ["Strong home corner attack", "Opponent concedes many corners"],
        "line": 10.5,
        "market_id": 6,
        "label_id": 2
    }
}
```

PREDICTION_COR_UNDER
```json
{
    "type_id": 100,
    "team_id": 0,
    "developer_name": "PREDICTION_COR_UNDER",
    "value": 8.5,
    "meta": {
        "reasons": ["Both teams have weak corner attacks (<0.9)"],
        "line": 10.5,
        "market_id": 6,
        "label_id": 1
    }
}
```

PREDICTION_COR_CONFIDENCE
```json
{
    "type_id": 101,
    "team_id": 0,
    "developer_name": "PREDICTION_COR_CONFIDENCE",
    "value": 66,
    "meta": {
        "note": "Base Confidence=60 (Slight xMetric advantage), Bonus=6, Final Confidence=66"
    }
}
```

PREDICTION_YC_OVER
```json
{
    "type_id": 104,
    "team_id": 0,
    "developer_name": "PREDICTION_YC_OVER",
    "value": 5.2,
    "meta": {
        "reasons": ["High card tendency by leader", "Opponent draws fouls"],
        "line": 4.25,
        "market_id": 203,
        "label_id": 2
    }
}
```

PREDICTION_YC_UNDER
```json
{
    "type_id": 105,
    "team_id": 0,
    "developer_name": "PREDICTION_YC_UNDER",
    "value": 3.8,
    "meta": {
        "reasons": ["Both teams have disciplined defenses (<0.9)"],
        "line": 4.25,
        "market_id": 203,
        "label_id": 1
    }
}
```

PREDICTION_YC_CONFIDENCE
```json
{
    "type_id": 106,
    "team_id": 0,
    "developer_name": "PREDICTION_YC_CONFIDENCE",
    "value": 59,
    "meta": {
        "note": "Base Confidence=50 (Close match), Bonus=9, Final Confidence=59"
    }
}
```

PREDICTION_TOTAL_ATTACK_PRESSURE
```json
{
    "type_id": 115,
    "team_id": 0,
    "developer_name": "PREDICTION_TOTAL_ATTACK_PRESSURE",
    "value": 4.8,
    "meta": {
        "classification": "High"
    }
}
```