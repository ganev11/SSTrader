---
title: Insights
excerpt: >-
  Access a stream of AI-generated betting recommendations that combine real-time
  market data with natural language analysis to identify high-probability
  outcomes.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Insights Endpoint Documentation (/insights)

The Insights endpoint returns AI-generated analysis and betting recommendations. Each insight pairs a natural language explanation with a specific market selection (the `bet`).

The Insights endpoint provides a list of match fixtures, where each fixture contains an insights array. This structure allows you to display the AI analysis directly alongside the relevant match data, team names, match metrics and live scores.

***

## Query Parameters

| Parameter   | Type    | Default   | Description                                                   |
| ----------- | ------- | --------- | ------------------------------------------------------------- |
| model\_id   | string  |           | Comma-separated list of Models IDs (e.g., 1,2,3).             |
| market\_id  | string  |           | Comma-separated list of Market IDs (e.g., 1,3).               |
| status      | string  | See below | Comma-separated list of Statuses to filter by.                |
| is\_live    | integer | 0         | Use 1 to filter only for events currently in-play.            |
| expired     | integer | 0         | Use 1 to include outdated insights (useful for backend sync). |
| league\_id  | string  |           | Comma-separated list of League IDs.                           |
| fixture\_id | integer |           | Restrict results to a specific single fixture.                |
| language    | string  | en        | Language code for the generated content.                      |
| value\_from | number  |           | Minimum decimal odd value.                                    |
| value\_to   | number  |           | Maximum decimal odd value.                                    |
| limit       | integer | 100       | Max results (1–1000).                                         |

## Default Statuses

If no status is provided, the API filters by these active match states:
INPLAY\_1ST\_HALF, INPLAY\_2ND\_HALF, INPLAY\_ET, INPLAY\_PENALTIES, HT, EXTRA\_TIME\_BREAK, PEN\_BREAK, NOT\_STARTED.

***

## Insight Object

| Field       | Type    | Description                                                                |
| ----------- | ------- | -------------------------------------------------------------------------- |
| id          | integer | Unique internal identifier for the insight.                                |
| `market_id`  | integer | The ID of the market the insight refers to.                                |
| `value`       | number  | The current decimal odd value for the recommendation.                      |
| `sp`          | number  | Starting Price: The odd value at the exact time the insight was generated. |
| `created_at` | string  | ISO 8601 timestamp of generation.                                          |
| content     | object  | Dynamic. This can hold any key-value pairs.                                |
| model       | object  | Metadata about the AI model used (id, name, color).                        |
| bet         | object  | The recommended selection. Follows the Odd Schema.                         |

## Example JSON Response

```json
[
  {
    "id": 1039720,
    "date_time": "2026-04-27T15:00:00.000Z",
    "status": "FT",
    "country": {
      "id": 2,
      "name": "Poland",
      "alpha3": "POL"
    },
    "league": {
      "id": 158,
      "name": "Ekstraklasa",
      "level": 2
    },
    "season": {
      "id": 16267,
      "name": "2025/2026"
    },
    "sport": {
      "id": 1,
      "name": "Football"
    },
    "participants": [
      {
        "id": 1397,
        "name": "Piast Gliwice",
        "position": 16,
        "location": "home"
      },
      {
        "id": 772,
        "name": "Arka Gdynia",
        "position": 17,
        "location": "away"
      }
    ],
    "is_live": false,
    "language": "en",
    "scores": [],
    "periods": [],
    "insights": [
      {
        "id": 38666,
        "user_id": 5,
        "model_id": 15,
        "job_id": 43483,
        "market_id": 22,
        "value": 0,
        "suspend": 1,
        "sp": 2.5,
        "created_at": "2026-04-27T17:26:12.709Z",
        "language": "en",
        "content": {
          "text": "Arka are suddenly all over them, piling on dangerous breaks and testing the keeper with real purpose. Momentum’s flipped, Piast look rattled, and the away side appear poised to cash in and grab this next crucial goal."
        },
        "model": {
          "id": 15,
          "name": "test - next goal away",
          "color": "#000000"
        },
        "bet": {
          "odd_id": 90004836,
          "market_id": 22,
          "bookmaker_id": 4,
          "is_live": 1,
          "label_id": 2,
          "value": 0,
          "handicap": 2,
          "line": 2,
          "last_update": 1777316400,
          "suspend": 1,
          "sp": 2.571,
          "home_score": 0,
          "away_score": 0,
          "status": -1,
          "raw": {
            "event_id": 15078878,
            "market_id": 1540374267,
            "selection_id": 3897429409,
            "market_type_id": 8,
            "selection_type_id": 3
          },
          "outcome": "Lost",
          "market_description": "Which team will score the next goal",
          "market_name": "Second Goal",
          "label_name": "Arka Gdynia"
        }
      }
    ]
  }
]
```

<br />