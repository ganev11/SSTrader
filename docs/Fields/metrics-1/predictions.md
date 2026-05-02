---
title: Predictions
excerpt: Prediction metrics
deprecated: false
hidden: false
metadata:
  robots: index
---
## Overview — Prediction metrics
These metrics are structured numeric signals attached to a fixture's `metrics` array that
describe predicted match behavior (for example: goals, corners, yellow cards, shot scores,
and pressure).
Use these metrics in the frontend, filters, analytics dashboards, or betting pipelines to
display prediction scores, highlight advantages, show confidence, map predictions to market
lines, and provide short explanatory context for users.
When present, the optional `meta`
object supplies human-friendly details such as reasons, the derived betting line, market and
label ids, or concise notes useful for UI tooltips and downstream workflows.