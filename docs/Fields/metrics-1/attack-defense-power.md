---
title: Attack / Defense Power
deprecated: false
hidden: false
metadata:
  robots: index
---
Attack Power and Defense Power measures how dangerous a team is going forward vs the league average.

Both are devided into 3 types: `HOME`, `AWAY` and `OVERALL` (home + away). Each developer_name starts with the location (HOME/AWAY/OVERALL) followed by the statistic type (GOALS, CORNERS, CARDS etc.) and ends with the classification type (ATTACK_POWER or DEFENSE_POWER). For example, `HOME_GOALS_ATTACK_POWER` is the attack power of a team when playing at home based on goals.

> Alway use team_id or location to determine if the metric is for home or away team. Do not rely only on the developer_name prefix as **both teams have metrics for each location**.

## Goals Attack Power (AP)
1.00 = league average
Values above 1 = more goal chances than average
Classification type "Attack Power"

| Classification | Range |
| --- | --- |
| Explosive | 1.70+ |
| Strong | 1.30–1.70 |
| Solid | 1.10–1.29 |
| Low | 0.80–1.09 |
| Weak | Below 0.80 |
> Example: 1.12 = 12% more goal chances than league average

## Goals Defensive Power (DP)
Measures how solid a team is defensively vs the league average.
1.00 = league average
Lower values = better defense (fewer goals conceded)
Classification type "Defense Power"

| Classification | Range |
| --- | --- |
| Wall | Below 0.70 |
| Solid | 0.70–0.89 |
| Stable | 0.90–1.04 |
| Leaky | 1.05–1.29 |
| Fragile | 1.30+ |

> Example: 0.85 = 15% fewer goals conceded than league average (solid defense)

## Corners Attack Power (AP)
Measures how many corner chances a team creates vs the league average.
1.00 = league average
Values above 1 = more corner chances than average
Classification type "Corners Winning Label"

| Classification | Range |
| --- | --- |
| Constantly | 1.50+ |
| Very Often | 1.25–1.49 |
| Frequently | 1.10–1.24 |
| Often | 0.95–1.09 |
| Sometimes | 0.85–0.94 |
| Occasionally | 0.70–0.84 |
| Rarely | Below 0.70 |
> Example: 1.12 = 12% more corner chances than league average

## Corners Defensive Power (DP)
Measures how many corner chances a team concedes vs the league average.
1.00 = league average
Lower values = better defense (fewer corners conceded)
Classification type "Corners Conceding Label"

| Classification | Range |
| --- | --- |
| Rarely | Below 0.70 |
| Occasionally | 0.70–0.84 |
| Sometimes | 0.85–0.94 |
| Often | 0.95–1.09 |
| Frequently | 1.10–1.24 |
| Very Often | 1.25–1.49 |
| Constantly | 1.50+ |

> Example: 0.85 = 15% fewer corner chances conceded than league average (solid defense)

## Yellow Cards Attack Power (AP)
Measures how many yellow cards a team receives vs the league average.
Classification type "Discipline Label"

| Classification | Range |
| --- | --- |
| Excellent | Below 0.70 |
| Very Good | 0.70–0.84 |
| Good | 0.85–0.94 |
| Fair | 0.95–1.09 |
| Poor | 1.10–1.24 |
| Very Poor | 1.25+ |

## Yellow Cards Defensive Power (DP)
Measures how many yellow cards a team concedes vs the league average.
Classification type "Aggression Label"

| Classification | Range |
| --- | --- |
| Calm | Below 0.70 |
| Controlled | 0.70–0.84 |
| Moderate | 0.85–0.94 |
| Physical | 0.95–1.09 |
| Aggressive | 1.10–1.24 |
| Intense | 1.25+ |

## Example Interpretation

Comparing two teams based on their Goals AP and DP.
Metrics are from fixture: Manchester City vs Crystal Palace - 2026-05-13

Manchester City has a Home AP of 1.59 and a Home DP of 0.58. This means that at home, Manchester City creates 59% more goal chances than the league average (strong attack) and concedes 42% fewer goals than the league average (wall defense). Overall, Manchester City is likely to be a strong contender in their home matches.

Crystal Palace has an Away AP of 0.94 and an Away DP of 0.87. This indicates that when playing away, Crystal Palace creates 6% fewer goal chances than the league average (stable attack) and concedes 13% fewer goals than the league average (solid defense). While their attack is slightly below average, their defense is relatively strong for away matches.

```json
{
    "metrics": [
        {
            "team_id": 9,
            "type_id": 235,
            "developer_name": "HOME_GOALS_ATTACK_POWER",
            "value": 1.59,
            "location": "home",
            "meta": {
                "classification": "Strong",
                "classification_type": "Attack Power"
            },
            "classification": "Strong", // Deprecated: use meta.classification instead
            "classification_type": "Attack Power" // Deprecated: use meta.classification_type instead
        },
        {
            "team_id": 9,
            "type_id": 236,
            "developer_name": "HOME_GOALS_DEFENSE_POWER",
            "value": 0.58,
            "location": "home",
            "meta": {
                "classification": "Wall",
                "classification_type": "Defense Power"
            },
            "classification": "Wall", // Deprecated: use meta.classification instead
            "classification_type": "Defense Power" // Deprecated: use meta.classification_type instead
        },
        {
            "team_id": 48,
            "type_id": 237,
            "developer_name": "AWAY_GOALS_ATTACK_POWER",
            "value": 0.94,
            "location": "away",
            "meta": {
                "classification": "Stable",
                "classification_type": "Attack Power"
            },
            "classification": "Low", // Deprecated: use meta.classification instead
            "classification_type": "Attack Power" // Deprecated: use meta.classification_type instead
        },
        {
            "team_id": 48,
            "type_id": 238,
            "developer_name": "AWAY_GOALS_DEFENSE_POWER",
            "value": 0.87,
            "location": "away",
            "meta": {
                "classification": "Solid",
                "classification_type": "Defense Power"
            },
            "classification": "Solid", // Deprecated: use meta.classification instead
            "classification_type": "Defense Power" // Deprecated: use meta.classification_type instead
        }
    ]
}
```