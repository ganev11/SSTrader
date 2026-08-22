---
title: Squads
excerpt: >-
  The registered season roster for both teams in a fixture, with each player's
  position and name in every spelling the API holds.
deprecated: false
hidden: false
metadata:
  robots: index
---
# Squads

`include=squads`

> ### 🧪 Beta
>
> The `squads` include is in beta. It is stable enough to build on, but the response shape and the depth of coverage may still change — in particular, position coverage is expected to improve. Pin nothing to the exact field set, and check for `null` on optional fields rather than assuming they are present.

The `squads` include attaches the **registered squad** for both teams: every player on the books for the fixture's season, with the position they are registered under and their name in all the spellings the API holds.

This is the season roster, not the team sheet for this match. It is available as soon as a fixture is scheduled, which is what makes it useful long before kick-off — see [Squads vs lineups](#squads-vs-lineups).

> **Availability:** Only returned when `include=squads` is added to a request to `GET /fixtures`. Not available on `/fixtures/search` or `/livescores`.

---

## Squads vs lineups

The two look similar and answer different questions.

| | `squads` | `lineups` |
|---|---|---|
| What it is | Everyone registered for the season | The team sheet for this one match |
| Available | As soon as the fixture is scheduled | ~1 hour before kick-off, sometimes later |
| Position means | What the player is registered as | Where they are actually playing today |
| Typical size | ~25–30 players per team | 11 starters + substitutes |

A full-back deployed in midfield is a `DEFENDER` in `squads` and a `MIDFIELDER` in `lineups`. Neither is wrong — they answer different questions.

Request both together (`include=lineups,squads`) when you want the full roster *and* today's team sheet. Doing so also enriches the squad array — see [`source`](#source).

---

## Object Schema

`squads` is a flat array covering **both** teams — entries are not grouped, use `team_id` to split them.

| Field            | Type    | Description |
|------------------|---------|-------------|
| `team_id`        | integer | The team this player is registered with. Match against `participants[].id` to tell home from away. |
| `season_id`      | integer | The season this entry belongs to. |
| `player_id`      | integer | The player's id. Stable across fixtures and seasons. |
| `in_squad`       | integer | `1` if currently part of the squad, `0` if no longer at the club. **See the note below.** |
| `source`         | string  | `squad` or `lineup` — see [`source`](#source). |
| `type_id`        | integer | Type id of the registered position. `null` when unavailable — see [Positions](#positions). |
| `developer_name` | string  | Developer name of the registered position. `null` under the same conditions. |
| `common_name`    | string  | Short form, usually initial + surname. |
| `display_name`   | string  | The name normally shown in match coverage. |
| `fullname`       | string  | Full registered name. |
| `firstname`      | string  | Given name(s). |
| `lastname`       | string  | Family name(s). |
| `birthdate`      | string  | Date of birth as `YYYY-MM-DD`. |

Any name field may be `null` where the provider holds no value for it.

### `in_squad`

> **The array is not filtered on this.** Around a fifth of entries have `in_squad: 0` — players registered against the season who have since left, typically sold or loaned out mid-season. **Filter on `in_squad === 1` if you want a current roster.**

Keeping the `0` rows is deliberate: they are what lets you resolve a `player_id` from earlier in the season, or explain a name that appears in older data.

### `source`

Every entry carries where it came from:

| `source` | Meaning |
|----------|---------|
| `squad` | From the registered season roster. |
| `lineup` | Named in this fixture's team sheet but **not** in the registered squad — usually a very recent signing or an academy call-up who has not been added to the roster yet. |

`lineup` entries only appear when `include=lineups` is requested alongside `squads`. Without it, a brand-new signing may be missing from the array entirely even though he is playing.

For a player present in both, the squad row wins: only its *empty* fields are filled in from the team sheet, so a name that differs between the two is never overwritten.

### Positions

`type_id` and `developer_name` describe the position the player is **registered** under:

| `type_id` | `developer_name` |
|-----------|------------------|
| `372`     | `GOALKEEPER` |
| `373`     | `DEFENDER` |
| `374`     | `MIDFIELDER` |
| `375`     | `ATTACKER` |
| `376`     | `UNKNOWN` |

> **A `null` position means "not available", not "no position".** A small share of players — roughly one in a hundred — come back with `type_id: null` and `developer_name: null`. Two causes: no position on record at all, or a *more granular* position (Centre Back, Defensive Midfield, Left Wing) that has no id in the public type space yet. In the second case the club knows the position and this API cannot yet express it. Widening this is one of the reasons the include is still beta.
>
> Treat `null` as unknown and fall back to the player's position in `lineups` where you have it.

---

## Example

`GET /fixtures?fixture_id=1078883&include=squads`

```json
"squads": [
  {
    "team_id": 4009,
    "season_id": 25580,
    "player_id": 184521,
    "in_squad": 1,
    "source": "squad",
    "type_id": 375,
    "developer_name": "ATTACKER",
    "common_name": "M. Silva Jaimes",
    "display_name": "Miguel Silva",
    "fullname": "Miguel Alejandro Silva Jaimes",
    "firstname": "Miguel",
    "lastname": "Silva Jaimes",
    "birthdate": "1998-03-14"
  },
  {
    "team_id": 4009,
    "season_id": 25580,
    "player_id": 512044,
    "in_squad": 1,
    "source": "lineup",
    "type_id": 374,
    "developer_name": "MIDFIELDER",
    "common_name": "J. Okoro",
    "display_name": "James Okoro",
    "fullname": "James Chidi Okoro",
    "firstname": "James",
    "lastname": "Okoro",
    "birthdate": "2005-01-19"
  }
]
```

The second entry is a player named in today's team sheet who is not yet on the registered roster — he is only present because `include=lineups` was requested too.

---

## Real-World Use Cases

### Team roster pages

Filter to `in_squad === 1`, split on `team_id`, group by `developer_name`, and you have a squad list by position — available for any scheduled fixture, without waiting for team sheets.

### Player metrics

`squads` is a required companion to [`player_metrics`](player-metrics.md), which identifies players by `player_id` alone. Join the two on `player_id` to attach names and positions to the ratings.

### Recognising players across sources

The five name variants exist because sources spell the same player differently. They let you match a name from an external feed — a bookmaker's goalscorer market, a news article — onto a `player_id`, even when that source spells it differently.

### Pre-match previews

Because squads are available days ahead while lineups are not, this is the only way to talk about who *might* play well before kick-off. Combine with `player_metrics` to rank a squad's likely threats before the team sheet lands.

---

## Notes

- Squad data is cached for up to an hour, so a transfer completed very recently may not be reflected immediately.
- A fixture whose season is not yet known returns an empty array.
- A handful of entries are **coaching staff rather than players**. They are not filtered out, and they arrive with a `null` position like any other unmapped role, so they are not distinguishable by `developer_name` alone. They are rare (well under 0.1% of entries), but if you are counting squad size or picking a random player, exclude anyone whose position is `null`.
