---
title: Authentication
deprecated: false
hidden: false
metadata:
  robots: index
---
<br />

This API is secured using an **API Key** provided by **SSTrader**.

Your API key must be included in **every request**. Requests without a valid key will be rejected.

***

## API Key Formats (Supported)

You can authenticate using either:

1. **Bearer Token** (recommended)
2. **Query Parameter** (`api_key`)

***

## Bearer Token (Recommended)

Send the API key in the `Authorization` header as a Bearer token.

### Request Header

```curl
Authorization: Bearer YOUR_API_KEY
```

## Query Parameter (api\_key)

You can also pass the API key as a URL query parameter.
