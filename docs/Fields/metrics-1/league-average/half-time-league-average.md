---
title: Half-Time League Average
excerpt: >-
  Reference for HT league_average developer types — naming convention, value
  semantics, how they differ from full-time averages, and usage with
  /fixtures/search.
deprecated: false
hidden: false
metadata:
  robots: index
---
# Half-Time League Average Types

`GET /types?developer_type=league_average`

Half-time league average types represent **per-match averages calculated from the first half only**, across all fixtures played in a league season. They are identified by the `HT_` prefix and follow the same storage and filtering rules as full-time league averages.

***

## Naming Convention

Every half-time `developer_name` follows a four-part pattern:

```
HT_{SCOPE}_{METRIC}_{AGGREGATION}
```

| Part          | Values                                                                                                   | Meaning                                                            |
| ------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| `HT`          | _(fixed)_                                                                                                | Marks the value as a **first-half** average                        |
| `SCOPE`       | `HOME`                                                                                                   | Average from **home team first-half performances** only            |
|               | `AWAY`                                                                                                   | Average from **away team first-half performances** only            |
|               | `OVERALL`                                                                                                | Combined average: `(HOME + AWAY) / 2`                              |
| `METRIC`      | `GOALS`, `CORNERS`, `YELLOWCARDS`, `SHOTS_ON_TARGET`, `SHOTS_OFF_TARGET`, `ATTACKS`, `DANGEROUS_ATTACKS` | The measured statistic                                             |
| `AGGREGATION` | `SCORED`                                                                                                 | Average number of events that occurred per match in the first half |

**Examples:**

| `developer_name`                | Meaning                                                                |
| ------------------------------- | ---------------------------------------------------------------------- |
| `HT_HOME_GOALS_SCORED`          | Average first-half goals scored by the home team per match             |
| `HT_AWAY_CORNERS_SCORED`        | Average first-half corners taken by the away team per match            |
| `HT_OVERALL_YELLOWCARDS_SCORED` | Average first-half yellow cards per side per match `(HOME + AWAY) / 2` |

***

## Full Type Reference

| `type_id` | `developer_name`                      | Scope   | Metric            |
| --------- | ------------------------------------- | ------- | ----------------- |
| 324       | `HT_HOME_GOALS_SCORED`                | Home    | Goals             |
| 325       | `HT_AWAY_GOALS_SCORED`                | Away    | Goals             |
| 326       | `HT_OVERALL_GOALS_SCORED`             | Overall | Goals             |
| 327       | `HT_HOME_CORNERS_SCORED`              | Home    | Corners           |
| 328       | `HT_AWAY_CORNERS_SCORED`              | Away    | Corners           |
| 329       | `HT_OVERALL_CORNERS_SCORED`           | Overall | Corners           |
| 330       | `HT_HOME_YELLOWCARDS_SCORED`          | Home    | Yellow Cards      |
| 331       | `HT_AWAY_YELLOWCARDS_SCORED`          | Away    | Yellow Cards      |
| 332       | `HT_OVERALL_YELLOWCARDS_SCORED`       | Overall | Yellow Cards      |
| 333       | `HT_HOME_ATTACKS_SCORED`              | Home    | Attacks           |
| 334       | `HT_AWAY_ATTACKS_SCORED`              | Away    | Attacks           |
| 335       | `HT_OVERALL_ATTACKS_SCORED`           | Overall | Attacks           |
| 336       | `HT_HOME_SHOTS_ON_TARGET_SCORED`      | Home    | Shots on Target   |
| 337       | `HT_AWAY_SHOTS_ON_TARGET_SCORED`      | Away    | Shots on Target   |
| 338       | `HT_OVERALL_SHOTS_ON_TARGET_SCORED`   | Overall | Shots on Target   |
| 339       | `HT_HOME_SHOTS_OFF_TARGET_SCORED`     | Home    | Shots off Target  |
| 340       | `HT_AWAY_SHOTS_OFF_TARGET_SCORED`     | Away    | Shots off Target  |
| 341       | `HT_OVERALL_SHOTS_OFF_TARGET_SCORED`  | Overall | Shots off Target  |
| 342       | `HT_HOME_DANGEROUS_ATTACKS_SCORED`    | Home    | Dangerous Attacks |
| 343       | `HT_AWAY_DANGEROUS_ATTACKS_SCORED`    | Away    | Dangerous Attacks |
| 344       | `HT_OVERALL_DANGEROUS_ATTACKS_SCORED` | Overall | Dangerous Attacks |

***

## Values and the `/fixtures/search` Endpoint

### How values are stored

All half-time average values are stored as **integers scaled by 1000**, identical to full-time averages.

| Stored value | Real-world average |
| ------------ | ------------------ |
| `500`        | 0.5 per first half |
| `800`        | 0.8 per first half |
| `1200`       | 1.2 per first half |

### The OVERALL ÷ 2 rule

`HT_OVERALL_*` values equal `(HT_HOME + HT_AWAY) / 2`. They represent the **average contribution per side**, not the full first-half match total. The same rule as full-time `OVERALL_*` applies.

> **When filtering by first-half match total, divide your target by 2.**

**Example — find leagues averaging 1.0 total first-half goals per match:**

`1.0 / 2 = 0.5` per side → `HT_OVERALL_GOALS_SCORED` stored as `500`.

```
GET /fixtures/search?where[HT_OVERALL_GOALS_SCORED]=500:9999
```

### HOME and AWAY filters

`HT_HOME_*` and `HT_AWAY_*` values represent a single side's contribution and require no halving.

```
# Leagues where home teams average more than 0.8 first-half goals
GET /fixtures/search?where[HT_HOME_GOALS_SCORED]=800:9999

# Leagues where away teams average fewer than 3 first-half corners
GET /fixtures/search?where[HT_AWAY_CORNERS_SCORED]=0:2999
```

***

## Differences from Full-Time Averages

| Aspect                | Full-time (`*_SCORED`)        | Half-time (`HT_*_SCORED`)                          |
| --------------------- | ----------------------------- | -------------------------------------------------- |
| Period covered        | Entire match                  | First half only                                    |
| `type_id` range       | 303–323                       | 324–344                                            |
| Typical goals value   | \~1.2–1.6 per side            | \~0.5–0.8 per side                                 |
| Typical corners value | \~4–6 per side                | \~2–3 per side                                     |
| Developer name prefix | _(none)_                      | `HT_`                                              |
| Calculation source    | `fixture_trends` (full match) | `fixture_trends` joined with `periods.type_id = 1` |

***

## Practical Use Cases

### 1. Find high first-half scoring leagues

Target leagues where first-half goals are likely, useful for HT result markets.

```
GET /fixtures/search?window=48
  &where[HT_OVERALL_GOALS_SCORED]=600:9999
  &include=odds
```

### 2. Find leagues with active first-half corner markets

```
GET /fixtures/search?window=24
  &where[HT_OVERALL_CORNERS_SCORED]=2500:9999
  &status=NOT_STARTED
```

### 3. Compare first-half vs full-time goal rates

Fetch both `HT_HOME_GOALS_SCORED` (type 324) and `HOME_GOALS_SCORED` (type 303) together to compute the first-half scoring ratio for each league.

```
GET /fixtures?league_id=8&include=metrics
  &filter[metrics]=types:303,304,324,325
```

A ratio above \~0.55 means the league tends to produce goals early — valuable for live betting models.

***

## UI Implementation Notes

- **Display values:** Divide stored integer by `1000` (`800 → 0.8`).
- **Labels:** Prefix the metric name with "HT" or "1st Half" in the UI to distinguish from full-time figures.
- **OVERALL filter inputs:** When accepting a "first-half total" from the user (e.g. "1.0 goals"), divide by `2` before sending (`1.0 → 0.5 → 500` in API units).
- **HOME/AWAY filter inputs:** Divide by `1000` only — no halving required.
- **Range sliders:** Scale by `1000`. A reasonable UI range for first-half goals is `0–2.0` per side (`0–2000` in API units).

<br />
