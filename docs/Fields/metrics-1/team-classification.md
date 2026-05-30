---
title: Team Classification
deprecated: false
hidden: false
metadata:
  robots: index
---
## `TEAM_GOALS_CLASSIFICATION`

A numeric metric that classifies a team's overall goal-scoring profile by combining their attack and defense power relative to the league average.

**Scope:** `team`<br />**Locations:** `home`, `away`

***

### Value

An integer from `0` to `3`:

| Value | `meta.classification` | Label        |
| ----- | --------------------- | ------------ |
| `0`   | `Weak`                | Weak Team    |
| `1`   | `Average`             | Average Team |
| `2`   | `Strong`              | Strong Team  |
| `3`   | `Top`                 | Top Team     |

***

### How It Works

Classification is derived from two underlying values calculated against the league baseline (`1.0`):

- `attack_power` — how prolific the team is at scoring goals vs. league average
- `defense_power` — how well the team concedes vs. league average (lower = better)

Both values come from the team's **overall** goals stats (combining home and away).

**Attack categories:**

| Range       | Category       |
| ----------- | -------------- |
| > 1.20      | Top Attack     |
| 1.05 – 1.20 | Strong Attack  |
| 0.90 – 1.05 | Average Attack |
| \< 0.90     | Weak Attack    |

**Defense categories:**

| Range       | Category        |
| ----------- | --------------- |
| \< 0.80     | Top Defense     |
| 0.80 – 0.95 | Strong Defense  |
| 0.95 – 1.10 | Average Defense |
| > 1.10      | Weak Defense    |

The two categories are combined to produce the final classification. For example, Top Attack + Top Defense → **Top Team (3)**; Weak Attack + Weak Defense → **Weak Team (0)**.

***

### Example Response

```json
{
  "team_id": 50366,
  "type_id": 277,
  "developer_name": "TEAM_GOALS_CLASSIFICATION",
  "value": 3,
  "meta": {
    "classification": "Top",
    "classification_type": "Team Goals Classification"
  },
  "location": "home"
}
```

### Usage

Use `meta.classification` for display labels and `value` for numeric comparisons.

**Pre-match strength display** — Show a badge (Weak / Average / Strong / Top) next to each team name before a fixture to give users an instant read on the matchup.

**Mismatch detection** — Compare `value` between home and away teams. A difference of 2+ (e.g. `3` vs `1`) indicates a clear favourite and can be surfaced as a confidence signal.

**Fixture filtering** — Let users filter or sort fixtures by team strength. For example, show only fixtures where both teams have `value >= 2` to focus on high-quality matches.

**Prediction models** — Use `value` as a numerical feature in goal or outcome prediction pipelines alongside other attack and defense power metrics.
