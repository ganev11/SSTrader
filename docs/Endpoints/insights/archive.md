---
title: Archive
excerpt: Returns paginated insights history
deprecated: false
hidden: false
metadata:
  robots: index
---
# GET /insights/archive

Returns paginated insight history for the authenticated user.

All insight records are returned as-is — no suspend/expired filtering and no deduplication is applied. One fixture can appear with more than one insight in the `insights[]` array (e.g. the same fixture was processed by multiple models or on multiple runs).

***

## Query Parameters

| Parameter   | Type    | Required | Default | Description                                                        |
| ----------- | ------- | -------- | ------- | ------------------------------------------------------------------ |
| `model_id`  | string  | No       | —       | Comma-separated model IDs to filter by (e.g. `1,3`)                |
| `language`  | string  | No       | `en`    | Language code for insight content. Also accepted as `lang`         |
| `time_from` | string  | No       | —       | ISO 8601 timestamp. Return insights with `created_at >= time_from` |
| `time_to`   | string  | No       | —       | ISO 8601 timestamp. Return insights with `created_at <= time_to`   |
| `page`      | integer | No       | `1`     | Page number                                                        |
| `per_page`  | integer | No       | `25`    | Results per page. Maximum `100`                                    |
| `order`     | string  | No       | `desc`  | Sort direction on `created_at`. Accepted values: `asc`, `desc`     |

### Time range notes

- Both `time_from` and `time_to` are optional and independent. Either can be used alone.
- If both are provided, `time_from` must not be after `time_to` — a `400` is returned otherwise.
- `time_to` without `time_from` is most useful with `order=desc` (returns the most recent insights before the cutoff).

### Pagination notes

Pagination operates at the **insight level**: `per_page` controls the maximum number of insight records returned. Because insights are grouped by fixture in the response, the number of fixtures per page varies.

***

## Response

**200 OK**

```json
{
  "fixtures": [
    {
      "id": 12345,
      "insights": [
        {
          "id": 1,
          "model_id": 2,
          "market_id": 1,
          "value": 1.85,
          "created_at": "2025-06-01T14:00:00.000Z",
          "model": {
            "id": 2,
            "name": "Model Alpha",
            "color": "#ff5733"
          },
          "bet": { ... }
        },
        {
          "id": 2,
          "model_id": 3,
          "market_id": 1,
          "value": 2.10,
          "created_at": "2025-06-01T15:30:00.000Z",
          "model": { ... },
          "bet": { ... }
        }
      ]
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 25,
    "has_more": true,
    "order": "desc"
  }
}
```

**400 Bad Request** — `time_from` is after `time_to`

```json
{ "error": "time_from must not be after time_to" }
```

**403 Forbidden** — missing or insufficient role

```json
{ "error": "Unauthorized" }
```

**500 Internal Server Error**

```json
{ "error": "Internal server error" }
```

***

## Examples

### Latest 25 insights (default)

```
GET /insights/archive
```

### Filter by model, last page of results before a date

```
GET /insights/archive?model_id=2,5&time_to=2025-06-01T00:00:00Z&order=desc
```

### Specific date window, oldest first

```
GET /insights/archive?time_from=2025-05-01T00:00:00Z&time_to=2025-05-31T23:59:59Z&order=asc&per_page=50
```

### Page through all history

```
GET /insights/archive?page=1&per_page=100&order=desc
GET /insights/archive?page=2&per_page=100&order=desc
```

<br />