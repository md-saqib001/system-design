# System Design Mastery

A structured, long-form study of how production software systems are designed, built, operated and scaled — from operating-system and network fundamentals to distributed consensus, complete High-Level Designs, Low-Level Designs and a production-oriented capstone.

The notes aim to explain **mechanisms, not answers**: for each concept, what it is, why it exists, how it works internally, who performs each operation, where state lives, what fails, and what the trade-offs are.

## Scope

| Track | Coverage |
|---|---|
| Backend engineering | OS and network primitives, Node.js runtime, HTTP/TLS, API design, authentication, production architecture, observability, containers, deployment |
| Database engineering | PostgreSQL internals, indexing and query execution, transactions, MVCC, locking, WAL and recovery, replication, partitioning, sharding, storage engines, NoSQL selection |
| Distributed systems | Partial failure, retries and idempotency, delivery semantics, consistency models, CAP/PACELC, replication, quorums, clocks, 2PC, sagas, distributed locks, consensus and Raft |
| Caching and messaging | Cache patterns and failure modes, Redis, RabbitMQ, Kafka, outbox, CDC, CQRS, event sourcing |
| Scalability and reliability | Load balancing, CDNs, bottleneck analysis, rate limiting, circuit breakers, HA, DR, SLOs, capacity estimation |
| HLD | 12 complete end-to-end system designs with estimation, data models, failure analysis and evolution |
| LLD | OOD, SOLID, UML, C++ concurrency, 14 design patterns, 11 implemented case studies |

## Repository Map

| Folder | Contents |
|---|---|
| [`00-roadmap`](./00-roadmap) | Course roadmap and progress tracker |
| [`01-backend-foundations`](./01-backend-foundations) | How a backend request actually executes |
| [`02-http-and-apis`](./02-http-and-apis) | HTTP, TLS, REST, auth, real-time communication |
| [`03-databases`](./03-databases) | PostgreSQL internals, indexing, transactions, scaling |
| [`04-backend-architecture`](./04-backend-architecture) | Production backend architecture and operations |
| [`05-lld-foundations`](./05-lld-foundations) | Object-oriented design, SOLID, C++ concurrency |
| [`06-distributed-systems`](./06-distributed-systems) | Distributed systems fundamentals |
| [`07-caching-and-messaging`](./07-caching-and-messaging) | Caching and asynchronous systems |
| [`08-scalability-and-reliability`](./08-scalability-and-reliability) | Scalability, reliability, performance |
| [`09-advanced-distributed-systems`](./09-advanced-distributed-systems) | Microservices, multi-region, search, storage |
| [`10-lld-design-patterns`](./10-lld-design-patterns) | Design patterns and LLD case studies |
| [`11-hld-case-studies`](./11-hld-case-studies) | Complete HLD case studies |
| [`12-capstone-and-interviews`](./12-capstone-and-interviews) | Architecture decision-making, mocks, capstone |
| [`project`](./project) | The running project that evolves across chapters into the capstone |

Each chapter contains:

- `chapter-roadmap.md` — teaching units, dependencies, assignments and outcomes
- `topics/` — one engineering document per teaching unit
- `interview-questions.md` — cumulative question bank (basic, why, follow-up, trap, scenario, design, trade-off, troubleshooting) with full answers and common wrong answers
- `assignments/` — specifications, actual experimental outputs and analysis
- `implementations/` — code
- `decisions/` — Architecture Decision Records

## Principles

- Depth over breadth: one concept at a time, explained to the level of the mechanism.
- Evidence over assertion: performance and failure claims are backed by experiments wherever practical, and outputs recorded in the notes are real.
- Decisions are documented as ADRs with alternatives and the conditions under which they would change.

## Status

See [`00-roadmap/progress-tracker.md`](./00-roadmap/progress-tracker.md).