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
| `season_id`  | `integer` | The ID of the season to filter fixtures by.             |
| `date`       | `string`  | Date in `YYYY-MM-DD` format.                            |
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

**This endpoint returns paginated results.** Responses include up to 25 results per page by default. Use the `page`, `per_page`, and `order` query parameters to navigate through results. Check the `has_more` field in the `pagination` object to determine if additional pages are available.

| Field      | Type    | Description                                                          |
| ---------- | ------- | -------------------------------------------------------------------- |
| `page`     | integer | The current page number                                              |
| `per_page` | integer | Number of results returned on this page                              |
| `has_more` | boolean | `true` if more pages are available, `false` if this is the last page |
| `order`    | string  | Sort order — `asc` or `desc`                                         |

See the [Pagination guide](/docs/pagination) for details on navigating pages and controlling result size.

***

## Date Range

| Parameter    | Type                | Description                                                               |
| :----------- | :------------------ | :------------------------------------------------------------------------ |
| `start_date` | `string` (ISO 8601) | The beginning of the date range in UTC format (YYYY-MM-DDTHH:mm:ss.sssZ). |
| `end_date`   | `string` (ISO 8601) | The end of the date range in UTC format.                                  |

Validation Constraints
To ensure data integrity and system performance, the following rules are enforced:

- Data range works only when **both** `start_date` and `end_date` are set.
- Logical Order: `start_date` must be chronologically before or equal to end\_date.
- Historical Limit: `start_date` cannot be more than 30 days in the past relative to the current server time.
- Format: Both dates must be valid ISO 8601 strings. Invalid dates (e.g., 2023-13-45) will be rejected.

***

## Examples

### 1. Match Day Overview with Odds

Retrieve all matches for a specific date with current betting market data:
`GET /fixtures?date=2024-10-25&include=odds`

### 2. Deep Analytical View

Get a specific fixture with advanced metrics, fair value calculations, and market movement:
`GET /fixtures?fixture_id=88421&include=metrics,fair_odds,movement`

***

## Example Base Response

`GET /fixtures?fixture_id=123`

```json
{
  "fixtures": [
    {
      "id": 1048750,
      "date_time": "2026-05-09T11:30:00.000Z",
      "status": "FT",
      "country": {
        "id": 46,
        "name": "England",
        "alpha3": "GBR"
      },
      "league": {
        "id": 3,
        "name": "Premier League",
        "level": 1
      },
      "season": {
        "id": 16310,
        "name": "2025/2026",
        "starting_at": "2025-08-15",
        "ending_at": "2026-05-24",
        "is_current": true
      },
      "group": null,
      "stage": {
        "id": 45421,
        "name": "Regular Season"
      },
      "sport": {
        "id": 1,
        "name": "Football"
      },
      "participants": [
        {
          "id": 8,
          "name": "Liverpool FC",
          "position": 4,
          "location": "home"
        },
        {
          "id": 18,
          "name": "Chelsea",
          "position": 9,
          "location": "away"
        }
      ],
      "is_live": false,
      "language": "en",
      "scores": [
        {
          "fixture_id": 1048750,
          "team_id": 8,
          "value": 1,
          "developer_name": "1ST_HALF"
        },
        {
          "fixture_id": 1048750,
          "team_id": 8,
          "value": 1,
          "developer_name": "2ND_HALF"
        },
        {
          "fixture_id": 1048750,
          "team_id": 8,
          "value": 1,
          "developer_name": "CURRENT"
        },
        {
          "fixture_id": 1048750,
          "team_id": 8,
          "value": 0,
          "developer_name": "2ND_HALF_ONLY"
        },
        {
          "fixture_id": 1048750,
          "team_id": 18,
          "value": 1,
          "developer_name": "1ST_HALF"
        },
        {
          "fixture_id": 1048750,
          "team_id": 18,
          "value": 1,
          "developer_name": "2ND_HALF"
        },
        {
          "fixture_id": 1048750,
          "team_id": 18,
          "value": 1,
          "developer_name": "CURRENT"
        },
        {
          "fixture_id": 1048750,
          "team_id": 18,
          "value": 0,
          "developer_name": "2ND_HALF_ONLY"
        }
      ],
      "periods": [
        {
          "period_id": 2069767,
          "fixture_id": 1048750,
          "started": 1778326228,
          "ended": 1778329075,
          "counts_from": 0,
          "ticking": 0,
          "sort_order": 1,
          "time_added": 2,
          "period_length": 45,
          "minutes": 47,
          "seconds": 27,
          "developer_name": "1ST_HALF"
        },
        {
          "period_id": 2070025,
          "fixture_id": 1048750,
          "started": 1778329971,
          "ended": 1778333166,
          "counts_from": 45,
          "ticking": 0,
          "sort_order": 2,
          "time_added": 7,
          "period_length": 45,
          "minutes": 98,
          "seconds": 15,
          "developer_name": "2ND_HALF"
        }
      ]
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 25,
    "has_more": false,
    "order": "asc"
  }
}
```

> **Note:** If no score or period data is currently available for a fixture, these fields will return as empty arrays `[]`. Advanced data like **statistics** or **odds** are only appended when using the `include` parameter.

<br />