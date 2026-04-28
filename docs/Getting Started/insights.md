---
title: Insights
excerpt: >-
  Access a stream of AI-generated betting recommendations that combine real-time
  market data with natural language analysis to identify high-probability
  outcomes.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Insights Endpoint Documentation (/insights)

The Insights endpoint returns AI-generated analysis and betting recommendations. Each insight pairs a natural language explanation with a specific market selection (the `bet`).

------------------------------
## 🛠️ Query Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| market_id | string | | Comma-separated list of Market IDs (e.g., 1,3). |
| status | string | See below | Filter by match Status. |
| is_live | integer | 0 | Use 1 to filter only for events currently in-play. |
| expired | integer | 0 | Use 1 to include outdated insights (useful for backend sync). |
| league_id | string | | Comma-separated list of League IDs. |
| fixture_id | integer | | Restrict results to a specific single fixture. |
| language | string | en | Language code for the generated content. |
| value_from | number | | Minimum decimal odd value. |
| value_to | number | | Maximum decimal odd value. |
| limit | integer | 100 | Max results (1–1000). |

## 📋 Default Statuses

If no status is provided, the API filters by these active match states:
INPLAY_1ST_HALF, INPLAY_2ND_HALF, INPLAY_ET, INPLAY_PENALTIES, HT, EXTRA_TIME_BREAK, PEN_BREAK, NOT_STARTED.

------------------------------
## 📦 Response Schema## Insight Object

| Field | Type | Description |
|---|---|---|
| id | integer | Unique internal identifier for the insight. |
| market_id | integer | The ID of the market the insight refers to. |
| value | number | The current decimal odd value for the recommendation. |
| sp | number | Starting Price: The odd value at the exact time the insight was generated. |
| created_at | string | ISO 8601 timestamp of generation. |
| content | object | Contains the text field with the AI analysis. |
| model | object | Metadata about the AI model used (id, name, color). |
| bet | object | The recommended selection. Follows the Odd Schema. |