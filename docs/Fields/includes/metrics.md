---
title: Metrics
excerpt: Provides AI-driven, machine-learned insights/metrics.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Metrics

The `metrics` include provides AI-driven, machine-learned insights—such as predictions and smart money indicators—categorized by domain-specific groups to help identify advanced match trends.

> Availability: This data is only included in the response when explicitly requested using the query parameter `include=metrics`.

## Object Schema

| Field           | Type    | Description                                                                           |
| --------------- | ------- | ------------------------------------------------------------------------------------- |
| team\_id        | integer | The ID of the team the metric relates to (uses 0 for neutral or match-level metrics). |
| type\_id        | integer | The unique ID associated with the metric type.                                        |
| developer\_name | string  | The unique constant identifier (e.g., PREDICTION\_GL\_CONFIDENCE).                    |
| value           | number  | The calculated value or probability of the metric.                                    |
| meta            | object  | Optional. Additional context such as reasons, line, or labels. Omitted if null.       |

> Dynamic Metadata
> The `meta` object is dynamic. Its properties (such as line, reasons, or note) vary based on the `developer_name`. Developers should implement flexible parsing for this object as fields are specific to the individual metric type.

## Filtering Metrics

You can filter metrics by specific IDs or entire domain groups using the following patterns:

- By Type IDs: filter\[metrics]=types:1,2,3
- By Developer Types: filter\[metrics]=developer\_types:predictions,smartmoney

## Example Response

```json
"metrics": [
  {
    "team_id": 0,
    "type_id": 105,
    "developer_name": "PREDICTION_GL_CONFIDENCE",
    "value": 70,
    "meta": {
      "note": "Base Confidence=70 (Clear xMetric advantage), Bonus=0, Final Confidence=70"
    }
  },
  {
    "team_id": 0,
    "type_id": 106,
    "developer_name": "PREDICTION_GL_UNDER",
    "value": 2.93,
    "meta": {
      "line": 3,
      "reasons": ["Both teams create few dangerous attacks"],
      "market_id": 3
    }
  }
]
```

<br />
