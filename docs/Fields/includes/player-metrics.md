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

Everything below rests on one idea: **a player is compared only against others in the same position, in the same league.** A defender's numbers are measured against other defenders, never against strikers. Without that, every striker would look like a terrible defender.

That makes the scale simple:

| Value | Meaning |
|-------|---------|
| `1.00` | Exactly average for this player's position |
| `1.35` | 35% above the positional average |
| `0.80` | 20% below the positional average |

Values are bounded to **0.25 – 2.50**, so one freak match cannot send a rating to an extreme.

`PLAYER_SST_RATING` is the exception: it is an absolute **0 – 99** score where **50 is average**, in the style of a game rating.

### Which direction is good

| Metric | Higher means |
|--------|--------------|
| Impact | **Better** — more effective at their position's core job |
| SST Rating | **Better** — stronger player overall |
| Discipline | **Worse** — their fouls turn into bookings more often |
| Aggression | Neither — more physical, which is an asset or a liability depending on the market |

Aggression and Discipline are deliberately separate. A player can be highly aggressive and still well disciplined: he commits plenty of fouls but rarely gets booked for them. Reading Aggression alone as "dirty player" will mislead you.

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
| `value`          | number  | The rating. See [Reading the values](#reading-the-values). |
| `meta`           | object  | `classification` (the label) and `classification_type` (the attribute's display name). |

### Metric types

| `type_id` | `developer_name` | What it measures |
|-----------|------------------|------------------|
| `380` | `PLAYER_IMPACT_INDEX` | How well the player does the main job of their position. What that job *is* depends on the position — see below. |
| `381` | `PLAYER_AGGRESSION_INDEX` | How many bookable situations the player gets into: fouls and tackles. |
| `382` | `PLAYER_DISCIPLINE_INDEX` | How often those situations actually become a yellow card. Higher is worse. |
| `383` | `PLAYER_SST_RATING` | Overall 0–99 rating combining the three above, weighted for the player's position. |

### The Impact metric

All four positions share one metric type, but it means something different for each. `meta.classification_type` tells you which:

| Position | `classification_type` | What a high value means |
|----------|----------------------|-------------------------|
| Attacker | `Threat` | Shoots often, hits the target, scores |
| Midfielder | `Control` | Creates chances, plays into the final third, wins the ball back |
| Defender | `Wall` | Clears, blocks, intercepts, wins duels |
| Goalkeeper | `Shield` | Saves a high share of the shots faced |

So a `1.40` Impact on a defender means a strong defender, and a `1.40` on a striker means a dangerous striker — both are "40% above average at their own job".

### Labels

Every row carries a plain-language label in `meta.classification`, so you can use the wording directly without applying your own thresholds.

| Impact | Aggression | Discipline | SST Rating |
|--------|-----------|------------|------------|
| `Elite` | `Intense` | `Reckless` | `Elite` |
| `Strong` | `Aggressive` | `Poor` | `Excellent` |
| `Solid` | `Physical` | `Fair` | `Strong` |
| `Average` | `Moderate` | `Good` | `Solid` |
| `Low` | `Controlled` | `Very Good` | `Average` |
| `Quiet` | `Calm` | `Excellent` | `Developing` |

Best at the top for Impact and SST Rating; worst at the top for Discipline. Aggression runs most physical at the top.

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
    "value": 1.42,
    "meta": { "classification": "Strong", "classification_type": "Threat" }
  },
  {
    "team_id": 4009,
    "player_id": 184521,
    "type_id": 381,
    "developer_name": "PLAYER_AGGRESSION_INDEX",
    "value": 0.83,
    "meta": { "classification": "Controlled", "classification_type": "Aggression" }
  },
  {
    "team_id": 4009,
    "player_id": 184521,
    "type_id": 382,
    "developer_name": "PLAYER_DISCIPLINE_INDEX",
    "value": 1.12,
    "meta": { "classification": "Fair", "classification_type": "Discipline" }
  },
  {
    "team_id": 4009,
    "player_id": 184521,
    "type_id": 383,
    "developer_name": "PLAYER_SST_RATING",
    "value": 63,
    "meta": { "classification": "Solid", "classification_type": "SST Rating" }
  }
]
```

Read together: a forward who is a well above-average goal threat, not especially physical, picks up cards slightly more often than most when he does foul, and rates 63 overall.

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

Join `player_metrics` to `squads` on `player_id` and render a card per player: the SST Rating as the headline number, the three indices as bars, and `meta.classification` as the caption under each. No threshold logic needed on your side.

### Ranking a squad

Sort a team's players by `PLAYER_SST_RATING` to surface its most dangerous names, or by `PLAYER_IMPACT_INDEX` within a position to find, say, the best defender on the pitch.

### Card markets

`PLAYER_AGGRESSION_INDEX` and `PLAYER_DISCIPLINE_INDEX` together are the pair that matters for booking markets. A player who is high on both — plenty of fouls *and* a high conversion into cards — is a very different proposition from one who is high on Aggression alone.

### Goalscorer markets

`Threat` on an attacker is a direct read on shot volume and finishing relative to other attackers in the league, which gives context to an anytime-scorer price.

### Matchup context

Compare one side's attackers' `Threat` against the other side's defenders' `Wall` to characterise a fixture before looking at odds — both are on the same 1.00-is-average scale, so they are directly comparable.
