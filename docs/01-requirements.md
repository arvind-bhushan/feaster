# Feaster — Requirements & Scope (Day 1)

The first step of designing any backend: decide **what requests it must answer** before writing a line of code. Two kinds of requirements.

## Functional requirements (what it does)
| # | Actor | Capability |
|---|---|---|
| 1 | Customer | Sign up / log in |
| 2 | Customer | Browse the list of restaurants |
| 3 | Customer | View a restaurant's menu |
| 4 | Customer | Place an order (pay, safely) |
| 5 | Customer | Track order status in real time |
| 6 | Restaurant | See incoming orders & update their status |
| 7 | System | Assign a delivery partner after an order is placed |

## Non-functional requirements (how well it does it)
| Quality | What it means for Feaster | How we'll achieve it |
|---|---|---|
| **Fast** | Menus/restaurants load instantly | Redis caching of hot reads |
| **Correct under concurrency** | No double-charge, no overselling | Postgres transactions + idempotency |
| **Resilient** | A slow delivery step never blocks/loses an order | Event-driven queue (NATS) + worker |
| **Observable** | We can see latency, throughput, errors | Prometheus metrics + structured logs |
| **Secure** | Only logged-in users act; passwords safe | JWT auth + bcrypt hashing |

## Core entities (the main "nouns" — first draft, refined on Day 2)
`User` · `Restaurant` · `MenuItem` · `Order` · `OrderItem` · `DeliveryPartner`

## Out of scope (kept deliberately focused)
Payments integration (simulated), maps/routing, ratings/reviews, promotions. Feaster focuses on the **core ordering + delivery engine** — the part with the interesting engineering.

---
*Next (Day 2): turn these requirements into concrete API endpoints + a data model.*
