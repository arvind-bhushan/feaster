# Design Decisions & Trade-offs

This log records **why** each choice was made — the reasoning, not just the what. (This is the file interviewers love: it shows engineering judgment.)

---
## Day 1 — Project framing & scope

**Decision: Build a food-ordering & delivery platform ("Feaster").**
Why: the domain is familiar (so effort goes into engineering, not figuring out requirements), but it naturally requires every important backend concept — transactions (checkout), caching (menus), real-time (tracking), event-driven design (delivery assignment), and concurrency (simultaneous orders). One project, full coverage.

**Decision: Language = Go.**
Why: strong concurrency model (goroutines) fits an order/delivery system with many simultaneous operations; statically typed and fast; the target skill for backend/infra roles.

**Decision: Clean/layered architecture (handler → service → repository).**
Why: separates transport (HTTP) from business logic from data access, so each is testable and swappable. Chosen up front so the codebase stays maintainable as features grow.

**Decision: Start with requirements before code.**
Why: a backend exists to answer specific requests; you can't design endpoints or a schema without first knowing the functional requirements. Design-first prevents rework.

**Trade-off noted:** payments, maps, and reviews are out of scope — deliberately, to keep focus on the core ordering/delivery engine where the hard engineering lives.

---
## Day 2 — API design & transport

**Decision: RESTful API with resource-noun paths + HTTP verbs.**
Why: predictable, standard, self-documenting. `GET /restaurants` and `POST /orders` are instantly understandable; the method carries the action so paths stay clean nouns. Chosen over RPC-style `/getRestaurants` naming.

**Decision: All endpoints over HTTP/TCP; live order tracking over WebSockets.**
Why: ordering touches money/inventory — data must arrive complete and in order, which is exactly TCP's guarantee (reliable, ordered, connection-based via the 3-way handshake). UDP (fast but lossy) is wrong here; it's for real-time media, not transactions. Order *status* updates use WebSockets (persistent connection, also over TCP) so the client sees live changes without polling.

**Decision: `{id}` path params to address specific resources.**
Why: `/orders/{id}` cleanly identifies one order; RESTful and cache-friendly.

---
## Day 3 — API contract & status codes

**Decision: Define exact request/response JSON per endpoint, with correct status codes, before coding.**
Why: the JSON is the contract the client and the Go structs both follow. Locking it now prevents rework and makes the code a direct translation of the design.

**Decision: Correct, specific HTTP status codes (201 create, 401 vs 403, 404, 409 conflict).**
Why: codes are the client's machine-readable outcome. `201` (not `200`) for creation, `401` (not logged in) vs `403` (logged in but not allowed), `409` for out-of-stock — precise codes let clients react correctly and signal engineering care.

**Decision: One consistent error shape `{ "error": { "code", "message" } }` everywhere.**
Why: clients handle all errors uniformly by parsing `error.code`, instead of a different shape per endpoint.

**Decision: Model order status as an explicit state machine (placed→preparing→out_for_delivery→delivered).**
Why: makes valid transitions clear and prevents illegal jumps (e.g. delivered→placed); the PATCH /status endpoint will validate against it.

**Noted risk: POST /orders is not idempotent** — a retry would duplicate the order/charge. Flagged now; addressed with idempotency keys + a DB transaction later (Day 29/51).

---
## Day 4 — Transport security (TLS/HTTPS)

**Decision: Feaster serves plain HTTP in local dev; TLS is terminated at the proxy/load balancer in production.**
Why: the app handles passwords + JWTs, so production traffic MUST be HTTPS (TLS gives encryption, integrity, and server authentication via a CA-signed certificate — preventing sniffing, tampering, and impersonation). But the Go app itself doesn't need to do TLS: in production a reverse proxy / load balancer / CDN (e.g. Nginx, Cloudflare) terminates TLS and forwards plain HTTP to the app internally. Locally there's no network to sniff, so `localhost:8080` over HTTP is fine and production-correct. This keeps the app code simple and matches how real deployments work.

**Implication for tokens/cookies:** because production is HTTPS end-to-user, the JWT in the `Authorization` header (and any cookies) is encrypted in transit — a network eavesdropper can't steal it. Cookies (if used) will be set `Secure` (HTTPS-only) + `httpOnly` (JS can't read them).

**Design phase complete.** Requirements → API → contract → transport all defined. Next: code.

---
*(New decisions appended per day as the design evolves.)*
