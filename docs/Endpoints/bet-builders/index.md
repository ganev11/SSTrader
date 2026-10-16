---
title: Bet Builders
excerpt: >-
  AI-selected bet builder suggestions — multiple selections from the same match
  combined into a single bet, with a natural-language explanation for the
  combination.
deprecated: false
hidden: false
metadata:
  robots: index
---

<Callout icon="🛑" theme="error">
  ### **NB**

  This endpoint is in beta testing.
</Callout>

`GET /bet-builders`

A bet builder combines multiple selections from the **same match** into one bet (e.g. Match Winner + Over 2.5 Goals for the same fixture).

Results use the same layout as `/insights`: a list of fixtures, where each fixture contains a `bet_builders` array. This lets you display bet builder suggestions directly alongside the relevant match data, team names, and live scores.

---

## Query Parameters

| Parameter    | Type    | Default | Description |
|--------------|---------|---------|-------------|
| `model_id`   | string  | —       | Comma-separated list of Model IDs (e.g. `1,2,3`) |
| `language`   | string  | `en`    | Language code for the generated content. Also accepted as `lang` |
| `fixture_id` | integer | —       | Restrict results to a specific single fixture |
| `league_id`  | string  | —       | Comma-separated list of League IDs |
| `is_live`    | integer | —       | Use `1` to filter only for events currently in-play, `0` for events not yet started. Omitted by default — no filtering on match state. |

```
GET /bet-builders?model_id=12&language=en
```

> **Always active:** Results only ever include active, non-suspended bet builders — there's no way to include suspended or expired ones yet. That's coming with a dedicated `/bet-builders/archive` endpoint (TODO), mirroring `/insights/archive`.

> **Pagination:** Not implemented yet (TODO) — a fixed internal result limit is applied for now.

---

## Bet Builder Object

| Field            | Type            | Description |
|------------------|-----------------|--------------|
| `id`             | integer         | Unique identifier for this bet builder. |
| `user_id`        | integer         | The owning user's ID — always your own, since results are scoped to the authenticated user. |
| `model_id`       | integer         | The Model that generated this suggestion. |
| `is_live`        | integer         | `1` if this bet builder was generated for an in-play match, `0` for pre-match. |
| `value`          | number / null   | Current combined decimal odds. `null` while the combination is still being priced. |
| `sp`             | number / null   | Starting Price — the combined odds at the time this bet builder was generated. |
| `suspend`        | integer         | Boolean flag (`0` or `1`). If `1`, this bet builder is temporarily unavailable (e.g. one of its selections is currently suspended). Treat this as "wait", not "cancelled" — it can flip back to `0` once the selection becomes available again. |
| `status`         | integer         | Settlement status. Same codes as the [Odd Object](/docs/odds) `status` field. |
| `last_priced_at` | string / null   | ISO 8601 timestamp of the last time `value`/`sp` were updated. |
| `raw`            | object          | Bookmaker-specific reference data for this exact combination of selections. Opaque and varies by bookmaker; returns `{}` when not available. |
| `meta`           | object          | Additional bookmaker-specific metadata (e.g. promotional price boosts). Shape may vary and is not guaranteed to be present. |
| `created_at`     | string          | ISO 8601 timestamp of generation. |
| `language`       | string          | Language code of the `content` below. Matches the request's `language`/`lang` param. |
| `content`        | object          | Dynamic. The natural-language explanation for the combination — can hold any key-value pairs. |
| `model`          | object          | Metadata about the model used (`id`, `name`, `color`). |
| `selections`     | array           | The individual picks that make up this bet builder — see below. |

### Selection Object

Each entry in `selections` follows the [Odd Object](/docs/odds) schema (`odd_id`, `fixture_id`, `market_id`, `value`, `suspend`, `status`, `outcome`, `market_name`, `label_name`, etc.).

---

## Example JSON Response

```json
{
  "fixtures": [
    {
      "id": 1080495,
      "date_time": "2026-07-16T17:00:00.000Z",
      "status": "NOT_STARTED",
      "country": {
        "id": 81,
        "name": "Norway",
        "alpha3": "NOR"
      },
      "league": {
        "id": 155,
        "name": "Eliteserien",
        "level": 1
      },
      "season": {
        "id": 17734,
        "name": "2026",
        "starting_at": "2026-03-14",
        "ending_at": "2026-12-06",
        "is_current": true
      },
      "group": null,
      "stage": {
        "id": 48471,
        "name": "Regular Season"
      },
      "sport": {
        "id": 1,
        "name": "Football"
      },
      "participants": [
        {
          "id": 411,
          "name": "Valerenga IF",
          "position": 11,
          "location": "home"
        },
        {
          "id": 349,
          "name": "Aalesunds",
          "position": 12,
          "location": "away"
        }
      ],
      "is_live": false,
      "language": "en",
      "scores": [],
      "periods": [],
      "bet_builders": [
        {
          "id": 3,
          "user_id": 2,
          "model_id": 164,
          "is_live": 0,
          "value": 2.71,
          "sp": 2.71,
          "suspend": 0,
          "status": 0,
          "last_priced_at": "2026-07-16T11:58:08.450Z",
          "raw": {
            "Bets": [
              {
                "Type": "Single",
                "MaxStake": 0,
                "MinStake": 0,
                "TrueOdds": 2.71,
                "DisplayOdds": "2.71",
                "NumberOfBets": 1,
                "SelectionsMapped": [
                  {
                    "Id": "0VS0ML864430244147548160H|0QA864430244147548195Q1714Q0"
                  }
                ]
              }
            ],
            "Selections": [
              {
                "Id": "0VS0ML864430244147548160H|0QA864430244147548195Q1714Q0",
                "TrueOdds": 2.71,
                "BetslipLine": "Valerenga | Yes",
                "DecimalOdds": "2.71",
                "DisplayOdds": "2.71",
                "IsEarlyPayout": false
              },
              {
                "Id": "0ML864430244147548160H",
                "TrueOdds": 1.69,
                "BetslipLine": "Valerenga",
                "DecimalOdds": "1.69",
                "DisplayOdds": "1.69",
                "IsEarlyPayout": false
              },
              {
                "Id": "0QA864430244147548195Q1714Q0",
                "TrueOdds": 1.48,
                "BetslipLine": "Yes",
                "DecimalOdds": "1.48",
                "DisplayOdds": "1.48",
                "IsEarlyPayout": false
              }
            ],
            "AdditionalInfo": {
              "0ML864430244147548160H": {
                "MarketTypeId": "ML0"
              },
              "0QA864430244147548195Q1714Q0": {
                "MarketTypeId": "QA158"
              },
              "0VS0ML864430244147548160H|0QA864430244147548195Q1714Q0": {
                "MarketTypeId": "RVMML0|QA158"
              }
            },
            "NonActiveSelections": {}
          },
          "meta": {},
          "created_at": "2026-07-16T11:58:08.450Z",
          "language": "en",
          "content": {
            "text": "Valerenga should boss this at home, their front line creating better chances all season. Aalesunds still carry enough threat to nick one, but Valerenga’s stronger engine, higher tempo and tighter back line point to a home win with both teams to score."
          },
          "model": {
            "id": 164,
            "name": "BB - all leagues",
            "color": "#ed145b"
          },
          "selections": [
            {
              "odd_id": 99053176,
              "fixture_id": 1080495,
              "market_id": 1,
              "bookmaker_id": 7,
              "is_live": 0,
              "label_id": 1,
              "value": 1.69,
              "handicap": 0,
              "line": 0,
              "last_update": 1784201658,
              "suspend": 0,
              "sp": 1.67,
              "home_score": 0,
              "away_score": 0,
              "status": 0,
              "raw": {
                "event_id": "864430242864062464",
                "market_id": "0ML864430244147548160",
                "selection_id": "0ML864430244147548160H",
                "is_betbuilder": true,
                "market_type_id": "ML0",
                "selection_type_id": 1
              },
              "outcome": "Pending",
              "market_description": "Predict the result of the match",
              "market_name": "Match Winner",
              "label_name": "Valerenga IF"
            },
            {
              "odd_id": 99053173,
              "fixture_id": 1080495,
              "market_id": 53,
              "bookmaker_id": 7,
              "is_live": 0,
              "label_id": 1,
              "value": 1.48,
              "handicap": 0,
              "line": 0,
              "last_update": 1783945606,
              "suspend": 0,
              "sp": 1.51,
              "home_score": 0,
              "away_score": 0,
              "status": 0,
              "raw": {
                "event_id": "864430242864062464",
                "market_id": "0QA864430244147548195",
                "selection_id": "0QA864430244147548195Q1714Q0",
                "is_betbuilder": true,
                "market_type_id": "QA158",
                "selection_type_id": 0
              },
              "outcome": "Pending",
              "market_description": "Will both teams score in the match",
              "market_name": "Both Teams to Score",
              "label_name": "Yes"
            }
          ]
        }
      ]
    }
  ]
}
```

---

## GET /bet-builders/:id

Returns a single bet builder by id, as a flat object (not nested under a fixture). Includes expired/suspended bet builders regardless of the `expired`/`suspend` filters.

```
GET /bet-builders/501?language=en
```

---

## Response Codes

**200 OK** — see example above.

**403 Forbidden** — missing or invalid authentication

```json
{ "error": "Unauthorized" }
```

**404 Not Found** — `GET /bet-builders/:id` only, when the id doesn't exist or doesn't belong to you

```json
{ "error": "Bet builder not found" }
```

**500 Internal Server Error**

```json
{ "error": "Internal server error" }
```
