---
title: Participants
excerpt: Details about a specific team or competitor involved in the match.
deprecated: false
hidden: false
metadata:
  robots: index
---
## 📄 Example JSON

This JSON object represents a typical response for a Participants following your schema.

```json
"participants": [
        {
          "id": 1,
          "name": "Manchester United",
          "position": 5,
          "location": "home"
        },
        {
          "id": 2,
          "name": "Liverpool",
          "position": 2,
          "location": "away"
        }
]
```
***

## Field Descriptions

The table below breaks down the requirements and constraints for each field in the Participant schema.

| Field    | Type    | Description      | 
| -------- | ------- | ------------------------------------------------- |
| id       | integer | Unique identifier for the participant.            |
| name     | string  | Localized name of the team or competitor.         |
| position | integer | The team's current standing/rank in their league.rank. |
| location | string  | Indicates if the team is hosting or visiting.      |

***


<br />