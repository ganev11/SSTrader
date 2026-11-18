---
title: Odds Alerts
deprecated: false
hidden: false
metadata:
  robots: index
---
---
title: Odds Alerts
description: Sudden price drops, price spikes and line moves on a fixture's odds, with the data to chart each move and the odd that triggered it.
---

# Odds Alerts

`GET /football/odds-alerts` · `GET /football/odds-alerts/{id}` · `include=odds_alerts` · basketball: `GET /basketball/odds-alerts`

An odds alert records one sudden move in a bookmaker's prices: a price that fell or rose sharply within a few minutes, or a main line that moved. Each alert says what moved, from where to where, when, at what score, and keeps the price path so you can draw it.

## Alert types

| `type` | What happened | `direction` |
|---|---|---|
| `odds_drop` | The price of one selection on one line fell sharply within a short time window. | `down` |
| `odds_spike` | The price of one selection on one line rose sharply within a short time window, while the score did not change. | `up` |
| `line_move` | The main line of a market moved compared with the start of the time window. | `up` / `down` = the line went up / down |
| `line_move_reference` | The main line moved compared with a reference point, such as its pre-match peak, with no time window. | `up` / `down` |

### Patterns

`patterns` describes the shape of a move inside its type. An alert with a pattern is still an alert of its type, so filtering on `type=odds_drop` also returns drops that carry `money_burst`.

| Pattern | On type | Meaning |
|---|---|---|
| `money_burst` | `odds_drop` | The price zig-zagged on its way down (down, up, down) inside the window, a typical sign of repeated money hitting the selection. |

## Alert object

Alerts are always returned inside their fixture, as `fixture.odds_alerts`, so they carry no `fixture_id`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Alert id. |
| `type` | string | One of the alert types above. |
| `patterns` | string[] | Patterns of this move, often empty. |
| `direction` | string | `down` or `up`, see the types table. |
| `is_live` | integer | `1` = the move happened in play, `0` = pre-match. |
| `bookmaker_id` | integer | Bookmaker whose prices moved. |
| `market_id` | integer | Market, as in the odds include. |
| `market_name` | string | Market name in the requested `language` (football). |
| `label_id` | integer | Selection, as in the odds include (on a goal line `2` = Over, `1` = Under). |
| `label_name` | string | Selection name in the requested `language`, at `line_to` (football). |
| `odd_id` | integer | The odd whose quote triggered the alert. Matches `odd_id` in the `odds` and `movement` includes. For a line move it is the odd of the new line. |
| `odd_id_from` | integer \| null | The odd the move is measured from: the same odd for a drop or spike, the old line's odd for a line move. |
| `line_from`, `line_to` | number \| null | Line before and after (`null` for markets without a line). Equal for drops and spikes. |
| `price_from`, `price_to` | number | Decimal price before and after. For a line move: the price at the old line and at the new one. |
| `change.price_pct` | number | Price change in percent, one decimal (2.02 → 1.90 = `-5.9`). |
| `change.line` | number \| null | `line_to - line_from`. |
| `score` | object \| null | `{ "home": 1, "away": 0 }` at the moment of the alert; `null` pre-match. |
| `window_start_at` | integer \| null | Unix seconds: start of the window the move is measured over. `null` for `line_move_reference`. |
| `detected_at` | integer | Unix seconds: when the triggering quote was received, on the same clock as `last_update` in `movement`. |
| `created_at` | integer | Unix seconds: when the alert was stored. |
| `meta` | object | Chart data (below). Left out with `meta=0`. |

### Chart data

`meta.series` holds the price path that led to the alert, as columns of equal length:

| Column | Description |
|---|---|
| `t` | Unix seconds, oldest first. |
| `price` | Decimal price at `t`. |
| `line` | Line at `t` (optional). |
| `odd_id` | Odd quoted at `t` (optional; changes when the main line switches). |
| `suspended` | `1` while the market was suspended (optional). |

`meta.marks` lists events to draw on the chart, such as goals. Other keys in `meta` may appear and should be ignored if unknown.

To follow the price **after** the alert, request the fixture with `include=movement` and `filter[movement]=since:<detected_at>` and read the rows of `odd_id` (see [Odds Movement](./odds-movement.md)). To settle the alert as a bet, read the odd's `status` in the odds include once the fixture has finished.

## List alerts

`GET /football/odds-alerts`

Returns fixtures, each with its alerts in `odds_alerts`. The full fixture object is the same as in `GET /football/fixtures` (teams, league, score, status), so a list of alerts needs no second request.

- Pages count **alerts**, not fixtures: `per_page=50` returns 50 alerts, grouped into however many fixtures they belong to.
- Fixtures are ordered by their newest alert on the page; alerts inside a fixture are newest first.
- Without `fixture_id`, the list covers a time range: by default the last 24 hours, at most 31 days.

| Parameter | Description |
|---|---|
| `fixture_id` | Comma-separated fixture ids. With it, no time range is applied unless you set one. |
| `league_id` | Comma-separated league ids. |
| `type` | Comma-separated types. |
| `patterns` | Comma-separated patterns. By default an alert must have all of them. |
| `patterns_match` | `all` (default) or `any`. Needs `patterns`. |
| `direction` | `up` or `down`. |
| `is_live` | `0` pre-match, `1` in play. |
| `bookmaker_id`, `market_id` | Comma-separated ids. |
| `from`, `to` | Unix seconds or ISO 8601 date-time, compared with `detected_at`. |
| `language` | Language of team, league and market names. Default `en`. |
| `meta` | `0` leaves out `meta` (smaller responses for a list view). Default `1`. |
| `page`, `per_page` | Default `1` / `50`, `per_page` at most `100`. See [Pagination](./pagination.md). |

Unknown values return `400`.

```
GET /football/odds-alerts?type=odds_drop&patterns=money_burst&is_live=1&meta=0&language=en
```

```json
{
  "fixtures": [
    {
      "id": 1208520,
      "status": "INPLAY_2ND_HALF",
      "participants": [ "..." ],
      "odds_alerts": [
        {
          "id": "41",
          "type": "odds_drop",
          "patterns": ["money_burst"],
          "direction": "down",
          "is_live": 1,
          "bookmaker_id": 2,
          "market_id": 3,
          "market_name": "Over/Under",
          "label_id": 2,
          "label_name": "Over 2.5",
          "odd_id": 88782664,
          "odd_id_from": 88782664,
          "line_from": 2.5,
          "line_to": 2.5,
          "price_from": 2.02,
          "price_to": 1.9,
          "change": { "price_pct": -5.9, "line": 0 },
          "score": { "home": 1, "away": 0 },
          "window_start_at": 1776776527,
          "detected_at": 1776776759,
          "created_at": 1776776761
        }
      ]
    }
  ],
  "pagination": { "page": 1, "per_page": 50, "total": 1, "total_pages": 1, "has_more": false, "order": "desc" }
}
```

(The fixture object is shortened; it has every field of `GET /football/fixtures`.)

## One alert

`GET /football/odds-alerts/{id}` returns the same shape with one fixture holding one alert, `meta` included: `{ "fixtures": [ { ..., "odds_alerts": [ { ... } ] } ] }`. Takes `language`. Unknown id: `404`.

## On a fixture

`include=odds_alerts` on `GET /football/fixtures` adds `odds_alerts` to the fixture, oldest first. It works on a **single fixture** only: the request must have exactly one `fixture_id`, otherwise `400`. For alerts across many fixtures use `GET /football/odds-alerts`.

```
GET /football/fixtures?fixture_id=1208520&include=odds_alerts&filter[odds_alerts]=types:odds_drop,line_move;meta:0
```

`filter[odds_alerts]` narrows it, keys separated by `;`, values by `,`:

| Key | Values |
|---|---|
| `types` | alert types |
| `patterns` | patterns, all must be present |
| `bookmakers`, `markets` | ids |
| `is_live` | `0` or `1` |
| `since` | Unix seconds, alerts with `detected_at` at or after it |
| `meta` | `0` leaves out `meta` |

New alerts also arrive on the live stream (SSE) as `odds_alerts` in the fixture's update. The same alert can be sent more than once, so keep them by `id`.
