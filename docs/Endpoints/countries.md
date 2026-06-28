---
title: Countries
excerpt: Returns a list of all countries.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Query Parameters

| Parameter  | Type   | Required | Default | Description                        |
| ---------- | ------ | -------- | ------- | ---------------------------------- |
| `language` | string | No       | `en`    | Language code for the country name |

***

## Response

**200 OK**

```json
{
  "countries": [
    { "id": 1, "name": "England", "alpha3": "ENG" },
    { "id": 2, "name": "Spain", "alpha3": "ESP" }
  ]
}
```

**403 Forbidden** — missing or insufficient role

```json
{ "error": "Unauthorized" }
```

**500 Internal Server Error**

```json
{ "error": "Internal Server Error" }
```

***

## Examples

### All countries (default language)

```
GET /countries
```

### All countries, Spanish names

```
GET /countries?language=es
```

<br />
