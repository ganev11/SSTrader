---
title: Attack / Defense Power
deprecated: false
hidden: false
metadata:
  robots: index
---
Attack Power and Defense Power measures how dangerous a team is going forward vs the league average.

Both are devided into 3 types: `HOME`, `AWAY` and `OVERALL` (home + away). Each developer_name starts with the location (HOME/AWAY/OVERALL) followed by the statistic type (GOALS, CORNERS, CARDS etc.) and ends with the classification type (ATTACK_POWER or DEFENSE_POWER). For example, `HOME_GOALS_ATTACK_POWER` is the attack power of a team when playing at home based on goals.

## Goals Attack Power (AP)
1.00 = league average
Values above 1 = more goal chances than average
Classification type "Attack Power"

| Classification | Range |
| --- | --- |
| Top | 1.80+ |
| Strong | 1.30–1.79 |
| Stable | 0.90–1.29 |
| Weak | 0.60–0.89 |
| Poor | Below 0.60 |
> Example: 1.12 = 12% more goal chances than league average

## Goals Defensive Power (DP)
Measures how solid a team is defensively vs the league average.
1.00 = league average
Lower values = better defense (fewer goals conceded)
Classification type "Defense Power"

| Classification | Range |
| --- | --- |
| Top | Below 0.60 |
| Strong | 0.60–0.89 |
| Stable | 0.90–1.29 |
| Weak | 1.30–1.79 |
| Poor | 1.80+ |

> Example: 0.85 = 15% fewer goals conceded than league average (solid defense)


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