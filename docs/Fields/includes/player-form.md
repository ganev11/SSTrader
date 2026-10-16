---
title: Player Form
excerpt: >-
  Each squad player's last five appearances before a fixture, with minutes,
  shots, goals, assists, cards, fouls, tackles and saves in each game.
deprecated: false
hidden: false
metadata:
  robots: index
---
`include=player_form`

The `player_form` include returns the recent games of every player in both squads: his last **5 appearances** before this fixture's kick-off, with his statistics in each one. It is meant to sit next to the matching [player metric](/docs/player-metrics). For example, you can show the goals from his last five games next to `PLAYER_EXPECTED_GOALS`, or his shots next to `PLAYER_EXPECTED_SHOTS`.

> **Requires `squads`.** Request them together: `include=squads,player_form`. Entries identify players by `player_id` only; `squads` carries their names and positions.

> **Availability:** Only returned when `include=player_form` is added to a request to `GET /fixtures`. Not available on `/fixtures/search` or `/livescores`.

## Fixed per fixture

The window ends at **this fixture's kick-off**, so a fixture returns the same entries before, during and after the match. Querying a match from last month shows the player's form as it was going into that match, not his form today.

## What counts as an appearance

- A finished game (including extra time and penalties) in **any competition**: league, cups and internationals.
- The player was on the pitch: `MINUTES_PLAYED` above `0`. An unused substitute did not play and gets no entry.
- Within the 365 days before kick-off. A player returning from a long absence can have fewer than 5 entries, and one with no appearance in that window has none.

---

## Object Schema

`player_form` is a flat array covering **both** teams' current squads. Entries are not grouped: use `team_id` and `player_id` to split them. Each entry is one past appearance of one player:

| Field        | Type    | Description |
|--------------|---------|-------------|
| `team_id`    | integer | The player's team **in this fixture**. Match it against `participants[].id` to tell home from away. |
| `player_id`  | integer | The player. Join to `squads[].player_id` for name and position. |
| `fixture_id` | integer | The past fixture the statistics come from. |
| `date_time`  | string  | Kick-off of the past fixture, ISO 8601 in UTC. |
| `statistics` | array   | The player's statistics in that game, see below. |

Entries are ordered by `team_id`, then `player_id`, then `date_time` **newest first**.

### Statistic object

| Field            | Type    | Description |
|------------------|---------|-------------|
| `type_id`        | integer | The unique ID of the statistic type. |
| `developer_name` | string  | Constant-style identifier, see the table below. |
| `value`          | integer | The player's count in that game (minutes for `MINUTES_PLAYED`). |

| `type_id` | `developer_name`  | Sits next to |
|-----------|-------------------|--------------|
| `42`      | `MINUTES_PLAYED`  | `PLAYER_EXPECTED_MINUTES` |
| `11`      | `SHOTS_TOTAL`     | `PLAYER_EXPECTED_SHOTS` |
| `22`      | `SHOTS_ON_TARGET` | `PLAYER_EXPECTED_SHOTS_ON_TARGET` |
| `15`      | `GOALS`           | `PLAYER_EXPECTED_GOALS` |
| `17`      | `ASSISTS`         | `PLAYER_EXPECTED_ASSISTS` |
| `20`      | `YELLOWCARDS`     | `PLAYER_EXPECTED_BOOKING_POINTS` |
| `19`      | `REDCARDS`        | `PLAYER_EXPECTED_BOOKING_POINTS` |
| `21`      | `YELLOWRED_CARDS` | `PLAYER_EXPECTED_BOOKING_POINTS` |
| `31`      | `FOULS`           | `PLAYER_EXPECTED_FOULS` |
| `73`      | `TACKLES`         | `PLAYER_EXPECTED_TACKLES` |
| `32`      | `SAVES`           | `PLAYER_EXPECTED_SAVES` |

### Missing data versus zero

**A statistic is present only when it was recorded for that game.** When it is present, `0` is a real zero. When it is absent, the game's data does not cover it. Never read an absent statistic as `0`.

Coverage differs per competition and per statistic. A game can carry shots but no tackles, or cards but no shots. So look statistics up by `developer_name` rather than by position in the array, and when summing over the last five games, count how many games actually carried the statistic.

- `MINUTES_PLAYED` is always present.
- `SAVES` is returned for the goalkeeper of that game only.
- Card statistics are left out for a team in a game where a card was shown that the data cannot attribute to a player, because that card may have been his.

---

## Example

```
GET /fixtures?fixture_id=19134520&include=squads,player_form
```

```json
"player_form": [
  {
    "team_id": 4009,
    "player_id": 184521,
    "fixture_id": 19134512,
    "date_time": "2026-10-11T19:00:00.000Z",
    "statistics": [
      { "type_id": 42, "developer_name": "MINUTES_PLAYED",  "value": 90 },
      { "type_id": 11, "developer_name": "SHOTS_TOTAL",     "value": 3 },
      { "type_id": 22, "developer_name": "SHOTS_ON_TARGET", "value": 2 },
      { "type_id": 15, "developer_name": "GOALS",           "value": 1 },
      { "type_id": 17, "developer_name": "ASSISTS",         "value": 0 },
      { "type_id": 20, "developer_name": "YELLOWCARDS",     "value": 0 },
      { "type_id": 19, "developer_name": "REDCARDS",        "value": 0 },
      { "type_id": 21, "developer_name": "YELLOWRED_CARDS", "value": 0 },
      { "type_id": 31, "developer_name": "FOULS",           "value": 1 },
      { "type_id": 73, "developer_name": "TACKLES",         "value": 2 }
    ]
  }
]
```

To show "goals in the last 5" next to expected goals, take the player's entries and sum the `GOALS` values of the entries that carry one.
