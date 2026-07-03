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

> **Currently supported:** `country`, `league`, `team`. Additional types (fixtures) will be
> added over time.

---

## Query Parameters

| Parameter  | Type    | Required | Default | Description |
|------------|---------|----------|---------|-------------|
| `q`        | string  | Yes      | —       | The search text. |
| `types`    | string  | Yes      | —       | Comma-separated list of entity types to search. Currently: `country`, `league`, `team`. |
| `limit`    | integer | No       | `5`     | Maximum results per type. 1–20. |
| `language` | string  | No       | `en`    | Language code for localized entity names. |

```
GET /search?q=spain&types=country
GET /search?q=espa&types=country&limit=3&language=bg
GET /search?q=premier&types=league
GET /search?q=tottenham&types=team
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
  ]
}
```

Each item includes a `score` between 0 and 1 — higher means a closer match to `q`. Results
within a type are ordered by `score`, descending. `league` results use the same shape as
`GET /regions`.

If `types=country,league,team` were requested, the response would include a `countries`, a
`leagues`, and a `teams` key (each only present if that type was requested).

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
{ "error": "Unsupported type(s): fixture. Supported: country, league, team" }
```
