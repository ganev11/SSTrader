---
title: Match Events
excerpt: >-
  Goals, cards, substitutions, missed penalties, shootout kicks and VAR reviews
  of a football fixture, live and after the match.
deprecated: false
hidden: false
metadata:
  robots: index
---
---
title: Match Events
description: Goals, cards, substitutions, missed penalties, shootout kicks and VAR reviews of a football fixture, live and after the match.
---

# Match Events

`include=events`

The `events` include returns the timeline of a football match: goals, own goals, penalties, missed penalties, cards, substitutions, penalty shootout kicks and VAR reviews, each with its team, player(s) and minute.

> **One fixture per request.** `include=events` needs exactly one `fixture_id`: `GET /football/fixtures?fixture_id=1189574&include=events`. Any other request with it returns **400** (`include=events needs exactly one fixture_id`).

> **Availability:** `GET /football/fixtures` only. Not available on `/football/fixtures/search` or `/football/livescores`, and not for basketball.

## Live, and corrected afterwards

Events are added during the match, usually a few seconds after the score changes. They are the data provider's account of the match and can be corrected later:

- An event can be **replaced**, **removed**, or appear **twice** for a short time.
- A goal that VAR **disallows** gets a `VAR` event with subtype `GOAL_DISALLOWED`, and its goal event is usually removed.
- A `VAR` or `VAR_CARD` event with subtype `REVIEW` means a review is **in progress**; its outcome follows as another event.

So use `events` to show and explain what happened, and read the score from the fixture's `scores`, not by counting goal events.

---

## Object Schema

`events` is an array, ordered by period, `minute`, `extra_minute`, then `id`.

| Field                    | Type            | Description |
|--------------------------|-----------------|-------------|
| `id`                     | integer         | The event's id. New events get higher ids: see `since_event_id` below. |
| `type_id`                | integer \| null | The event type, ids `426`–`437` (see the table below). |
| `developer_name`         | string \| null  | The event type's name, e.g. `GOAL`. |
| `subtype_id`             | integer \| null | A detail of the event, ids `438`–`479`, `428` or `429`: card reason, goal kind, penalty outcome, substitution reason or VAR outcome. Often null. |
| `subtype_developer_name` | string \| null  | The subtype's name, e.g. `GOAL_DISALLOWED`. |
| `team_id`                | integer \| null | The team of the event. For an `OWNGOAL`, the team **credited** with the goal (the scorer's opponent). |
| `side`                   | string \| null  | `home` or `away`: `team_id` in this fixture. |
| `player_id`              | integer \| null | The player. For a substitution, the player coming **on**. |
| `player_name`            | string \| null  | The player's name. Present even when the fixture has no lineups. |
| `related_player_id`      | integer \| null | A second player: the assist of a goal, the player going **off** in a substitution. |
| `related_player_name`    | string \| null  | The second player's name. |
| `period`                 | string \| null  | `1ST_HALF`, `2ND_HALF`, `ET` or `PENALTY_SHOOTOUT`. |
| `minute`                 | integer \| null | Match minute. |
| `extra_minute`           | integer \| null | Added time: `3` for 45+3. |
| `result`                 | string \| null  | The score after a goal, home first (`2-1`). Null on other events. |
| `info`                   | string \| null  | Free text from the data provider, e.g. `Offside`, `Penalty`, `Video review`. Can be empty. Do not parse it: use `subtype_id`. |

`type_id` and `subtype_id` are null only for a type we do not publish yet. Treat an unknown id or name as forward compatibility rather than an error. All types are listed by `GET /football/types?developer_types=event,sub_event`.

### Event types

| `type_id` | `developer_name`        | Meaning |
|-----------|-------------------------|---------|
| 426       | `VAR`                   | VAR review. The subtype is the outcome, or `REVIEW` while in progress. |
| 427       | `GOAL`                  | Goal |
| 428       | `OWNGOAL`               | Own goal |
| 429       | `PENALTY`               | Goal from a penalty |
| 430       | `MISSED_PENALTY`        | Missed penalty |
| 431       | `SUBSTITUTION`          | Substitution |
| 432       | `YELLOWCARD`            | Yellow card |
| 433       | `REDCARD`               | Straight red card |
| 434       | `YELLOWREDCARD`         | Second yellow, and so a red card |
| 435       | `PENALTY_SHOOTOUT_MISS` | Shootout kick missed |
| 436       | `PENALTY_SHOOTOUT_GOAL` | Shootout kick scored |
| 437       | `VAR_CARD`              | VAR review of a card |

### VAR outcomes (subtypes)

| `subtype_id` | `subtype_developer_name` | On |
|--------------|--------------------------|----|
| 455          | `GOAL_DISALLOWED`        | `VAR` |
| 477          | `GOAL_AWARDED`           | `VAR` |
| 473          | `PENALTY_CONFIRMED`      | `VAR` |
| 474          | `PENALTY_CANCELLED`      | `VAR` |
| 475          | `CARD_ADJUSTED`          | `VAR_CARD` |
| 476          | `RED_CARD_CANCELLED`     | `VAR_CARD`, `VAR` |
| 478          | `REVIEW`                 | `VAR`, `VAR_CARD`: review in progress |

The other subtypes are card reasons (`FOUL`, `ARGUMENT`, `HANDBALL`, `TIME_WASTING`, `SIMULATION`, `VIOLENT_CONDUCT`, ...), goal kinds (`RIGHT_FOOT_SHOT`, `LEFT_FOOT_SHOT`, `HEADER`, `FREE_KICK`, ...; also `PENALTY` and `OWNGOAL`), penalty outcomes (`PENALTY_SAVED_BY_GOALKEEPER`, `PENALTY_SHOT_OFF_TARGET`, `WOODWORK`) and substitution reasons (`TACTICAL_SUB`, `SUB_BECAUSE_OF_INJURY`).

---

## Filtering — `filter[events]`

| Key              | Example                | Description |
|------------------|------------------------|-------------|
| `types`          | `types:426,437`        | Event or subtype ids. An event matches when its `type_id` **or** its `subtype_id` is listed. |
| `since_event_id` | `since_event_id:3553900` | Only events with an `id` above this one. |

Keys are separated by `;`, values inside a key by `,`. Unknown keys, ids that are not event or subtype ids, and values that are not whole numbers return **400**.

```
# Every VAR review of the match
GET /football/fixtures?fixture_id=1189574&include=events&filter[events]=types:426,437

# Only disallowed goals
GET /football/fixtures?fixture_id=1189574&include=events&filter[events]=types:455

# Events added since the last one you hold
GET /football/fixtures?fixture_id=1189574&include=events&filter[events]=since_event_id:3553900
```

`since_event_id` returns only events with a higher id, so an event that is corrected in place or **removed** is not reported. To see the full current list, request without `since_event_id`.

---

## Example

```json
{
  "fixtures": [
    {
      "id": 1189574,
      "events": [
        {
          "id": 3553717,
          "type_id": 428,
          "developer_name": "OWNGOAL",
          "subtype_id": 428,
          "subtype_developer_name": "OWNGOAL",
          "team_id": 23350,
          "side": "home",
          "player_id": 369795,
          "player_name": "Nené",
          "related_player_id": 395166,
          "related_player_name": "Lê Văn Đô",
          "period": "1ST_HALF",
          "minute": 4,
          "extra_minute": null,
          "result": "1-0",
          "info": "Own Goal"
        },
        {
          "id": 3553959,
          "type_id": 426,
          "developer_name": "VAR",
          "subtype_id": 455,
          "subtype_developer_name": "GOAL_DISALLOWED",
          "team_id": 23350,
          "side": "home",
          "player_id": 158516,
          "player_name": "Nguyễn Quang Hải",
          "related_player_id": null,
          "related_player_name": null,
          "period": "2ND_HALF",
          "minute": 90,
          "extra_minute": null,
          "result": null,
          "info": "Goal"
        }
      ]
    }
  ]
}
```
