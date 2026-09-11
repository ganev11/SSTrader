---
title: Lineups
excerpt: >-
  Starting XI, bench and sidelined players for both teams in a fixture, with
  shirt numbers, formation slots and player details.
deprecated: false
hidden: false
metadata:
  robots: index
---

`include=lineups`

The `lineups` include attaches the team sheets for a fixture: every player named by either team, with the slot they occupy (starting XI, bench or sidelined), their shirt number, their position in this match, and their name in all the spellings the API holds.

Lineups are published by the data provider shortly before kick-off — typically around an hour before, sometimes later for smaller competitions. Requesting the include for a fixture whose team sheets have not been announced yet returns an empty array.

> **Availability:** Only returned when `include=lineups` is added to a request to `GET /fixtures`. Not available on `/fixtures/search` or `/livescores`.

> **Knowing whether a team sheet is confirmed — or is ever coming.** An empty `lineups` array on its own cannot tell you whether the sheets are merely late or will never be published, and a populated one cannot tell you whether it is the confirmed XI or a provisional sheet. The `LINEUP_CONFIRMED` entry in [`include=metadata`](/docs/metadata) (`type_id` `402`) answers both. See [Reading it with `LINEUP_CONFIRMED`](#reading-it-with-lineup_confirmed) below.

---

## Object Schema

`lineups` is a flat array covering **both** teams — entries are not grouped, use `team_id` to split them. Each entry describes one slot in one team's sheet:

| Field                | Type    | Description |
|----------------------|---------|-------------|
| `team_id`            | integer | The team this entry belongs to. Match it against `participants[].id` to tell home from away. |
| `number`             | integer | Shirt number. `null` when the provider did not supply one. |
| `formation_position` | integer | The player's slot index within the team's formation, for starting players. `null` for bench entries and where the provider did not supply one. |
| `type_id`            | integer | Type id of the slot itself — see the table below. |
| `developer_name`     | string  | Developer name of the slot: `LINEUP`, `BENCH` or `SIDELINED`. |
| `player`             | object  | The player filling the slot — see [Player object](#player-object). |

### Slot types

| `type_id` | `developer_name` | Meaning |
|-----------|------------------|---------|
| `377`     | `LINEUP`         | Named in the starting XI. |
| `378`     | `BENCH`          | Named as a substitute. |
| `379`     | `SIDELINED`      | Unavailable for this match (injury, suspension). Rarely present — most feeds report team sheets as starters and substitutes only. |

Entries are ordered by team, then by slot type (starters before bench), then by formation position and shirt number.

### Player object

| Field            | Type    | Description |
|------------------|---------|-------------|
| `player_id`      | integer | The player's id. Stable across fixtures and seasons. |
| `type_id`        | integer | Type id of the player's position **in this match**. |
| `developer_name` | string  | Developer name of that position: `GOALKEEPER`, `DEFENDER`, `MIDFIELDER`, `ATTACKER` or `UNKNOWN`. |
| `common_name`    | string  | Short form, usually initial + surname — e.g. `M. Silva Jaimes`. |
| `display_name`   | string  | The name normally shown in match coverage — e.g. `Miguel Silva`. |
| `fullname`       | string  | Full registered name. |
| `firstname`      | string  | Given name(s). |
| `lastname`       | string  | Family name(s). |
| `birthdate`      | string  | Date of birth as `YYYY-MM-DD`. |

Any field on the player object may be `null` where the provider holds no value for it. The four name variants exist because sources spell the same player differently; having all of them is what makes it possible to recognise a player across feeds.

> **Note:** `player.type_id` / `player.developer_name` are the player's position **in this fixture**, which can differ from the position he is registered under for the season — a full-back deployed in midfield is reported as a midfielder here.

The player object uses the same field names as an entry in the `squads` include, so the two can be compared field for field.

---

## Request Examples

**Team sheets for a single fixture:**

```
GET /fixtures?fixture_id=1105585&include=lineups
```

**Team sheets alongside odds and statistics:**

```
GET /fixtures?fixture_id=1105585&include=lineups,odds,statistics
```

**Every fixture on a date, with team sheets where they have been announced:**

```
GET /fixtures?start_date=2026-08-05T00:00:00Z&end_date=2026-08-05T23:59:59Z&include=lineups
```

---

## Example Response

```json
"lineups": [
  {
    "team_id": 4009,
    "number": 1,
    "formation_position": 1,
    "type_id": 377,
    "developer_name": "LINEUP",
    "player": {
      "player_id": 293067,
      "type_id": 372,
      "developer_name": "GOALKEEPER",
      "common_name": "M. Silva Jaimes",
      "display_name": "Miguel Silva",
      "fullname": "Miguel Alejandro Silva Jaimes",
      "firstname": "Miguel Alejandro",
      "lastname": "Silva Jaimes",
      "birthdate": "1998-03-14"
    }
  },
  {
    "team_id": 4009,
    "number": 21,
    "formation_position": 2,
    "type_id": 377,
    "developer_name": "LINEUP",
    "player": {
      "player_id": 95690,
      "type_id": 373,
      "developer_name": "DEFENDER",
      "common_name": "A. González Sibulo",
      "display_name": "Alexander David González Sibulo",
      "fullname": "Alexander David González Sibulo",
      "firstname": "Alexander David",
      "lastname": "González Sibulo",
      "birthdate": "1996-11-02"
    }
  },
  {
    "team_id": 631,
    "number": 45,
    "formation_position": null,
    "type_id": 378,
    "developer_name": "BENCH",
    "player": {
      "player_id": 441628,
      "type_id": 374,
      "developer_name": "MIDFIELDER",
      "common_name": "E. Martinho Silva",
      "display_name": "Edson Martinho da Silva",
      "fullname": "Edson Martinho Silva",
      "firstname": "Edson",
      "lastname": "Martinho Silva",
      "birthdate": "2004-06-27"
    }
  }
]
```

> An empty array (`"lineups": []`) means no team sheets have been published for that fixture yet — expected for matches that are still more than an hour or so away.

---

## Reading it with `LINEUP_CONFIRMED`

`include=lineups` tells you what has been published. The `LINEUP_CONFIRMED` entry in [`include=metadata`](/docs/metadata) tells you what that publication *means*. Request both together:

```
GET /fixtures?fixture_id=1105585&include=lineups,metadata
```

`LINEUP_CONFIRMED` is fixture-wide, so its entry carries `team_id: null`:

```json
{
  "team_id": null,
  "type_id": 402,
  "developer_name": "LINEUP_CONFIRMED",
  "value": 1,
  "meta": { "confirmed": true }
}
```

The two includes combine into five states. The last two are the ones worth handling explicitly:

| `LINEUP_CONFIRMED` | `lineups` | Meaning |
|---|---|---|
| `confirmed: true` | populated | The confirmed starting XI. Safe to present as the team sheet. |
| `confirmed: true` | `[]` | Confirmed upstream but the sheet has not reached us. Uncommon — a feed gap, not a normal pre-match state. |
| `confirmed: false` | `[]` | Not confirmed yet. Team sheets are expected closer to kick-off. |
| `confirmed: false` | populated | A **provisional** sheet — published but not confirmed, and still subject to change. Do not present it as the starting XI. |
| *entry absent* | `[]` | No lineup coverage for this fixture. Team sheets are very unlikely to arrive at any point, including after kick-off. |

That last row is the important one: a missing `LINEUP_CONFIRMED` entry is **not** the same as `confirmed: false`. Fixtures with no entry almost never receive team sheets at all, so treating them as "still waiting" means polling a fixture that will never fill in.

---

## Real-World Use Cases

### Team sheet display

Split the array on `team_id`, then on `developer_name`, to render the classic two-column starting XI plus substitutes, using `formation_position` for the pitch layout and `number` for shirt numbers.

### Knowing when to stop waiting

Rather than re-requesting `lineups` for every upcoming fixture, request `metadata` alongside it and let `LINEUP_CONFIRMED` drive the polling: fixtures with no entry can be dropped from the schedule entirely, `confirmed: false` fixtures are worth re-checking as kick-off approaches, and `confirmed: true` fixtures are done.

### Availability checks before betting

Confirm a key player is in the starting XI (`developer_name` = `LINEUP`) rather than on the bench or sidelined before showing markets that depend on him — a striker starting on the bench changes the value of a goalscorer bet considerably.

Gate this on `LINEUP_CONFIRMED` being `true`. Acting on a provisional sheet means acting on a team that may still change before kick-off.

### Line-up-driven market context

Combine `lineups` with `odds` to explain price movement: a strong XI on one side, or a first-choice goalkeeper missing, often lines up with the shift a user is looking at.

### Recognising players across sources

The four name variants on `player` let you match a player named by an external feed — a bookmaker's goalscorer market, a news article — to a `player_id`, even when that source spells the name differently.
