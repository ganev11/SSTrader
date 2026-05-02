---
title: Statistics
excerpt: >-
  The statistics include provides granular, real-time and post-match performance
  metrics, such as possession and shots, for each team.
deprecated: false
hidden: false
metadata:
  robots: index
---

## Statistics Schema Documentation
The statistics include provides granular match data for each team. These metrics allow you to track performance indicators such as possession, shots, and corners in real-time or post-match.

> Availability: This data is only included in the response when explicitly requested using the query parameter include=statistics.

## Object Schema

| Field | Type | Description |
|---|---|---|
| team_id | integer | The unique ID of the team the statistic belongs to. |
| value | number | The numerical value of the specific metric (e.g., total count or percentage). |
| type_id | integer | The unique ID associated with the statistic type. |
| developer_name | string | The unique, constant-style identifier used for programming logic (e.g., CORNERS, SHOTS_TOTAL). |

## Key Features

* Developer-Friendly: Use the developer_name field for consistent data mapping across different matches and leagues.
* Team-Specific: Data is categorized by team_id, allowing for easy comparison between the home and away sides.
* Comprehensive: Covers all match actions including offensive, defensive, and discipline-related metrics.

## Example Response

```json
"statistics": [
  {
    "team_id": 108,
    "value": 6,
    "type_id": 34,
    "developer_name": "CORNERS"
  },
  {
    "team_id": 108,
    "value": 20,
    "type_id": 52,
    "developer_name": "SHOTS_TOTAL"
  }
]
```

## Filtering Statistics

You can narrow down the returned statistics to specific metrics by applying a filter. This is highly recommended to reduce payload size and improve performance if you only need specific data points (like Corners or Possession).
## Filter Pattern
Use the filter[statistics]=types: key followed by a comma-separated list of type_id values.

| Parameter | Format | Description |
|---|---|---|
| filter[statistics] | types:id1,id2,id3 | Filters the statistics array to only include the specified type_id values. |

## Example Request

To fetch only specific statistics (e.g., where Type IDs are 2, 3, and 4):
GET /v1/fixtures?include=statistics&filter[statistics]=types:2,3,4

## Example Filtered Response
The API will only return the metrics matching your requested IDs:

```json
"statistics": [
  {
    "team_id": 108,
    "value": 8,
    "type_id": 2,
    "developer_name": "SHOTS_OFF_TARGET"
  },
  {
    "team_id": 108,
    "value": 12,
    "type_id": 3,
    "developer_name": "SHOTS_ON_TARGET"
  }
]
```