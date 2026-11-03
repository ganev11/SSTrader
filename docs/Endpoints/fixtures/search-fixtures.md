---
title: Search
excerpt: >-
  Find fixtures by time, competition, team, bookmaker coverage and by metric or
  statistic values
deprecated: false
hidden: false
metadata:
  robots: index
---
# Search Fixtures

`GET /football/fixtures/search` · basketball: `GET /basketball/fixtures/search` ([differences](./basketball/fixtures-search.md))

Finds fixtures by **values we store for them** — expected goals, power metrics, classifications,
expected values, live statistics — together with the usual time, league and team filters. For
example: "upcoming matches where both teams' expected goals are at least 1.8, highest total
first", or "today's live matches with at least 3 corners in the last 10 minutes".

The fixture objects are the same as on `/football/fixtures`. Each one also carries
[`search_values`](#search_values): the values that the request filtered or sorted on.

> **Old path.** `GET /fixtures/search` (without `/football`) is the previous version of this
> endpoint. It keeps working unchanged for now but is deprecated: move to
> `/football/fixtures/search`. Differences: hour windows here include matches in play, values are
> in display units (not raw integers), every unknown parameter is a `400`, sorting puts missing
> values last, and `has_more` is exact.

---

## Time

Use **one** of `window`, `for_date` or `from` + `to`. Without any of them, `window=24`.

| Parameter | Example | Selects |
|---|---|---|
| `window` | `48` | Matches kicking off in the next N hours (1–168), **plus the matches in play now**. Default statuses: in play + `NOT_STARTED`. |
| `window` | `live` | Matches in play now. Default statuses: the in-play ones. |
| `for_date` | `2026-11-07` | Kick-off on that UTC day, past or future. Default statuses: all except `DELETED`. |
| `from`, `to` | `from=2026-11-05&to=2026-11-12` | Kick-off in that range (dates or ISO 8601 date-times, UTC unless a zone is given; a date in `to` means the end of that day). At most 14 days long, starting no more than 30 days ago. Default statuses as `for_date`. |

A `NOT_STARTED` match whose kick-off passed less than 15 minutes ago is still in an hour window
(its feed has not gone live yet).

## Narrowing

| Parameter | Example | Notes |
|---|---|---|
| `status` | `NOT_STARTED,HT` | Replaces the default statuses. Unknown statuses are a `400`. [Status list](https://docs.sstrader.com/docs/states). |
| `league_id` | `8,564` | Our league ids. |
| `country_id` | `41` | Our country ids (the league's country). |
| `team_id` | `19` | Matches where the team plays home **or** away. |
| `fixture_id` | `1177600,1129264` | Search only these fixtures (still inside the time selection). |
| `league_level` | `1,2` or `all` | Default `1,2,3`, except when `league_id`, `team_id` or `fixture_id` is given: then every level. |
| `has_odds` | `7` | Only matches with active (not suspended) odds from one of these bookmakers. |
| `filter[integration]` | see [integration](./integration.md) | Your own platform's ids, including `mapped_only`. |

Every list takes at most 100 ids.

## Value filters — `where[NAME]`

```
where[NAME]=min:max[:SIDE]
```

- `NAME` is a `developer_name` from [`/football/types`](./types-league-average.md) (only types that
  have a `type_id`, i.e. the ones the `metrics` and `statistics` includes return).
- `min` and `max` are inclusive and may have decimals. Leave one empty for an open range:
  `2.5:` is "at least 2.5", `:1` is "at most 1".
- Values are in the **units the response shows**: metrics as returned by `include=metrics`,
  statistics as returned by `include=statistics`.
- Up to 10 filters; all must match.

| `SIDE` | Tests |
|---|---|
| `MATCH` (default) | The match-level row (`team_id` 0), e.g. `FT_TOTAL_GOALS_V2`, expected values, league averages. Old name `LEAGUE` is not accepted. |
| `HOME` / `AWAY` | That team's value. |
| `TOTAL` | home + away |
| `DIFF` | home − away |
| `GAP` | \|home − away\| |
| `MAX` / `MIN` | The larger / smaller of the two teams' values. |
| `EITHER` | At least one team's value is in the range. |
| `BOTH` | Both teams' values are in the range. |

**Statistics are stored per team only**, so `MATCH` is a `400` for a statistic — use `TOTAL`.

**A statistic the feed did not send counts as 0** once the match has feed statistics at all (the
feed leaves out values still at 0, and `LAST_N_*` values are written only when not 0). So
`where[GOALS]=0:0:BOTH` finds goalless matches. A match with no feed statistics has no value: it
never matches a statistic filter. Metrics are never filled in: a match without that metric does
not match.

```
# Both teams expected to score at least 1.8 (V2 expected goals)
where[FT_EXPECTED_GOALS_V2]=1.8::BOTH

# Live: at least 3 corners in the last 10 minutes, either team
window=live&where[LAST_10_CORNERS]=3::TOTAL

# Big favourites: home goals classification 3 against an away team at 0
where[TEAM_GOALS_CLASSIFICATION]=3:3:HOME&where[TEAM_GOALS_CLASSIFICATION]=0:0:AWAY
```

## Sorting — `order[NAME]`

```
order[NAME]=ASC|DESC[:SIDE]
```

Up to 3 keys, applied in the order given. `SIDE` as above except `EITHER` / `BOTH` (default
`MATCH`). `order[date_time]=ASC|DESC` sorts by kick-off. Matches without a value always come
last, in both directions. Ties end with kick-off time and fixture id, so pages never overlap.
Default: kick-off time ascending.

```
order[FT_EXPECTED_GOALS_V2]=DESC:TOTAL
order[EV_EUROPE_ML_HOME]=DESC
```

## `search_values`

Every fixture carries the values of each type named in `where[]` / `order[]`, so you can show
why it matched:

```json
"search_values": {
  "FT_EXPECTED_GOALS_V2": { "home": 2.53, "away": 2.521, "match": null }
}
```

`match` is the `team_id` 0 value (always `null` for statistics). Absent when no `where` / `order`
is given.

## Includes

`include=odds,metrics,fair_odds,statistics,metadata,integration` — same as on
`/football/fixtures`, with the same `filter[metrics]` (`types`, `developer_types`),
`filter[statistics]` (`types`, `developer_types`) and `filter[odds]` (`bookmakers`, `markets`,
`is_live`, `suspend`). A `filter[...]` needs its include. Any other include is a `400`
(`lineups`, `squads` and the player includes are on `/football/fixtures` only).

`fair_odds` is priced from the `FT_EXPECTED_GOALS` metrics and returned as `fairOdds` /
`fairOddsV2`, as on `/football/fixtures`.

## Other parameters

| Parameter | Default | Notes |
|---|---|---|
| `language` | `en` | Two-letter code for team, league and country names. |
| `page` | `1` | 1–1000. |
| `per_page` | `25` | 1–50. |

Anything else — including `sport_id` and `lang` — is a `400`.

## Response

```json
{
  "fixtures": [
    {
      "id": 1175258,
      "date_time": "2026-11-03T12:00:00.000Z",
      "status": "NOT_STARTED",
      "league": { "id": 1234, "name": "U18 Premier League", "level": 3 },
      "participants": [
        { "id": 71, "name": "Leeds United U18", "location": "home" },
        { "id": 72, "name": "Blackburn U18", "location": "away" }
      ],
      "search_values": {
        "FT_EXPECTED_GOALS_V2": { "home": 2.53, "away": 2.521, "match": null }
      }
    }
  ],
  "pagination": { "page": 1, "per_page": 25, "has_more": false }
}
```

`has_more` is exact: `true` only when the next page has at least one fixture.

## Errors

`400` for any invalid or unknown parameter, include, filter key, status, side or type name — the
message says which. `403` without a premium subscription.

## Examples

```
# Saturday, top-tier matches the sportsbook prices, highest expected total goals first
GET /football/fixtures/search?for_date=2026-11-07&league_level=1&has_odds=7&order[FT_EXPECTED_GOALS_V2]=DESC:TOTAL

# Arsenal's next matches
GET /football/fixtures/search?window=168&team_id=19

# Yesterday's matches with at least 14 corners
GET /football/fixtures/search?for_date=2026-11-01&status=FT,FT_AET,FT_PEN&where[CORNERS]=14::TOTAL&order[CORNERS]=DESC:TOTAL

# Best home expected value in the next 48 hours, with odds and metrics
GET /football/fixtures/search?window=48&order[EV_EUROPE_ML_HOME]=DESC&include=odds,metrics&filter[odds]=bookmakers:2;markets:1
```
