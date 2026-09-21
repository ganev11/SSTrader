---
title: 'Integration '
deprecated: false
hidden: false
metadata:
  robots: index
---
# Your platform's internal ids

Two fields carry your own platform's identifiers, so you can join SSTrader data to your system
without a second lookup or a mapping table of your own:

| Field | On | Carries |
|---|---|---|
| `integration` | each fixture | your event id and its surrounding event detail |
| `raw` | each odd | your market and selection ids for that price |

They are independent — request either, or both.

Every id in `integration` can also be sent back as a **filter**, so you can query with your own
ids instead of pulling a list and matching on your side — see "Filtering by your own ids" below.

---

# Event ids on fixtures (`include=integration`)

Attaches your platform's own event payload to each fixture.

Off by default. It costs nothing unless you ask for it.

## Requesting it

```
GET /v1/fixtures?league_id=8&include=integration
GET /v1/fixtures?fixture_id=1150093,1150091&include=integration
GET /v1/fixtures?league_id=8&include=odds,metrics,integration
```

`include=integration` combines freely with the other modules.

Access is granted per API key, and your key resolves to your integration automatically — you do
not normally need to name one. If you do, `filter[integration]=providers:first` selects it
explicitly. Only one integration is served per request: if several are listed, the first is used
and the rest ignored.

Asking for an integration your key is not entitled to returns `403`; asking for one that does not
exist returns `400`. Both are raised before any fixture data is loaded.

## What you get back

Each fixture gains an `integration` object holding your feed's payload for that event, with
`provider` naming the integration that answered.

```jsonc
{
  "fixtures": [
    {
      "id": 1147719,
      "date_time": "2026-10-07T23:30:00.000Z",
      "participants": [ /* ... */ ],
      "integration": {
        "provider": "first",
        "home_team": "Botafogo RJ",
        "away_team": "Vasco da Gama",
        "country_name": "Brazil",
        "league_name": "Serie A",
        "date_time": "2026-10-07T23:30:00.000Z",
        "ext_event_id": "887346598991204352",
        "ext_home_id": "122061",
        "ext_away_id": "2028",
        "ext_country_id": "29",
        "ext_league_id": "801347706789560320",
        "is_betbuilder": true
      }
    }
  ],
  "pagination": { "page": 1, "per_page": 25, "has_more": false, "order": "asc" }
}
```

### Three things to handle

**Ids are JSON strings, and that is deliberate.** `ext_event_id` and the other external ids are
64-bit integers. A value like `887346598991204352` cannot survive `JSON.parse` in a language with
IEEE-754 numbers — JavaScript silently rounds it to `887346598991204400`, and you would corrupt one
id in every batch with no error raised anywhere. Sending them quoted means every digit reaches you
intact. Compare them as strings, or parse them with a bigint-aware reader; do not cast them to a
double.

**Fields vary by event.** The payload is your feed's own record of that event, and not every event
carries every field. Read the fields you need defensively rather than assuming they are present.

**A missing `integration` key is normal.** A fixture your feed does not carry simply has no
`integration` key — not `null`, not an empty object. SSTrader's fixture coverage is wider than any
single integration's, so expect this on a routine listing and treat the key's presence as the test
for whether the event exists on your side.

## Which event you get when there are several

When an event is dropped and republished under a new id, both records can end up mapped to the
same fixture. You always get the **most recently updated** one, resolved deterministically —
repeat the same request and the same event id comes back every time.

If you need the superseded ids as well, ask us; they exist, they are just not exposed here.

---

# Filtering by your own ids

Every id we hand you in the `integration` object can also be sent back as a filter, so you can
query in your own vocabulary instead of pulling a list and matching on your side.

## Syntax

One parameter, `filter[integration]`. Segments are separated by `;`, values inside a segment by `,`.

```
GET /v1/fixtures?filter[integration]=ext_league_ids:860235927002570752
GET /v1/fixtures?filter[integration]=ext_event_ids:886928362927575040,886956863147749376
GET /v1/fixtures?filter[integration]=ext_country_ids:65&include=integration
GET /v1/insights?filter[integration]=ext_team_ids:300
GET /v1/bet-builders?filter[integration]=ext_master_league_ids:38
```

You do not need `providers:` — your API key already resolves to your integration. Add it only if
you want to be explicit: `filter[integration]=providers:first;ext_league_ids:…`.

Filtering does **not** require `include=integration`. They are independent: filter to choose which
fixtures come back, include to get your ids attached to them. Combining both is usually what you
want.

## The filters

| Filter | Takes | Returns |
|---|---|---|
| `ext_event_ids` | your event ids | those exact fixtures |
| `ext_league_ids` | your league ids | every fixture in the matching leagues |
| `ext_master_league_ids` | your master league ids | same, keyed on the stable id |
| `ext_country_ids` | your country ids | every league we hold under that country |
| `ext_team_ids` | your team ids | fixtures with that team on **either** side |
| `ext_home_ids` / `ext_away_ids` | your team ids | that team at home / away only |
| `mapped_only` | `1` or `0` | narrows to fixtures your feed carries (`/fixtures` only) |

One of the three league-shaped keys, and one of the three team-shaped keys, per request. Sending
two of either returns `400` — a league *and* a country is a union to one reader and an intersection
to another, and there is nothing in the request to say which you meant.

## Where each one works

| | `/fixtures` | `/insights` | `/bet-builders` | `/fixtures/search` | `/insights/archive` |
|---|:--:|:--:|:--:|:--:|:--:|
| `ext_event_ids` | ✓ | ✓ | ✓ | — | — |
| `ext_league_ids` | ✓ | ✓ | ✓ | ✓ | — |
| `ext_master_league_ids` | ✓ | ✓ | ✓ | ✓ | — |
| `ext_country_ids` | ✓ | ✓ | ✓ | ✓ | — |
| `ext_team_ids` / `ext_home_ids` / `ext_away_ids` | ✓ | ✓ | ✓ | — | — |
| `mapped_only` | ✓ | — | — | — | — |

A dash is a `400`, never a silently ignored filter. `/fixtures/search` selects by metric value
ranges and has no event or team predicate; `/insights/archive` filters on model and time only.

## Three things to handle

**Send ids as strings.** Exactly as you receive them. `886928362927575040` does not survive
`JSON.parse` in a language with IEEE-754 numbers — it becomes `886928362927575100`, silently, in
one id per batch. This is the same reason we quote them in the response.

**An unknown id returns an empty list, not an error.** If we have no mapping for a league or team
you name, you get `{"fixtures": []}`. That is not a failure — it means we have not yet seen enough
of your events in that league to map it, and coverage grows over time. It is never an *unfiltered*
list: a filter that matches nothing returns nothing.

**A filter we do not support returns `400`.** We would rather refuse than quietly drop it and hand
you back the full schedule, which you would have no way to distinguish from a real result. The same
goes for naming the same thing twice: `league_id` together with a league filter, or `fixture_id`
together with `ext_event_ids`.

## One of your leagues can be several of ours

Where you carry a regionalised division as a single league, we split it into groups. Spain's
Tercera is one league to you and **18** to us; Italy's Serie D is one to you and 9 to us.

`ext_league_ids` handles this for you — it returns fixtures from all of them, and you never see our
ids unless you ask for them. It is worth knowing only because the result can be larger than a
one-league query would suggest.

## Prefer `ext_master_league_ids`

Your platform reissues `ext_league_id` from time to time. `ext_master_league_id` does not change,
and it is the id your provider recommends keying on.

Both work, and a retired `ext_league_id` keeps resolving — we accumulate them rather than replacing
them, so an id you cached months ago still finds its league. But a filter written against the
master id will not need revisiting.

## `mapped_only` — your events, not ours

Our fixture coverage runs further ahead than your platform's publishing schedule. You list an event
roughly two to three days before kickoff; inside that window you have 75–85% of our fixtures, but
seven days out you have around 5%.

So by default a league filter returns fixtures you cannot act on yet:

```
GET /v1/fixtures?filter[integration]=ext_country_ids:65
    → the full schedule, including fixtures not yet on your platform

GET /v1/fixtures?filter[integration]=ext_country_ids:65;mapped_only:1
    → only the ones you carry
```

Both are useful — the first is a view of what is coming, the second is what your users can bet on
today. Full coverage is the default because that is how our `league_id` filter already behaves.

`mapped_only` needs a league, country or team filter alongside it; on its own it would mean "every
fixture you have ever carried", which is not a query we serve. Note also that the result grows on
its own as kickoff approaches and you publish more events, so do not treat it as a stable set.

## Pagination

Filters narrow the query itself, so pages come back full and pagination behaves normally.

One thing to know: `has_more` is inferred from whether the page came back full, not from a count.
If your result is an exact multiple of `per_page`, the last page will report `has_more: true` and
the next page will be empty. **Stop on an empty page, not only on `has_more: false`.**

## Worked example

Your league page, showing only what you can price, with your ids attached:

```
GET /v1/fixtures
  ?filter[integration]=ext_master_league_ids:38;mapped_only:1
  &include=integration,odds
  &filter[odds]=bookmakers:<your bookmaker_id>
  &per_page=50
```

That gives you the fixtures, your event ids on each one, and your selection ids on every price —
a full round trip without ever touching our ids.

---

# Selection ids on odds (`raw`)

Every odd carries a `raw` object holding your platform's own references for that exact price —
enough to place the selection on your side, or to deep-link to it.

Unlike `integration`, `raw` needs no separate opt-in: it is part of the odd, and arrives with
`include=odds`.

## Requesting it

```
GET /v1/fixtures?league_id=8&include=odds
GET /v1/fixtures?league_id=8&include=odds&filter[odds]=bookmakers:<your bookmaker_id>
```

**Scope to your own `bookmaker_id`.** Without `filter[odds]=bookmakers:…` you receive prices from
every bookmaker we carry, each with its own `raw` in its own format. Filtering is both less data
and the only way to be sure the `raw` you are reading is yours to use.

You can combine the two features on one call, which is usually what you want — the fixture's
`integration` gives you the event, and each odd's `raw` gives you the selection within it:

```
GET /v1/fixtures?league_id=8&include=integration,odds&filter[odds]=bookmakers:<your bookmaker_id>
```

## What you get back

```jsonc
{
  "odd_id": 117213472,
  "fixture_id": 1129928,
  "market_id": 353,
  "bookmaker_id": 7,
  "label_id": 1,
  "value": 4.43,
  "line": 0,
  "suspend": 1,
  "player_id": 370446,
  "market_name": "Goalscorers (Golden Sub)",
  "label_name": "First",
  "raw": {
    "event_id": "881531765548929024",
    "market_id": "0QA883694268420845586",
    "market_type_id": "QA6431",
    "selection_id": "0QA883694268420845586Q1Q250956",
    "selection_type_id": 0,
    "ext_player_id": 250956,
    "player_name": "Ricardo Pepi"
  }
}
```

| Field | Meaning |
|---|---|
| `event_id` | your id for the event this price belongs to |
| `market_id` | your id for this specific market instance on this event |
| `market_type_id` | your id for the kind of market, shared across events |
| `selection_id` | your id for this exact selection — the value to place a bet against |
| `selection_type_id` | your id for the kind of selection within the market |
| `ext_player_id` | your player id; present only on player markets |
| `player_name` | the player as your feed names them; present only on player markets |
| `is_betbuilder` | present when the selection is eligible for a bet-builder combination |

The surrounding fields are SSTrader's own: `market_id`, `label_id` and `player_id` at the top level
are **our** numbering and unrelated to the identically named ids inside `raw`. Read ids from `raw`
when talking to your platform, and the top-level ones when talking to us.

### Three things to handle

**Treat every id in `raw` as an opaque string.** Some are alphanumeric
(`"0QA883694268420845586"`). Others are long integers — `event_id` runs to 18 digits, past what an
IEEE-754 double can hold, so it is sent quoted for exactly the reason described above. Do not
parse these into numbers, do not normalise their case or padding, and do not assume the format is
stable between market types. Compare and store them as text.

**`raw` varies by bookmaker.** Each bookmaker's `raw` is its own provider's reference data, so the
set of fields differs between them and some carry no `raw` at all (`{}`). The fields above describe
your own integration's shape; do not write code that reads another bookmaker's `raw` expecting the
same keys.

**Not every field is on every selection.** `ext_player_id`, `player_name` and `is_betbuilder` only
appear where they apply. Read defensively.

## Joining odds back to the event

`raw.event_id` on an odd is the same id as `integration.ext_event_id` on its fixture, so you can
match a price to an event entirely in your own vocabulary, without going through our `fixture_id`.

One exception worth handling: when an event is republished under a new id, prices written before
the republish keep the **superseded** `event_id`, while the fixture's `integration` reports the
newest. The two then disagree for that event. It is uncommon — around 1% of events — but if you
join strictly on `event_id` you will drop those prices. Join on our `fixture_id` when you need
completeness, and use `event_id` when you need your own vocabulary.
