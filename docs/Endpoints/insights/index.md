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

# Insights

Access a stream of AI-generated betting recommendations that combine real-time market data with natural language analysis to identify high-probability outcomes.

## Insights Endpoint Documentation (/football/insights)

The Insights endpoint returns AI-generated analysis and betting recommendations. Each insight pairs a natural language explanation with a specific market selection (the `bet`).

The Insights endpoint provides a list of match fixtures, where each fixture contains an insights array. This structure allows you to display the AI analysis directly alongside the relevant match data, team names, match metrics and live scores.

***

## Query Parameters

| Parameter             | Type    | Default   | Description                                                                                             |
| --------------------- | ------- | --------- | ------------------------------------------------------------------------------------------------------- |
| `model_id`            | string  |           | Comma-separated list of Models IDs (e.g., 1,2,3).                                                       |
| `market_id`           | string  |           | Comma-separated list of Market IDs (e.g., 1,3).                                                         |
| `filter[markets]`     | string  |           | Markets with their selections and price range, e.g. `4:2@1.9-;53:1`. See below.                         |
| `status`              | string  | See below | Comma-separated list of Statuses to filter by.                                                          |
| `is_live`             | integer |           | `1` = only events currently in-play, `0` = only events not started yet. Omit for both.                  |
| `expired`             | integer | 0         | Use 1 to include outdated insights (useful for backend sync).                                           |
| `league_id`           | string  |           | Comma-separated list of League IDs.                                                                     |
| `fixture_id`          | integer |           | Restrict results to a specific single fixture.                                                          |
| `language`            | string  | en        | Language code for the generated content.                                                                |
| `value_from`          | number  |           | Minimum decimal odd value.                                                                              |
| `value_to`            | number  |           | Maximum decimal odd value.                                                                              |
| `include`             | string  |           | `integration` attaches your platform's own event payload to each fixture (see the Integration guide).   |
| `filter[integration]` | string  |           | Filter by your platform's own ids: events, leagues, countries, teams (see the Integration guide).       |
| `light`               | integer | 0         | Use 1 to skip fixture statistics and metrics, for fast list views that only need the insights and bets. |
| `page`                | integer | 1         | Page number (see Pagination below).                                                                     |
| `per_page`            | integer | 100       | Insights (not fixtures) per page. Maximum `100`.                                                        |

## Default Statuses

If no status is provided, the API filters by these active match states:
INPLAY\_1ST\_HALF, INPLAY\_2ND\_HALF, INPLAY\_ET, INPLAY\_PENALTIES, HT, EXTRA\_TIME\_BREAK, PEN\_BREAK, NOT\_STARTED.

## Market Filter (`filter[markets]`)

One segment per market, separated by `;`: `market_id[:label_id,label_id...][@min-max]`.

| Segment        | Keeps                                                      |
| -------------- | ---------------------------------------------------------- |
| `4:2@1.9-`     | Match Goals (4), Over (label 2), price ≥ 1.90              |
| `53:1@1.9-3.0` | Both Teams To Score (53), Yes (label 1), price 1.90 – 3.00 |
| `1@-2.5`       | Match Result (1), every selection, price ≤ 2.50            |
| `58`           | Double Chance (58), every selection, any price             |

An insight is returned when it matches **any** segment. Prices are decimal odds, inclusive; leave out `min` or `max` for no limit on that side. It combines with the other filters, which all still apply. A malformed segment, or the same market twice, returns `400`.

## Pagination

Every response is paged and carries a `pagination` object. To get more than `per_page` insights, request the next page:

```json
"pagination": {
  "page": 1,
  "per_page": 100,
  "total": 572,
  "total_pages": 6,
  "has_more": true
}
```

| Field         | Type    | Description                                                                                     |
| ------------- | ------- | ----------------------------------------------------------------------------------------------- |
| `page`        | integer | The current page number                                                                         |
| `per_page`    | integer | The page size applied                                                                           |
| `total`       | integer | Number of insights matching your filters, across all pages (counted at the time of the request) |
| `total_pages` | integer | `ceil(total / per_page)`; `0` when nothing matches                                              |
| `has_more`    | boolean | `true` only when at least one more insight exists past this page                                |

- Pages count **insights**, newest first, not fixtures. Insights are grouped by fixture in the response, so the number of fixtures per page varies and one fixture can span two pages.
- `has_more` is exact: keep requesting `page + 1` while it is `true`.
- `total` and `total_pages` count only the insights your filters let through.
- Insights are published, suspended and expire continuously, so `total` can change from one page to the next and an insight can move across a page boundary. Narrow with `fixture_id`, `league_id` or `model_id` when you need a complete, consistent view.
- Every page is full except the last.
- `limit` is no longer supported. It is ignored, not refused — use `per_page` and `page` instead.

***

## Insight Object

| Field        | Type    | Description                                                                |
| ------------ | ------- | -------------------------------------------------------------------------- |
| `id`         | integer | Unique internal identifier for the insight.                                |
| `market_id`  | integer | The ID of the market the insight refers to.                                |
| `value`      | number  | The current decimal odd value for the recommendation.                      |
| `sp`         | number  | Starting Price: The odd value at the exact time the insight was generated. |
| `created_at` | string  | ISO 8601 timestamp of generation.                                          |
| `content`    | object  | Dynamic. This can hold any key-value pairs.                                |
| `model`      | object  | Metadata about the AI model used (id, name, color).                        |
| `bet`        | object  | The recommended selection. Follows the Odd Schema.                         |

## Example JSON Response

`GET /football/insights?page=1&per_page=100`

```json
{
  "fixtures": [
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
          "fixture_id": 1039720,
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
  ],
  "pagination": {
    "page": 1,
    "per_page": 100,
    "total": 1,
    "total_pages": 1,
    "has_more": false
  }
}
```
