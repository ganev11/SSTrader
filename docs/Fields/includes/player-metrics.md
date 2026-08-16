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

The `player_metrics` include returns two different kinds of number for every player in both squads:

| Family | `type_id` | What it is | Scale |
|---|---|---|---|
| **Ratings** | 380–383 | How good the player had been going into this fixture | Integer 1–99, 50 = average |
| **Expected metrics** | 384–385 | What the player is predicted to do **in this fixture** | Decimal count (shots, minutes) |

They live in the same array and share the same row shape, so the one thing to get right is telling them apart — use `developer_name`, or the `type_id` ranges above. Everything else follows from which family a row belongs to.

Both are available from the moment a fixture is scheduled, so you do not need to wait for team sheets. Expected metrics additionally tell you how to *revise* them once a team sheet arrives — see [Applying a confirmed lineup](#applying-a-confirmed-lineup).

## Every value belongs to one fixture, and freezes at kick-off

This applies to **both** families and is the single most useful thing to know about them:

- **Values are attached to a fixture, not to a player.** Ask for the same player in two different matches and you get two different sets of numbers.
- **They stop moving at kick-off.** What you get back is the last state before the match started, and it is never revised afterwards.

For the ratings that means every fixture carries the player's form *as it stood going into that match* — so a fixture from March still reports what the player looked like in March, not what he looks like today. His most recent fixture is his current card; the earlier ones are the historical record.

For the expected metrics it means the same thing and matters more, since they were only ever pre-match statements. A prediction quietly updated after the result is known would be worthless for judging how good the predictions are.

> **A fixture that was never seen before kick-off carries no player metrics at all.** There was no pre-match state to record. This is rare, but check for an empty array rather than assuming every fixture has one.

> **Requires `squads`.** Request them together: `include=squads,player_metrics`. Metric rows identify players by `player_id` only; `squads` carries their names and positions.

> **Availability:** Only returned when `include=player_metrics` is added to a request to `GET /fixtures`. Not available on `/fixtures/search` or `/livescores`.

---

# Part 1 — Ratings

`PLAYER_IMPACT_INDEX` · `PLAYER_AGGRESSION_INDEX` · `PLAYER_DISCIPLINE_INDEX` · `PLAYER_SST_RATING`

## Reading the values

Every rating is an integer from **1 to 99**. Two rules cover all four:

1. **50 is average**, and average means *for this player's position, in this league*. A defender is measured against other defenders, never against strikers — without that, every striker would look like a terrible defender.
2. **Higher is always better.** This holds for all four ratings, Discipline included.

| Value | Meaning |
|-------|---------|
| `50` | Exactly average for this player's position |
| `73` | Well above average |
| `39` | Below average |

As a rough guide, about 10 points is a meaningful gap, and the middle 80% of players in a league fall between roughly 25 and 80. Values are bounded, so a single freak match cannot push a player to an extreme.

### What each rating means

| Metric | A high value means |
|--------|--------------------|
| Impact | More effective at their position's core job |
| Aggression | Competes more physically — more fouls and tackles |
| Discipline | **Stays out of the book** — rarely booked for the fouls they commit |
| SST Rating | Stronger player overall |

> **Aggression and Discipline are separate on purpose.** A player can score high on *both*: he commits plenty of fouls but rarely gets carded for them. Reading a high Aggression as "dirty player" will mislead you — that is what Discipline is for, and on this scale a *low* Discipline is the one to watch.

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

# Part 2 — Expected metrics

`PLAYER_EXPECTED_SHOTS` · `PLAYER_EXPECTED_MINUTES`

These are **predictions for the specific fixture**, not ratings. The value is a plain decimal count in the metric's own unit:

| Metric | `type_id` | Unit | Typical range |
|--------|-----------|------|---------------|
| `PLAYER_EXPECTED_SHOTS` | `384` | Shots attempted | Median `0.42`, 95th percentile `1.63`, rarely above `4` |
| `PLAYER_EXPECTED_MINUTES` | `385` | Minutes on the pitch | `0`–`90+`, median `47.8` |

Expected Shots counts **every attempt** — on target or not, blocked shots included.

> **Not on the 1–99 scale.** An Expected Shots of `1.29` is not "very poor". It is 1.29 shots. Applying the ratings scale to these values, or the reverse, is the one mistake worth guarding against, and it is why the two families are marked by `developer_name` rather than left to context.

### How much they move between fixtures

Expected metrics vary far more from match to match than the ratings do, because opponent, venue and the player's chance of starting all feed the model directly. Measured across one player's six fixtures, expected shots ranged from `1.068` to `1.781` — a 67% spread — while his ratings drifted by a couple of points.

So do not carry an expected value from one fixture to another. Ask for the fixture you actually care about.

## The team-sheet problem

We publish expected metrics as soon as a fixture is scheduled — days ahead, when player markets first open. At that point nobody knows whether a given player will start, be on the bench, or be left out entirely, and that single unknown dominates everything else. A striker who takes four shots per start is worth almost nothing if he does not play.

So the published `value` **assumes the team sheet is unknown** and blends the possibilities:

```
value = p_start × if_starts + (1 − p_start) × if_benched
```

All three inputs ship in `meta` on every expected-metric row, so the number is never a black box:

| `meta` field | Meaning |
|---|---|
| `if_starts` | The expected value **if the player is named in the starting XI** |
| `if_benched` | The expected value **if the player starts on the bench** |
| `p_start` | The probability that he starts, which produced the blend |

> **`if_benched` already includes the chance he never comes on.** It is not simply "if_starts, but shorter" — most benched players do not appear at all, and that is priced in. Use it as published; do not scale it down further.

`PLAYER_EXPECTED_MINUTES` follows exactly the same structure, and it is worth reading first whenever another expected metric looks surprisingly low. A low expected count usually means low expected minutes rather than a poor player.

## Applying a confirmed lineup

Team sheets are typically published 15–60 minutes before kick-off. When one arrives, you do **not** need to wait for us to recompute and republish — that is the entire reason all three numbers are exposed. Substitute the case you now know is true:

| The team sheet says | Use |
|---|---|
| In the starting XI | `meta.if_starts` |
| On the bench | `meta.if_benched` |
| Not in the squad | `0` for every expected metric |

Because `p_start` is published alongside, you can also see how much the confirmation moved things. A player at `p_start: 0.95` barely changes when confirmed; one at `0.45` roughly doubles.

`p_start` is useful on its own as a rotation read — a nominal first-choice player sitting at `0.5` is a rotation risk the ratings will not show you.

## Planned expected metrics

Expected Shots is the first of a wider set. The rest are **not implemented yet — no `type_id` is assigned, and nothing for them is returned today.** They are listed so you can see where the family is going, not so you can code against them:

| Planned | Unit | Notes |
|---------|------|-------|
| Expected Goals (xG) | Goals | |
| Expected Assists (xA) | Assists | |
| Expected Goal Involvements (xGI) | Goals + assists | Derived from xG and xA rather than modelled separately |
| Expected Shots on Target (xSoT) | Shots on target | Will be modelled as a share of Expected Shots, so `xSoT ≤ xSh` always holds |
| Expected Booking Points (xBP) | Points | Yellow 10, straight red 25, second yellow 35 |
| Expected Fouls (xF) | Fouls committed | |
| Expected Tackles (xT) | Tackles | |
| Expected Passes (xP) | Passes completed | |
| Expected Saves (xS) | Saves | Goalkeepers only |

Two things are worth knowing in advance:

- **They will share the shape documented above.** Same row structure, same decimal `value`, same `meta` with `if_starts` / `if_benched` / `p_start`. Code written against Expected Shots will handle them unchanged.
- **Coverage will differ per metric.** Each depends on its own underlying stat being recorded, and leagues carry different subsets — a competition that reports shots may not report tackles. Expect the player count to vary between metrics in the same fixture, which is another reason to look rows up by `developer_name` rather than by position in the array.

> **Expected Booking Points and Expected Fouls are the least certain of the set.** Both depend heavily on the referee, and referee appointments arrive late: around half of fixtures have one assigned within 24 hours of kick-off, under a third at two days, and almost none beyond that. Expect these two to behave like the team sheet does — a usable number early, sharpened once the appointment is known.

---

## Object schema

`player_metrics` is a flat array covering **both** squads — entries are not grouped, use `team_id` to split them, and join to `squads[]` on `player_id` for names and positions.

| Field            | Type    | Description |
|------------------|---------|-------------|
| `team_id`        | integer | The team the player is registered with. Match against `participants[].id` to tell home from away. |
| `player_id`      | integer | The player. Join to `squads[].player_id`. |
| `type_id`        | integer | Type id of the metric — see the table below. |
| `developer_name` | string  | Developer name of the metric. **This is what tells the two families apart.** |
| `value`          | number  | Integer 1–99 for ratings; a decimal count for expected metrics. |
| `meta`           | object  | **Only present on expected metrics** (384, 385). Absent entirely from ratings. |

### Metric types

| `type_id` | `developer_name` | Family | Meaning |
|-----------|------------------|--------|---------|
| `380` | `PLAYER_IMPACT_INDEX` | Rating | Effective at the main job of their position — what that job *is* depends on the position, see [above](#the-impact-metric). |
| `381` | `PLAYER_AGGRESSION_INDEX` | Rating | Competes physically: lots of fouls and tackles. |
| `382` | `PLAYER_DISCIPLINE_INDEX` | Rating | Rarely booked for the fouls they commit. |
| `383` | `PLAYER_SST_RATING` | Rating | Strong overall, combining the three above, weighted for their position. |
| `384` | `PLAYER_EXPECTED_SHOTS` | Expected | Shots attempted in this fixture. |
| `385` | `PLAYER_EXPECTED_MINUTES` | Expected | Minutes played in this fixture. |

### `meta` (expected metrics only)

| Field | Type | Description |
|---|---|---|
| `if_starts` | number | Expected value conditional on starting. |
| `if_benched` | number | Expected value conditional on starting on the bench, already accounting for not being brought on. |
| `p_start` | number | Probability of starting, `0`–`1`. |

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
  },
  {
    "team_id": 4009,
    "player_id": 184521,
    "type_id": 384,
    "developer_name": "PLAYER_EXPECTED_SHOTS",
    "value": 1.293,
    "meta": { "if_starts": 1.632, "if_benched": 0.158, "p_start": 0.77 }
  },
  {
    "team_id": 4009,
    "player_id": 184521,
    "type_id": 385,
    "developer_name": "PLAYER_EXPECTED_MINUTES",
    "value": 65.953,
    "meta": { "if_starts": 82.949, "if_benched": 9.053, "p_start": 0.77 }
  }
]
```

Read together, and knowing from `squads[]` that this player is an `ATTACKER`: a well above-average goal threat (73), not especially physical (41), slightly more prone to picking up a card than most when he does foul (43), rating 63 overall.

For this match he is a likely but not certain starter (`p_start` 0.77), which works out at 1.29 expected shots and 66 expected minutes. If the team sheet names him in the XI, those become **1.63 shots and 83 minutes**; if he is benched, **0.16 and 9**.

The blend is checkable by hand: `0.77 × 1.632 + 0.23 × 0.158 = 1.293`.

> **Not every squad player appears, and the two families have separate bars.** Players without enough recent playing time are omitted rather than given a placeholder — newly signed and youth players are the usual cases. Expected metrics need a longer and richer history than the ratings do, so **a player can have all four ratings and no expected metrics**. Currently around 63% of rated players also carry expected metrics. Always look rows up by `player_id` **and** `developer_name` rather than assuming a fixed six rows per player.

---

## Filtering

`filter[player_metrics]` narrows which metrics come back:

```
filter[player_metrics]=types:380,383
```

returns only Impact and SST Rating for each player, which is usually all a listing view needs.

```
filter[player_metrics]=types:384,385
```

returns only the expected metrics, which is what a player-props view wants.

---

## Real-World Use Cases

### Player cards

Join `player_metrics` to `squads` on `player_id` and render a card per player: the SST Rating as the headline number and the other three ratings as 1–99 bars. Since every rating shares one scale and one direction, a single bar component works for all four. Do **not** feed the expected metrics into that component — they are counts, not scores.

### Ranking a squad

Sort a team's players by `PLAYER_SST_RATING` to surface its most dangerous names, or by `PLAYER_IMPACT_INDEX` within a position to find, say, the best defender on the pitch.

### Shots markets

`PLAYER_EXPECTED_SHOTS` is the direct input for "player to have 1+ / 2+ shots" markets. Two rules make it usable:

- **Before the team sheet**, compare the published `value` against the market price — it already carries the selection risk the price should also reflect.
- **After the team sheet**, switch to `meta.if_starts` or `meta.if_benched`. Prices often move slower than lineups do, and this is where the gap opens.

Cross-check against `PLAYER_EXPECTED_MINUTES` before acting on a low number: a low expected count on a strong player is usually a minutes story, not a form story.

### Reacting to lineup news

`meta.p_start` ranks a squad by rotation risk before any lineup is announced, and once one is, the two conditionals give you the revised numbers immediately — no second API call, no waiting for a republish.

### Card markets

`PLAYER_AGGRESSION_INDEX` and `PLAYER_DISCIPLINE_INDEX` together are the pair that matters for booking markets, and it is the **combination** that carries the signal:

| Aggression | Discipline | Reading |
|-----------|-----------|---------|
| High | **Low** | The genuine card risk — fouls a lot *and* gets booked for it |
| High | High | Physical but gets away with it; far less exposed than the raw foul count suggests |
| Low | Low | Fouls rarely, but is booked when he does |
| Low | High | Minimal exposure |

Note the direction: it is a **low** Discipline that flags risk, not a high one. Weight the pair by `PLAYER_EXPECTED_MINUTES` — a reckless player expected to play 20 minutes is a smaller risk than a moderate one playing 90.

### Goalscorer markets

Impact on an attacker is a direct read on shot volume and finishing relative to other attackers in the league. Paired with `PLAYER_EXPECTED_SHOTS` it separates the two halves of an anytime-scorer price: how many chances he is expected to get in this match, and how well he converts them relative to his peers.

### Matchup context

Compare one side's attackers' Impact against the other side's defenders' Impact to characterise a fixture before looking at odds — both are on the same 50-is-average scale, so they are directly comparable even though one measures shooting and the other blocking.
