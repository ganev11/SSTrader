---
title: Types
excerpt: Here you can find all types SSTrader included in the API.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Get Types
Use this endpoint `/types` to retrieve a list of available Types supported by the SSTrader Football API. Types are used to categorize data points like periods, positions, and statistics.

To see how these types are structured, check out the [Types API Reference](/reference/gettypes).

## Query Parameters

| Parameter | Type | Description |
|---|---|---|
| `developer_types` | string | Optional. Filter types by their domain-specific group (e.g., period). You can provide a single value or a comma-separated list (e.g., period,statistics). |

## Endpoint Behavior

* Filtering: If the `developer_types` parameter is provided, the API returns only the types belonging to those specific groups.
* Response: Returns an array of type objects.

## Response Schema
The response is a JSON array of objects, each containing:

* type_id integer: The unique ID for the type.
* developer_name string: The unique, constant-style identifier used for programming logic.
* developer_description string: A brief explanation of the type's purpose.
* developer_type string: The category or domain this type belongs to (e.g., period).

## Example Response

```json
{
  "types": [
    {
      "type_id": 1,
      "developer_name": "1ST_HALF",
      "developer_description": "First half of the match",
      "developer_type": "period"
    },
    {
      "type_id": 2,
      "developer_name": "2ND_HALF",
      "developer_description": "Second half of the match",
      "developer_type": "period"
    }
  ]
}
```