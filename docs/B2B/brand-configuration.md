---
title: Brand configuration
deprecated: false
hidden: false
metadata:
  robots: index
---
# Brand configuration

All of your brands share one API key and one set of insights and bet builders. A brand's
**configuration** decides which part of that content the brand receives: which markets, which
selections within them, and at what prices.

Your back office (for example your feature-configuration system) saves each brand's
configuration with our API. Afterwards, when you request content and name the brand, you get only
what that brand is configured for.

A brand is identified by your **`domain_id` + `agent_id`** pair.

> Content endpoints live under a sport prefix (`/v1/football/insights`). The unprefixed paths
> (`/v1/insights`, `/v1/bet-builders`) are deprecated aliases that return identical responses.
> `/v1/integrations/*` has no sport prefix.

```
1. GET  /v1/integrations/{integration}/football/catalog         what can be configured
2. PUT  /v1/integrations/{integration}/brands/{domain_id}/{agent_id}/football/insights
3. GET  /v1/football/insights?filter[integration]=domain_id:{domain_id};agent_id:{agent_id}
```

`{integration}` is the integration name we gave you (e.g. `first`). Every request uses your API
key: `Authorization: Bearer <key>`.

The `/integrations` endpoints need an account we have enabled for brand configuration. A key that
can read content but was not enabled gets `403` there. Ask us to enable the account whose key your
back office uses.

These endpoints are meant to be called **from your servers**. Brands never call them directly.

---

## Sports and products

Configuration is kept separately for each sport and each product:

| Sport | Products |
|---|---|
| `football` | `insights`, `bet_builders`, `acca` |
| `basketball` | coming soon |

Each product is saved on its own, so saving a brand's bet builder configuration never changes its
insights configuration.

`acca` can already be configured, but accumulators are not published yet. A saved `acca`
configuration takes effect once they are.

---

## 1. Catalog

```
GET /v1/integrations/{integration}/football/catalog
```

Lists every market each product can contain, the selections of each market, and the allowed range
for the number of legs. Build your configuration form from this response rather than hardcoding
ids: it grows when we add markets, and a new sport comes with its own catalog.

```jsonc
{
  "sport": "football",
  "products": {
    "insights": {
      "markets": [
        { "market_id": 1, "name": "Match Result (1X2)",
          "selections": [{ "label_id": 1, "name": "Home" }, { "label_id": 0, "name": "Draw" }, { "label_id": 2, "name": "Away" }] },
        { "market_id": 3, "name": "Goal Line (Asian Total Goals)",
          "selections": [{ "label_id": 2, "name": "Over" }, { "label_id": 1, "name": "Under" }] },
        { "market_id": 4, "name": "Match Goals (Over/Under)",
          "selections": [{ "label_id": 2, "name": "Over" }, { "label_id": 1, "name": "Under" }] },
        { "market_id": 53, "name": "Both Teams To Score",
          "selections": [{ "label_id": 1, "name": "Yes" }, { "label_id": 2, "name": "No" }] }
        // ...
      ]
    },
    "bet_builders": { "markets": [ /* ... */ ], "limits": { "legs": { "min": 2, "max": 10 } } },
    "acca":         { "markets": [ /* ... */ ], "limits": { "legs": { "min": 2, "max": 20 } } }
  }
}
```

Market 3 (Asian goal line) and market 4 (European match goals) are separate markets. Configure
whichever ones the brand offers.

---

## 2. Saving a brand's configuration

```
PUT    /v1/integrations/{integration}/brands/{domain_id}/{agent_id}/football/{product}
GET    /v1/integrations/{integration}/brands/{domain_id}/{agent_id}/football/{product}
DELETE /v1/integrations/{integration}/brands/{domain_id}/{agent_id}/football/{product}
GET    /v1/integrations/{integration}/brands/{domain_id}/{agent_id}          all products of one brand
GET    /v1/integrations/{integration}/brands?page=1&per_page=100             all configured brands
```

`domain_id` and `agent_id` may contain `A-Z a-z 0-9 _ . -`, up to 64 characters each.

### Insights

```jsonc
PUT /v1/integrations/first/brands/12/345/football/insights
{
  "enabled": true,
  "markets": [
    { "market_id": 4,  "label_ids": [2],       "price": { "min": 1.90 } },
    { "market_id": 53, "label_ids": [1],       "price": { "min": 1.90, "max": 3.00 } },
    { "market_id": 1,  "label_ids": [1, 0, 2], "price": { "min": 1.50, "max": 4.00 } }
  ]
}
```

| Field | Required | Meaning |
|---|---|---|
| `enabled` | yes | `false` switches the product off for this brand |
| `markets` | yes | one entry per market the brand offers |
| `markets[].market_id` | yes | from the catalog |
| `markets[].label_ids` | no | the ticked selections. Leave it out for every selection of the market |
| `markets[].price.min` / `.max` | no | decimal odds, inclusive. Leave either out for no limit on that side |

An insight is delivered when it matches **any** entry: same market, one of the ticked selections,
price within the range.

### Bet builders and accas

```jsonc
PUT /v1/integrations/first/brands/12/345/football/bet_builders
{
  "enabled": true,
  "markets": [
    { "market_id": 4,  "label_ids": [2], "price": { "min": 1.20 } },
    { "market_id": 53, "label_ids": [1, 2] },
    { "market_id": 1 }
  ],
  "legs":        { "min": 2, "max": 4 },
  "total_price": { "min": 3.00, "max": 15.00 }
}
```

The same fields as insights, plus:

| Field | Required | Meaning |
|---|---|---|
| `legs.min` / `.max` | no | number of selections, within the catalog's `limits.legs` |
| `total_price.min` / `.max` | no | the combined price, decimal odds |

**Every leg** must match one of the `markets` entries, and `price` there applies to that leg's own
price. A bet builder with even one leg outside the brand's markets or selections is not delivered.
The number of legs and the combined price must also be within `legs` and `total_price`. `acca`
takes exactly the same fields.

### Responses

`PUT` replaces the whole product configuration and returns what was stored. `GET` returns the same
shape. `config` is exactly what `PUT` accepts, so you can redraw your form from it and send it back.

```jsonc
{
  "domain_id": "12",
  "agent_id": "345",
  "sport": "football",
  "product": "insights",
  "config": { "enabled": true, "markets": [ /* ... */ ] },
  "created_at": "2026-10-22T09:14:03.120Z",
  "updated_at": "2026-10-22T09:14:03.120Z"
}
```

A saved configuration takes effect on the next content request.

| Status | When |
|---|---|
| `400` | invalid body: unknown field, a market not in the product's catalog, a selection that does not belong to its market, the same market twice, `min` greater than `max`, a value out of range. The message names the field |
| `403` | your key is not entitled to this integration |
| `404` | unknown sport or product; or, on `GET`/`DELETE`, nothing is saved for that brand and product |

`DELETE` returns the product to "not configured" (see below).

---

## 3. Reading content for a brand

Name the brand in `filter[integration]`:

```
GET /v1/football/insights?filter[integration]=domain_id:12;agent_id:345
GET /v1/football/bet-builders?filter[integration]=domain_id:12;agent_id:345
```

The brand selector combines with every other parameter of these endpoints, including the filters
by your own ids and `include=integration`:

```
GET /v1/football/insights?filter[integration]=domain_id:12;agent_id:345;ext_master_league_ids:38&include=integration
```

### Rules

- **Not configured means nothing.** A brand with no saved configuration for the product, with
  `enabled: false`, or with an empty `markets` list gets an empty list: `{"fixtures": []}`.
- **The configuration only narrows.** Other request parameters are applied on top of it. They can
  remove items from the brand's set but never add any. For example, if a brand is configured for
  BTTS only, `market_id=1` returns an empty list, not Match Result insights. This includes the
  public `filter[markets]` filter (same market / selection / price idea, sent per request).
- **Send `domain_id` and `agent_id` together**, one value each. Sending only one returns `400`.
- **Only `GET /v1/football/insights` and `GET /v1/football/bet-builders` apply brand configurations.** Every other
  endpoint answers `400` when it is given `domain_id` / `agent_id`, so a brand is never served
  unfiltered content by mistake.
- Without a brand selector, these endpoints behave exactly as they do today.

Prices in the configuration are compared with the prices the content is generated from, which are
your own odds.
