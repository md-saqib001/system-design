# Chapter 1 — Backend Engineering Foundations: Interview Questions

> Cumulative question bank for Chapter 1. Questions are numbered continuously across the chapter and appended as each unit is completed.
> Chapter roadmap: [`chapter-roadmap.md`](./chapter-roadmap.md)

## How to Use This File

1. Read the question and answer it **aloud or in writing before reading the answer**.
2. Compare your answer with the reasoning, not just the conclusion.
3. Check the **common incorrect answers**. If yours matches one, record the concept in the Weak Concepts table of [`progress-tracker.md`](../00-roadmap/progress-tracker.md).

**Categories:** Basic · Why · Follow-up · Trap · Scenario · Design · Trade-off · Troubleshooting

## Index

| Unit | Questions | Topic note |
|---|---|---|
| 1.1 What Is Backend Engineering | Q1–Q16 | [01-what-is-backend-engineering.md](./topics/01-what-is-backend-engineering.md) |

---

## Unit 1.1 — What Is Backend Engineering

### Q1. What is backend engineering, and how does it differ from backend development?

**Category:** Basic

**Answer:**
The backend is the operator-controlled part of a system. It holds the authoritative state, enforces the rules, keeps the secrets, and serves clients over a network. **Backend development** is implementing functionality: endpoints, business logic, queries. **Backend engineering** is guaranteeing that functionality keeps its required properties (correctness, availability, latency, durability, security) while many clients use it concurrently, while components fail, while load grows, and while the code changes.

**Reasoning:**
The distinction is about the questions asked. A developer asks "does this endpoint return the right result?" An engineer also asks "what happens when two of these run at once, when the database is slow, when we run ten copies, when we deploy mid-request, and when the input is malicious?"

**Follow-ups:**
- Name one property that is easy to achieve on a laptop and hard in production. *(Correctness under concurrency; availability during deploys; durability through crashes.)*
- Which components besides the application server belong to "the backend"? *(Load balancers, databases, caches, queues, workers, object storage, observability systems.)*

**Common incorrect answers:**
- *"Backend engineering is writing APIs and connecting them to a database."* This describes the entry point and one dependency. It omits concurrency control, failure handling, asynchronous processing, infrastructure and operations, which are where most of the engineering effort goes.
- *"Backend engineering is Node.js / Java / Go development."* This confuses a tool with the discipline. The same problems exist in every language.

---

### Q2. What precisely distinguishes the frontend from the backend?

**Category:** Basic

**Answer:**
The **trust boundary**: who controls the environment the code runs in. Frontend code runs in an environment the user controls (browser, mobile device), where code can be inspected and modified and requests can be forged. Backend code runs in an environment the operator controls. Consequently, the frontend's decisions are advisory and the backend's decisions are authoritative.

**Reasoning:**
"Frontend is UI, backend is logic" fails as a definition, because frontends contain plenty of logic. What matters is enforceability. Logic placed in the frontend can be bypassed. Logic placed in the backend cannot be bypassed by a client.

**Follow-ups:**
- Is a backend-for-frontend (BFF) service frontend or backend? *(Backend: it runs in the operator's environment, even though it is shaped around a UI.)*
- Is a desktop application that talks to your API trusted? *(No. It runs on the user's machine, so it is on the untrusted side.)*

**Common incorrect answers:**
- *"The frontend is what the user sees; the backend is everything else."* This doesn't explain why validation must be repeated server-side. The visibility framing misses the trust framing.

---

### Q3. Why can't a multi-user application keep its authoritative state on client devices?

**Category:** Why

**Answer:**
Because shared state that is modified concurrently needs **one authority that orders conflicting operations**. If each device keeps its own copy, two devices can make conflicting decisions, such as both booking the last seat, and no component is entitled to resolve the conflict. Client devices are also untrusted (state can be tampered with), unreliable (devices are lost or destroyed, so durability fails), and not always online (no one can coordinate with an offline device).

**Reasoning:**
There are only two ways out: put the authority in one trusted place (a backend), or make the devices agree among themselves via a protocol (peer-to-peer consensus or CRDT-based merging). The second is much harder, and some invariants, such as "a seat is sold at most once", cannot be enforced without coordination.

**Follow-ups:**
- Which invariants can local-first CRDT apps preserve, and which can't they? *(They can merge commutative updates such as adding items to a set or editing text. They can't enforce uniqueness or scarcity constraints such as one ticket per seat without coordination. Covered in Ch 6.)*
- What component in a typical backend actually orders the conflicting writes? *(The database's concurrency control: locks or MVCC conflict detection.)*

**Common incorrect answers:**
- *"Because client devices have less storage."* Storage capacity is a minor factor. The fundamental issues are authority, trust and durability.

---

### Q4. The frontend already validates input. Why must the backend validate it again?

**Category:** Why

**Answer:**
Because the backend cannot know which software produced a request. Any program can send bytes to the server's address and port: `curl`, Postman, a script, or a modified copy of your own frontend. Frontend validation improves user experience (immediate feedback, fewer wasted requests). Only backend validation provides guarantees.

**Reasoning:**
Validation is a security and correctness control only if the attacker cannot skip it. The client is entirely under the attacker's control, so client-side checks can always be skipped.

**Follow-ups:**
- Should the backend accept the price of an item from the request body? *(No. It should derive the price from authoritative data. Client-supplied values may be used only as identifiers or as expectations to compare against, never as authoritative values.)*
- Can authentication tell you which client app sent a request? *(No. Authentication identifies the principal, meaning the user or service. Client attestation mechanisms raise the cost of impersonating an app, but they do not make the client trusted.)*

**Common incorrect answers:**
- *"Backend validation is a fallback in case frontend validation has a bug."* This frames it as redundancy. In fact it is the only enforcing control. The frontend check is the optional one.
- *"Using HTTPS prevents tampering, so frontend validation is enough."* HTTPS protects data in transit from third parties. It does nothing to stop the user, who controls one endpoint of the encrypted connection, from sending whatever they like.

---

### Q5. Two users tap "Book" on the last seat at the same instant. Which component decides who gets it, and how?

**Category:** Follow-up

**Answer:**
In a correctly built system, the **database's concurrency control** decides. For example, the application runs a transaction that takes a row lock (`SELECT … FOR UPDATE`) or issues a single conditional update (`UPDATE seats SET status='BOOKED' WHERE id='F12' AND status='AVAILABLE'`). The database serialises the two operations. The first to acquire the lock or apply the update wins. The second either waits and then sees the seat already booked, or its conditional update affects zero rows. The application then turns that outcome into a success response for one user and a conflict response (e.g., `409`) for the other.

**Reasoning:**
Neither client can decide, because they are untrusted and unaware of each other. The application processes can't decide on their own either, because there may be many instances and requests interleave even within one instance. The decision must happen at the single component that holds the authoritative copy of the data and can apply operations atomically.

**Follow-ups:**
- What if the seats table is sharded across several databases? *(Each seat should live on exactly one shard, so the same mechanism applies within that shard. Operations spanning shards need distributed coordination, covered in Ch 6.)*
- Which approach is better under very high contention, row locks or conditional updates? *(Covered in Ch 3: optimistic vs pessimistic concurrency.)*

**Common incorrect answers:**
- *"Whichever request reaches the server first."* "Reaching the server first" has no meaning across several instances and interleaved handlers. Without an atomic operation at the data layer, both requests can succeed.
- *"Node.js is single-threaded, so the requests are processed one after another."* See Q7.

---

### Q6. If the application tier is "stateless", where did the state go, and why move it there?

**Category:** Follow-up

**Answer:**
The state moved into dedicated stateful systems: databases (authoritative data), caches such as Redis (derived or session data), message queues (pending work) and object storage (files). It moved there so that application instances become **interchangeable**. Any instance can serve any request, instances can be added or removed freely, and a crashed instance loses nothing. The stateful systems are built specifically for durability, replication and concurrency control, which application processes are not.

**Reasoning:**
"Stateless backend" is shorthand that is easy to misread. The system is fundamentally stateful. Its value *is* its state. Statelessness is a property of one tier, achieved by concentrating state in components designed to manage it.

**Follow-ups:**
- What new problems does this create? *(The stateful components become bottlenecks and single points of failure, so they must be scaled and replicated. That is the subject of Ch 3, 7 and 8.)*
- Is an in-process cache acceptable in a stateless application tier? *(Yes, if it holds only derived data that can be lost or briefly out of date without breaking correctness. Covered in Ch 7.)*

**Common incorrect answers:**
- *"Stateless means the backend doesn't store any data."* This contradicts the purpose of a backend. Only the application processes avoid holding state between requests.

---

### Q7. Node.js runs JavaScript on a single thread. Can a Node.js backend have race conditions?

**Category:** Trap

**Answer:**
**Yes.** Single-threaded means only one piece of JavaScript executes at any instant. It does not mean only one request is in progress. Each `await` suspends the current handler and returns control to the event loop, which can then run other requests' handlers. If a handler checks a condition (`await db.getSeat()`), then acts on it (`await db.markBooked()`) in a separate step, another request can run its own check in between. Both see the seat as available, and both book it. With several Node.js processes or machines, the interleaving also happens in true parallel.

**Reasoning:**
A race condition requires two things: a sequence of operations that is not atomic, and other operations that can interleave with it. Threads are one source of interleaving. Asynchronous suspension points are another. What Node.js does rule out is data races on JavaScript memory *within a synchronous block*: no other JavaScript runs until the current synchronous code finishes or reaches an `await`.

```js
// Race-prone even on one thread
const seat = await db.getSeat(id);          // suspension point
if (seat.status === 'AVAILABLE') {
  await db.markBooked(id, userId);          // suspension point
}
```

**Follow-ups:**
- Is a JavaScript mutex (e.g., a promise-based lock) a correct fix? *(Only within one process. It fails as soon as there are two instances. The correct fix is atomicity in the shared data store, covered in Ch 3.)*
- Is `counter++` on a module-level variable safe in Node.js? *(Yes within one process, because it is synchronous. But the counter is per-process and is lost on restart.)*

**Common incorrect answers:**
- *"No, because there is only one thread."* This is the trap. It equates "single-threaded" with "sequential request processing".
- *"Only if you use worker threads."* Worker threads add another source of interleaving, but they aren't required for this race.

---

### Q8. "Our application servers are stateless, so we can scale indefinitely by adding servers." Is this true?

**Category:** Trap

**Answer:**
**No.** Statelessness lets the *application tier* scale horizontally, but every instance still depends on shared stateful components: the database, cache, queue and external providers. Adding application servers increases the load on those components, such as more database connections and more queries per second, until one of them becomes the bottleneck. Beyond that point, adding application servers increases latency and error rates rather than throughput.

**Reasoning:**
Horizontal scaling moves the bottleneck; it doesn't remove it. For example, 50 instances each holding a pool of 20 database connections opens 1,000 connections against one PostgreSQL server, which uses one OS process per connection (Ch 3). That alone can exhaust the database's memory.

**Follow-ups:**
- What would you scale next once the database becomes the bottleneck? *(Connection pooling, caching, read replicas, then partitioning or sharding: Ch 3, 7, 8.)*
- Are there limits even for the application tier itself? *(Load-balancer capacity, network bandwidth, coordination costs, and cost per request.)*

**Common incorrect answers:**
- *"Yes, that's the point of stateless design."* This ignores shared dependencies.

---

### Q9. A mobile app embeds a third-party API key, heavily obfuscated, to call a paid service directly. Is this acceptable?

**Category:** Trap

**Answer:**
**No**, if the key grants anything the operator wouldn't hand to every user. Anything shipped to a client can be extracted from the binary or captured from the app's network traffic. Obfuscation only raises the effort required. The correct design keeps the secret on the backend: the app calls your backend, which authenticates the user, applies authorization and rate limits, and then calls the third-party service with the secret.

**Reasoning:**
Secret custody is one of the fundamental reasons a backend exists (Unit 1.1 §2). Keys designed to be public, such as publishable payment keys or restricted map keys, are an explicit exception. They are scoped so that exposing them is harmless.

**Follow-ups:**
- What extra benefits does proxying through the backend give? *(Per-user rate limiting, auditing, the ability to rotate keys without shipping a new app version, and cost control.)*
- What is the cost of proxying? *(An extra network hop, and the backend must scale with that traffic.)*

**Common incorrect answers:**
- *"It's fine if the key is obfuscated and the traffic uses HTTPS."* The user controls the device and one end of the TLS connection, so they can read both the binary and the decrypted traffic.

---

### Q10. Review this handler. What will go wrong in production?

**Category:** Scenario

```js
const bookedBy = {};

app.post('/book', async (req, res) => {
  const { seatId, price } = req.body;
  const seat = await db.getSeat(seatId);
  if (seat.status !== 'AVAILABLE') return res.status(409).json({ error: 'taken' });
  await paymentGateway.charge(req.user.id, price);
  await db.markBooked(seatId, req.user.id);
  bookedBy[seatId] = req.user.id;
  res.json({ ok: true });
});
```

**Answer:**

| Issue | Failure | Fix (direction) |
|---|---|---|
| `price` taken from the request | Users pay any amount | Derive price server-side |
| Check (`getSeat`) and act (`markBooked`) are separate | Concurrent requests both pass the check; double booking | Atomic conditional update, row lock, or unique constraint |
| No timeout on `charge` | A slow gateway holds requests open, exhausting connections and concurrency; the service degrades | Timeouts and circuit breakers (Ch 6, Ch 8) |
| No idempotency key on `charge` | Client or network retries cause double charges | Idempotency keys (Ch 2, Ch 6) |
| Charge happens before booking, without compensation | If `markBooked` fails, the user is charged with no seat | Reserve the seat first, then charge, then confirm; or compensate (Ch 9, sagas) |
| `bookedBy` held in process memory | Different per instance; lost on restart or deploy | Store in the database |
| Missing null check on `seat` | Nonexistent `seatId` causes an unhandled error (500) instead of `404` | Validate the input and handle the missing resource |

**Reasoning:**
Each defect is invisible with one user, one process, a healthy gateway and friendly input. Each is exposed by one of the four forces: concurrency, failure, scale and change, plus adversarial input.

**Follow-ups:**
- Which bug do you fix first, and why? *(Usually the double-charge and double-booking bugs, because they cause financial loss and damage users' trust directly. Trusting the client price is equally critical, since it is trivially exploitable.)*
- How would you detect each of these in production? *(Reconciliation jobs comparing payments and bookings, unique-constraint violation alerts, latency and timeout metrics on the gateway, logs keyed by request ID.)*

**Common incorrect answers:**
- Listing only the race condition. Strong answers cover trust, concurrency, failure, scale and correctness of the operation order.

---

### Q11. Design question: a team wants to build a shared to-do list for families without writing any backend code. What options exist, and how would you choose?

**Category:** Design

**Answer:**
Options, roughly in increasing control and effort:

1. **Backend-as-a-Service** (Firebase/Supabase). Authentication, a database, real-time sync and access rules come as configuration. Best when the rules are simple ("only family members can read or write their list") and time-to-market dominates.
2. **Local-first with a sync service** (CRDT-based). Each device holds the list and works offline; a relay merges changes. Good fit because to-do edits merge well (add, complete, rename), but rules like "max 100 items" or strict permissions are harder to enforce.
3. **Serverless functions plus a managed database.** Custom logic without managing servers.
4. **Conventional backend.** Full control, highest operational effort.

Choose by asking: *What invariants must hold?* (Permissions are essential; strict uniqueness or scarcity isn't.) *Is offline use important?* *How much custom logic and integration is expected?* *What are the cost and lock-in tolerances?* For a family to-do list, BaaS or local-first are reasonable. A conventional backend is justified if the product is expected to grow into complex logic.

**Reasoning:**
"No backend code" rarely means "no backend". BaaS is a backend someone else operates, with your rules expressed as configuration. The trust-boundary principle still applies: access rules must be enforced by the BaaS, not by the client app.

**Follow-ups:**
- With Firebase, where must the "only family members can read this list" rule live? *(In the server-side security rules, never only in the app.)*
- What would push you off BaaS later? *(Complex transactional logic, cost at scale, compliance or data-residency requirements, vendor lock-in.)*

**Common incorrect answers:**
- *"Store it in each device's local storage and sync via email or files."* No authority, no conflict resolution, no access control.

---

### Q12. What are the trade-offs between a centralised backend and a peer-to-peer or local-first architecture?

**Category:** Trade-off

**Answer:**

| Aspect | Centralised backend | Peer-to-peer / local-first |
|---|---|---|
| Correctness of conflicting updates | Simple: one authority orders operations | Hard: requires consensus or merge semantics; some invariants unenforceable |
| Rule enforcement and security | Strong: trusted environment | Weak or complex: peers are untrusted |
| Availability | Authority is a single point of failure unless replicated | No single point of failure; works offline |
| Latency | Network round trip to the authority | Local operations are instant |
| Scalability | Authority must be scaled | Load is spread across peers |
| Operational burden | Operator runs infrastructure | Less central infrastructure; more protocol complexity |

**Reasoning:**
Centralising authority buys simple, strong correctness at the cost of availability and scaling difficulty. Decentralising buys availability and offline capability at the cost of weaker guarantees and more complexity. Most production systems are centralised but replicated, which brings back some decentralisation problems (Ch 6).

**Follow-ups:**
- Which architecture suits a bank ledger? A collaborative text editor? *(A ledger needs strong invariants, so centralised. An editor benefits from offline editing and mergeable operations, so local-first or hybrid.)*

**Common incorrect answers:**
- *"Peer-to-peer is always more scalable, so it's better."* This ignores the cost of reaching agreement and of enforcing invariants.

---

### Q13. Your service has a 99.9% availability target. How much downtime does that allow, and what does moving to 99.99% imply?

**Category:** Trade-off

**Answer:**
99.9% allows (1 − 0.999) × 8,760 h = **8.76 hours per year**, or about **43 minutes per 30-day month**. 99.99% allows about **52.6 minutes per year**, or about **4.3 minutes per month**. Moving to 99.99% means humans can no longer be part of the recovery path for most failures: failover must be automated, deployments must cause zero downtime, every component needs redundancy across failure domains, and dependencies must themselves be more available than the target. Each additional nine costs disproportionately more in engineering, infrastructure and operational discipline.

**Reasoning:**
A single 10-minute incident uses about 23% of a 99.99% monthly budget. If detecting a problem and paging an engineer takes 5 minutes, manual recovery alone exceeds the budget.

**Follow-ups:**
- If your service calls three dependencies in series, each 99.9% available, what is the best availability you can achieve? *(Roughly 0.999³ ≈ 99.7%, assuming independent failures. Covered in Ch 8.)*
- Is availability measured as uptime or as successful requests? *(Both definitions are used; request-based is generally more meaningful. Covered in Ch 8, SLIs.)*

**Common incorrect answers:**
- *"99.9% and 99.99% are basically the same."* They differ by a factor of ten in allowed downtime.
- Using 365 × 24 × 60 incorrectly, or confusing per-year and per-month budgets.

---

### Q14. Name the system properties you would ask about before designing any backend, and why each matters.

**Category:** Basic

**Answer:**
Latency (how fast each operation must be, as percentiles), throughput (how many operations per second), availability (how much downtime is acceptable), durability (whether acknowledged data may ever be lost), consistency (whether readers must see the latest state), scalability (expected growth), security (threat model, sensitive data), observability (how failures will be detected and diagnosed), maintainability (how often it will change, team size) and cost.

**Reasoning:**
These properties decide the architecture far more than the features do. A booking system that tolerates a few seconds of stale reads but never double booking leads to a different design from a social feed that tolerates staleness but must be very fast. Asking about these first is what separates system design from listing components.

**Follow-ups:**
- Which property usually conflicts with consistency? *(Availability during network partitions, and latency: CAP and PACELC in Ch 6.)*

**Common incorrect answers:**
- Listing only "scalability and performance". This is incomplete, and it skips the properties (consistency, durability) that most often drive design decisions.

---

### Q15. After moving from one application instance to three behind a load balancer, users report being logged out randomly, and an admin dashboard shows different "currently booked" counts on each refresh. Nothing else changed. What is happening, and how would you confirm it?

**Category:** Troubleshooting

**Answer:**
The application keeps state **in process memory**: sessions in an in-memory store, booking information in a module-level object. Each instance has its own copy. The load balancer sends successive requests from the same user to different instances. An instance that didn't create the session doesn't recognise it, so the user appears logged out. Each instance's in-memory booking data differs, so the dashboard changes depending on which instance served the request.

**How to confirm:**
1. Add the serving instance's identifier to responses (e.g., an `X-Instance-Id` header) or to logs, and correlate logouts with a change of instance between consecutive requests.
2. Inspect the session middleware configuration. The default in-memory store (e.g., `express-session` with no store configured) is per-process.
3. Reproduce by pinning to one instance (should work) vs. round-robin (should fail).

**Fix direction:**
Move session state to a shared store (Redis or the database) or use self-contained tokens. Move booking data to the database. Session affinity ("sticky sessions") at the load balancer hides the symptom, but state is still lost when an instance restarts or is deployed. Covered in Unit 1.16.

**Reasoning:**
"Nothing else changed except the number of instances" points directly at state that depends on the instance.

**Common incorrect answers:**
- *"The load balancer is dropping cookies."* Possible in theory, but it doesn't explain the inconsistent dashboard counts. One root cause explains both symptoms.
- *"Enable sticky sessions."* This treats the symptom and leaves the system fragile to restarts, deploys and uneven load.

---

### Q16. During a payment-gateway slowdown, some customers were charged twice, and the booking API's latency rose until almost every request timed out, including endpoints unrelated to payments. Explain both effects and how you would investigate.

**Category:** Troubleshooting

**Answer:**

**Double charges:** Calls to the gateway were slow. Either the client, a proxy or the application retried the booking request (or the charge call) after a timeout, while the original charge was still in progress or had already succeeded. Without an **idempotency key**, the gateway treats each retry as a new charge.

**Service-wide latency:** The charge calls had **no timeout**, or one that was too long. Each in-flight booking request held resources while it waited: an open client connection, memory, possibly a database connection checked out from the pool. As slow requests piled up, shared resources (database pool, outbound connection limits, file descriptors, memory) ran out, so unrelated endpoints that needed those resources also stalled. One slow dependency spread to the whole service.

**Investigation:**
1. Correlate duplicate charges with request logs by user and seat, and look for multiple requests or retries close together.
2. Check retry configuration at every layer: client, load balancer, HTTP client library.
3. Inspect gateway-call latency metrics and timeout settings.
4. Inspect database-pool and connection metrics during the incident: active vs idle connections, wait time.
5. Check event-loop lag and memory metrics for the Node.js processes.

**Fix direction:**
Idempotency keys on charge requests; strict timeouts on outbound calls; circuit breakers that fail fast when the gateway is unhealthy; bulkheads that isolate payment traffic from other endpoints (Ch 2, 6, 8).

**Reasoning:**
Timeouts and retries interact: no timeout leads to resource exhaustion, while timeouts with retries but no idempotency lead to duplicate side effects. Both must be designed together.

**Common incorrect answers:**
- *"The gateway charged twice by mistake."* Possible, but the most common cause is unprotected retries, and the evidence (duplicate requests in logs) usually shows it.
- *"Increase the timeout."* A longer timeout makes resource exhaustion worse.
