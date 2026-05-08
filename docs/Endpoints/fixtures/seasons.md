---
title: Seasons
excerpt: >-
  The seasons include provides a historical and upcoming breakdown of match days
  for the league. This is useful for building calendar views or identifying
  active competition dates.
deprecated: false
hidden: true
metadata:
  robots: index
---
## Field Definitions

| Field | Type | Description |
|---|---|---|
| id | integer | Unique identifier for the specific league season. |
| name | string | The display name of the season (e.g., "2025/2026"). |
| days | array[string] | A list of dates where at least one match is scheduled. |

------------------------------
## Date Format

The dates within the days array are strings formatted according to ISO 8601 standards (YYYY-MM-DD).

------------------------------
## Example JSON Response

```json
{
  "seasons": [
    {
      "id": 17004,
      "name": "2025/2026",
      "days": [
        "2025-09-11",
        "2025-09-12",
        "2025-09-13"
      ]
    },
    {
      "id": 14489,
      "name": "2024/2025",
      "days": [
        "2024-08-19",
        "2024-08-20"
      ]
    }
  ]
}
```