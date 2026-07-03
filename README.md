# Feaster

A food-ordering & delivery platform backend, built in **Go**. Feaster handles the full lifecycle of an order — from browsing restaurants to safe checkout to real-time delivery tracking — with the reliability, caching, and observability of a production system.

> Status: 🟡 In active development. Built incrementally with documented design decisions.

## The problem
Placing a food order touches **money** and **inventory**, so the hard engineering problems are:
- **Concurrency-safe checkout** — never double-charge a customer, never oversell a dish, even under simultaneous orders.
- **Speed** — restaurant and menu data is read constantly; it must be cached, not hit the database every time.
- **Real-time tracking** — customers expect live order status without refreshing.
- **Resilience** — a slow delivery-assignment step must never block or lose an order.

Feaster is built to solve each of these the way a real platform would.

## Features
- 👤 **Accounts & auth** — sign up / log in (JWT-based), for both customers and restaurants
- 🍽️ **Discovery** — browse restaurants, view menus (cached for speed)
- 🛒 **Checkout** — place an order inside a database transaction (atomic, no double-charge, stock-respecting)
- 📦 **Order tracking** — live status over WebSockets: `placed → preparing → out for delivery → delivered`
- 🏪 **Restaurant view** — restaurants receive and update incoming orders
- 🚚 **Delivery assignment** — event-driven: placing an order emits an event; a background worker assigns delivery asynchronously

## Non-functional goals
Fast (Redis caching) · reliable (transactions, retries, idempotency) · observable (Prometheus metrics + structured logs) · concurrent (handles many simultaneous orders).

## Architecture
```
   Customer app / Restaurant app          (clients)
              │  HTTP / WebSocket
              ▼
   ┌────────────────────────────┐
   │        Feaster API         │   Go · clean architecture
   │   handler → service → repo │
   │  auth · restaurants · menu │
   │  orders · live tracking    │
   └───────┬───────────┬────────┘
           │           │
       PostgreSQL     Redis            (source of truth + cache)
           │
        NATS queue ─► delivery worker  (async delivery assignment)
           │
      gRPC ↔ internal services · Prometheus metrics
```

## Tech stack
**Go** · chi (routing) · PostgreSQL (+ pgvector for dish search) · Redis · NATS · gRPC · WebSockets · Prometheus · Docker · GitHub Actions (CI)

## Getting started
```bash
docker compose up      # spins up API + Postgres + Redis  (coming soon)
```

## Documentation
- [Requirements & scope](docs/01-requirements.md)
- [Design decisions & trade-offs](DECISIONS.md)

---
*A ground-up exploration of production backend engineering: clean architecture, concurrency-safe data access, event-driven design, and observability.*
