---
title: Archive
excerpt: >-
  Paginated bet builder history for the authenticated user — every bet builder
  in a time range, including settled, suspended and expired ones.
deprecated: false
hidden: false
metadata:
  robots: index
---

# GET /bet-builders/archive

Returns paginated bet builder history for the authenticated user.

Unlike [`GET /bet-builders`](/docs/bet-builders) — which only ever returns active, non-expired suggestions — the archive returns **every** bet builder in the time range, regardless of settlement, suspend or expired state. Use it to build a results/history view: past picks, how they settled, and the odds they were generated at.

Results use the same layout as `GET /bet-builders`: a list of fixtures, where each fixture contains a `bet_builders` array, plus a `pagination` object.

---

## Query Parameters

| Parameter   | Type    | Required | Default | Description |
|-------------|---------|----------|---------|-------------|
| `model_id`  | string  | No       | —       | Comma-separated model IDs to filter by (e.g. `1,3`) |
| `language`  | string  | No       | `en`    | Language code for the generated content. Also accepted as `lang` |
| `time_from` | string  | No       | —       | ISO 8601 timestamp. Return bet builders with `created_at >= time_from` |
| `time_to`   | string  | No       | —       | ISO 8601 timestamp. Return bet builders with `created_at <= time_to` |
| `page`      | integer | No       | `1`     | Page number |
| `per_page`  | integer | No       | `25`    | Results per page. Maximum `100` |
| `order`     | string  | No       | `desc`  | Sort direction on `created_at`. Accepted values: `asc`, `desc` |

```
GET /bet-builders/archive?model_id=168&page=1&per_page=25
```

### Time range notes

- Both `time_from` and `time_to` are optional and independent. Either can be used alone.
- If both are provided, `time_from` must not be after `time_to` — a `400` is returned otherwise.
- `time_to` without `time_from` is most useful with `order=desc` (returns the most recent bet builders before the cutoff).
- The range applies to `created_at` (when the bet builder was generated), **not** to the fixture's kick-off time.

### Pagination notes

Pagination operates at the **bet builder level**: `per_page` controls the maximum number of bet builder records returned. Because they are grouped by fixture in the response, the number of fixtures per page varies — a fixture with three bet builders consumes three of your `per_page` budget.

`has_more` is exact: `true` only when at least one more bet builder exists past this page, so a last page that happens to be exactly full reports `false`.

`total` is the number of bet builders matching your filters across all pages, and `total_pages` is `ceil(total / per_page)`. Like the pages themselves, they count bet builders, not fixtures. With a fixed `time_to` in the past the archive does not change, so `total` stays the same while you page; an open-ended range can grow as new bet builders are created.

### Language notes

Only bet builders that have a translation in the requested `language` are returned. A bet builder whose content was never generated in that language is omitted from the archive entirely — it is not returned with empty content. If a history view looks unexpectedly short, retry with `language=en`.

---

## Bet Builder Object

Identical to the object returned by [`GET /bet-builders`](/docs/bet-builders) — same fields (`id`, `model_id`, `is_live`, `value`, `sp`, `suspend`, `status`, `last_priced_at`, `raw`, `meta`, `created_at`, `language`, `content`, `model`, `selections`). See that page for the full field reference.

Fields that matter most in an archive context:

| Field            | Description |
|------------------|-------------|
| `status`         | Settlement status of the combination — `0` pending, `1` won, `-1` lost, `-3` void. This is the field to read for "did it win?". Every selection must win for `1`; a single void selection voids the whole bet builder. |
| `sp`             | Starting Price — the combined decimal odds at the time the bet builder was generated. This is the price to use when calculating historical returns. |
| `value`          | Last known combined decimal odds. For a settled bet builder this is the final price seen before settlement, not the generation price — use `sp` for performance maths. |
| `suspend`        | State at the time of the last pricing update. In an archive result a `1` here is historical noise, not an instruction — it doesn't mean the bet was void. |
| `selections`     | The individual picks, each with its own `status` and `outcome`, so you can show which leg of the combination failed. |

> **Expired bet builders are included.** There's no `expired` flag in the response — an expired bet builder is returned exactly like any other historical one.

---

## Response

**200 OK**

```json
{
  "fixtures": [
    {
      "id": 1080495,
      "date_time": "2026-07-16T17:00:00.000Z",
      "status": "FINISHED",
      "league": {
        "id": 155,
        "name": "Eliteserien",
        "level": 1
      },
      "participants": [
        { "id": 411, "name": "Valerenga IF", "location": "home" },
        { "id": 349, "name": "Aalesunds", "location": "away" }
      ],
      "is_live": false,
      "language": "en",
      "bet_builders": [
        {
          "id": 3,
          "user_id": 2,
          "model_id": 168,
          "is_live": 0,
          "value": 2.71,
          "sp": 2.71,
          "suspend": 0,
          "status": 1,
          "last_priced_at": "2026-07-16T16:58:08.450Z",
          "raw": { },
          "meta": {},
          "created_at": "2026-07-16T11:58:08.450Z",
          "language": "en",
          "content": {
            "text": "Valerenga should boss this at home, their front line creating better chances all season..."
          },
          "model": {
            "id": 168,
            "name": "BB - all leagues",
            "color": "#ed145b"
          },
          "selections": [
            {
              "odd_id": 99053176,
              "fixture_id": 1080495,
              "market_id": 1,
              "value": 1.69,
              "sp": 1.67,
              "status": 1,
              "outcome": "Won",
              "market_name": "Match Winner",
              "label_name": "Valerenga IF"
            },
            {
              "odd_id": 99053173,
              "fixture_id": 1080495,
              "market_id": 53,
              "value": 1.48,
              "sp": 1.51,
              "status": 1,
              "outcome": "Won",
              "market_name": "Both Teams to Score",
              "label_name": "Yes"
            }
          ]
        }
      ]
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 25,
    "total": 48,
    "total_pages": 2,
    "has_more": true,
    "order": "desc"
  }
}
```

A page past the end returns the same envelope with no fixtures; `total` still reports the full count:

```json
{
  "fixtures": [],
  "pagination": { "page": 3, "per_page": 25, "total": 48, "total_pages": 2, "has_more": false, "order": "desc" }
}
```

**400 Bad Request** — `time_from` is after `time_to`

```json
{ "error": "time_from must not be after time_to" }
```

**403 Forbidden** — missing or invalid authentication

```json
{ "error": "Unauthorized" }
```

**500 Internal Server Error**

```json
{ "error": "Internal server error" }
```

---

## Examples

### Latest 25 bet builders (default)
```
GET /bet-builders/archive
```

### History for a single model
```
GET /bet-builders/archive?model_id=168&page=1&per_page=25
```

### Filter by several models, most recent before a date
```
GET /bet-builders/archive?model_id=164,168&time_to=2026-07-01T00:00:00Z&order=desc
```

### Specific date window, oldest first
```
GET /bet-builders/archive?time_from=2026-06-01T00:00:00Z&time_to=2026-06-30T23:59:59Z&order=asc&per_page=50
```

### Page through all history
```
GET /bet-builders/archive?page=1&per_page=100&order=desc
GET /bet-builders/archive?page=2&per_page=100&order=desc
```

---

## Related

- [`GET /bet-builders`](/docs/bet-builders) — active bet builders only, plus the full field reference
- [`GET /insights/archive`](/docs/insights-archive) — the equivalent endpoint for insights, with the same parameters and pagination semantics
