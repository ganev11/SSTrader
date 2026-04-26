---
title: States
excerpt: Fixture status definitions
deprecated: false
hidden: false
metadata:
  robots: index
---
Fixtures transition through various **statuses** — including scheduled, finished, or cancelled — identified by unique state codes (developer_name). These states allow for real-time tracking of match progress and enable data filtering via the API.

| State              | Description                                         |
| ------------------ | --------------------------------------------------- |
| `ABANDONED`        | Match abandoned and will resume later.              |
| `AWAITING_UPDATES` | Awaiting data or connectivity updates.              |
| `AWARDED`          | Result awarded by administrative decision.          |
| `BREAK`            | Regular time finished and awaiting extra time.      |
| `CANCELLED`        | Match permanently cancelled.                        |
| `DELETED`          | Fixture marked deleted.                             |
| `DELAYED`          | Kick-off has been delayed.                          |
| `EXTRA_TIME_BREAK` | Short break between extra-time halves.              |
| `FT`               | Match ended after 90 minutes.                       |
| `FT_AET`           | Match ended after extra time (120 minutes).         |
| `FT_PEN`           | Match finished after a penalty shootout.            |
| `HT`               | Match is at half-time.                              |
| `INPLAY_1ST_HALF`  | First half in progress.                             |
| `INPLAY_2ND_HALF`  | Second half in progress.                            |
| `INPLAY_ET`        | Extra time is in progress.                          |
| `INPLAY_PENALTIES` | Penalty shootout is in progress.                    |
| `INTERRUPTED`      | Match temporarily interrupted (e.g., weather).      |
| `NOT_STARTED`      | Fixture hasn't started.                             |
| `PENDING`          | Fixture pending data or verification.               |
| `PEN_BREAK`        | Waiting for the penalty shootout to start.          |
| `POSTPONED`        | Match postponed to a later date.                    |
| `SUSPENDED`        | Match suspended and will resume later.              |
| `TBA`              | Date and time not yet confirmed.                    |
| `WALKOVER`         | Victory awarded due to opponent absence (walkover). |

<br />