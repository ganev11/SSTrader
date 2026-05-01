---
title: Periods
excerpt: >-
  The periods array provides technical timing data for each segment of the
  match, including timestamps, added time, and clock state.
deprecated: false
hidden: false
metadata:
  robots: index
---

## Field Definitions

| Field           | Type    | Description                                                               |
| --------------- | ------- | ------------------------------------------------------------------------- |
| period_id      | integer | Unique identifier for this specific period instance.                      |
| fixture\_id     | integer | Unique identifier for the associated match.                               |
| started         | integer | Unix timestamp indicating when the period began.                          |
| ended           | integer | Unix timestamp indicating when the period finished (null if live).        |
| counts\_from    | integer | The minute the clock starts at for this period (e.g., 45 for 2nd half).   |
| ticking         | integer | Boolean-style flag (0 or 1) indicating if the clock is currently running. |
| sort\_order     | integer | The chronological sequence of the period.                                 |
| time\_added     | integer | The amount of injury/stoppage time added by the referee.                  |
| period\_length  | integer | The regulation duration of the period in minutes (usually 45).            |
| minutes         | integer | Total minutes elapsed since the start of the match.                       |
| seconds         | integer | Additional seconds elapsed beyond the full minutes.                       |
| developer\_name | enum    | Machine-readable name (matches the keys in the Score schema).             |

------------------------------

## Developer Name Values

The developer_name field uses the following constant values to describe the match state:

| type_id | Value | Description |
|---|---|---|
| 1 | 1ST_HALF | Total goals scored during the first half. |
| 2 | 2ND_HALF | The total score at the end of the second half. |
| 3 | 2ND_HALF_ONLY | Goals scored exclusively during the second half period. |
| 4 | ET_1ST_HALF | Goals scored during the first period of extra time. |
| 5 | ET_2ND_HALF | Goals scored during the second period of extra time. |
| 6 | ET | Total score at the conclusion of all extra time periods. |
| 7 | PENALTY_SHOOTOUT | Total successful penalties recorded in a shootout. |
| 8 | CURRENT | The total live score for the team at the current moment. |

> This types are from developer_type: period

## Example JSON

```json
{
    "periods": [
        {
            "period_id": 2045635,
            "fixture_id": 1033370,
            "started": 1776875170,
            "ended": 1776878125,
            "counts_from": 0,
            "ticking": 0,
            "sort_order": 1,
            "time_added": 4,
            "period_length": 45,
            "minutes": 49,
            "seconds": 15,
            "developer_name": "1ST_HALF"
        }
    ]
}
```

<br />