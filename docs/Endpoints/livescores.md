---
title: Livescores
excerpt: Get all Inplay fixtures.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Endpoint

`/livescores`

## Query Parameters

| Parameter   | Type      | Description                                             |
| :---------- | :-------- | :------------------------------------------------------ |
| `sport_id`  | `integer` | The ID of the sport. Defaults to `1` (Football/Soccer). |
| `league_id` | `string`  | Comma-separated list of league IDs (e.g., `12,45`).     |
| `language`  | `string`  | Localization code for names (default: `en`).            |
| `include`   | `string`  | Comma-separated list of data modules to expand.         |

***

## The `include` Parameter

Use the `include` parameter to customize the data payload. This allows you to fetch exactly what you need in a single request.

### Supported Includes

| Value        | Description                                                                                                      |   |
| :----------- | :--------------------------------------------------------------------------------------------------------------- | - |
| `odds`       | Injects betting odds from available bookmakers into each fixture object.                                         |   |
| `statistics` | Adds match statistics (e.g., attacks, shots on goal, possession). Returns live data if the match is in progress. |   |
| `metrics`    | Adds advanced algorithmic metrics and calculated performance indicators for the fixture.                         |   |

> The `odds` include will always return only odds with property `is_live` = 1. This means that you can work ONLY with inPlay match odds in this endpoint.

## Pagination

No

***

## Examples

### 1. Live Matches with Betting Odds

Retrieve all inplay matches with current betting market data:
`GET /livescores?include=odds`

### 2. Live Matches Analysis

Get live matches with real-time statistics to track performance:<br />`GET /livescores?include=statistics`

### 3. Deep Analytical View

Get a specific fixture with advanced metrics and odds for in-depth analysis:<br />
`GET /livescores?include=metrics,odds,statistics`

***

## Example Base Response

`GET /livescores`

```json
{
  "fixtures": [
    {
      "id": 123,
      "date_time": "2024-10-25T19:45:00.000Z",
      "status": "INPLAY_1ST_HALF",
      "is_live": true,
      "language": "en",
      "sport": {
        "id": 1,
        "name": "Football"
      },
      "country": {
        "id": 42,
        "name": "England",
        "alpha3": "GBR"
      },
      "league": {
        "id": 8,
        "name": "Premier League",
        "level": 1
      },
      "season": {
        "id": 21024,
        "name": "2024/2025"
      },
      "participants": [
        {
          "id": 1,
          "name": "Manchester United",
          "position": 5,
          "location": "home"
        },
        {
          "id": 2,
          "name": "Liverpool",
          "position": 2,
          "location": "away"
        }
      ],
      "scores": [],
      "periods": []
    }
  ]
}
```

> **Note:** If no score or period data is currently available for a fixture, these fields will return as empty arrays `[]`. Advanced data like **statistics** or **odds** are only appended when using the `include` parameter.

<br />