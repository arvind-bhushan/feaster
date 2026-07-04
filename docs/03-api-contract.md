# Feaster — API Contract: Request/Response Shapes (Day 3)

The exact JSON each endpoint accepts and returns — the contract the Go structs (Day 5–6) will match. Every endpoint lists its success code and its failure codes (correct HTTP status codes are part of the design).

Convention: all bodies are `application/json`. Authenticated endpoints require `Authorization: Bearer <jwt>`.

---
## Auth

### POST /signup → 201 Created
```json
// request
{ "name": "Arvind", "email": "arvind@x.com", "password": "secret123", "role": "customer" }
// response 201
{ "user_id": 1, "name": "Arvind", "email": "arvind@x.com", "role": "customer" }
```
Errors: `400` invalid input · `409` email already registered

### POST /login → 200 OK
```json
// request
{ "email": "arvind@x.com", "password": "secret123" }
// response 200
{ "token": "eyJhbG...", "expires_in": 3600 }
```
Errors: `400` missing fields · `401` wrong email/password

---
## Restaurants & Menu (reads)

### GET /restaurants → 200 OK
```json
{ "restaurants": [
  { "id": 42, "name": "Paradise Biryani", "cuisine": "Biryani", "is_open": true },
  { "id": 43, "name": "Domino's", "cuisine": "Pizza", "is_open": false }
]}
```

### GET /restaurants/{id} → 200 OK
```json
{ "id": 42, "name": "Paradise Biryani", "cuisine": "Biryani", "is_open": true, "rating": 4.3 }
```
Errors: `404` no such restaurant

### GET /restaurants/{id}/menu → 200 OK
```json
{ "restaurant_id": 42, "items": [
  { "id": 101, "name": "Chicken Biryani", "price": 320, "available": true },
  { "id": 105, "name": "Raita", "price": 40, "available": true }
]}
```
Errors: `404` no such restaurant

---
## Orders (customer)

### POST /orders → 201 Created  (auth required)
```json
// request
{ "restaurant_id": 42, "items": [
  { "menu_item_id": 101, "quantity": 2 },
  { "menu_item_id": 105, "quantity": 1 }
]}
// response 201
{ "order_id": 5001, "status": "placed", "total": 680, "created_at": "2026-07-03T18:20:00Z" }
```
Errors: `400` empty items/bad quantity · `401` no token · `404` restaurant/dish missing · `409` dish out of stock

> ⚠️ POST is **not idempotent** — retrying "place order" would create duplicate orders/charges. This is why checkout needs idempotency protection (Day 29/51).

### GET /orders/{id} → 200 OK  (auth required)
```json
{ "order_id": 5001, "restaurant_id": 42, "status": "preparing", "total": 680,
  "items": [ { "name": "Chicken Biryani", "quantity": 2, "price": 320 } ],
  "created_at": "2026-07-03T18:20:00Z" }
```
Errors: `401` no token · `403` not your order · `404` no such order

### GET /orders → 200 OK  (auth required — my order history)
```json
{ "orders": [ { "order_id": 5001, "status": "preparing", "total": 680 } ] }
```

---
## Restaurant side (auth required, role=restaurant)

### GET /restaurants/{id}/orders → 200 OK
```json
{ "orders": [ { "order_id": 5001, "status": "placed", "total": 680 } ] }
```
Errors: `401` no token · `403` not your restaurant

### PATCH /orders/{id}/status → 200 OK
```json
// request
{ "status": "out_for_delivery" }   // allowed: preparing | out_for_delivery | delivered
// response 200
{ "order_id": 5001, "status": "out_for_delivery" }
```
Errors: `400` invalid status value · `403` not your restaurant · `404` no such order

---
## Real-time

### WS /orders/{id}/track  (auth required)
Server pushes a message on every status change:
```json
{ "order_id": 5001, "status": "out_for_delivery", "at": "2026-07-03T18:40:00Z" }
```

---
## Order status lifecycle (state machine)
```
placed → preparing → out_for_delivery → delivered
   └────────────── cancelled (from placed/preparing only)
```

## Standard error shape (all 4xx/5xx)
```json
{ "error": { "code": "OUT_OF_STOCK", "message": "Chicken Biryani is unavailable" } }
```
Why a consistent shape: clients can handle all errors uniformly (parse `error.code`), instead of guessing per-endpoint.

---
*Next (Day 5): write the Go structs matching these shapes + the first `net/http` server serving `GET /restaurants`.*
