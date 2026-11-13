---
title: Search Players
excerpt: >-
  Find players in upcoming fixtures by their player metrics — expected shots,
  goals, booking points, card risk, ratings — with the usual time, league and
  team filters.
deprecated: false
hidden: true
metadata:
  robots: index
---
---
title: Search Players
description: Find players in upcoming fixtures by their player metrics — expected shots, goals, booking points, card risk, ratings — with the usual time, league and team filters.
---

# Search Players

`GET /football/players/search`

Finds **players** by the [player metrics](./player-metrics.md) we publish for them: expected
shots, goals, assists, booking points, card risk, ratings and the rest. For example: "attackers
likely to start this weekend, highest expected goals first", or "players with a card risk of at
least 80 in tonight's top-flight matches".

The fixtures are selected exactly as on [`/football/fixtures/search`](./search.md), with the same
time, status, league, country, team, fixture, league level and bookmaker parameters and the same
defaults. Players are then searched inside those fixtures.

Football only for now.

---

## One result = one player in one fixture

Player metrics belong to a fixture, not to a player: the opponent, the venue and the chance of
starting all go into them. So **a player with two matches in the time selection appears twice**,
once per match, each time with that match's values. With a one-day selection that hardly ever
happens. Over a week it is common (a midweek cup tie plus the weekend league game, or a club match
plus an international).

`distinct=player` keeps **one result per player**: his earliest fixture among those that pass
your `where[]` filters. Filtering comes first, so a player whose next match does not pass but
whose second one does is returned with the second one. `total` and `has_more` then count players.

## Time and fixtures

Exactly as on [fixture search](./search.md#time): one of `window` (default `24`, or `live`),
`for_date`, or `from` + `to`; `status` replaces the default statuses.

| Parameter | Example | Notes |
|---|---|---|
| `league_id` | `8,564` | Our league ids. |
| `country_id` | `41` | The league's country. |
| `team_id` | `19` | Players **of** these teams, not their opponents. |
| `fixture_id` | `1177600` | Only these fixtures. |
| `player_id` | `154421,37612345` | Only these players. |
| `position` | `ATTACKER,MIDFIELDER` | `GOALKEEPER`, `DEFENDER`, `MIDFIELDER`, `ATTACKER`: the player's position in that team's squad for the season. |
| `league_level` | `1,2` or `all` | Default `1,2,3`, except when `league_id`, `team_id`, `fixture_id` or `player_id` is given: then every level. |
| `has_odds` | `7` | Only matches with active odds from one of these bookmakers. |

Every list takes at most 100 ids.

Player metrics freeze at kick-off, so matches in play (included by an hour `window`) carry their
pre-match values. Add `status=NOT_STARTED` to leave them out.

## Value filters — `where[NAME]`

```
where[NAME]=min:max
```

- `NAME` is a player metric `developer_name` (the `player_metrics` include lists them all, e.g.
  `PLAYER_EXPECTED_GOALS`, `PLAYER_CARD_RISK`, `PLAYER_SST_RATING`), or one of two values from
  the expected metrics' `meta`:
  - `P_START`: the chance of starting, 0–1.
  - `P_PLAY`: the chance of playing at all, 0–1.
- `min` and `max` are inclusive and may have decimals. Leave one empty for an open range.
- Values are in the units of `include=player_metrics` (decimal counts, 1–99 ratings).
- **No sides**: a player has one value per fixture. `min:max:HOME` is a `400`.
- A player without the value does not match. Not every player has every metric: for example,
  `PLAYER_EXPECTED_SAVES` exists for goalkeepers only.
- Up to 10 filters; all must match.

## Sorting — `order[NAME]`

```
order[NAME]=ASC|DESC
```

Up to 3 keys, in the order given; `NAME` as in `where[]`, or `order[date_time]` for kick-off.
Players without the value come last in both directions. Ties end with kick-off time, fixture id
and player id, so pages never overlap. Default: kick-off ascending.

## Starting players — `P_START` and `basis`

The published expected metrics are **averaged over whether the player starts**: a striker with a
50% chance of starting shows roughly half the expected shots he would have as a starter. Ranked as
published, values mix "dangerous" with "likely to play". Two ways to separate them:

- **Filter on the chance of starting:** `where[P_START]=0.7:` keeps likely starters.
- **Rank on the value if he starts:** `basis=if_starts` makes `where[]`, `order[]` and
  `search_values` use each expected metric's `meta.if_starts` instead of its value.
  `basis=if_benched` uses `meta.if_benched`.

The ratings (`PLAYER_*_INDEX`, `PLAYER_SST_RATING`) and `PLAYER_CARD_RISK` have no conditional
and always use their value.

`p_start` is a model estimate made before team sheets are out. Once the lineup is confirmed, read
it from the fixture's `lineups` / `metadata` (`LINEUP_CONFIRMED`) and see
[Applying a confirmed lineup](./player-metrics.md#applying-a-confirmed-lineup).

## Includes

`include=player_metrics` adds every player metric row of that player in that fixture, in the same
shape as the `player_metrics` include on `/football/fixtures` (`type_id`, `developer_name`,
`value`, `meta`). Any other include, and any `filter[...]`, is a `400`.

## Other parameters

| Parameter | Default | Notes |
|---|---|---|
| `basis` | `value` | `value`, `if_starts` or `if_benched`, see above. |
| `distinct` | (none) | `player`, see above. |
| `language` | `en` | Team, league and country names. |
| `page` | `1` | 1–1000. |
| `per_page` | `25` | 1–50. |

Anything else, including `sport_id`, is a `400`.

## Response

```json
{
  "players": [
    {
      "player": { "id": 203987, "name": "Fahd El Khoumisti", "common_name": "F. El Khoumisti", "position": "ATTACKER" },
      "team": { "id": 854, "name": "Caen" },
      "location": "away",
      "p_start": 0.88,
      "p_play": 0.926,
      "search_values": { "PLAYER_EXPECTED_GOALS": 0.749 },
      "fixture": {
        "id": 1197302,
        "date_time": "2026-11-14T15:00:00.000Z",
        "status": "NOT_STARTED",
        "league": { "id": 111, "name": "Coupe de France", "level": 2, "sub_type": "domestic_cup" },
        "participants": [
          { "id": 4949, "name": "Choisy au Bac", "position": 1, "location": "home" },
          { "id": 854, "name": "Caen", "position": 2, "location": "away" }
        ]
      }
    }
  ],
  "pagination": { "page": 1, "per_page": 25, "total": 3736, "total_pages": 150, "has_more": true, "order": "desc" }
}
```

- Only players with player metrics are found. In a match where one side has none (often an
  amateur club in a cup tie), that side's players never appear.
- `player.position` is `null` when the feed has none for him.
- `p_start` / `p_play` come from the expected metrics and are `null` for a player who has ratings
  only.
- `search_values` holds the value of each name in `where[]` / `order[]`, on the requested `basis`.
  It is absent when neither is given.
- `fixture` is the same object as on `/football/fixtures/search` (no includes).
- `pagination` works as on fixture search. `has_more` is exact. `total` counts results (players
  with `distinct=player`) at the time of the request. `order` is the direction of the first
  `order[]` key.

## Errors

`400` for any invalid or unknown parameter, include, status, position or metric name; the message
says which, and an unknown metric name lists the valid ones. `403` without a premium subscription.

## Examples

```
# This weekend's likely starters, highest expected goals first, one row per player
GET /football/players/search?from=2026-11-14&to=2026-11-16&status=NOT_STARTED&where[P_START]=0.7:&order[PLAYER_EXPECTED_GOALS]=DESC&distinct=player

# Attackers ranked by expected shots if they start, top tiers, next 72 hours
GET /football/players/search?window=72&position=ATTACKER&basis=if_starts&order[PLAYER_EXPECTED_SHOTS]=DESC

# Booking candidates in matches the sportsbook prices
GET /football/players/search?window=24&has_odds=7&where[PLAYER_CARD_RISK]=80:&order[PLAYER_EXPECTED_BOOKING_POINTS]=DESC&include=player_metrics

# One team's players in its next match, best rated first
GET /football/players/search?window=168&team_id=19&order[PLAYER_SST_RATING]=DESC&distinct=player
```