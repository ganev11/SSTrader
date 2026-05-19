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

## Error Codes

When making a request, a code response will always be returned. The following table lists all possible HTTP response codes for any request made to the API:

| Code    | Description                                                                                                                        |
| :------ | :--------------------------------------------------------------------------------------------------------------------------------- |
| **200** | **OK**: The request was successful and the data has been returned.                                                                 |
| **201** | **Created**: The request has been fulfilled and has resulted in one or more new resources being created. |
| **400** | **Bad Request**: The request was invalid or malformed. See the response body for specific validation errors.                       |
| **401** | **Unauthorized**: Missing or invalid authentication credentials.                                                                   |
| **403** | **Forbidden**: Your current subscription plan does not have permission to access this resource.                                    |
| **409** | **Conflict**: Resource creation request fails because it conflicts with the current state of the server, such as a duplicate unique constraint
| **429** | **Too Many Requests**: You have reached your hourly API rate limit. Refer to the `meta` section in your headers for limit details. |
| **500** | **Internal Server Error**: A server-side error occurred. If this persists, please contact our support team.                        |

Check our **status page** to see if we are aware of any possible issues.