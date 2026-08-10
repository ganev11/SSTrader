---
title: Player Metrics
excerpt: >-
  Position-relative ratings for every squad player in a fixture — Impact,
  Aggression, Discipline and an overall SST Rating, each with a plain-language
  label.
deprecated: false
hidden: false
metadata:
  robots: index
---

# Player Metrics

`include=player_metrics`

The `player_metrics` include rates every player in both squads on four attributes: how well they do the main job of their position, how physical they are, how disciplined they are, and an overall score combining the three.

Ratings describe a player's recent form in the competition, not a single match, so they are available from the moment a fixture is scheduled — you do not need to wait for team sheets.

> **Requires `squads`.** Request them together: `include=squads,player_metrics`. Metric rows identify players by `player_id` only; `squads` carries their names and positions.

> **Availability:** Only returned when `include=player_metrics` is added to a request to `GET /fixtures`. Not available on `/fixtures/search` or `/livescores`.

---

## Reading the values

Every value is an integer from **1 to 99**. Two rules cover all four metrics:

1. **50 is average**, and average means *for this player's position, in this league*. A defender is measured against other defenders, never against strikers — without that, every striker would look like a terrible defender.
2. **Higher is always better.** This holds for all four metrics, Discipline included.

| Value | Meaning |
|-------|---------|
| `50` | Exactly average for this player's position |
| `73` | Well above average |
| `39` | Below average |

As a rough guide, about 10 points is a meaningful gap, and the middle 80% of players in a league fall between roughly 25 and 80. Values are bounded, so a single freak match cannot push a player to an extreme.

### What each metric means

| Metric | A high value means |
|--------|--------------------|
| Impact | More effective at their position's core job |
| Aggression | Competes more physically — more fouls and tackles |
| Discipline | **Stays out of the book** — rarely booked for the fouls they commit |
| SST Rating | Stronger player overall |

> **Aggression and Discipline are separate on purpose.** A player can score high on *both*: he commits plenty of fouls but rarely gets carded for them. Reading a high Aggression as "dirty player" will mislead you — that is what Discipline is for, and on this scale a *low* Discipline is the one to watch.

---

## Object Schema

`player_metrics` is a flat array covering **both** squads — entries are not grouped, use `team_id` to split them, and join to `squads[]` on `player_id` for names and positions.

Each player has **four** rows:

| Field            | Type    | Description |
|------------------|---------|-------------|
| `team_id`        | integer | The team the player is registered with. Match against `participants[].id` to tell home from away. |
| `player_id`      | integer | The player. Join to `squads[].player_id`. |
| `type_id`        | integer | Type id of the metric — see the table below. |
| `developer_name` | string  | Developer name of the metric. |
| `value`          | integer | The rating, 1–99. See [Reading the values](#reading-the-values). |

There is no `meta` object and no label — the value is the whole payload.

### Metric types

| `type_id` | `developer_name` | A high value means |
|-----------|------------------|--------------------|
| `380` | `PLAYER_IMPACT_INDEX` | Effective at the main job of their position — what that job *is* depends on the position, see below. |
| `381` | `PLAYER_AGGRESSION_INDEX` | Competes physically: lots of fouls and tackles. |
| `382` | `PLAYER_DISCIPLINE_INDEX` | Rarely booked for the fouls they commit. |
| `383` | `PLAYER_SST_RATING` | Strong overall, combining the three above, weighted for their position. |

### The Impact metric

All four positions share one metric type, but it measures a different job for each. Derive which one applies from the player's position in `squads[]`:

| `squads[].developer_name` | Impact measures | A high value means |
|---------------------------|-----------------|--------------------|
| `ATTACKER` | Threat | Shoots often, hits the target, scores |
| `MIDFIELDER` | Control | Creates chances, plays into the final third, wins the ball back |
| `DEFENDER` | Wall | Clears, blocks, intercepts, wins duels |
| `GOALKEEPER` | Shield | Saves a high share of the shots faced |

So an Impact of `73` on a defender means a strong defender, and `73` on a striker means a dangerous striker — both are "well above average at their own job". Because everything is position-relative, the two numbers are directly comparable even though they measure different things.

---

## Example

`GET /fixtures?fixture_id=1078883&include=squads,player_metrics`

```json
"player_metrics": [
  {
    "team_id": 4009,
    "player_id": 184521,
    "type_id": 380,
    "developer_name": "PLAYER_IMPACT_INDEX",
    "value": 73
  },
  {
    "team_id": 4009,
    "player_id": 184521,
    "type_id": 381,
    "developer_name": "PLAYER_AGGRESSION_INDEX",
    "value": 41
  },
  {
    "team_id": 4009,
    "player_id": 184521,
    "type_id": 382,
    "developer_name": "PLAYER_DISCIPLINE_INDEX",
    "value": 43
  },
  {
    "team_id": 4009,
    "player_id": 184521,
    "type_id": 383,
    "developer_name": "PLAYER_SST_RATING",
    "value": 63
  }
]
```

Read together, and knowing from `squads[]` that this player is an `ATTACKER`: a well above-average goal threat (73), not especially physical (41), slightly more prone to picking up a card than most when he does foul (43), rating 63 overall.

> **Not every squad player appears.** Players without enough recent playing time to rate are omitted rather than given a placeholder — expect fewer players in `player_metrics` than in `squads`. Newly signed and youth players are the usual cases. Always look players up by `player_id` rather than assuming the arrays line up.

---

## Filtering

`filter[player_metrics]` narrows which metrics come back:

```
filter[player_metrics]=types:380,383
```

returns only Impact and SST Rating for each player, which is usually all a listing view needs.

---

## Real-World Use Cases

### Player cards

Join `player_metrics` to `squads` on `player_id` and render a card per player: the SST Rating as the headline number and the other three as 1–99 bars. Since every value shares one scale and one direction, a single bar component works for all four.

### Ranking a squad

Sort a team's players by `PLAYER_SST_RATING` to surface its most dangerous names, or by `PLAYER_IMPACT_INDEX` within a position to find, say, the best defender on the pitch.

### Card markets

`PLAYER_AGGRESSION_INDEX` and `PLAYER_DISCIPLINE_INDEX` together are the pair that matters for booking markets, and it is the **combination** that carries the signal:

| Aggression | Discipline | Reading |
|-----------|-----------|---------|
| High | **Low** | The genuine card risk — fouls a lot *and* gets booked for it |
| High | High | Physical but gets away with it; far less exposed than the raw foul count suggests |
| Low | Low | Fouls rarely, but is booked when he does |
| Low | High | Minimal exposure |

Note the direction: it is a **low** Discipline that flags risk, not a high one.

### Goalscorer markets

Impact on an attacker is a direct read on shot volume and finishing relative to other attackers in the league, which gives context to an anytime-scorer price.

### Matchup context

Compare one side's attackers' Impact against the other side's defenders' Impact to characterise a fixture before looking at odds — both are on the same 50-is-average scale, so they are directly comparable even though one measures shooting and the other blocking.
