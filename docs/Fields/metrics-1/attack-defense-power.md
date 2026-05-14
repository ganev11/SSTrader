---
title: Attack / Defense Power
deprecated: false
hidden: false
metadata:
  robots: index
---
Attack Power and Defense Power measures how dangerous a team is going forward vs the league average.
Both are devided into 3 types: home, away and overall (home + away).

## Goals Attack Power (AP)
1.00 = league average
Values above 1 = more goal chances than average
| Label | Range |
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
| Label | Range |
| --- | --- |
| Top | Below 0.60 |
| Strong | 0.60–0.89 |
| Stable | 0.90–1.29 |
| Weak | 1.30–1.79 |
| Poor | 1.80+ |

> Example: 0.85 = 15% fewer goals conceded than league average (solid defense)


## Example Interpretation

Comparing two teams based on their Home AP and Home DP: Manchester City vs Crystal Palace - 2026-05-13

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
            "type_id": 237,
            "developer_name": "AWAY_GOALS_ATTACK_POWER",
            "value": 1.45,
            "location": "home",
            "meta": {
                "classification": "Strong",
                "classification_type": "Attack Power"
            },
            "classification": "Strong", // Deprecated: use meta.classification instead
            "classification_type": "Attack Power" // Deprecated: use meta.classification_type instead
        },
        {
            "team_id": 48,
            "type_id": 235,
            "developer_name": "HOME_GOALS_ATTACK_POWER",
            "value": 0.82,
            "location": "away",
            "meta": {
                "classification": "Low",
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

Team A has a Home AP of 1.20 and a Home DP of 0.80. This means that at home, Team A creates 20% more goal chances than the league average (strong attack) and concedes 20% fewer goals than the league average (strong defense). Overall, Team A is likely to be a strong contender in their home matches.