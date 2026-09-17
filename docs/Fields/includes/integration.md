---
title: 'Integration '
deprecated: false
hidden: false
metadata:
  robots: index
---
# Integration mapping data (`include=integration`)

Attaches your platform's own event payload to each fixture, so you can join SSTrader fixtures to
your internal ids without a second lookup or a mapping table of your own.

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
