---
title: Odd Movement
excerpt: >-
  A historical log of price fluctuations and market status updates used to
  analyze market trends and perform accurate backtesting.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Odd Movement Schema Documentation

The `movement` array tracks the historical fluctuations of a specific odd over time. This chronological log is essential for backtesting strategies, identifying "steam" (sharp money moves), and analyzing market closing lines.

> Availability: This data is only included in the response when explicitly requested using the query parameter include=movement.

------------------------------

## Field Definitions

| Field | Type | Description |
|---|---|---|
| odd_id | integer | The unique identifier of the odd being tracked. |
| value | number | The decimal price of the odd at that specific point in time. |
| suspend | integer | Status flag: 1 if the market was locked/suspended, 0 if active. |
| last_update | integer | Unix timestamp representing when this specific price change occurred. |

------------------------------

## Backtesting Example

By sorting the movement array by last_update, you can visualize the market trend:

* Opening Odd: 1.403 (1776776759)
* Drift: The price increased over time to 1.50.
* Closing State: The final entry shows the market suspended (suspend: 1) at 1.50, likely just before the event started or during a major match incident.


## Example JSON response

```json
{
   "movement": [
        {
          "odd_id": 88782664,
          "value": 1.403,
          "suspend": 0,
          "last_update": 1776776759
        }
   ]
}
```