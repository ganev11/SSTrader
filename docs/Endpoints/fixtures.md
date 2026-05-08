---
title: Fixtures
excerpt: >-
  The `/fixtures` endpoint is the central hub for retrieving sports match data.
  While a standard request returns core match details (teams, time, score), you
  can use the `include` parameter to enrich the response with deep analytical
  and market data.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Query Parameters

| Parameter    | Type      | Description                                             |
| :----------- | :-------- | :------------------------------------------------------ |
| `sport_id`   | `integer` | The ID of the sport. Defaults to `1` (Football/Soccer). |
| `league_id`  | `string`  | Comma-separated list of league IDs (e.g., `12,45`).     |
| `fixture_id` | `string`  | Comma-separated list of specific fixture IDs.           |
| `date`       | `string`  | Date in `YYYY-MM-DD` format.                            |
| `is_live`    | `integer` | Set to `1` to filter for currently active matches.      |
| `language`   | `string`  | Localization code for names (default: `en`).            |
| `include`    | `string`  | Comma-separated list of data modules to expand.         |

***

## The `include` Parameter

Use the `include` parameter to customize the data payload. This allows you to fetch exactly what you need in a single request.

### Supported Includes

| Value        | Description                                                                                                                           |
| :----------- | :------------------------------------------------------------------------------------------------------------------------------------ |
| `odds`       | Injects betting odds from available bookmakers into each fixture object.                                                              |
| `statistics` | Adds match statistics (e.g., attacks, shots on goal, possession). Returns live data if the match is in progress.                      |
| `metrics`    | Adds advanced algorithmic metrics and calculated performance indicators for the fixture.                                              |
| `fair_odds`  | Adds "Fair Value" pricing. This calculates what the odds should be based on statistical models, helping identify value in the market. |
| `movement`   | Includes historical odds movement data, showing how the market lines have changed over time.                                          |

## Pagination

**This endpoint returns paginated results.** Responses include up to 50 results per page by default. Use the `page`, `per_page`, and `order` query parameters to navigate through results. Check the `has_more` field in the `pagination` object to determine if additional pages are available.

See the [Pagination guide](/docs/pagination) for details on navigating pages and controlling result size.

***

## Examples

### 1. Match Day Overview with Odds

Retrieve all matches for a specific date with current betting market data:
`GET /fixtures?date=2024-10-254&include=odds`

### 2. Live Match Analysis

Get live matches with real-time statistics to track performance:<br />`GET /fixtures?is_live=1&include=statistics`

### 3. Deep Analytical View

Get a specific fixture with advanced metrics, fair value calculations, and market movement:
`GET /fixtures?fixture_id=88421&include=metrics,fair_odds,movement`

***

## Example Base Response

`GET /fixtures?fixture_id=123`

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