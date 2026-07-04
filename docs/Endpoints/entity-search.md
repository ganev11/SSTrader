---
title: Entity Search
excerpt: >-
  Find countries, teams, leagues, and fixtures by free-text query, grouped by
  entity type.
deprecated: false
hidden: false
metadata:
  robots: index
---

`GET /search`

A general-purpose lookup endpoint for finding entities by name. Unlike exact-match endpoints
(e.g. `/countries`), this endpoint tolerates misspellings, abbreviations, and alternate
phrasings — useful for search-as-you-type UI and for mapping a free-text mention onto a
canonical entity ID that can be passed to other endpoints.

Results are grouped by entity type. Only the types you request are included in the response.

> **Currently supported:** `country`, `league`, `team`, `fixture`.
>
> `fixture` search only covers matches in a rolling window from a few hours ago to about a week
> ahead, in top-tier leagues — i.e. upcoming fixtures and recently finished ones.

---

## Query Parameters

| Parameter  | Type    | Required | Default | Description |
|------------|---------|----------|---------|-------------|
| `q`        | string  | Yes      | —       | The search text. |
| `types`    | string  | Yes      | —       | Comma-separated list of entity types to search. Currently: `country`, `league`, `team`, `fixture`. |
| `limit`    | integer | No       | `5`     | Maximum results per type. 1–20. |
| `language` | string  | No       | `en`    | Language code for localized entity names. |

```
GET /search?q=spain&types=country
GET /search?q=espa&types=country&limit=3&language=bg
GET /search?q=premier&types=league
GET /search?q=tottenham&types=team
GET /search?q=tottenham+arsenal&types=fixture
```

---

## Response

```json
{
  "countries": [
    { "id": 41, "name": "Spain", "alpha3": "ESP", "score": 0.94 }
  ],
  "leagues": [
    {
      "region": { "id": 2, "name": "Europe" },
      "country": { "id": 251, "name": "England", "alpha3": "ENG" },
      "league": { "id": 8, "name": "Premier League", "level": 1 },
      "season": {
        "id": 123,
        "name": "2025/2026",
        "starting_at": "2025-08-15",
        "ending_at": "2026-05-24",
        "is_current": true
      },
      "sport": { "id": 1, "name": "Football" },
      "count": 12,
      "live": 1,
      "popular": true,
      "level": 1,
      "has_search": true,
      "score": 0.91
    }
  ],
  "teams": [
    {
      "id": 6,
      "name": "Tottenham Hotspur",
      "short_code": "TOT",
      "national_team": false,
      "country": { "id": 46, "name": "England", "alpha3": "GBR" },
      "score": 0.97
    }
  ],
  "fixtures": [
    {
      "id": 1033370,
      "date_time": "2026-05-24T15:00:00Z",
      "status": "NOT_STARTED",
      "is_live": false,
      "sport": { "id": 1, "name": "Football" },
      "country": { "id": 46, "name": "England", "alpha3": "GBR" },
      "league": { "id": 8, "name": "Premier League", "level": 1 },
      "participants": [
        { "id": 6, "name": "Tottenham Hotspur", "location": "home" },
        { "id": 19, "name": "Arsenal", "location": "away" }
      ],
      "score": 0.95
    }
  ]
}
```

Each item includes a `score` between 0 and 1 — higher means a closer match to `q`. Results
within a type are ordered by `score`, descending. `league` results use the same shape as
`GET /regions`; `fixture` results use the same shape as `GET /fixtures` items.

If `types=country,league,team,fixture` were requested, the response would include a `countries`,
a `leagues`, a `teams`, and a `fixtures` key (each only present if that type was requested).

---

## Error Responses

| Status | Condition |
|--------|-----------|
| `400`  | Missing `q`. |
| `400`  | Missing `types`. |
| `400`  | `types` includes an unsupported value. |
| `403`  | Missing or insufficient authentication. |
| `500`  | Search is temporarily unavailable, or an internal error occurred. |

**Example error:**

```json
{ "error": "Unsupported type(s): player. Supported: country, league, team, fixture" }
```
