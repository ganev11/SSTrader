---
title: League Average
excerpt: >-
  Reference for league_average developer types returned by the /types endpoint —
  naming convention, value semantics, and usage with /fixtures/search.
deprecated: false
hidden: false
metadata:
  robots: index
---
# League Average Types

`GET /types?developer_type=league_average`

League average types represent **per-match averages** calculated across all fixtures played in a league season. They are attached to fixtures as metrics and can be used as filter criteria in [`/fixtures/search`](/docs/search-fixtures).

***

## Naming Convention

Every `developer_name` follows a strict three-part pattern:

```
{SCOPE}_{METRIC}_{AGGREGATION}
```

| Part          | Values                                                                                                   | Meaning                                                                   |
| ------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `SCOPE`       | `HOME`                                                                                                   | Average calculated from **home team performances only** across the league |
|               | `AWAY`                                                                                                   | Average calculated from **away team performances only** across the league |
|               | `OVERALL`                                                                                                | Combined average: `(HOME + AWAY) / 2`                                     |
| `METRIC`      | `GOALS`, `CORNERS`, `YELLOWCARDS`, `SHOTS_ON_TARGET`, `SHOTS_OFF_TARGET`, `ATTACKS`, `DANGEROUS_ATTACKS` | The measured statistic                                                    |
| `AGGREGATION` | `SCORED`                                                                                                 | Average number of events that occurred per match                          |

**Examples:**

| `developer_name`                | Meaning                                                          |
| ------------------------------- | ---------------------------------------------------------------- |
| `HOME_GOALS_SCORED`             | Average goals scored by the home team per match in this league   |
| `AWAY_GOALS_SCORED`             | Average goals scored by the away team per match in this league   |
| `OVERALL_GOALS_SCORED`          | Average total goals per match in this league `(HOME + AWAY) / 2` |
| `HOME_CORNERS_SCORED`           | Average corners taken by the home team per match                 |
| `AWAY_DANGEROUS_ATTACKS_SCORED` | Average dangerous attacks by the away team per match             |

***

## Full Type Reference

| `type_id` | `developer_name`                   | Scope   | Metric            |
| --------- | ---------------------------------- | ------- | ----------------- |
| 303       | `HOME_GOALS_SCORED`                | Home    | Goals             |
| 304       | `AWAY_GOALS_SCORED`                | Away    | Goals             |
| 305       | `OVERALL_GOALS_SCORED`             | Overall | Goals             |
| 306       | `HOME_CORNERS_SCORED`              | Home    | Corners           |
| 307       | `AWAY_CORNERS_SCORED`              | Away    | Corners           |
| 308       | `OVERALL_CORNERS_SCORED`           | Overall | Corners           |
| 309       | `HOME_YELLOWCARDS_SCORED`          | Home    | Yellow Cards      |
| 310       | `AWAY_YELLOWCARDS_SCORED`          | Away    | Yellow Cards      |
| 311       | `OVERALL_YELLOWCARDS_SCORED`       | Overall | Yellow Cards      |
| 312       | `HOME_DANGEROUS_ATTACKS_SCORED`    | Home    | Dangerous Attacks |
| 313       | `AWAY_DANGEROUS_ATTACKS_SCORED`    | Away    | Dangerous Attacks |
| 314       | `OVERALL_DANGEROUS_ATTACKS_SCORED` | Overall | Dangerous Attacks |
| 315       | `HOME_ATTACKS_SCORED`              | Home    | Attacks           |
| 316       | `AWAY_ATTACKS_SCORED`              | Away    | Attacks           |
| 317       | `OVERALL_ATTACKS_SCORED`           | Overall | Attacks           |
| 318       | `HOME_SHOTS_ON_TARGET_SCORED`      | Home    | Shots on Target   |
| 319       | `AWAY_SHOTS_ON_TARGET_SCORED`      | Away    | Shots on Target   |
| 320       | `OVERALL_SHOTS_ON_TARGET_SCORED`   | Overall | Shots on Target   |
| 321       | `HOME_SHOTS_OFF_TARGET_SCORED`     | Home    | Shots off Target  |
| 322       | `AWAY_SHOTS_OFF_TARGET_SCORED`     | Away    | Shots off Target  |
| 323       | `OVERALL_SHOTS_OFF_TARGET_SCORED`  | Overall | Shots off Target  |

***

## Values and the `/fixtures/search` Endpoint

### How values are stored

All league average values are stored as **integers scaled by 1000**. A value of `2200` represents an average of `2.2` events per match.

| Stored value | Real-world average |
| ------------ | ------------------ |
| `1000`       | 1.0 per match      |
| `1500`       | 1.5 per match      |
| `2200`       | 2.2 per match      |
| `3450`       | 3.45 per match     |

### The OVERALL ÷ 2 rule

`OVERALL_*` values equal `(HOME + AWAY) / 2`. This means they represent the **average contribution per side**, not the full match total.

> **When filtering by match total, divide your target value by 2.**

**Example — find leagues averaging 2.2 total goals per match:**

A match with 2.2 total goals has on average `2.2 / 2 = 1.1` goals from each side, so `OVERALL_GOALS_SCORED` is stored as `1100`.

```
# Leagues with average total goals between 2.0 and 2.5 per match
GET /fixtures/search?where[OVERALL_GOALS_SCORED]=1000:1250
```

**Example — find leagues with high corner counts (10+ total per match):**

```
GET /fixtures/search?where[OVERALL_CORNERS_SCORED]=5000:9999
```

This rule applies to **all** `OVERALL_*` types without exception.

### HOME and AWAY filters

`HOME_*` and `AWAY_*` values do **not** require division — they already represent a single side's contribution.

```
# Leagues where home teams average more than 1.5 goals
GET /fixtures/search?where[HOME_GOALS_SCORED]=1500:9999

# Leagues where away teams average fewer than 1 yellow card
GET /fixtures/search?where[AWAY_YELLOWCARDS_SCORED]=0:999
```

***

## Practical Application & Use Cases

### 1. Find high-scoring leagues (Over 2.5 goals market)

To target leagues where both teams are likely to score, filter for `OVERALL_GOALS_SCORED` above `1250` (= 2.5 total goals ÷ 2).

```
GET /fixtures/search?window=48
  &where[OVERALL_GOALS_SCORED]=1250:9999
  &include=odds
  &filter[odds]=markets:1
```

### 2. Find defensive leagues (Under 2.0 goals market)

```
GET /fixtures/search?window=24
  &where[OVERALL_GOALS_SCORED]=0:1000
  &status=NOT_STARTED
```

### 3. Find corner-heavy leagues

Useful for corners betting markets. 9+ total corners per match → `OVERALL_CORNERS_SCORED` > `4500`.

```
GET /fixtures/search?window=24
  &where[OVERALL_CORNERS_SCORED]=4500:9999
```

### 4. Compare home vs. away scoring strength in a league

A large gap between `HOME_GOALS_SCORED` and `AWAY_GOALS_SCORED` indicates strong home advantage in that league. Fetch fixtures with metrics included and compare the two values client-side.

```
GET /fixtures?league_id=8&include=metrics
  &filter[metrics]=types:303,304
```

### 5. Find card-heavy leagues for disciplinary markets

```
GET /fixtures/search?window=48
  &where[OVERALL_YELLOWCARDS_SCORED]=1750:9999
```

This targets leagues averaging 3.5+ yellow cards per match (1750 × 2 = 3500 = 3.5 cards).

### 6. Build a league stats dashboard

Use all `OVERALL_*` types to build a sortable league comparison table. Fetch fixtures for a date range with `include=metrics` and group by `league.id`, reading the metric values for each fixture.

```
GET /fixtures?start_date=2026-06-01T00:00:00Z&end_date=2026-06-07T23:59:59Z
  &include=metrics
  &filter[metrics]=developer_types:league_average
```

***

## UI Implementation Notes

- **Display values:** Always divide the stored integer by `1000` before displaying (`2200 → 2.2`).
- **OVERALL filter inputs:** When accepting a "total match average" from the user (e.g. "2.5 goals"), divide by `2` before sending to the API (`2.5 → 1.25 → 1250` in API units).
- **HOME/AWAY filter inputs:** Divide by `1000` only — no halving required.
- **Range sliders:** Scale the slider range by `1000`. For goals a reasonable UI range is `0–4.0` per side (`0–4000` in API units).

<br />
