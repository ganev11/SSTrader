---
title: Types
excerpt: Here you can find all types SSTrader included in the API.
deprecated: false
hidden: false
metadata:
  robots: index
---
## Types Reference

In SSTrader, Types provide a standardized way to categorize and identify data points across the Football API. Each type serves as a unique identifier for specific metrics, periods, or entity roles.

## Schema Definition
Every type object returned by the API includes the following core attributes:

* type_id: integer
The unique numeric identifier for the specific type.
* developer_name: string (Unique)
A constant-style, unique string identifier (e.g., PENALTIES). This is the most reliable field for use in your application logic.
* developer_description: string
A short, human-readable explanation of what this type represents and how it should be interpreted.
* developer_type: string
The domain-specific grouping for the type. This categorizes the type into a functional area, such as period, statistic, or event.

## Implementation Example
When processing statistics or match data, you should map these types to your internal models using the developer_name (or type_id) to ensure your integration remains robust even if display names are updated.
