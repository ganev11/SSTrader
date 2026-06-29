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

> **Currently supported:** `country`. Additional types (teams, leagues, fixtures) will be
> added over time.

***

## Query Parameters

| Parameter  | Type    | Required | Default | Description                                                           |
| ---------- | ------- | -------- | ------- | --------------------------------------------------------------------- |
| `q`        | string  | Yes      | —       | The search text.                                                      |
| `types`    | string  | Yes      | —       | Comma-separated list of entity types to search. Currently: `country`. |
| `limit`    | integer | No       | `5`     | Maximum results per type. 1–20.                                       |
| `language` | string  | No       | `en`    | Language code for localized entity names.                             |

```
GET /search?q=spain&types=country
GET /search?q=espa&types=country&limit=3&language=bg
```

***

## Response

```json
{
  "countries": [
    { "id": 41, "name": "Spain", "alpha3": "ESP", "score": 0.94 }
  ]
}
```

Each item includes a `score` between 0 and 1 — higher means a closer match to `q`. Results
within a type are ordered by `score`, descending.

If `types=country,team` were requested, the response would include both a `countries` and a
`teams` key (each only present if that type was requested).

***

## Error Responses

| Status | Condition                                                         |
| ------ | ----------------------------------------------------------------- |
| `400`  | Missing `q`.                                                      |
| `400`  | Missing `types`.                                                  |
| `400`  | `types` includes an unsupported value.                            |
| `403`  | Missing or insufficient authentication.                           |
| `500`  | Search is temporarily unavailable, or an internal error occurred. |

**Example error:**

```json
{ "error": "Unsupported type(s): team. Supported: country" }
```

<br />
