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
| **Expected metrics** | 384–388 | What the player is predicted to do **in this fixture** | Decimal count (shots, goals, assists, involvements, minutes) |

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

`PLAYER_EXPECTED_SHOTS` · `PLAYER_EXPECTED_GOALS` · `PLAYER_EXPECTED_ASSISTS` ·
`PLAYER_EXPECTED_GOAL_INVOLVEMENTS` · `PLAYER_EXPECTED_MINUTES`

These are **predictions for the specific fixture**, not ratings. The value is a plain decimal count in the metric's own unit:

| Metric | `type_id` | Unit | Typical range |
|--------|-----------|------|---------------|
| `PLAYER_EXPECTED_SHOTS` | `384` | Shots attempted | Usually below `1`; above `3` is rare |
| `PLAYER_EXPECTED_GOALS` | `386` | Goals scored | Usually well below `0.5`; above `1` is rare |
| `PLAYER_EXPECTED_ASSISTS` | `387` | Assists | Lower than Expected Goals for most players; creators are the exception |
| `PLAYER_EXPECTED_GOAL_INVOLVEMENTS` | `388` | Goals + assists | Always the sum of the two above, exactly |
| `PLAYER_EXPECTED_MINUTES` | `385` | Minutes on the pitch | `0`–`90+` |

Expected Shots counts **every attempt** — on target or not, blocked shots included.

> **Expected Goals never exceeds Expected Shots**, on the published value and on both `meta` conditionals. A goal has to start as a shot, so the ordering always holds and you can rely on it.

Dividing the two gives an implied conversion rate — how likely each of a player's attempts is to go in. Across a squad that typically lands somewhere around one goal in six to one in ten attempts, and it is a fair way to separate a high-volume shooter from a clinical finisher.

> **Not on the 1–99 scale.** An Expected Shots of `1.29` is not "very poor". It is 1.29 shots. Applying the ratings scale to these values, or the reverse, is the one mistake worth guarding against, and it is why the two families are marked by `developer_name` rather than left to context.

### How much they move between fixtures

Expected metrics vary far more from match to match than the ratings do, because the opponent, the venue and the player's chance of starting all bear on them directly. Across a few fixtures the same player's expected shots can differ by well over half, while his ratings drift by only a point or two.

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
| `if_benched` | The expected value **if the player does not start** |
| `p_start` | The probability that he starts, which produced the blend |
| `p_play` | The probability he takes **any** part in the match |

> **`if_benched` already includes the chance he never comes on.** It is not simply "if_starts, but shorter" — most benched players do not appear at all, and that is priced in. Use it as published; do not scale it down further.

`p_play` is the number betting markets need, because essentially every player market is **void if the player takes no part**. It is always at least `p_start`, and the gap between them is the chance he appears as a substitute. See [Pricing markets](#pricing-markets) below.

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

Expected Shots, Goals, Assists, Goal Involvements and Minutes are live today. The rest of the family is not. The rest are **not implemented yet — no `type_id` is assigned, and nothing for them is returned today.** They are listed so you can see where the family is going, not so you can code against them:

| Planned | Unit | Notes |
|---------|------|-------|
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
| `386` | `PLAYER_EXPECTED_GOALS` | Expected | Goals scored in this fixture. Never exceeds Expected Shots. |
| `387` | `PLAYER_EXPECTED_ASSISTS` | Expected | Assists in this fixture. |
| `388` | `PLAYER_EXPECTED_GOAL_INVOLVEMENTS` | Expected | Goals plus assists. Exactly `386` + `387`. |

### `meta` (expected metrics only)

| Field | Type | Description |
|---|---|---|
| `if_starts` | number | Expected value conditional on starting. |
| `if_benched` | number | Expected value conditional on not starting, already accounting for not being brought on. |
| `p_start` | number | Probability of starting, `0`–`1`. |
| `p_play` | number | Probability of taking any part in the match, `0`–`1`. Always ≥ `p_start`. |

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
    "meta": { "if_starts": 1.632, "if_benched": 0.158, "p_start": 0.77, "p_play": 0.834768 }
  },
  {
    "team_id": 4009,
    "player_id": 184521,
    "type_id": 385,
    "developer_name": "PLAYER_EXPECTED_MINUTES",
    "value": 65.953,
    "meta": { "if_starts": 82.949, "if_benched": 9.053, "p_start": 0.77, "p_play": 0.834768 }
  }
]
```

Read together, and knowing from `squads[]` that this player is an `ATTACKER`: a well above-average goal threat (73), not especially physical (41), slightly more prone to picking up a card than most when he does foul (43), rating 63 overall.

For this match he is a likely but not certain starter (`p_start` 0.77), which works out at 1.29 expected shots and 66 expected minutes. If the team sheet names him in the XI, those become **1.63 shots and 83 minutes**; if he is benched, **0.16 and 9**.

The blend is checkable by hand: `0.77 × 1.632 + 0.23 × 0.158 = 1.293`.

> **Not every squad player appears, and the two families have separate bars.** Players without enough recent playing time are omitted rather than given a placeholder — newly signed and youth players are the usual cases. Expected metrics need a longer and richer history than the ratings do, so **a player can have all four ratings and no expected metrics** — this is common rather than exceptional. Always look rows up by `player_id` **and** `developer_name` rather than assuming a fixed six rows per player.

---

## Filtering

`filter[player_metrics]` narrows which metrics come back:

```
filter[player_metrics]=types:380,383
```

returns only Impact and SST Rating for each player, which is usually all a listing view needs.

```
filter[player_metrics]=types:384,385,386
```

returns the expected metrics for shots and minutes; add `386` for goals. A player-props view usually wants all three.

---

## Real-World Use Cases

### Player cards

Join `player_metrics` to `squads` on `player_id` and render a card per player: the SST Rating as the headline number and the other three ratings as 1–99 bars. Since every rating shares one scale and one direction, a single bar component works for all four. Do **not** feed the expected metrics into that component — they are counts, not scores.

### Ranking a squad

Sort a team's players by `PLAYER_SST_RATING` to surface its most dangerous names, or by `PLAYER_IMPACT_INDEX` within a position to find, say, the best defender on the pitch.

### Shots markets

`PLAYER_EXPECTED_SHOTS` is the direct input for "player to have 1+ / 2+ shots" markets. See [Shots lines](#shots-lines-over-05-15-25) for the exact conversion — it is a negative binomial per branch, not a Poisson on the blended value. Two further rules make it usable:

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

`PLAYER_EXPECTED_GOALS` is the direct input for anytime-scorer prices, but converting it takes two steps that are easy to skip — and skipping either loses money. See [Pricing markets](#pricing-markets) for the full derivation; the short version is that you must convert **each branch separately** and then divide by `p_play`.

The same before/after team-sheet rule applies, and it bites hardest here: a fringe striker's published value is dominated by the chance he does not start, so confirmation can move it sharply. Switch to `meta.if_starts` the moment the XI is known.

Read alongside `PLAYER_EXPECTED_SHOTS`, the pair separates the two halves of a scorer price — how many chances a player is expected to get, and how likely each is to go in. A high xSh with a low ratio is a volume shooter; the reverse is a clinical finisher who needs fewer chances.

### Matchup context

Compare one side's attackers' Impact against the other side's defenders' Impact to characterise a fixture before looking at odds — both are on the same 50-is-average scale, so they are directly comparable even though one measures shooting and the other blocking.

---

## Pricing markets

Turning an expected value into a fair price is not `1 − exp(−value)`. Two corrections come first, and the second is much larger than the first.

**1. Convert each branch, then blend — never the other way round.** Starting and being benched are different worlds, and the conversion from a mean to a probability is curved, so collapsing them first overstates the price.

**2. Divide by the probability the bet stands.** Almost every player market is void if the player takes no part, so a fair price is conditional on him appearing:

```
fair probability = P(event) / P(bet stands)
```

Use `p_play` when the market voids on non-participation, or `p_start` when it voids unless the player starts. Both conventions exist; the two fields are exactly those two rules.

### Anytime goalscorer, worked through

Given a row with `p_start: 0.5208`, `p_play: 0.6557`, `if_starts: 0.226`, `if_benched: 0.021`:

```js
const { p_start, p_play, if_starts, if_benched } = meta;

// The chance he appears as a substitute is the gap between the two probabilities.
const p_sub = p_play - p_start;

// if_benched already has "might not come on" folded in, so undo it to get the rate
// that applies once he IS on the pitch.
const lambda_sub = if_benched / (p_sub / (1 - p_start));

const p_scores = p_start * (1 - Math.exp(-if_starts))
               + p_sub   * (1 - Math.exp(-lambda_sub));

const fair_odds = p_play / p_scores;     // 5.70
```

For comparison, on the same row: reading `value` as a probability gives 7.81, and `1 − exp(−value)` gives 8.32. Both are far enough off to erase any edge — the void adjustment alone is worth more than a typical bookmaker's margin.

### Shots lines: over 0.5, 1.5, 2.5

Shots markets void the same way, so the structure is identical — convert each branch, mix, divide by `p_play`. The only difference is the count distribution.

**Do not fit the spread against the published `value`.** Measured that way shot counts look heavily overdispersed, around 1.8× variance-to-mean, and that number is misleading: most of it is the start/bench mixture itself. `value` blends "started and played 90 minutes" with "came on for ten", two genuinely different distributions, and their spread is not shot randomness. Once you convert each branch separately — which the method above already requires — the remaining spread is mild, close to **1.19** variance-to-mean.

That leaves a negative binomial with a single dispersion constant:

```js
const D = 1.19;

// P(X >= n) for one branch. Setting r = mu/(D-1) makes the success probability exactly
// 1/D regardless of mu, which is why one constant works at any expected count.
function atLeast(n, mu) {
    if (mu <= 0) return 0;
    const r = mu / (D - 1), p = 1 / D, q = 1 - p;
    let term = Math.pow(p, r), cdf = term;
    for (let j = 1; j < n; j++) { term *= ((r + j - 1) / j) * q; cdf += term; }
    return 1 - cdf;
}

const { p_start, p_play, if_starts, if_benched } = meta;
const p_sub      = p_play - p_start;
const lambda_sub = if_benched / (p_sub / (1 - p_start));

// "over 1.5 shots" is P(X >= 2).
const p_over  = (p_start * atLeast(2, if_starts) + p_sub * atLeast(2, lambda_sub)) / p_play;
const fair    = p_play / (p_start * atLeast(2, if_starts) + p_sub * atLeast(2, lambda_sub));
```

How the two families compare against settled results, as a percentage error on the fair probability:

| Line | Poisson | Negative binomial |
|---|---|---|
| over 0.5 | +4% | **−1%** |
| over 1.5 | +3% | +3% |
| over 2.5 | **−6%** | **0%** |

Poisson is not catastrophic, but it is biased in a consistent and exploitable direction — too generous on low lines, too stingy on high ones. The negative binomial removes most of that.

### Assists, and goal involvements

`PLAYER_EXPECTED_ASSISTS` prices "anytime assist" exactly like anytime goalscorer — Poisson per branch, mixed, divided by `p_play`. Assists are rarer than goals for most players, so the rate is low enough that the Poisson approximation is comfortable.

**Expected goal involvements is published directly** as `PLAYER_EXPECTED_GOAL_INVOLVEMENTS` (`388`). You no longer need to add the rows yourself, though you still can — the identity holds exactly:

```
value(388)      = value(386)      + value(387)
if_starts(388)  = if_starts(386)  + if_starts(387)
if_benched(388) = if_benched(386) + if_benched(387)
```

That is exact on the published 3dp numbers, not approximate: the sum is taken from the same rounded conditionals the other two rows carry, so it cannot drift from its own addends.

**It is a count, not a probability, and the difference matters here more than anywhere else.** Goals and assists are separate events, so their expected counts add — but the probability of *either* is not the sum of the two probabilities. Convert at the branch level, exactly as for the other metrics:

```js
// per branch, never on the blended value
const p_involved = 1 - Math.exp(-gi_if_starts);
```

Two caveats worth carrying:

- A player **cannot assist his own goal**, so the two events are mildly negatively correlated within a match. That does not affect the published value — expectation is linear whatever the correlation — but treating them as independent when converting slightly overstates "score or assist". Small next to the void adjustment, and real.
- The row is emitted **only when both addends are**. Coverage differs per metric, and a league recording goals but not assists would otherwise produce an "involvements" figure that quietly means goals only. If `388` is missing while `386` is present, that is why.

### Other markets

**Goals stay Poisson.** At the rates involved the two families barely differ for "at least one", and the anytime-scorer method above needs no dispersion term.

**Cards run the opposite way.** A player almost never receives more than one yellow, so a card market is closer to a coin flip than a count, and `1 − exp(−λ)` will *understate* it. Do not reuse the shots recipe there.

**Tackles, fouls and passes** behave like shots — expect overdispersion within a branch — but we have not measured their constants, so treat `D = 1.19` as specific to shots rather than a general figure.

### After the team sheet

Once the XI is confirmed the mixture collapses and the arithmetic gets simpler — use `if_starts` with `P(bet stands) = 1`, or `if_benched` for a named substitute. Note that a confirmed substitute has a **higher** chance of appearing than `p_play` implies: `p_play` is a pre-match number that includes the possibility of being left out of the squad altogether, which the team sheet has now ruled out.
