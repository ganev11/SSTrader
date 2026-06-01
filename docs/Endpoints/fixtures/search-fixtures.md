---
title: Search
excerpt: >-
  Query upcoming and live fixtures by time window, status, and value-based
  metric or statistic filters.
deprecated: false
hidden: false
metadata:
  robots: index
---
# Search Fixtures

`GET /fixtures/search`

A powerful query endpoint for discovering fixtures that match specific metric or statistic thresholds. Unlike `/fixtures`, which retrieves fixtures by date or ID, `/search` lets you filter by **calculated values** — for example, "find all upcoming matches where the home team's attack power is between 1100 and 1800."

Results are paginated and the response structure is identical to `/fixtures`.

---

## Query Parameters

### Time Window

| Parameter | Type   | Default | Description |
|-----------|--------|---------|-------------|
| `window`  | string | `24`    | Defines the time scope of the search. |

**Valid values:** `live`, `3`, `6`, `12`, `24`, `48`, `168`

| Value  | Behaviour |
|--------|-----------|
| `live` | No date range applied. Only the `status` filter drives results. Defaults to all in-play statuses. |
| `3`–`168` | Matches fixtures from **now** up to **+N hours** from now. Defaults to live + `NOT_STARTED` statuses. |

```
GET /fixtures/search?window=6
GET /fixtures/search?window=live
```

---

### League Filter

| Parameter  | Type   | Default | Description |
|------------|--------|---------|-------------|
| `league_id` | string | —      | Comma-separated list of league IDs to restrict the search scope. |

```
GET /fixtures/search?league_id=8,34,55
```

---

### Status Filter

| Parameter | Type   | Default | Description |
|-----------|--------|---------|-------------|
| `status`  | string | See window | Comma-separated list of fixture statuses to include. |

When omitted:
- `window=live` → defaults to all in-play statuses (`INPLAY_1ST_HALF`, `INPLAY_2ND_HALF`, `INPLAY_ET`, `INPLAY_PENALTIES`, `HT`, `EXTRA_TIME_BREAK`, `PEN_BREAK`)
- Any numeric window → defaults to in-play statuses **plus** `NOT_STARTED`

```
GET /fixtures/search?window=24&status=NOT_STARTED,HT
```

👉 **[View all status definitions](https://docs.sstrader.com/docs/states)**

---

### Value Filters — `where[developer_name]`

The most powerful feature of this endpoint. Each `where` filter narrows results to fixtures where the named metric or statistic falls within a given range. **All provided filters must match** (AND logic). Up to **10 filters** are allowed per request.

**Format:**

```
where[DEVELOPER_NAME]=min:max[:location]
```

| Part       | Type    | Required | Description |
|------------|---------|----------|-------------|
| `min`      | integer | Yes      | Minimum value (inclusive). |
| `max`      | integer | Yes      | Maximum value (inclusive). |
| `location` | string  | No       | Perspective for the filter. One of `HOME`, `AWAY`, `LEAGUE`. Default: `LEAGUE`. |

**Location values:**

| Value    | Description |
|----------|-------------|
| `HOME`   | Apply the filter to the home team's value. |
| `AWAY`   | Apply the filter to the away team's value. |
| `LEAGUE` | Apply the filter at league/match level (team_id = 0). |

The `DEVELOPER_NAME` must match a value in the `developer_name` field of the `/types` endpoint.

**Examples:**

```
# Fixtures where home team goals classification is between 3000 and 3000 (3000 is top team)
GET /fixtures/search?where[TEAM_GOALS_CLASSIFICATION]=3000:3000:HOME

# Combine multiple filters (all must match)
GET /fixtures/search?where[TEAM_GOALS_CLASSIFICATION]=3000:3000:HOME&where[TEAM_GOALS_CLASSIFICATION]=0:0:AWAY

# League-level metric range (default location)
GET /fixtures/search?where[OVERALL_GOALS_SCORED]=2300:3000
```

> **Note:** Duplicate `where[X]` keys with different locations are fully supported — each is treated as a separate filter condition.

---

### Custom Ordering — `order[developer_name]`

Sort results by any metric or statistic value instead of the default `date_time ASC`.

**Format:**

```
order[DEVELOPER_NAME]=DIR[:location]
```

| Part       | Type   | Required | Description |
|------------|--------|----------|-------------|
| `DIR`      | string | Yes      | Sort direction: `ASC` or `DESC`. |
| `location` | string | No       | Which team's value to sort by. One of `HOME`, `AWAY`, `LEAGUE`. Default: `LEAGUE`. |

Only the **first** `order[...]` parameter is applied; subsequent ones are ignored.

**Examples:**

```
# Sort by home team goals classification (highest first)
GET /fixtures/search?order[TEAM_GOALS_CLASSIFICATION]=DESC:HOME

# Sort by away team classification ascending
GET /fixtures/search?order[TEAM_GOALS_CLASSIFICATION]=ASC:AWAY
```

---

### Data Enrichment — `include`

| Parameter | Type   | Default | Description |
|-----------|--------|---------|-------------|
| `include` | string | —       | Comma-separated list of modules to attach to each fixture. |

**Allowed values:** `odds`, `metrics`, `statistics`

> **Note:** `fair_odds` and `movement` are not available on this endpoint.

```
GET /fixtures/search?window=24&include=odds,metrics
```

Using an include unlocks the corresponding `filter[entity]` parameter (see below).

---

### Include Filters

#### `filter[metrics]`

Refine which metrics are returned when `include=metrics`.

| Format | Example | Description |
|--------|---------|-------------|
| `types:ID,...` | `types:1,2,3` | Return only metrics matching these Type IDs. |
| `developer_types:GROUP,...` | `developer_types:predictions,smart_money` | Return metrics belonging to these domain groups. |

```
GET /fixtures/search?include=metrics&filter[metrics]=developer_types:predictions
```

#### `filter[statistics]`

Refine which statistics are returned when `include=statistics`.

```
GET /fixtures/search?include=statistics&filter[statistics]=types:5,6
```

#### `filter[odds]`

Refine which odds are returned when `include=odds`.

Use a **comma (,)** for OR logic within a key and a **semicolon (;)** for AND logic between keys.

```
GET /fixtures/search?include=odds&filter[odds]=bookmakers:2,3;markets:1,3
```

| Key          | Description |
|--------------|-------------|
| `bookmakers` | Comma-separated bookmaker IDs. |
| `markets`    | Comma-separated market IDs. |
| `is_live`    | `1` for live odds, `0` for pre-match. |
| `suspend`    | `1` for suspended odds, `0` for active. |

---

### Pagination

| Parameter  | Type    | Default | Constraints | Description |
|------------|---------|---------|-------------|-------------|
| `page`     | integer | `1`     | ≥ 1         | Page number. |
| `per_page` | integer | `25`    | 1–50        | Results per page. Maximum is **50**. |

---

### Localization

| Parameter  | Type   | Default | Description |
|------------|--------|---------|-------------|
| `language` | string | `en`    | Language code for localized team, league, and country names. Alias: `lang`. |

---

## Request Examples

**Fixtures in the next 24 hours that haven't started yet:**

```
GET /fixtures/search?window=24&status=NOT_STARTED
```

**Live fixtures in specific leagues with odds:**

```
GET /fixtures/search?window=live&league_id=8,34&include=odds&filter[odds]=bookmakers:2,3;markets:1
```

**Upcoming fixtures where the home team's classification is in a high-value range, sorted by that value:**

```
GET /fixtures/search?window=48
  &where[TEAM_GOALS_CLASSIFICATION]=3000:3000:HOME
  &order[TEAM_GOALS_CLASSIFICATION]=DESC:HOME
  &include=metrics
  &filter[metrics]=developer_types:predictions
```

**Multi-condition search with full enrichment:**

```
GET /fixtures/search?window=12
  &where[TEAM_GOALS_CLASSIFICATION]=2000:3000:HOME
  &where[HOME_GOALS_ATTACK_POWER]=1200:2000:HOME
  &include=odds,metrics,statistics
  &per_page=50
  &page=1
```

---

## Response

```json
{
  "fixtures": [
    {
      "id": 1033370,
      "date_time": "2026-06-01T15:00:00.000Z",
      "status": "NOT_STARTED",
      "is_live": false,
      "language": "en",
      "sport": { "id": 1, "name": "Football" },
      "country": { "id": 41, "name": "England", "alpha3": "ENG" },
      "league": { "id": 8, "name": "Premier League", "level": 1 },
      "season": {
        "id": 17760,
        "name": "2025/2026",
        "starting_at": "2025-08-09",
        "ending_at": "2026-05-24",
        "is_current": true
      },
      "participants": [
        { "id": 85, "name": "Manchester City", "position": 1, "location": "home" },
        { "id": 33, "name": "Arsenal", "position": 2, "location": "away" }
      ],
      "scores": [],
      "periods": []
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 25,
    "has_more": false
  }
}
```

### Pagination Object

| Field      | Type    | Description |
|------------|---------|-------------|
| `page`     | integer | The current page number. |
| `per_page` | integer | Results returned on this page. |
| `has_more` | boolean | `true` if more pages exist (determined by whether the count of results equals `per_page`). |

> **Note:** The `order` field is not included in the search endpoint's pagination object.

When no fixtures match the query, the endpoint returns immediately without querying enrichment modules:

```json
{ "fixtures": [], "pagination": { "page": 1, "per_page": 25, "has_more": false } }
```

---

## Error Responses

| Status | Condition |
|--------|-----------|
| `400`  | Invalid `window` value. |
| `400`  | Malformed `where[X]` filter (missing or non-numeric min/max). |
| `400`  | More than 10 `where` filters provided. |
| `403`  | Missing or insufficient authentication. |
| `500`  | Internal server error. |

**Example error:**

```json
{ "error": "Invalid window. Valid values: live, 3, 6, 12, 24, 48, 168" }
```

---

## Notes

- Value filters (`where[X]`) search across **fixture_metrics**, **fixture_stats**, and **fixture_stats_live** tables, so both pre-match metrics and in-play statistics can be used as filter criteria.
