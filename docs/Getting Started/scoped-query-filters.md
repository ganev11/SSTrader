---
title: Scoped Query Filters
excerpt: >-
  This system uses URL query parameters to pass structured filters to the
  backend. It groups criteria by entity (like `odds`) to ensure your queries
  remain organized and easy to parse.
deprecated: false
hidden: false
metadata:
  robots: index
---
## 1. URL Pattern

To apply filters, use the following syntax:
`?filter[entity]=key:value1,value2;key2:value`

**Delimiter Rules:**

- `:`**&#x20;(Colon)**: Connects a filter key to its values.
- `,`**&#x20;(Comma)**: Separates multiple values for the same key (Logical `OR`).
- `;`**&#x20;(Semicolon)**: Separates different filter keys (Logical `AND`).

***

## 2. Detailed Example

**Query:**
`?filter[odds]=bookmakers:2;markets:1,2,3,6,203;is_live:0`

**Breakdown:**

- **Entity**: `odds`
- **Filter 1**: `bookmakers` is set to `2`.
- **Filter 2**: `markets` includes IDs `1`, `2`, `3`, `6`, and `203`.
- **Filter 3**: `is_live` is set to `0` (False).

**Logical Interpretation:**

- **Odds** must satisfy all of the following:
  - Belong to **bookmaker 2**.
  - Be in **markets 1, 2, 3, 6, or 203**.
  - Be **not live (inPlay)** (`is_live=0`).

<br />