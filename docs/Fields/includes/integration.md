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
