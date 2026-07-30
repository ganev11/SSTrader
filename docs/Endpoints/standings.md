---
title: Standings
excerpt: Returns league table standings for a given season.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Endpoint

```
GET /standings
```

## Query parameters

| Parameter   | Type    | Required | Description                                                     |
| ----------- | ------- | -------- | --------------------------------------------------------------- |
| `season_id` | integer | Yes      | The season to retrieve standings for.                           |
| `language`  | string  | No       | Localization code for team names. Alias: `lang`. Default: `en`. |

## Response

A single `standings` array. Each entry is one team's row in a league table.

```json
{
  "standings": [
    {
      "season": { "id": 271, "name": "2025/2026" },
      "stage": { "id": 12345, "name": "Regular Season" },
      "group": null,
      "position": 1,
      "team": { "id": 50, "name": "Arsenal" },
      "points": 45,
      "result": "Champions League",
      "statistics": [
        { "team_id": 50, "type_id": 1, "developer_name": "OVERALL_MATCHES_PLAYED", "value": 20 },
        { "team_id": 50, "type_id": 2, "developer_name": "OVERALL_WON", "value": 15 }
      ]
    }
  ]
}
```

### Fields

| Field        | Description                                                                                                                                                                                                                             |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `season`     | The season this row belongs to (`id`, `name`).                                                                                                                                                                                          |
| `stage`      | The competition stage this table belongs to (e.g. "Regular Season", "Group Stage"). `null` if the competition has no stages.                                                                                                            |
| `group`      | The group within the stage (e.g. "Group A" in a cup competition). `null` if not applicable.                                                                                                                                             |
| `position`   | The team's rank in this table.                                                                                                                                                                                                          |
| `team`       | `id` and localized `name` of the team.                                                                                                                                                                                                  |
| `points`     | Total points.                                                                                                                                                                                                                           |
| `result`     | Qualification/relegation note for this position (e.g. "Champions League", "Relegation"), if any.                                                                                                                                        |
| `statistics` | Breakdown stats for this team in this table (matches played, won/drawn/lost, goals scored/against, etc.) — same shape as the `statistics` include on `/fixtures`. Available `developer_name` values can be discovered via `GET /types`. |

## Notes

- A season can have more than one table — e.g. a regular season plus a separate playoff stage, or several groups in a cup competition. All tables are returned together in one flat `standings` array; group by `stage.id` and `group.id` on the client if you need to render separate tables.
- An unknown or not-yet-populated `season_id` returns `{ "standings": [] }`, not an error.

## Errors

| Status | Cause                                       |
| ------ | ------------------------------------------- |
| `400`  | `season_id` missing or not a valid integer. |
| `403`  | Missing or insufficient role.               |

<br />
