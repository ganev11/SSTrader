---
title: Pagination
deprecated: false
hidden: false
metadata:
  robots: index
---
Responses from endpoints that return large datasets use pagination to keep requests fast and manageable. Instead of returning all results at once, data is split into pages of up to **50 results** per page.

***

## Default Behavior

By default, every paginated request returns:

| Parameter  | Default | Description                         |
| ---------- | ------- | ----------------------------------- |
| `page`     | `1`     | The current page number             |
| `per_page` | `50`    | Results returned per page (max: 50) |
| `order`    | `asc`   | Sort order of results               |

***

## Pagination Response Object

Every paginated response includes a `pagination` object at the end:

```json
{
  "pagination": {
      "page": 1,
      "per_page": 50,
      "has_more": true,
      "order": "asc"
  }
}
```

| Field      | Type    | Description                                                          |
| ---------- | ------- | -------------------------------------------------------------------- |
| `page`     | integer | The current page number                                              |
| `per_page` | integer | Number of results returned on this page                              |
| `has_more` | boolean | `true` if more pages are available, `false` if this is the last page |
| `order`    | string  | Sort order — `asc` or `desc`                                         |

> `has_more` is determined by whether the number of results returned equals `per_page`. If you receive fewer results than `per_page`, you've reached the last page.

***

## Navigating Pages

To move to the next page, add `&page=2` to your request:

```
GET /fixtures?page=2
```

Keep incrementing `page` and check `has_more` after each response. Once `has_more` is `false`, you've retrieved all available data.

***

## Controlling Results Per Page

You can request between **1 and 50** results per page using the `per_page` parameter:

```
GET /fixtures?per_page=25&page=1
```

The maximum value is **50**. Note that `per_page` only affects the base entity — included relations are not paginated.

***

> **Rate limiting:** Each page request counts as a separate API call. Fetching pages 1 through 5 counts as 5 calls against your rate limit.

<br />
