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
| **Expected metrics** | 384–394 | What the player is predicted to do **in this fixture** | Decimal count (shots, shots on target, goals, assists, involvements, minutes, fouls, tackles, passes, saves), or booking points |

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

`PLAYER_EXPECTED_SHOTS` · `PLAYER_EXPECTED_SHOTS_ON_TARGET` · `PLAYER_EXPECTED_GOALS` ·
`PLAYER_EXPECTED_ASSISTS` · `PLAYER_EXPECTED_GOAL_INVOLVEMENTS` · `PLAYER_EXPECTED_MINUTES` ·
`PLAYER_EXPECTED_BOOKING_POINTS` · `PLAYER_EXPECTED_FOULS` · `PLAYER_EXPECTED_TACKLES`

These are **predictions for the specific fixture**, not ratings. The value is a plain decimal count in the metric's own unit — with one exception, Expected Booking Points, which is a points score rather than a count:

| Metric | `type_id` | Unit | Typical range |
|--------|-----------|------|---------------|
| `PLAYER_EXPECTED_SHOTS` | `384` | Shots attempted | Usually below `1`; above `3` is rare |
| `PLAYER_EXPECTED_SHOTS_ON_TARGET` | `389` | Attempts hitting the target | Roughly a third of Expected Shots |
| `PLAYER_EXPECTED_GOALS` | `386` | Goals scored | Usually well below `0.5`; above `1` is rare |
| `PLAYER_EXPECTED_ASSISTS` | `387` | Assists | Lower than Expected Goals for most players; creators are the exception |
| `PLAYER_EXPECTED_GOAL_INVOLVEMENTS` | `388` | Goals + assists | Always the sum of the two above, exactly |
| `PLAYER_EXPECTED_MINUTES` | `385` | Minutes on the pitch | `0`–`90+` |
| `PLAYER_EXPECTED_BOOKING_POINTS` | `390` | **Booking points** (yellow 10, red 25, second yellow 35) | Around `2.5` for a regular; capped at `35` |
| `PLAYER_EXPECTED_FOULS` | `391` | Fouls committed | Around `1` for a regular starter; goalkeepers near `0` |
| `PLAYER_EXPECTED_TACKLES` | `392` | Tackles | Similar to fouls for defenders and midfielders, lower for forwards — **but see the note on competitions below** |
| `PLAYER_EXPECTED_PASSES` | `393` | **Passes completed** | Much the largest of these numbers — around `30` for a full-match player, and highest for defenders |
| `PLAYER_EXPECTED_SAVES` | `394` | Saves | Around `3` for a full-match goalkeeper. **Goalkeepers only — no row is returned for anyone else** |

Expected Shots counts **every attempt** — on target or not, blocked shots included. Expected Shots on Target counts the subset that hits the target, on the same definition the match feed uses.

> **The three shooting metrics are always ordered:**
>
> ```
> Expected Goals  ≤  Expected Shots on Target  ≤  Expected Shots
> ```
>
> A goal has to be on target, and an on-target attempt has to be a shot. This holds on the published `value` **and** on both `meta` conditionals, so you can rely on it rather than defending against it.

The two ratios in that chain are the two halves of finishing, and they answer different questions:

| Ratio | What it measures |
|---|---|
| `xSoT / xSh` | **Accuracy** — how often his attempts hit the target |
| `xG / xSoT` | **Conversion** — how often the ones on target go in |

Their product, `xG / xSh`, is the overall strike rate, which across a squad typically lands somewhere around one goal in six to one in ten attempts. Splitting it in two is more informative than the single number: a player who shoots a lot from distance and one who gets fewer but better chances can share a strike rate while looking nothing alike here.

> **Not on the 1–99 scale.** An Expected Shots of `1.29` is not "very poor". It is 1.29 shots. Applying the ratings scale to these values, or the reverse, is the one mistake worth guarding against, and it is why the two families are marked by `developer_name` rather than left to context.

### Expected Booking Points

`PLAYER_EXPECTED_BOOKING_POINTS` (390) is the odd one in this family: every other expected metric counts a thing that happened, and this one is a **score**. It uses the standard disciplinary scale:

| Card | Points |
|---|---|
| Yellow | 10 |
| Straight red | 25 |
| Second yellow | 35 |

A second yellow is 10 + 25 because the first booking still counts. So does a player who is booked and then sent off for a separate offence — both routes reach 35, which is the **maximum any one player can score**.

That changes what a typical value looks like. A regular midfielder sits near **2.5**, not near 0.25, and 8 is a high number rather than an impossible one. If you are feeding this into anything that also reads Expected Shots, keep the scales apart.

> **The points total on its own cannot be inverted.** It blends two different events — a booking and a dismissal — at a 1:2.5 weight, so a player at 2.5 points might be a 25% booking risk who is never sent off, or a 15% booking risk who plays on the edge. Those price very differently. So both components ship in `meta` as **`p_booked`** and **`p_sent_off`**, and those, not the points total, are what you price a card market off:
>
> ```
> value = 10 × p_booked + 25 × p_sent_off
> ```
>
> The identity holds up to rounding — `value` is published to 3 decimals and the two probabilities to 5, so a reconstruction can differ by one unit in `value`'s last digit and no more. `p_booked` is the "player to be carded" price directly; remember to divide by `p_play`, since card markets void if he takes no part.

**Two things set it apart from the rest of the family, and both are worth knowing before you price off it.**

*The referee is in it now, but only once he is appointed.* This is the biggest thing to understand about Expected Booking Points, so it is worth setting out slowly.

**Referees are not interchangeable.** Some hand out a lot of cards, some hardly any. Among officials with at least 20 matches, the strictest tenth show roughly 75% more cards per match than the most lenient tenth. Who is in the middle is the single largest influence on how many cards a match produces — larger than anything about the players.

**Knowing who it is does not change the average.** Over a season the strict referees and the lenient ones cancel out. Ask "how many cards in a typical match?" and the answer is the same whether or not anyone has been appointed yet. So this feature does not move the overall level of card prices, and it was not meant to.

**It changes the answer for each individual match.** You are not pricing a typical match, you are pricing one match. If Saturday's official is a strict one, that match will run well above the competition's norm; if he is lenient, well below. Previously both got the competition's average. Now they get their own numbers.

It is the same as a weather forecast. Checking it does not change the average temperature of the city. It still tells you whether to take a coat *tomorrow*.

> **One thing worth knowing if you priced off an earlier version.** Before the referee was included, the model leaned on how many fouls a competition produces as a stand-in — more fouls, more cards. That works in general and fails on a specific and common type of official: one who blows the whistle constantly but rarely reaches for a card. On those matches the old model read the high foul count and predicted *more* cards where fewer actually came.
>
> So the change is not "vague number becomes sharper number". For unusual referees the earlier figure could lean the wrong way. **Card prices carried over from before this release are worth re-requesting**, not just refreshing for precision.

**How it is weighted.** The model reads how far the appointed official's record sits from his competition's average, discounted by how much of a record he has. A referee we have seen twice barely moves the number; one we have seen fifty times moves it a lot. That keeps a new or rarely-seen official from swinging a price on almost no evidence.

**The practical consequence: this metric sharpens as kick-off approaches.** Appointments are published days out at most — a week before a match there is usually no official named, and the number you get is the competition's average behaviour. When the appointment lands the value updates, and for an unusually strict or lenient referee it can move materially. Nothing about the players has changed; the model simply knows more.

Two things follow for how you use it. **Re-request the fixture close to kick-off** if the card price matters, rather than carrying an early number forward — `meta.referee_known` tells you which state a row is in. And **do not read the early number as wrong**: it is the honest expectation given an unknown official, and it is deliberately identical to what you would get for a perfectly average one, so the value does not drift for its own sake.

*Coverage is narrower.* Cards arrive in their own feed bundle, and a fixture can report shots without reporting cards. Expect noticeably fewer players to carry a `390` row than a `384` row in the same fixture — look rows up by `developer_name` and treat the metric as absent rather than zero when it is missing.

### Expected Fouls and Expected Tackles

`PLAYER_EXPECTED_FOULS` (391) and `PLAYER_EXPECTED_TACKLES` (392) are plain counts and behave like
Expected Shots — same row shape, same `meta`, same decimal `value`. Two things are specific to
them.

**Expected Fouls sharpens as kick-off approaches, for the same reason booking points does.**
Everything written above about Expected Booking Points and the referee applies here:
`meta.referee_known` says which state a row is in, and the same advice follows — re-request close
to kick-off if the number matters, and do not read the early value as wrong.

Two differences are worth knowing. **Who the official is, is a more consistent trait for fouls
than for cards** — a referee's foul rate relative to his competition repeats from one half of his
season to the other more reliably than his card rate does, which is unsurprising: whether a
challenge is *called* a foul is his decision outright, while a card is a second decision layered
on top. **But the appointment moves the number less than it moves a card price**, because the
spread between officials is narrower here: the strictest tenth call roughly 30% more fouls per
match than the most lenient tenth, against roughly 65% more cards.

So the appointment matters in both directions and matters more for `390` than for `391`. What it
is *not* is optional on either — a value computed without an official is the competition's normal
behaviour, and for an unusual referee that is the wrong answer for that specific match rather
than a slightly blurred one.

**Expected Tackles has no referee in it at all, deliberately.** A tackle is something a player
does, not something an official awards, and we measured the referee's influence on it as
indistinguishable from noise. A `392` row therefore carries no `referee_known` field and does not
sharpen at appointment time — it is as good a week out as it is an hour out.

> **Compare tackles within a competition, not across them.** What a match scorer records as a
> "tackle" varies between competitions far more than a foul does, because a foul has already been
> judged by the referee and a tackle has not. The model accounts for the competition it is
> predicting, so the number is right for that match — but the same player would read differently
> in a different league, and a cross-competition ranking on `392` measures recording convention as
> much as it measures the player. Fouls are much less affected; shots and goals barely at all.

*Coverage.* Fouls are carried on essentially every fixture that reports shots, so a `391` row is
the most widely available of the expected metrics. Tackles are carried slightly less often, and
unevenly — a few competitions do not report them at all, so no player in them will have a `392`
row. Look rows up by `developer_name` and treat a missing metric as absent rather than zero.

### Expected Passes

`PLAYER_EXPECTED_PASSES` (393) is the newest of the family and behaves differently enough from the
rest to be worth reading before you use it. It is the only one where the typical value is in the
tens rather than around one, and three things follow from that.

**It counts passes COMPLETED, not attempted.** Around 80% of attempted passes are completed, so if
you are pricing against a market quoted on attempts, this number will look about a fifth low and
nothing is wrong. There is no attempted-passes metric today.

**Read it against the player's position, not against other players.** This is the one expected
metric whose position ordering runs opposite to the rest of the family:

| Position | Typical completions per 90 |
|---|---|
| Defenders | highest — roughly `38` |
| Midfielders | roughly `32` |
| Goalkeepers | roughly `19` |
| Forwards | lowest — roughly `17` |

Defenders complete well over twice what forwards do. So a forward on `20` is having an unusually
involved match while a defender on `30` is having a quiet one, which is the reverse of how you
would read Expected Fouls or Expected Tackles on the same two players.

**Treat the number as a central estimate, not as a tight one.** Every expected metric is an
average over how the match might go, but the spread around it is wider here than for any other
metric in the family. The reason is possession: how much of the ball a side ends up with on the
day is a large part of how many passes its players complete, and it is genuinely not knowable
before kick-off — a team protecting a lead and the same team chasing one play very different
matches.

Concretely, if you are pricing an over/under: **the realised count scatters roughly twice as
widely as a Poisson assumption would imply**, and the multiple grows with the value — it is
smallest for low-minute substitutes and largest for the high-volume midfielders and defenders
these markets are usually quoted on. Do not derive a spread from the square root of the value.
Goals and assists are close to Poisson and shots are mildly over-dispersed; passes are not in
that range, and the difference is large enough to move a price rather than shade it.

**No referee term, and none needed.** Unlike Expected Booking Points and Expected Fouls, this
value does not sharpen when the officiating appointment lands — we measured the referee's
influence on passing as indistinguishable from noise. A `393` row carries no `referee_known`
field and is as good a week out as an hour out, subject only to the team-sheet question below.

*Coverage.* Passes are the best-covered statistic in the feed — carried on essentially every
fixture that reports shots — so `393` is the most widely available expected metric we publish.

*Stability.* It is also the most predictable. How much a player passes is a function of his role,
and roles change slowly, so a player's own history is a much stronger guide here than it is for
shots or cards. Expect `393` to move less between fixtures than the shooting metrics do, once
expected minutes are held constant.

### Expected Saves

`PLAYER_EXPECTED_SAVES` (394) is the newest of the family and the only one published for a single
position. Three things about it are unlike every other expected metric, and all three matter
before you price anything off it.

**It exists for goalkeepers and nobody else.** A fixture carries roughly four `394` rows against
roughly forty of every other expected metric. There is no row for an outfield player — not a zero
row, no row at all — because a striker's expected saves is undefined rather than zero. This is
the clearest reason to look rows up by `developer_name` rather than by position in the array.

**It depends on the opponent at least as much as on the goalkeeper**, in the way Expected Assists
depends on the finisher. A keeper cannot make a save nobody attempts, so the strongest thing we
know before kick-off is how much on-target shooting the opposing side does and how much his own
defence tends to concede. The keeper's own record matters, but it is not the largest of the three
inputs — it is roughly the smallest, which is the reverse of every other metric here. In
particular, **a high `394` is not a compliment.** Keepers behind weak defences post the highest
values in the league.

**Treat it as a centre, not a forecast.** This is the least predictable value we publish, and the
reason is football rather than modelling. Saves are a low-frequency event driven almost entirely
by what the opposition happens to do on the day: most of the variation in a keeper's save count
from match to match cannot be known in advance by anyone. The number is the average over how the
match might go, and a realised `1` or a realised `6` against a published `3` are both completely
ordinary outcomes. If you are pricing an over/under, size the spread generously.

**No referee term, and none is possible.** A referee decides whether a challenge is a foul; he has
no influence over whether a shot is on target. We measured his effect on saves as indistinguishable
from noise — the cleanest such result of any metric in the family — so a `394` row carries no
`referee_known` field and does not sharpen when the appointment lands.

*Coverage.* Saves are carried on about 97.5% of the fixtures that report shots, reaching 331
competitions. That is the narrowest coverage of the count metrics — passes reach essentially every
fixture — so expect `394` to be missing somewhat more often than `391`, `392` or `393`.

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

## The expected-metric family is complete

All eleven expected metrics — Shots, Shots on Target, Goals, Assists, Goal Involvements, Minutes,
Booking Points, Fouls, Tackles, Passes and Saves — are live today. Nothing further is planned.

Two properties hold across all of them and are worth relying on:

- **They share one shape.** Same row structure, same decimal `value`, same `meta` with `if_starts`
  / `if_benched` / `p_start`. Code written against Expected Shots handles every one of them
  unchanged.
- **Coverage differs per metric.** Each depends on its own underlying statistic being recorded,
  and competitions carry different subsets — one that reports shots may not report tackles, and
  Expected Saves is the narrowest of the set. So the player count varies between metrics in the
  same fixture, which is why rows should be looked up by `developer_name` rather than by position
  in the array.

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
| `meta`           | object  | **Only present on expected metrics** (384–394). Absent entirely from ratings. |

### Metric types

| `type_id` | `developer_name` | Family | Meaning |
|-----------|------------------|--------|---------|
| `380` | `PLAYER_IMPACT_INDEX` | Rating | Effective at the main job of their position — what that job *is* depends on the position, see [above](#the-impact-metric). |
| `381` | `PLAYER_AGGRESSION_INDEX` | Rating | Competes physically: lots of fouls and tackles. |
| `382` | `PLAYER_DISCIPLINE_INDEX` | Rating | Rarely booked for the fouls they commit. |
| `383` | `PLAYER_SST_RATING` | Rating | Strong overall, combining the three above, weighted for their position. |
| `384` | `PLAYER_EXPECTED_SHOTS` | Expected | Shots attempted in this fixture, on target or not. |
| `385` | `PLAYER_EXPECTED_MINUTES` | Expected | Minutes played in this fixture. |
| `386` | `PLAYER_EXPECTED_GOALS` | Expected | Goals scored in this fixture. Never exceeds Expected Shots on Target. |
| `387` | `PLAYER_EXPECTED_ASSISTS` | Expected | Assists in this fixture. |
| `388` | `PLAYER_EXPECTED_GOAL_INVOLVEMENTS` | Expected | Goals plus assists. Exactly `386` + `387`. |
| `389` | `PLAYER_EXPECTED_SHOTS_ON_TARGET` | Expected | Attempts on target in this fixture. Sits between `386` and `384`. |
| `390` | `PLAYER_EXPECTED_BOOKING_POINTS` | Expected | Booking points in this fixture. **Points, not cards** — see below. |
| `391` | `PLAYER_EXPECTED_FOULS` | Expected | Fouls committed in this fixture. Sharpens once the referee is appointed. |
| `392` | `PLAYER_EXPECTED_TACKLES` | Expected | Tackles in this fixture. Compare within a competition, not across them. |
| `393` | `PLAYER_EXPECTED_PASSES` | Expected | Passes **completed** in this fixture, not attempted. Read against the player's position, not against other players. |
| `394` | `PLAYER_EXPECTED_SAVES` | Expected | Saves in this fixture. **Goalkeepers only** — absent for every other player, and absent is not zero. |

### `meta` (expected metrics only)

| Field | Type | Description |
|---|---|---|
| `if_starts` | number | Expected value conditional on starting. |
| `if_benched` | number | Expected value conditional on not starting, already accounting for not being brought on. |
| `p_start` | number | Probability of starting, `0`–`1`. |
| `p_play` | number | Probability of taking any part in the match, `0`–`1`. Always ≥ `p_start`. |
| `p_booked` | number | **`390` only.** Probability he is shown a yellow card, `0`–`1`. Blended over the team sheet, like `value`. |
| `p_sent_off` | number | **`390` only.** Probability he is sent off by any route, `0`–`1`. Blended the same way. |
| `referee_known` | boolean | **`390` and `391` only.** Whether the officiating appointment was published when this value was computed. `false` means the figure reflects the competition's average refereeing and will move once the official is named. Absent on `392`, `393` and `394`, none of which has a referee term. |

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
    "type_id": 389,
    "developer_name": "PLAYER_EXPECTED_SHOTS_ON_TARGET",
    "value": 0.444,
    "meta": { "if_starts": 0.561, "if_benched": 0.054, "p_start": 0.77, "p_play": 0.834768 }
  },
  {
    "team_id": 4009,
    "player_id": 184521,
    "type_id": 385,
    "developer_name": "PLAYER_EXPECTED_MINUTES",
    "value": 65.953,
    "meta": { "if_starts": 82.949, "if_benched": 9.053, "p_start": 0.77, "p_play": 0.834768 }
  },
  {
    "team_id": 4009,
    "player_id": 184521,
    "type_id": 390,
    "developer_name": "PLAYER_EXPECTED_BOOKING_POINTS",
    "value": 1.005,
    "meta": {
      "if_starts": 1.269, "if_benched": 0.121, "p_start": 0.77, "p_play": 0.834768,
      "p_booked": 0.09108, "p_sent_off": 0.00377, "referee_known": false
    }
  }
]
```

Read together, and knowing from `squads[]` that this player is an `ATTACKER`: a well above-average goal threat (73), not especially physical (41), slightly more prone to picking up a card than most when he does foul (43), rating 63 overall.

For this match he is a likely but not certain starter (`p_start` 0.77), which works out at 1.29 expected shots and 66 expected minutes. If the team sheet names him in the XI, those become **1.63 shots and 83 minutes**; if he is benched, **0.16 and 9**.

The blend is checkable by hand: `0.77 × 1.632 + 0.23 × 0.158 = 1.293`, and the same arithmetic reproduces every other expected row.

Booking points work the same way but read on their own scale — `1.005` points is a mild card risk, not a near-certainty, and the two probabilities in `meta` say why: about a **9% chance of a yellow** and well under **1% of a red**. Those reconstruct the value, `10 × 0.09108 + 25 × 0.00377 = 1.005`, and `p_booked` is what you would price "to be carded" from, after dividing by `p_play`.

Of those 1.63 shots if he starts, 0.56 are expected on target — an accuracy of about 34%, right at the typical level, so his scorer price here rests on volume rather than on him being unusually accurate.

> **Not every squad player appears, and the two families have separate bars.** Players without enough recent playing time are omitted rather than given a placeholder — newly signed and youth players are the usual cases. Expected metrics need a longer and richer history than the ratings do, so **a player can have all four ratings and no expected metrics** — this is common rather than exceptional. Always look rows up by `player_id` **and** `developer_name` rather than assuming a fixed six rows per player.

---

## Filtering

`filter[player_metrics]` narrows which metrics come back:

```
filter[player_metrics]=types:380,383
```

returns only Impact and SST Rating for each player, which is usually all a listing view needs.

```
filter[player_metrics]=types:384,385,386,389
```

returns the expected metrics a player-props view usually needs: shots, minutes, goals and shots on target. Add `387` and `388` for assists and goal involvements.

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

**For an actual price, use `PLAYER_EXPECTED_BOOKING_POINTS` (390) instead.** The two indices are position-relative ratings that tell you *who* to look at; the expected metric is a number you can bet off, and it has already done the minutes weighting for you. Its `meta.p_booked` is the "to be carded" probability directly. The indices remain the better tool for scanning a squad, because they compare a player against his position rather than in absolute terms.

### Shots on target markets

`PLAYER_EXPECTED_SHOTS_ON_TARGET` is the direct input for "player to have 1+ / 2+ shots on target", one of the more liquid player markets. Everything said above about shots markets applies unchanged: compare the published `value` before the team sheet, switch to `meta.if_starts` after it.

What this metric adds beyond xSh is the **accuracy split**. Two players with the same expected shots can have quite different expected shots on target, and the shots-on-target market pays for exactly that difference while the shots market does not. Take `xSoT / xSh` per player and rank the squad by it: the high end is where an on-target price built off shot volume alone is likely to be wrong.

### Goalscorer markets

`PLAYER_EXPECTED_GOALS` is the direct input for anytime-scorer prices, but converting it takes two steps that are easy to skip — and skipping either loses money. See [Pricing markets](#pricing-markets) for the full derivation; the short version is that you must convert **each branch separately** and then divide by `p_play`.

The same before/after team-sheet rule applies, and it bites hardest here: a fringe striker's published value is dominated by the chance he does not start, so confirmation can move it sharply. Switch to `meta.if_starts` the moment the XI is known.

Read alongside `PLAYER_EXPECTED_SHOTS` and `PLAYER_EXPECTED_SHOTS_ON_TARGET`, the trio separates a scorer price into three parts rather than two — how many attempts a player is expected to get, how many of those hit the target, and how many of *those* go in. A high xSh with a low `xSoT / xSh` is a volume shooter whose scorer price is doing less work than it looks; a high `xG / xSoT` is a finisher who needs fewer chances.

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

### Shots on target lines

`PLAYER_EXPECTED_SHOTS_ON_TARGET` prices "1+ / 2+ shots on target" and the structure is identical to shots — convert each branch, mix, divide by `p_play`.

**What is not identical is the dispersion constant.** `D = 1.19` was fitted on shot attempts and does not transfer: on-target attempts are roughly a third as frequent, so the same player's count distribution is a different shape. Until we publish a measured constant for this metric, **Poisson is the honest default** — the rates are low enough that the two families barely separate at the 0.5 line, which is where most of the volume sits. Reusing `1.19` here would be borrowing a number from a distribution it was not measured on.

```js
const { p_start, p_play, if_starts, if_benched } = meta;   // from the 389 row
const p_sub      = p_play - p_start;
const lambda_sub = if_benched / (p_sub / (1 - p_start));

// "1+ shot on target" is P(X >= 1).
const p_on_target = p_start * (1 - Math.exp(-if_starts))
                  + p_sub   * (1 - Math.exp(-lambda_sub));
const fair        = p_play / p_on_target;
```

The higher the line, the more the missing dispersion term costs you — the same pattern the shots table shows, where Poisson runs stingy at 2.5. Treat 2+ and 3+ on-target prices as indicative until the constant is measured.

### Booking points, and cards

This one does **not** follow the Poisson pattern the rest of this section uses, and trying to make it fit is the mistake to avoid. Booking points are not a count — a player scores exactly one of **0, 10, 25 or 35**, and nothing else is reachable. So the distribution is a four-point one you can write down in full, and there is no dispersion constant to worry about.

`meta` gives you the two probabilities directly, and everything else follows:

```js
const { p_booked, p_sent_off, p_play } = meta;   // from the 390 row

// "To be carded" and "to be sent off", the two card markets.
// Both void if he takes no part, so both divide by p_play.
const fair_carded    = p_play / p_booked;
const fair_sent_off  = p_play / p_sent_off;
```

For a **booking-points line** you need the four outcome probabilities, which needs one more fact: how often a dismissal came with a booking attached. Measured across the feed, about **54% of dismissals are straight reds** (25 points) and **46% arrive via a booking** (35 points) — a second yellow, or a booking followed by a separate red.

```js
const STRAIGHT_RED_SHARE = 0.54;                 // measured, global, not per player

const p35 = (1 - STRAIGHT_RED_SHARE) * p_sent_off;
const p25 = STRAIGHT_RED_SHARE * p_sent_off;
const p10 = Math.max(0, p_booked - p35);         // booked, stayed on
const p0  = Math.max(0, 1 - p10 - p25 - p35);

// e.g. "over 10.5 booking points" = anything above a single yellow
const p_over_10_5 = p25 + p35;
const fair        = p_play / p_over_10_5;
```

Those four probabilities reproduce the published `value` exactly — `10·p10 + 25·p25 + 35·p35` is `10·p_booked + 25·p_sent_off` by construction — so you can check your implementation against the row you were given.

> **The split is a global constant, not a per-player one.** Some players really are more likely to pick up a straight red than a second yellow, and 0.54 does not know that. It matters only for lines that separate 25 from 35, which is a thin market; for "over 10.5" the two are on the same side and the constant cancels out entirely.

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
