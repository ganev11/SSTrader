---
title: Trends
excerpt: Per-minute time-series data for each team across the full duration of a match.
deprecated: false
hidden: false
metadata:
  robots: index
---

`include=trends`

The trends include attaches a per-minute breakdown of match events to each fixture. Each entry represents the **cumulative value** of a specific stat for a given team at a given minute of the match — for example, how many corners team A had by minute 34, or how many shots on target by minute 67.

Two computed stats are also appended for every minute: **Expected Goals (xG)** and a **Pressure Index**, both calculated from per-minute data and event details.

> **Availability:** Only returned when `include=trends` is added to the request. A default set of stats is returned when no filter is applied.

---

## Object Schema

Each entry in the `trends` array represents one stat for one team at one minute.

| Field            | Type    | Description                                                                 |
|------------------|---------|-----------------------------------------------------------------------------|
| `team_id`        | integer | The team the entry belongs to.                                              |
| `period_id`      | integer | The period (half) the entry belongs to. Disambiguates repeated minute values across periods. |
| `minute`         | integer | The match minute at which this value was recorded.                          |
| `type_id`        | integer | The external type ID of the stat. Use `/types` to look up names and IDs.   |
| `developer_name` | string  | The constant-style identifier for the stat (e.g. `CORNERS`, `ATTACKS`).    |
| `value`          | integer | The cumulative value of the stat at this minute. See [Value Encoding](#value-encoding) for `EXPECTED_GOALS`. |

---

## Minute Scope

Minute values are **global across the match**, not reset per period. However, the second half always begins at minute **45**, regardless of how much added time occurred in the first half.

| Period       | Minute range                          |
|--------------|---------------------------------------|
| 1st half     | `1` → `45+` (continues into added time) |
| 2nd half     | `45` → `90+` (always starts at 45)   |

**Key rules:**

- Minutes within a period count continuously until that period ends. A 1st half with 3 minutes of added time produces entries up to minute `48`.
- When the 2nd half kicks off, the minute counter **resets to 45** — not to wherever the 1st half ended. There is no gap or overlap in the stored data; use `period_id` to distinguish entries at the same minute value across the two halves.
- Use `period_id` as the disambiguator when plotting: entries at minute `45` can exist in both halves, and `period_id` tells you which is which.

### Value Continuity Across Periods

**Cumulative `value` fields are never reset between periods.** Each new period picks up where the previous one left off — a team that had 6 corners by the end of the first half will show 6+ corners at the start of the second half, not 0.

This applies to all periods, including additional time in cup matches where both teams are level after 90 minutes:

| Period            | Minute range                                 | Value behaviour                          |
|-------------------|----------------------------------------------|------------------------------------------|
| 1st half          | `1` → `45+`                                  | Starts from 0                            |
| 2nd half          | `45` → `90+`                                 | Continues from end of 1st half           |
| Extra time 1st    | `90` → `105+`                                | Continues from end of 2nd half           |
| Extra time 2nd    | `105` → `120+`                               | Continues from end of extra time 1st     |

When plotting a single stat across all periods, use the raw `value` directly — no per-period offset is needed. To isolate a period's contribution, subtract the last `value` of the preceding period from the first `value` of the current one.

---

## Available Stats

### Default Stats

Returned when no `filter[trends]` is provided.

| `type_id` | `developer_name`    | Description                                        |
|-----------|---------------------|----------------------------------------------------|
| `15`      | `GOALS`             | Cumulative goals scored.                           |
| `14`      | `PENALTIES`         | Penalty kicks awarded.                             |
| `53`      | `PENALTIES_MISSES`  | Penalties missed.                                  |
| `22`      | `SHOTS_ON_TARGET`   | Shots that required a save or resulted in a goal.  |
| `12`      | `SHOTS_OFF_TARGET`  | Shots that missed the target.                      |
| `11`      | `SHOTS_TOTAL`       | Total shots (on + off target).                     |
| `27`      | `SHOTS_INSIDEBOX`   | Shots taken from inside the penalty area.          |
| `28`      | `SHOTS_OUTSIDEBOX`  | Shots taken from outside the penalty area.         |
| `25`      | `BALL_POSSESSION`   | Percentage of ball possession.                     |
| `13`      | `CORNERS`           | Corner kicks taken.                                |
| `9`       | `ATTACKS`           | Total attacking moves recorded.                    |
| `10`      | `DANGEROUS_ATTACKS` | Attacking moves that reached a dangerous position. |
| `20`      | `YELLOWCARDS`       | Yellow cards received.                             |
| `19`      | `REDCARDS`          | Red cards received.                                |
| `16`      | `SUBSTITUTIONS`     | Substitutions made.                                |
| `23`      | `INJURIES`          | Injuries recorded.                                 |
| `117`     | `EXPECTED_GOALS`    | Calculated xG × 100 as an integer. See [Value Encoding](#value-encoding). |
| `118`     | `PRESSURE_INDEX`    | A composite pressure score reflecting how aggressively a team attacked in the recent minutes. |

### Computed Stats

Calculated from per-minute data and event details. Included in the default response and available via filter.

| `type_id` | `developer_name`  | Description                                                                                         |
|-----------|-------------------|-----------------------------------------------------------------------------------------------------|
| `117`     | `EXPECTED_GOALS`  | Expected goals (xG) at the given minute, based on shots, dangerous attacks, corners and penalties. Stored as an integer — divide by 100 to get the decimal xG. |
| `118`     | `PRESSURE_INDEX`  | A composite pressure score reflecting how aggressively a team attacked in the recent minutes.        |

---

## Value Encoding

### EXPECTED_GOALS

`EXPECTED_GOALS` values are stored as **integers scaled by 100**. To convert to the standard xG decimal, divide by `100`.

| `value` | xG decimal |
|---------|-----------|
| `18`    | `0.18`    |
| `75`    | `0.75`    |
| `142`   | `1.42`    |

**Example:**

```json
{
  "team_id": 85,
  "period_id": 2101938,
  "minute": 34,
  "type_id": 117,
  "developer_name": "EXPECTED_GOALS",
  "value": 18
}
```

This means the team had **0.18 xG** by minute 34.

---

## Filtering

Add `filter[trends]` to limit which stats are returned. Two filter formats are supported.

### By developer name — `developer_names`

Returns only the stats matching the listed names. **Takes priority over `types` when both are provided.**

```
filter[trends]=developer_names:GOALS,CORNERS,DANGEROUS_ATTACKS
```

### By type ID — `types`

Returns only the stats matching the listed external type IDs. Ignored when `developer_names` is also set.

```
filter[trends]=types:117,118
```

---

## Request Examples

**All trends with defaults (no filter):**

```
GET /fixtures?fixture_id=1063864&include=trends
```

**Only goals and corners per minute:**

```
GET /fixtures?fixture_id=1063864&include=trends&filter[trends]=developer_names:GOALS,CORNERS
```

**Expected Goals and Pressure Index only:**

```
GET /fixtures?fixture_id=1063864&include=trends&filter[trends]=developer_names:EXPECTED_GOALS,PRESSURE_INDEX
```

**Full trend picture for live match analysis:**

```
GET /fixtures?status=INPLAY_1ST_HALF,INPLAY_2ND_HALF&include=trends&filter[trends]=developer_names:ATTACKS,DANGEROUS_ATTACKS,SHOTS_ON_TARGET,SHOTS_OFF_TARGET,CORNERS,EXPECTED_GOALS,PRESSURE_INDEX
```

---

## Example Response

```json
"trends": [
  {
    "team_id": 85,
    "period_id": 2101938,
    "minute": 12,
    "type_id": 9,
    "developer_name": "ATTACKS",
    "value": 34
  },
  {
    "team_id": 85,
    "period_id": 2101938,
    "minute": 12,
    "type_id": 13,
    "developer_name": "CORNERS",
    "value": 2
  },
  {
    "team_id": 85,
    "period_id": 2101938,
    "minute": 12,
    "type_id": 117,
    "developer_name": "EXPECTED_GOALS",
    "value": 18
  },
  {
    "team_id": 85,
    "period_id": 2101938,
    "minute": 12,
    "type_id": 118,
    "developer_name": "PRESSURE_INDEX",
    "value": 42
  }
]
```

> `EXPECTED_GOALS` value `18` = **0.18 xG** (divide by 100).

---

## Real-World Use Cases

### Live match dashboard

Fetch all default trend stats for live matches and plot each stat as a time series. The `minute` field is your X-axis; the `value` is your Y-axis. Separate by `team_id` to show home vs. away lines side by side.

### Momentum detection

Use `DANGEROUS_ATTACKS` and `PRESSURE_INDEX` filtered by the last 10–15 minutes to detect which team is currently dominating. A rising `PRESSURE_INDEX` combined with a low `GOALS` value often signals an imminent scoring opportunity.

### xG chart

Request `EXPECTED_GOALS` (type_id `117`) for both teams across the full match. Plot cumulative xG per minute to show how the match evolved and whether the final score reflects the balance of play. Divide each `value` by `100` to get the decimal xG before plotting.

### Pre-match data review

Trends are stored for completed matches. Fetch historical fixtures with `include=trends` to analyse how a team typically builds up attacking pressure across a 90-minute window before predicting outcomes.

### Substitution impact analysis

Combine `ATTACKS` and `DANGEROUS_ATTACKS` before and after the minute of a `SUBSTITUTIONS` event to measure whether a tactical change shifted momentum for a team.