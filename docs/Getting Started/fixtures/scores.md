---
title: Scores
excerpt: >-
  The `scores` array provides a granular breakdown of points or goals scored by
  each team, categorized by specific periods of the match.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Field Definitions

| Field | Type | Description |
|---|---|---|
| fixture_id | integer | Unique identifier for the specific match/event. |
| team_id | integer | Unique identifier for the team the score belongs to. |
| value | integer | The numerical score/goals recorded for the period. |
| developer_name | enum | A machine-readable string identifying the time period. |

------------------------------
## Developer Name Values
The developer_name field uses the following constant values to describe the match state:

| Value | Description |
|---|---|
| 1ST_HALF | Total goals scored during the first half. |
| 2ND_HALF | The total score at the end of the second half. |
| 2ND_HALF_ONLY | Goals scored exclusively during the second half period. |
| ET_1ST_HALF | Goals scored during the first period of extra time. |
| ET_2ND_HALF | Goals scored during the second period of extra time. |
| ET | Total score at the conclusion of all extra time periods. |
| PENALTY_SHOOTOUT | Total successful penalties recorded in a shootout. |
| CURRENT | The total live score for the team at the current moment. |

------------------------------
## Example Implementation

```json
{
  "scores": [
    {
      "fixture_id": 1033370,
      "team_id": 3585,
      "value": 2,
      "developer_name": "1ST_HALF"
    },
    {
      "fixture_id": 1033370,
      "team_id": 3585,
      "value": 4,
      "developer_name": "CURRENT"
    }
  ]
}
```

<br />