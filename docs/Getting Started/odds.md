---
title: Odds
excerpt: >-
  This documentation provides a comprehensive breakdown of the Odd Object, which
  is the core entity used to track pricing, settlement, and market data for any
  betting selection within the SSTrader API.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Example Odd Object

```json
{
  "odd_id": 72919503,
  "fixture_id": 946305,
  "market_id": 203,
  "bookmaker_id": 2,
  "is_live": 0,
  "label_id": 2,
  "value": 2.1,
  "handicap": 3.5,
  "line": 3.5,
  "last_update": 1768081284,
  "suspend": 1,
  "sp": 1.8,
  "home_score": 0,
  "away_score": 0,
  "status": -1,
  "raw": {},
  "player_id": null,
  "outcome": "Lost",
  "market_description": "Total number of cards in the match",
  "market_name": "Asian Total Cards",
  "label_name": "Over 3.5"
}
```

## Field Definitions

Here are the field definitions for the **Odd Object** in Markdown format, organized by category for clarity.

### Core Identifiers

| Field          | Type           | Description                                                                                                                                                           |
| :------------- | :------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `odd_id`       | Integer        | Unique identifier for this specific odd record.                                                                                                                       |
| `fixture_id`   | Integer        | Identifier of the match (fixture) this odd belongs to.                                                                                                                |
| `market_id`    | Integer        | The category of the bet (e.g., `1` for Match Winner, `22` for Next Goal).                                                                                             |
| `bookmaker_id` | Integer        | The ID of the bookmaker providing this price.                                                                                                                         |
| `label_id`     | Integer        | Internal identifier for the specific selection type within a market.                                                                                                  |
| `player_id`     | Integer / null | Nullable. The unique identifier for a player. This is only populated when the odd is related to a specific player performance (e.g., First Goalscorer, Player Shots). |

### Pricing & Lines

| Field      | Type    | Description                                                                      |
| :--------- | :------ | :------------------------------------------------------------------------------- |
| `value`    | Float   | The current decimal odds (price).                                                |
| `line`     | Float   | The numerical threshold for the bet (e.g., `2.5` in an Over/Under market).       |
| `handicap` | Float   | The point spread or goal handicap (if applicable).                               |
| `sp`       | Float   | Starting Price. The opening odds or the first recorded value for the event.      |
| `suspend`  | Integer | Boolean flag (`0` or `1`). If `1`, the odd is currently locked and not tradable. |

### Match State

| Field         | Type    | Description                                                                                                                                                                                                 |
| :------------ | :------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `is_live`     | Integer | Indicates whether the odds are from a live, in-play event (`1`) or a pre-match event (`0`).                                                                                                                 |
| `home_score`  | Integer | The home team's current goal count at the time the odd was generated. **Note:** This is specific to Asian Handicap and Asian Total markets, representing the "base score" from which the bet is calculated. |
| `away_score`  | Integer | The away team's current goal count at the time the odd was generated. **Note:** This is specific to Asian Handicap and Asian Total markets, representing the "base score" from which the bet is calculated. |
| `last_update` | Integer | Unix timestamp indicating the last time this odd was modified (including changes to price, suspend status, or settlement status).                                                                           |

### Localized Display Fields (Enriched)

| Field                | Type    | Description                                                       |
| :------------------- | :------ | :---------------------------------------------------------------- |
| `status`             | Integer | The numerical result code of the bet.                             |
| `outcome`            | String  | Localized name of the status (e.g., "Won", "Half Lost", "Void").  |
| `market_name`        | String  | Localized name of the betting market (e.g., "Asian Total Cards"). |
| `market_description` | String  | A brief localized explanation of the market rules.                |
| `label_name`          | String  | The human-readable name of the specific pick (e.g., "Over 3.5").  |

### Settlement Status Codes (`status`)

The `status` integer indicates the result of the selection. The `outcome` string provides the localized translation of this code.

| Status |    Outcome    | Description                                             |
| :----- | :-----------: | :------------------------------------------------------ |
| **2**  |  **Half Won** | Half of the bet won; half was returned as a push.       |
| **1**  |    **Won**    | The selection was successful.                           |
| **0**  |  **Pending**  | The event is ongoing or settlement is not yet complete. |
| **-1** |    **Lost**   | The selection was unsuccessful.                         |
| **-2** | **Half Lost** | Half of the stake was lost; half was returned.          |
| **-3** |    **Void**   | The bet was cancelled and the stake was returned.       |

***

## Odds Entity Filters

When querying fixtures or odds, you can refine the results by applying specific keys to the `odds` entity scope. This allows you to limit the data returned to specific bookmakers, markets, or match states.

### Available Filter Keys for `odds`

| Key          | Type          | Description                                                           |
| :----------- | :------------ | :-------------------------------------------------------------------- |
| `bookmakers` | Integer(s)    | Filter by one or more Bookmaker IDs.                                  |
| `markets`    | Integer(s)    | Filter by specific Market IDs (e.g., `1` for Match Winner).           |
| `is_live`    | Boolean (0/1) | `1` returns only In-Play odds; `0` returns only Pre-match odds.       |
| `suspend`    | Boolean (0/1) | `1` returns only suspended/locked odds; `0` returns only active odds. |

### Practical Examples

**1. Filter by specific Bookmaker and Markets**
To get odds only for Bookmaker **2** and only for **Match Winner (1)** or **Over/Under (3)**:
`?filter[odds]=bookmakers:2;markets:1,3`

**2. Filter for Active Pre-match Odds**
To get odds that are **not live** and are **not suspended**:
`?filter[odds]=is_live:0;suspend:0`

**3. Comprehensive Selection**
To get odds for **Bookmaker 2**, within **Market 203**, that are currently **Live** and **Active**:
`?filter[odds]=bookmakers:2;markets:203;is_live:1;suspend:0`

### Summary of Delimiter Logic for Odds

- **OR Logic (Comma):** `markets:1,3` means "Market 1 **OR** Market 3".
- **AND Logic (Semicolon):** `is_live:1;suspend:0` means "Must be Live **AND** Must be Not Suspended".

***

<br />
