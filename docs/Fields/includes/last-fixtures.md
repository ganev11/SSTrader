---
title: Last Fixtures
excerpt: >-
  Recent home and away form for both teams in a fixture, with statistics and
  metrics for each historical match.
deprecated: false
hidden: false
metadata:
  robots: index
---
`include=last_fixtures`

The `last_fixtures` include attaches a "form guide" to each fixture: the home team's most recent matches **played at home**, and the away team's most recent matches **played away**, both restricted to fixtures that took place before this fixture's kick-off. Each historical match is enriched with its own `statistics` and `metrics`, so you can analyse recent venue-specific performance without making additional requests.

> **Availability:** Only returned when `include=last_fixtures` is added to a request to `GET /fixtures`. Not available on `/fixtures/search` or `/livescores`.

---

## Object Schema

`last_fixtures` is an object with two arrays:

| Field  | Type  | Description |
|--------|-------|-------------|
| `home` | array | Up to 10 most recent fixtures where the home team played **at home**, ordered most recent first. |
| `away` | array | Up to 10 most recent fixtures where the away team played **away**, ordered most recent first. |

Each entry in `home` and `away` has the same shape as a fixture returned from `/fixtures` (`id`, `date_time`, `status`, `participants`, `scores`, `periods`, `league`, `season`, etc.), plus two additional fields:

| Field        | Type  | Description |
|--------------|-------|-------------|
| `statistics` | array | Match statistics for that historical fixture — same shape as the `statistics` include. |
| `metrics`    | array | Calculated metrics for that historical fixture — same shape as the `metrics` include. |

> **Note:** `statistics` and `metrics` inside `last_fixtures` are always returned in full and are **not** affected by `filter[statistics]` or `filter[metrics]` on the outer request.

---

## Why Home/Away Form, Not Overall Form

`last_fixtures.home` only contains matches the home team played **at home**, and `last_fixtures.away` only contains matches the away team played **away**. This isolates venue-specific form — for example, a team's home scoring record vs. its away record — rather than mixing in results from the other venue that may not be representative of how the team performs in this fixture's setting.

---

## Request Examples

**Fixture with recent home/away form:**

```
GET /fixtures?fixture_id=1063864&include=last_fixtures
```

**Combine with the upcoming fixture's own statistics and metrics:**

```
GET /fixtures?fixture_id=1063864&include=last_fixtures,statistics,metrics
```

---

## Example Response

```json
"last_fixtures": {
  "home": [
    {
      "id": 1048750,
      "date_time": "2026-05-02T14:00:00.000Z",
      "status": "FT",
      "is_live": false,
      "league": { "id": 3, "name": "Premier League", "level": 1 },
      "season": {
        "id": 16310,
        "name": "2025/2026",
        "starting_at": "2025-08-15",
        "ending_at": "2026-05-24",
        "is_current": true
      },
      "participants": [
        { "id": 8, "name": "Liverpool FC", "position": 4, "location": "home" },
        { "id": 18, "name": "Chelsea", "position": 9, "location": "away" }
      ],
      "scores": [
        { "fixture_id": 1048750, "team_id": 8, "value": 2, "developer_name": "CURRENT" },
        { "fixture_id": 1048750, "team_id": 18, "value": 1, "developer_name": "CURRENT" }
      ],
      "periods": [],
      "statistics": [
        { "team_id": 8, "type_id": 34, "developer_name": "CORNERS", "value": 6 }
      ],
      "metrics": [
        { "team_id": 0, "type_id": 105, "developer_name": "PREDICTION_GL_CONFIDENCE", "value": 70 }
      ]
    }
  ],
  "away": []
}
```

> An empty array (e.g. `"away": []`) means no qualifying historical fixtures were found for that team at that venue — for example, a newly promoted team with no prior away matches in the dataset.

---

## Real-World Use Cases

### Recent form widget

Show the last 5–10 results for each team at their respective venue, deriving W/D/L badges from the `scores` of each `last_fixtures` entry.

### Venue-specific scoring trends

Aggregate `statistics` (e.g. `CORNERS`, `SHOTS_ON_TARGET`) across `last_fixtures.home` and `last_fixtures.away` to compare a team's attacking output at home vs. on the road.

### Pre-match context for predictions

Combine `last_fixtures` with the upcoming fixture's own `metrics` to give bettors context behind a prediction — for example, "Team A has scored in 8 of their last 10 home games."
