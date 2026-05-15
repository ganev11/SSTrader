---
title: Pressure Index
deprecated: false
hidden: false
metadata:
  robots: index
---
The Pressure Index is a dynamic, real-time metric that quantifies momentum, territorial control, and the intensity of a team’s physical presence. It serves as a barometer for both offensive dominance and defensive disruption.

## Game Flow Index

A composite model for estimating match intensity and tactical dominance, derived from SSTrader's per-team Pressure Index.

**Input signals**

### SSTrader Pressure Index (per team)

| Range | Label | Description |
|---|---|---|
| `0 – 10` | **No pressure** | Team is primarily defending, sitting deep |
| `10 – 30` | **Balanced** | Evenly contested, neither side dominates |
| `30 – 50` | **Pressing** | Team is actively pressing and attacking |
| `50 – 60` | **High pressure** | Sustained aggressive pressure on opponent |
| `60 – 80` | **Max attack** | Maximum attacking intensity, all-in |

---

**Derived calculations**

### Two-axis framework

- `Total Sum = Home Index + Away Index` — Determines overall match tempo  
- `Delta = |Home Index − Away Index|` — Determines tactical dominance

**Axis 1 — Tempo**

| Sum (Home + Away) | Tempo tag | Expected match character |
|---|---|---|
| `0 – 20` | **Dead Game** | Both teams passive, low energy, slow transitions |
| `20 – 40` | **Balanced Tempo** | Moderate pace, structured, few open chances |
| `40 – 70` | **Open Game** | Good flow, both sides creating, end-product likely |
| `70 – 100` | **High Intensity** | Fast, aggressive, chances at both ends |
| `100 – 140` | **All-Out War** | Max pressure from both sides, chaotic, high volume |

**Axis 2 — Control**

| Delta `|H − A|` | Control tag | Interpretation |
|---|---|---|
| `0 – 10` | **Neutral** | No dominant side, genuinely contested |
| `10 – 25` | **Slight edge** | One team has a marginal territorial advantage |
| `25 – 45` | **Clear control** | Structural superiority for one team |
| `45+` | **Total dominance** | One team completely imposing their game |

---

**Combined archetypes**

| Archetype | Condition | Description | Betting signal |
|---|---|---|---|
| **Trench War** | `Sum < 20` | Both teams defending. Minimal transitions, very few shots. | Goals under, corners under, 0–0 HT |
| **One-Sided Push** | `Sum 20–50`, `Delta > 25` | One team attacking relentlessly, the other sitting back and absorbing. | AH favor attacker, corners over, 1X2 lean |
| **Tactical Battle** | `Sum 20–50`, `Delta < 15` | Balanced, mid-block chess match. Few clear-cut chances but quality over quantity. | BTTS, 1–1 or 1–0, moderate corners |
| **Open Game** | `Sum 50–80` | Both teams pressing and creating. Open spaces, good tempo, multiple chances expected. | Goals over, BTTS yes, corners over |
| **End-to-End** | `Sum 80+` | Maximum bilateral pressure. Chaos, high volume, both teams vulnerable defensively. | Goals over, corners over, cards likely |

---

**Examples**

| Teams | Sum · Delta | Archetype |
|---:|---|---|
| Home 8 · Away 6 | Sum 14 · Delta 2 | **Trench War** |
| Home 45 · Away 5 | Sum 50 · Delta 40 | **One-Sided Push** |
| Home 28 · Away 25 | Sum 53 · Delta 3 | **Open Game** |
| Home 38 · Away 18 | Sum 56 · Delta 20 | **Open Game + Slight edge** |
| Home 62 · Away 55 | Sum 117 · Delta 7 | **End-to-End** |
| Home 22 · Away 20 | Sum 42 · Delta 2 | **Tactical Battle** |

---
