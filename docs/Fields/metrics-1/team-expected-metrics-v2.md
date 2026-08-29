---
title: Team Expected Metrics (V2)
excerpt: >-
  Per-team pre-match expectations for goals, corners, yellow cards and red cards
  — a new model published alongside the original FT\_EXPECTED\_* family, under
  its own developer names, with per-fixture confidence flags.
deprecated: false
hidden: false
metadata:
  robots: index
---
`include=metrics`

The `*_V2` family answers one question per team, per fixture: **how many of this thing is this
side expected to produce in this match?**

| Metric | `type_id` | Unit | Typical per side |
|---|---|---|---|
| `FT_EXPECTED_GOALS_V2` | `395` | Goals | Around `1.3`; `2.5` is a strong favourite |
| `FT_EXPECTED_CORNERS_V2` | `396` | Corners won | Around `5` |
| `FT_EXPECTED_YELLOWCARDS_V2` | `397` | Yellow cards shown to this side | Around `2` |
| `FT_EXPECTED_REDCARDS_V2` | `401` | Red cards shown to this side | Around `0.11` |

Every value is a plain decimal count in the metric's own unit, published to **3 decimals**. All
four are full-time figures and cover the whole match.

They are available from the moment a fixture is scheduled — no team sheet required — and they
are pre-match statements throughout: nothing is revised once the match has started.

> **These are new names, not new values on the old ones.** The original `FT_EXPECTED_GOALS`,
> `FT_EXPECTED_CORNERS` and `FT_EXPECTED_YELLOWCARDS` are unchanged and still published. Nothing
> you already read has moved. See [Migrating from the original family](#migrating-from-the-original-family).

---

## Reading a row

V2 metrics arrive in the same `fixture.metrics[]` array as everything else, in the same row
shape:

```json
{
  "team_id": 62,
  "type_id": 395,
  "developer_name": "FT_EXPECTED_GOALS_V2",
  "value": 1.482,
  "location": "home",
  "meta": {
    "estimated": false,
    "home_known": true,
    "away_known": true,
    "fitted_at": "2026-08-23T19:44:05+00:00"
  }
}
```

**Two rows per metric per fixture** — one for each side, distinguished by `team_id` and
`location`. A fixture with all four metrics therefore carries eight V2 rows.

Match rows by `developer_name` (or `type_id`) **and** `location`. Do not assume ordering, and do
not assume every metric is present — see below.

> **Match totals are the sum of the two sides.** Expected corners in the match is the home value
> plus the away value. The same holds for goals and for each card metric.

---

## Coverage is not uniform, and a missing row means absent

This is the single most important thing to handle, and the most likely source of a surprise.

**Goals are near-universal; corners and cards are not.** Goals are derived from a feed that
covers essentially every finished fixture, so `FT_EXPECTED_GOALS_V2` is available across roughly
1,250 competitions. Corners and cards come from match statistics, which are reported on a little
under half of fixtures — those three metrics cover roughly 600 competitions. **A fixture
routinely carries `FT_EXPECTED_GOALS_V2` and no `FT_EXPECTED_CORNERS_V2`.** That is the shape of
the data, not a failure.

**Some competitions are skipped entirely.** The model measures how good a side is relative to
the others it plays, which requires a competition where teams meet repeatedly. Leagues qualify.
Friendlies buckets, and knockout cups where two teams meet once or twice, do not — those return
**no V2 row at all**, for any of the four metrics.

The practical consequences:

> **Treat a missing row as absent, never as zero.** A zero expected-corners value would mean "we
> expect no corners", which is never what a missing row means.

> **The V2 family and the original family overlap without either containing the other.** A
> fixture can carry `FT_EXPECTED_GOALS` and no `FT_EXPECTED_GOALS_V2`, or the reverse. If you
> need a value for every fixture, read V2 first and fall back to the original.

> **Look rows up by name.** Never index into `metrics[]` by position, and never assume the four
> metrics arrive as a set.

---

## The `meta` block

Every V2 row carries a small `meta` object that says how much of the number was measured and how
much was assumed. It is worth reading before you price off the value.

| Field | Meaning |
|---|---|
| `home_known` | `true` when the home side had its own measured strength in this competition |
| `away_known` | Same, for the away side |
| `estimated` | `true` when at least one side was **not** known, so its figure falls back to the competition average |
| `fitted_at` | When the model behind this number was last refreshed |
| `referee_known` | **Yellow cards only.** Whether the match official was known at the time of the request |

### `home_known` / `away_known` / `estimated`

A side is "known" when it has enough history in this competition for the model to have measured
it. A side that is **not** known — a promoted club, a newly-relegated one, a team new to the
competition — still gets a published value, but that value is the **competition's average**
rather than anything specific to the team.

So a row like this:

```json
{ "estimated": true, "home_known": false, "away_known": true, "fitted_at": "…" }
```

means: trust the away side's number as measured; treat the home side's number as a placeholder.
The match is likely early in a promoted team's first season in this division.

Two things follow:

- **`estimated: true` does not mean the number is wrong.** It means it is the honest expectation
  for a team we have no record of, which is the competition average. It is the best available
  answer, not a broken one.
- **It fixes itself.** Once the team has played enough matches in the competition, the next
  refresh gives it its own figures and the flag goes to `false`. If you cache values, do not
  cache an estimated one for long.

> **`estimated` is only ever `true` where knowing the team would have changed the number.** For a
> metric where team identity carries no measurable signal, an unrecognised team is priced exactly
> as well as a recognised one, and the flag stays `false` rather than warning you off a number
> that is as good as any other.

### `fitted_at`

The timestamp of the model refresh behind this value. It applies to the whole competition, not
to this fixture — two fixtures in the same competition share it. Use it to tell whether a cached
value is stale after a refresh; it says nothing on its own about the quality of the number.

---

## Cards

Yellow and red cards are **two separate metrics**, deliberately. They behave very differently —
a side picks up roughly two yellows a match and a red about once every nine — and blending them
into one number makes it impossible to recover either. Each is counted as the match statistics
feed reports it.

Keeping them apart is what lets you settle whichever rule your market actually uses. The two
common ones:

| Settlement rule | Weights | Expected value for one side |
|---|---|---|
| Card points | Yellow 10, red 25 | `10 × FT_EXPECTED_YELLOWCARDS_V2 + 25 × FT_EXPECTED_REDCARDS_V2` |
| Card count | Yellow 1, red 2 | `FT_EXPECTED_YELLOWCARDS_V2 + 2 × FT_EXPECTED_REDCARDS_V2` |

Check the rule you are pricing against before you combine — the two conventions give noticeably
different numbers, and a single blended metric would have forced one of them on you.

> **A second yellow is not a third card.** Under both rules above a player dismissed for two
> bookings has already been counted once as a yellow, so adding the red on top is the whole of
> it. Confirm how your book treats it; some settle a second yellow as a red alone.

> **Red cards are small numbers and that is real.** Around `0.11` per side is the normal range,
> so most fixtures return something like `0.09`–`0.14` rather than a round figure. Three decimals
> is the floor at which the value stays useful, which is why the whole family publishes to three.
> Do not round it to two and do not mistake a small value for a missing one.

### The referee, on yellow cards only

Yellow cards are the one metric in this family that reads who is officiating.

**Referees are not interchangeable.** Some show a lot of cards, some hardly any, and who is in
the middle is among the largest single influences on how many cards a match produces.

**Knowing who it is does not change the average.** Over a season strict and lenient officials
cancel out, so the *typical* match is unaffected. What changes is the answer for **this** match:
a strict official pushes it well above the competition norm, a lenient one well below.

**So the value sharpens as kick-off approaches.** Appointments are usually published a day or
two out at most. Before then there is no official named, the metric is published at the
competition average, and `meta.referee_known` is `false`. When the appointment lands the value
updates — and for an unusually strict or lenient referee it can move materially. Nothing about
the teams has changed; the model simply knows more.

Two things follow for how you use it:

- **Re-request close to kick-off** if the card number matters, rather than carrying an early
  value forward. `meta.referee_known` tells you which state a row is in.
- **Do not read the early number as wrong.** It is deliberately identical to what a perfectly
  average official would produce, which is the correct expectation for an unknown one. The value
  does not drift for its own sake, and the movement when the appointment lands is the model
  working rather than a correction.

> **`referee_known` appears only on `FT_EXPECTED_YELLOWCARDS_V2`.** Goals, corners and red cards
> do not consult the official, so publishing the flag on them would imply he was considered and
> missing rather than never consulted. Its absence on those three is not ambiguity — it is the
> answer.
>
> **Red cards genuinely do not use it**, and that is measured rather than an oversight: a referee
> shows only a handful of reds a year, far too few to tell one official from another. So a red
> card figure does *not* sharpen when the appointment lands, and there is no reason to re-request
> it near kick-off the way there is for yellows.

---

## Migrating from the original family

The originals — `FT_EXPECTED_GOALS`, `FT_EXPECTED_CORNERS`, `FT_EXPECTED_YELLOWCARDS` and the
`HT_*` equivalents — are **unchanged, still published, and not deprecated by this release.** You
can hold both families side by side, compare them on your own fixtures, and migrate when you
choose.

They are separate names rather than new values on the same ids because **the two models disagree
by design**, and most where it matters:

- **The V2 family compounds rather than adds.** A strong attack meeting a weak defence multiplies
  through, so the extremes spread further apart than the original model puts them. Expect the
  biggest differences on mismatches and the smallest on evenly matched fixtures.
- **V2 cannot produce a negative expectation.** The original occasionally required clamping at
  zero; this one is bounded below by construction.
- **V2 covers far more competitions** — roughly 1,250 for goals against about 30 — but skips
  cups and friendlies, which the original does not.

Republishing that under the existing names would have silently moved every price already
computed against them, which is why it was not done.

**A practical migration:** read the `_V2` row where it exists, fall back to the original where it
does not, and keep both in your own store for a period so you can see where they part company on
fixtures you care about.

> **There is no `FT_EXPECTED_REDCARDS` to migrate from.** Red cards are new in this family — the
> original publishes goals, corners and yellow cards only, so there is nothing to compare against
> and nothing to switch over. Add it where you want it.

---

## Half-time metrics

`HT_EXPECTED_GOALS_V2`, `HT_EXPECTED_CORNERS_V2` and `HT_EXPECTED_YELLOWCARDS_V2` are **reserved
names that nothing currently publishes.** Do not code against them yet.

The two half-time card and corner metrics are unlikely to arrive: match statistics carry no
per-period corner or card counts, so there is nothing to build them on. Half-time goals are
possible and not yet built.

The original `HT_EXPECTED_*` metrics are unaffected and continue as before.

---

## Checklist

Before you ship against these:

- [ ] Rows are looked up by `developer_name` and `location`, never by position.
- [ ] A missing metric is handled as **absent**, not as `0`.
- [ ] You do not assume goals, corners and cards all arrive together.
- [ ] `meta.estimated` is surfaced or handled rather than ignored.
- [ ] Yellow card values are re-requested near kick-off if `meta.referee_known` is `false`.
- [ ] Yellow and red are combined with the weights your market actually settles on.
- [ ] Red card values are read at 3 decimals, and a small value is not treated as missing.
- [ ] Match totals are computed as home + away, not read from a single row.