---
title: Bet Builders
excerpt: >-
  AI-selected bet builder suggestions — multiple selections from the same match
  combined into a single bet, with a natural-language explanation for the
  combination.
deprecated: false
hidden: false
metadata:
  robots: index
---
<Callout icon="🛑" theme="error">
  ### **NB**

  This endpoint is in beta testing.
</Callout>

`GET /bet-builders`

A bet builder combines multiple selections from the **same match** into one bet (e.g. Match Winner + Over 2.5 Goals for the same fixture).

Results use the same layout as `/insights`: a list of fixtures, where each fixture contains a `bet_builders` array. This lets you display bet builder suggestions directly alongside the relevant match data, team names, and live scores.

---

## Query Parameters

| Parameter    | Type    | Default | Description |
|--------------|---------|---------|-------------|
| `model_id`   | string  | —       | Comma-separated list of Model IDs (e.g. `1,2,3`) |
| `language`   | string  | `en`    | Language code for the generated content. Also accepted as `lang` |
| `fixture_id` | integer | —       | Restrict results to a specific single fixture |
| `league_id`  | string  | —       | Comma-separated list of League IDs |
| `is_live`    | integer | —       | Use `1` to filter only for events currently in-play, `0` for events not yet started. Omitted by default — no filtering on match state. |

```
GET /bet-builders?model_id=12&language=en
```

> **Always active:** Results only ever include active, non-suspended bet builders — there's no way to include suspended or expired ones yet. That's coming with a dedicated `/bet-builders/archive` endpoint (TODO), mirroring `/insights/archive`.

> **Pagination:** Not implemented yet (TODO) — a fixed internal result limit is applied for now.

---

## Bet Builder Object

| Field            | Type            | Description |
|------------------|-----------------|--------------|
| `id`             | integer         | Unique identifier for this bet builder. |
| `user_id`        | integer         | The owning user's ID — always your own, since results are scoped to the authenticated user. |
| `model_id`       | integer         | The Model that generated this suggestion. |
| `is_live`        | integer         | `1` if this bet builder was generated for an in-play match, `0` for pre-match. |
| `value`          | number / null   | Current combined decimal odds. `null` while the combination is still being priced. |
| `sp`             | number / null   | Starting Price — the combined odds at the time this bet builder was generated. |
| `suspend`        | integer         | Boolean flag (`0` or `1`). If `1`, this bet builder is temporarily unavailable (e.g. one of its selections is currently suspended). Treat this as "wait", not "cancelled" — it can flip back to `0` once the selection becomes available again. |
| `status`         | integer         | Settlement status. Same codes as the [Odd Object](/docs/odds) `status` field. |
| `last_priced_at` | string / null   | ISO 8601 timestamp of the last time `value`/`sp` were updated. |
| `raw`            | object          | Bookmaker-specific reference data for this exact combination of selections. Opaque and varies by bookmaker; returns `{}` when not available. |
| `meta`           | object          | Additional bookmaker-specific metadata (e.g. promotional price boosts). Shape may vary and is not guaranteed to be present. |
| `created_at`     | string          | ISO 8601 timestamp of generation. |
| `language`       | string          | Language code of the `content` below. Matches the request's `language`/`lang` param. |
| `content`        | object          | Dynamic. The natural-language explanation for the combination — can hold any key-value pairs. |
| `model`          | object          | Metadata about the model used (`id`, `name`, `color`). |
| `selections`     | array           | The individual picks that make up this bet builder — see below. |

### Selection Object

Each entry in `selections` follows the [Odd Object](/docs/odds) schema (`odd_id`, `fixture_id`, `market_id`, `value`, `suspend`, `status`, `outcome`, `market_name`, `label_name`, etc.).

---

## Example JSON Response

```json
{
  "fixtures": [
    {
      "id": 1039720,
      "date_time": "2026-04-27T15:00:00.000Z",
      "status": "NOT_STARTED",
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
      "bet_builders": [
        {
          "id": 501,
          "user_id": 8831,
          "model_id": 12,
          "is_live": 0,
          "value": 4.8,
          "sp": 4.5,
          "suspend": 0,
          "status": 0,
          "last_priced_at": "2026-04-27T14:58:00.000Z",
          "raw": { "selection_ids": "3897429409|3314639099" },
          "meta": {},
          "created_at": "2026-04-27T14:00:00.000Z",
          "language": "en",
          "content": {
            "text": "Piast have been slow starters at home this season, and Arka's front two have been finding the net at a steady clip — backing the away side alongside goals in this one."
          },
          "model": { "id": 12, "name": "Home Value Builder", "color": "#2f7d32" },
          "selections": [
            {
              "odd_id": 90004836,
              "fixture_id": 1039720,
              "market_id": 1,
              "bookmaker_id": 4,
              "is_live": 0,
              "label_id": 2,
              "value": 2.1,
              "handicap": 0,
              "line": 0,
              "last_update": 1777316400,
              "suspend": 0,
              "sp": 2.2,
              "home_score": 0,
              "away_score": 0,
              "status": 0,
              "raw": {},
              "outcome": "Pending",
              "market_name": "Match Winner",
              "market_description": "Predict the result of the match",
              "label_name": "Arka Gdynia"
            },
            {
              "odd_id": 90004901,
              "fixture_id": 1039720,
              "market_id": 3,
              "bookmaker_id": 4,
              "is_live": 0,
              "label_id": 2,
              "value": 2.05,
              "handicap": 0,
              "line": 2.5,
              "last_update": 1777316400,
              "suspend": 0,
              "sp": 2.0,
              "home_score": 0,
              "away_score": 0,
              "status": 0,
              "raw": {},
              "outcome": "Pending",
              "market_name": "Over/Under",
              "market_description": "Total goals scored",
              "label_name": "Over 2.5"
            }
          ]
        }
      ]
    }
  ]
}
```

---

## GET /bet-builders/:id

Returns a single bet builder by id, as a flat object (not nested under a fixture). Includes expired/suspended bet builders regardless of the `expired`/`suspend` filters.

```
GET /bet-builders/501?language=en
```

---

## Response Codes

**200 OK** — see example above.

**403 Forbidden** — missing or invalid authentication

```json
{ "error": "Unauthorized" }
```

**404 Not Found** — `GET /bet-builders/:id` only, when the id doesn't exist or doesn't belong to you

```json
{ "error": "Bet builder not found" }
```

**500 Internal Server Error**

```json
{ "error": "Internal server error" }
```
