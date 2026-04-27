---
title: Scores
deprecated: false
hidden: false
metadata:
  robots: index
---
## Score Schema Documentation

The `scores` array provides a granular breakdown of points or goals scored by each team, categorized by specific periods of the match.
------------------------------
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

* 1ST_HALF: Total goals scored during the first half.
* 2ND_HALF: The score at the end of the second half (usually includes 1st half goals).
* 2ND_HALF_ONLY: Goals scored exclusively during the second half period.
* CURRENT: The total live score for the team at the current moment.

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
