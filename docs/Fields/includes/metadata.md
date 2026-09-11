---
title: Metadata
excerpt: Fixture-level metadata such as team kit and participant colors.
deprecated: false
hidden: false
metadata:
  robots: index
---

`include=metadata`

The `metadata` include attaches an array of fixture-level metadata entries to each fixture. The entries follow the same shape as other "type" based includes (`statistics`, `metrics`, `trends`) so they can be consumed with the same client-side logic.

> **Availability:** Only returned when `include=metadata` is added to a request to `GET /fixtures` or `GET /fixtures/search`. When not requested, the field is omitted entirely.

> **Scope:** Currently only **kit / participant colors** (`PARTICIPANT_COLORS`), **formations** (`FORMATION`) and **lineup confirmation** (`LINEUP_CONFIRMED`) are returned. Other raw metadata stored against a fixture (e.g. neutral venue, kickoff side, attendance) is not yet exposed via this include.

> **Entries are only present once known.** A formation is usually unavailable until the lineup is confirmed, roughly an hour before kickoff. Until then the fixture simply carries no `FORMATION` entry — treat a missing entry as "not yet known", and expect it to appear closer to kickoff. The same applies to colors. A fixture with nothing known yet returns `"metadata": []`.

> **`LINEUP_CONFIRMED` is the exception to that rule** — it is the one entry that is present *before* the thing it describes has happened. `value: 0` / `"confirmed": false` means the lineup is not confirmed yet and is a real answer, not a placeholder; a **missing** `LINEUP_CONFIRMED` entry means we hold no lineup information for that fixture at all. Do not read the two as the same thing.

> **Not a live signal.** Metadata is refreshed when a fixture is loaded, not pushed, so `LINEUP_CONFIRMED` can lag the actual confirmation by a few minutes. Use it to decide whether lineup-derived data (formations, squads) is worth reading — not as a trigger to schedule against.

---

## Object Schema

| Field            | Type    | Description                                                                 |
|------------------|---------|-------------------------------------------------------------------------------|
| `team_id`        | integer | The team this entry belongs to, or `null` when the entry is about the fixture as a whole rather than one side (`LINEUP_CONFIRMED`). |
| `type_id`        | integer | The external type ID. `348` = `PARTICIPANT_COLORS`, `349` = `FORMATION`.   |
| `developer_name` | string  | The constant-style identifier for the metadata type.                       |
| `value`          | integer | `null` for `PARTICIPANT_COLORS` and `FORMATION`. `1` / `0` for `LINEUP_CONFIRMED`. |
| `meta`           | object  | Context-specific payload. For `PARTICIPANT_COLORS`, contains `kit` and `participant`; for `FORMATION`, contains `formation`; for `LINEUP_CONFIRMED`, contains `confirmed`. See below. |

### `meta` for `PARTICIPANT_COLORS`

| Field         | Type   | Description                                                                                   |
|---------------|--------|------------------------------------------------------------------------------------------------|
| `kit`         | string | Comma-separated list of hex colors describing the team's kit (shirt, shorts, socks, sleeves, etc.). |
| `participant` | string | The primary/representative hex color for this team, used for charts, badges, and UI accents. |

### `meta` for `FORMATION`

| Field       | Type   | Description                                                              |
|-------------|--------|---------------------------------------------------------------------------|
| `formation` | string | The team's formation for this fixture, e.g. `"3-4-2-1"`.                |

### `meta` for `LINEUP_CONFIRMED`

| Field       | Type    | Description                                                                   |
|-------------|---------|--------------------------------------------------------------------------------|
| `confirmed` | boolean | Whether the starting lineups for this fixture have been confirmed. The same fact as `value` (`1`/`0`); read whichever suits your client. |

---

## Available Types

| `type_id` | `developer_name`     | `developer_type` | Description                                  |
|-----------|-----------------------|-------------------|-----------------------------------------------|
| `348`     | `PARTICIPANT_COLORS`  | `match_info`      | Home/away kit and participant colors for the fixture. One entry per team. |
| `349`     | `FORMATION`           | `match_info`      | Home/away formation for the fixture. One entry per team. |
| `402`     | `LINEUP_CONFIRMED`    | `metadata`        | Whether the starting lineups are confirmed. One entry per fixture, with `team_id: null`. |

---

## Request Examples

**Fixture with kit colors:**

```
GET /fixtures?fixture_id=19609153&include=metadata
```

**Combine with participants and statistics:**

```
GET /fixtures?fixture_id=19609153&include=metadata,statistics
```

**Search results with kit colors:**

```
GET /fixtures/search?window=24&include=metadata
```

---

## Example Response

```json
"participants": [
  { "id": 85, "name": "Team A", "position": 3, "location": "home" },
  { "id": 102, "name": "Team B", "position": 11, "location": "away" }
],
"metadata": [
  {
    "team_id": 85,
    "type_id": 348,
    "developer_name": "PARTICIPANT_COLORS",
    "value": null,
    "meta": {
      "kit": "#D6003D,#D6003D,#D6003D,#D6003D,#C40010,#0046A8,#C40010,#D6003D,#0A0A0A,#0A0A0A,#F0F0F0",
      "participant": "#C40010"
    }
  },
  {
    "team_id": 102,
    "type_id": 348,
    "developer_name": "PARTICIPANT_COLORS",
    "value": null,
    "meta": {
      "kit": "#F0F0F0,#F0F0F0,#F0F0F0,#F0F0F0,#C40010,#0046A8,#F0F0F0,#679FEA,#C40010,#679FEA,#679FEA",
      "participant": "#F0F0F0"
    }
  },
  {
    "team_id": 85,
    "type_id": 349,
    "developer_name": "FORMATION",
    "value": null,
    "meta": {
      "formation": "3-4-2-1"
    }
  },
  {
    "team_id": 102,
    "type_id": 349,
    "developer_name": "FORMATION",
    "value": null,
    "meta": {
      "formation": "4-3-3"
    }
  },
  {
    "team_id": null,
    "type_id": 402,
    "developer_name": "LINEUP_CONFIRMED",
    "value": 1,
    "meta": {
      "confirmed": true
    }
  }
]
```

Match `team_id` against `participants[].id` to determine which entry belongs to the home or away team. An entry whose `team_id` is `null` matches neither — it describes the fixture itself.

---

## Real-World Use Cases

### Match center theming

Use `participant` to color team badges, score bugs, and momentum bars to match each team's primary color.

### Kit preview

Use `kit` to render a simple jersey graphic (shirt/shorts/socks/sleeves) for each team ahead of kickoff.

### Lineup / formation display

Use `formation` to render each team's formation (e.g. on a pitch graphic) ahead of or during a match.

### Gating on confirmed lineups

Use `LINEUP_CONFIRMED` to decide what a pre-match view should show. While `confirmed` is `false`, label formations and squads as predicted and keep a "lineups not yet confirmed" state on screen; once it flips to `true`, the same data can be presented as the real starting eleven. It is also the cheapest way to filter a list of upcoming fixtures down to the ones whose lineups have landed.
