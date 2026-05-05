---
title: Regions
excerpt: >-
  Retrieve a hierarchical list of global regions, countries, and sports leagues
  to power your navigation menus and match filters.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Regions Endpoint Documentation

Use the Regions endpoint to retrieve a comprehensive, hierarchical list of all available betting areas. This endpoint maps the relationships between global regions, specific countries, and their respective sports leagues, making it the primary tool for building navigation menus or league filters in your application.

To see how these regions are structured, check out the [Regions API Reference](/reference/getregions).

***

## Query Parameters

| Parameter | Type | Default  | Description                                                                                          |
| --------- | ---- | -------- | ---------------------------------------------------------------------------------------------------- |
| status    | enum | upcoming | Filter leagues by activity. Use upcoming for active/near-future events or all for the full database. |
| language  | enum | en       | Returns localized names. Supported: en (English), es (Spanish), bg (Bulgarian).                      |
| league_level | string |   | Comma-separated list to filter by league tier. Use 1 for top-flight, 2 for second division, etc. Default is all levels.     |
| country_id | string |    | Comma-separated list of country IDs to filter leagues by specific nations. Default is all countries. |

***

## Response Schema

The response returns an array of objects, where each entry represents a unique "League Pathway" (Region > Country > League).

## Field Definitions

| Field   | Type    | Description                                                       |
| ------- | ------- | ----------------------------------------------------------------- |
| region  | object  | The broad geographical area (e.g., Europe, Asia, International).  |
| country | object  | The specific nation, including the ISO alpha3 code.               |
| league  | object  | The competition details, including the technical level.           |
| season  | object  | The current season details, name, starting_at and ending_at.           |
| sport   | object  | The type of sport (e.g., Football).                               |
| count   | integer | Total number of available fixtures for this league.               |
| live    | integer | Number of fixtures currently in-play.                             |
| popular | boolean | Flag for high-traffic leagues (e.g., Premier League, Bundesliga). |
| level   | integer | The tier of the league (1 = Top Flight, 5+ = Amateur/Regional).   |

***

## Example JSON Response

```json
{
  "regions": [
    {
      "region": {
        "id": 2,
        "name": "EUROPE - Main"
      },
      "country": {
        "id": 4,
        "name": "Germany",
        "alpha3": "DEU"
      },
      "league": {
        "id": 1310,
        "name": "Reg. Cup Rheinland",
        "level": 5
      },
      "season": {
        "id": 17760,
        "name": "2026",
        "starting_at": "2026-03-13",
        "ending_at": "2026-11-30",
        "is_current": 1
      },
      "sport": {
        "id": 1,
        "name": "Football"
      },
      "count": 12,
      "live": 1,
      "popular": false,
      "level": 5
    }
  ]
}
```

***

## Integration Tip

Use the level and popular fields to prioritize data display. Most users prefer to see Popular leagues or Level 1 competitions at the top of their search results.