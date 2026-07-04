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
*(New decisions appended per day as the design evolves.)*
