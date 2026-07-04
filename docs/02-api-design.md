# Feaster — API Design (Day 2)

The backend's job is to answer requests. This is the "menu of requests" Feaster accepts — each mapped from a requirement in [01-requirements.md](01-requirements.md).

Every endpoint = an HTTP **method** (the action) + a **path** (the resource, always a noun).

## Auth
| Method | Path | Purpose | Maps to req |
|---|---|---|---|
| POST | `/signup` | Create a customer/restaurant account | #1 |
| POST | `/login` | Authenticate, return a JWT token | #1 |

## Restaurants & Menu (browsing — reads)
| Method | Path | Purpose | Maps to req |
|---|---|---|---|
| GET | `/restaurants` | List all restaurants | #2 |
| GET | `/restaurants/{id}` | One restaurant's details | #2 |
| GET | `/restaurants/{id}/menu` | A restaurant's menu | #3 |

## Orders (customer)
| Method | Path | Purpose | Maps to req |
|---|---|---|---|
| POST | `/orders` | Place an order | #4 |
| GET | `/orders/{id}` | View one order + its status | #5 |
| GET | `/orders` | My order history | #5 |

## Restaurant side
| Method | Path | Purpose | Maps to req |
|---|---|---|---|
| GET | `/restaurants/{id}/orders` | Incoming orders for a restaurant | #6 |
| PATCH | `/orders/{id}/status` | Update order status | #6 |

## Real-time
| Protocol | Path | Purpose | Maps to req |
|---|---|---|---|
| WebSocket | `/orders/{id}/track` | Live order-status updates | #5 |

## REST conventions used here
- **GET** = read (safe, no side effects) · **POST** = create · **PATCH** = partial update · **PUT** = full replace · **DELETE** = remove.
- Paths name **resources (nouns)**, not actions: `/restaurants`, not `/getRestaurants`. The method is the verb.
- `{id}` is a **path parameter** — a placeholder for a specific resource's ID.

## Networking notes (why this rides on TCP)
All these endpoints use **HTTP over TCP** — correctness matters (an order must arrive complete and in order), so TCP's reliable, ordered delivery is required. The one exception in spirit is real-time media (not used here); live order *status* is fine over WebSockets (which also ride on TCP).

---
*Next (Day 3): HTTP in depth — status codes, headers — then define the request/response JSON shapes for these endpoints.*
