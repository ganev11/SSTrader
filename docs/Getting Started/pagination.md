---
title: Pagination
deprecated: false
hidden: false
metadata:
  robots: index
---
Endpoints that return large datasets are paginated to keep requests fast and manageable. Instead of returning every result at once, the data is split into pages, and each response tells you where you are and whether more pages remain.

***

## Request Parameters

| Parameter  | Default        | Description                                                                 |
| ---------- | -------------- | --------------------------------------------------------------------------- |
| `page`     | `1`            | The page to return, starting at `1`                                         |
| `per_page` | endpoint-based | Results per page. The default and maximum are listed in each endpoint's reference |

Some endpoints also accept an `order` parameter (`asc` or `desc`). Where an endpoint does not, its sort order is fixed, and the `order` field in the response tells you what it is.

***

## Pagination Response Object

Every paginated response includes a `pagination` object next to the results:

```json
{
  "pagination": {
    "page": 1,
    "per_page": 25,
    "total": 132,
    "total_pages": 6,
    "has_more": true,
    "order": "desc"
  }
}
```

| Field         | Type    | Description                                                                       |
| ------------- | ------- | --------------------------------------------------------------------------------- |
| `page`        | integer | The current page number                                                           |
| `per_page`    | integer | The page size applied to this request                                             |
| `total`       | integer | Number of results matching your request, across all pages                         |
| `total_pages` | integer | `ceil(total / per_page)`; `0` when nothing matches                                |
| `has_more`    | boolean | `true` only when at least one more result exists after this page                  |
| `order`       | string  | Sort direction of the results — `asc` or `desc`                                   |

Notes:

- **`has_more` is exact.** It is `true` only when the next page has at least one result, so you never need to request an empty page to find the end.
- **`total` is counted at the time of the request.** Live data (fixtures starting, insights being published or expiring) can change it between pages, and a result can move across a page boundary. Narrow your filters when you need a complete, consistent snapshot.
- **Every page is full except the last.**
- **`order` is always present**, also on endpoints where you cannot set it.
- Requesting a page past the last one returns an empty result list with `has_more: false`; it is not an error.

***

## Navigating Pages

To move to the next page, increase `page`:

```
GET /football/fixtures?page=2
```

Keep requesting `page + 1` while `has_more` is `true`. Once it is `false`, you have retrieved all available data. Alternatively, use `total_pages` to know up front how many requests you will need.

***

## Controlling Results Per Page

Set the page size with `per_page`:

```
GET /football/fixtures?per_page=50&page=1
```

`per_page` must be between `1` and the endpoint's maximum. `per_page` only applies to the base entity — included relations (for example `include=odds`) are not paginated.

***

> **Rate limiting:** Each page request counts as a separate API call. Fetching pages 1 through 5 counts as 5 calls against your rate limit.
