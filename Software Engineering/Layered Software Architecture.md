# Application Layers — Notes

## 1. What is a Layered Architecture?

A **layered architecture** organizes an application into horizontal stacks of responsibility. Each layer:

- Has a single, well-defined concern (UI, business rules, data, etc.)
- Only talks to the layer directly below it(strict layering) or to lower layers (relaxed layering)
- Hides its internals behind a contract/interface so layers can be swapped or tested independently

The benefit is separation of concerns: a UI redesign should not force a database migration, and vice versa.

---

## 2. The Classical Layers

```
┌────────────────────────────────────────────┐
│  Presentation Layer (UI / Client)          │  ← what the user sees
├────────────────────────────────────────────┤
│  API / Gateway Layer                       │  ← entry point for clients
├────────────────────────────────────────────┤
│  Business / "Application Logic" Layer      │  ← rules, workflows, use cases
├────────────────────────────────────────────┤
│  Data Access Layer (DAL / Repository)      │  ← reads/writes to persistence
├────────────────────────────────────────────┤
│  Persistence / Database Layer              │  ← where data actually lives
├────────────────────────────────────────────┤
│  Infrastructure Layer                      │  ← OS, networking, hosting
└────────────────────────────────────────────┘
```

---

## 3. Presentation Layer (Frontend / UI)

**Responsibility:** Render information and capture user input. **No business rules** should live here — that creates duplication when you add a second client (mobile, CLI).

### Tools and their intricacies

| Tool | What it is | Intricacy / gotcha |
|------|-----------|--------------------|
| **HTML / CSS** | Markup + styling primitives | CSS specificity & cascade order are the source of most "why isn't my style applying" bugs. Modern apps lean on Flexbox/Grid; older code uses floats. |
| **JavaScript / TypeScript** | The runtime language of the browser | TS adds compile-time types but **erases at runtime** — runtime data still needs validation (Zod, io-ts). |
| **React** | Component-based UI library | Re-renders on state change; misuse of `useEffect` and missing `key` props are the top sources of bugs. **State location matters** — lift it only as high as needed. |
| **Vue / Svelte / Angular** | Alternative frameworks | Svelte compiles away the framework; Angular ships an opinionated full kit; Vue sits in between. |
| **Next.js / Remix / Nuxt** | React/Vue meta-frameworks | Blur the line between presentation and API by adding **server components** and **server actions**. The "presentation layer" now spans client + server. |
| **Tailwind CSS** | Utility-first CSS | Removes naming overhead but couples markup to styling. Class strings can balloon — extract via `@apply` or component abstraction. |
| **State management (Redux, Zustand, TanStack Query)** | Client-side data store | Redux = predictable but boilerplate-heavy. TanStack Query handles **server state** (cache, refetch, dedupe) which is fundamentally different from UI state. |
| **Bundlers (Vite, webpack, esbuild, Turbopack)** | Pack source into deliverable assets | Tree-shaking, code-splitting, and source maps are the things to actually understand. Vite uses esbuild for dev and Rollup for prod — different behaviors. |

**Key intricacy:** *Hydration* — the server sends rendered HTML, then the client "wakes it up" with JS. Mismatches between server and client output cause hydration errors that are notoriously hard to debug.

---

## 4. API / Gateway Layer

**Responsibility:** Expose a contract over the network. Authenticate, validate input, route requests, shape responses. **It is not where business rules live** — it should delegate to the business layer.

### Tools and their intricacies

| Tool | What it is | Intricacy / gotcha |
|------|-----------|--------------------|
| **REST (over HTTP)** | Resource-based API style | "REST" in practice is usually just JSON-over-HTTP. True REST (HATEOAS) is rare. Versioning strategy (URL vs header) is a long-term commitment. |
| **GraphQL** | Query language for APIs | Solves over/under-fetching but introduces **N+1 problems** in resolvers — needs DataLoader. Caching is harder than REST because every query is unique. |
| **gRPC** | Binary RPC over HTTP/2 | Fast, strongly-typed via Protobuf. Browsers can't speak it directly — needs grpc-web proxy. Best for service-to-service. |
| **tRPC** | TypeScript end-to-end types | No code-gen — types flow from server to client via inference. Locks you into a TS-only ecosystem. |
| **Express / Fastify / Hono** | Node.js web frameworks | Express is ubiquitous but unmaintained for years. Fastify is faster + has built-in schema validation. Hono runs on edge runtimes (Workers, Deno). |
| **FastAPI** | Python async framework | Pydantic validation + auto OpenAPI docs. Async is required to get the perf — sync code blocks the event loop. |
| **Spring Boot** | Java enterprise framework | Annotation-driven, deep magic. Startup time and memory footprint are real concerns; GraalVM native-image helps. |
| **API Gateways (Kong, AWS API Gateway, Nginx)** | Reverse proxy + cross-cutting concerns | Handle rate-limiting, auth, routing **before** your app sees the request. Easy to accidentally duplicate logic between gateway and app. |

**Key intricacy:** *Idempotency*. POST is not idempotent by default; if a client retries a payment request, you may charge twice. Solve with idempotency keys.

---

## 5. Business / Application Logic Layer

**Responsibility:** Encode the actual rules of the domain — "an order over $1000 needs manager approval", "a user can't book two rooms at the same time". This is the **most valuable** layer; everything else is replaceable.

### Patterns and their intricacies

| Pattern | What it is | Intricacy / gotcha |
|---------|-----------|--------------------|
| **Service Layer** | Plain classes/functions encoding use cases | Easiest to start with. Watch out for "service" bloat — when one service does everything, split by use case. |
| **Domain-Driven Design (DDD)** | Models domain in code via Entities, Value Objects, Aggregates | Heavy upfront cost; pays off in complex domains (insurance, finance). Overkill for CRUD apps. |
| **CQRS** | Separate read and write models | Lets reads scale independently. Adds complexity — only worth it when read/write needs genuinely diverge. |
| **Event Sourcing** | Store every state change as an event | Perfect audit log, replayable history. **Schema evolution is painful** — old events can't be rewritten. |
| **Hexagonal / Clean Architecture** | Domain at center, adapters at edges | Keeps business logic free of framework imports. Many indirection layers — easy to over-engineer. |

### Tools

- **Language runtimes**: Node.js, Python (CPython, PyPy), JVM (Java, Kotlin, Scala), Go, Rust, .NET
- **Workflow engines**: Temporal, Airflow, Step Functions — for long-running, durable business processes
- **Rule engines**: Drools, JSON Logic — when business rules change frequently and shouldn't require deploys

**Key intricacy:** This layer should be **framework-agnostic** if possible. If you can't unit-test your business logic without spinning up a database or HTTP server, the layers are leaking.

---

## 6. Data Access Layer (DAL / Repository)

**Responsibility:** Translate between in-memory domain objects and persistent storage. Hide the storage technology from the business layer.

### Tools and their intricacies

| Tool | What it is | Intricacy / gotcha |
|------|-----------|--------------------|
| **ORM (Prisma, TypeORM, Hibernate, SQLAlchemy, ActiveRecord)** | Maps objects to relational rows | The **N+1 query problem** is the #1 footgun. Lazy loading hides it. Always inspect generated SQL for hot paths. |
| **Query builders (Knex, Kysely, jOOQ)** | Type-safe SQL composition without full ORM | Less magic, more control. You write SQL-shaped code; the builder protects against injection and typos. |
| **Raw SQL** | Direct queries | Best performance, full power. Loses type-safety unless paired with tools like sqlc or PgTyped. |
| **Repository pattern** | Interface around persistence | Lets you swap Postgres for Mongo (in theory). In practice, most teams never swap — but the testability win is real. |
| **Unit of Work** | Tracks changes, commits in one transaction | Hibernate/EF do this automatically. In Node ORMs you usually manage transactions explicitly. |

**Key intricacy:** *Transactions* and *isolation levels*. READ COMMITTED (Postgres default) allows non-repeatable reads. SERIALIZABLE prevents anomalies but can fail with serialization errors that the app must retry.

---

## 7. Persistence / Database Layer

**Responsibility:** Durable storage and retrieval of data with the consistency, availability, and performance the application needs.

### Categories and tools

| Category | Tools | When to use / intricacy |
|---------|-------|------------------------|
| **Relational (OLTP)** | PostgreSQL, MySQL, SQLite, SQL Server | Default choice. **Postgres** is the modern pick — has JSON, full-text search, extensions (PostGIS, pgvector). MySQL is widely deployed but has historically looser SQL conformance. |
| **Document** | MongoDB, Couchbase, DynamoDB | Schema flexibility at the cost of join power. Mongo's transactions are recent and slower than relational equivalents. |
| **Key-Value** | Redis, Memcached, etcd | Redis is single-threaded — a slow `KEYS *` command blocks everything. Use `SCAN`. Redis also doubles as a pub/sub and queue, but **persistence guarantees are weaker than a real DB**. |
| **Search** | Elasticsearch, OpenSearch, Meilisearch, Typesense | Eventual consistency with the source DB; an index lag of seconds is normal. Re-indexing is operationally expensive at scale. |
| **Columnar / Analytics (OLAP)** | ClickHouse, DuckDB, BigQuery, Snowflake | Aggregations over billions of rows in seconds; **bad at single-row updates**. Different tool for a different job vs OLTP. |
| **Graph** | Neo4j, Memgraph, Neptune | Worth it when relationships are the primary query (recommendations, fraud detection). For shallow joins, a relational DB is fine. |
| **Time-Series** | InfluxDB, TimescaleDB, Prometheus | Optimized for append-heavy, time-indexed data. Prometheus is pull-based; Influx is push-based — affects deployment shape. |
| **Vector** | pgvector, Pinecone, Weaviate, Milvus | For embedding-based similarity search (RAG, semantic search). Index choice (HNSW vs IVF) trades recall vs latency vs memory. |

**Key intricacies:**

- **Indexes are not free** — they speed reads but slow writes and consume disk. Every index should justify itself.
- **Migrations** are the highest-risk operation in production. Tools: Flyway, Liquibase, Alembic, Prisma Migrate, Atlas. Always think: can this run *online* without locking the table?
- **Connection pools**: Postgres connections are expensive (≈10 MB each). Use PgBouncer or RDS Proxy in serverless environments.

---

## 8. Infrastructure Layer

**Responsibility:** Run the code, route the traffic, store the bytes, recover from failure.

### Tools and their intricacies

| Tool | What it is | Intricacy / gotcha |
|------|-----------|--------------------|
| **Docker** | Container packaging | Image size matters — multi-stage builds and Alpine/distroless bases shrink them. `latest` tag in production is a trap. |
| **Kubernetes** | Container orchestration | Powerful and complex. Most teams do not need it; a managed PaaS is simpler. When you do need it, **resource limits/requests** and **liveness/readiness probes** are non-optional. |
| **Terraform / Pulumi / OpenTofu** | Infrastructure as Code | State files are the source of truth — losing one is catastrophic. Always store remotely with locking (S3 + DynamoDB, Terraform Cloud). |
| **CI/CD (GitHub Actions, GitLab CI, CircleCI, Jenkins)** | Automated build/test/deploy | Caching is the difference between a 2-minute and 20-minute pipeline. Secrets management — never echo them into logs. |
| **Cloud (AWS, GCP, Azure)** | Managed compute, storage, networking | Lock-in is real but often overstated. Use managed databases unless you have a strong reason — undifferentiated heavy lifting. |
| **Serverless (Lambda, Cloud Run, Cloudflare Workers)** | Run-on-demand functions | Cold starts, execution time limits, and **stateless-by-design** constrain what fits. Workers run on V8 isolates — tiny cold starts but a smaller API surface than Node. |
| **CDN (Cloudflare, Fastly, CloudFront)** | Edge caching of static + dynamic content | Cache invalidation is one of the two hard problems in CS. Stale-while-revalidate gives both freshness and speed. |
| **Observability (Prometheus + Grafana, Datadog, OpenTelemetry, Sentry)** | Metrics, logs, traces, errors | The "three pillars" — metrics for trends, logs for forensics, traces for cross-service flow. OpenTelemetry is the vendor-neutral standard worth investing in. |

**Key intricacy:** *The blast radius* of an infra change. A misconfigured load balancer or a bad migration can take down the entire stack regardless of how clean the upper layers are. Always have a rollback plan.

---

## 9. Cross-Cutting Concerns

Some things don't fit cleanly into one layer — they slice through all of them:

- **Authentication & Authorization** — Auth0, Clerk, Supabase Auth, Keycloak, custom JWT. Auth*entication* is "who are you"; Auth*orization* is "what can you do" — they are different.
- **Logging** — pino, winston, structlog. Always emit JSON in production for machine parsing.
- **Caching** — Redis, in-memory LRU, HTTP cache headers, CDN. Decide: cache at which layer? Closer to user = faster, harder to invalidate.
- **Validation** — Zod, Yup, Pydantic, Joi. Validate at the **boundary** (API entry, DB exit). Internal code can trust validated data.
- **Secrets management** — Vault, AWS Secrets Manager, doppler. Never commit `.env` files; use `.env.example`.

---

## 10. How the Layers Talk

```
User
  │
  ▼
[Presentation]  ── HTTP/WebSocket ──▶  [API Gateway]
                                            │
                                            ▼
                                     [Business Logic]
                                            │
                                            ▼
                                     [Data Access]
                                            │
                                            ▼
                                       [Database]
```

- **Synchronous**: HTTP, gRPC — caller waits for response. Simple but couples availability.
- **Asynchronous**: Message queues (RabbitMQ, Kafka, SQS), pub/sub. Decouples producers and consumers; introduces eventual consistency.
- **Streaming**: WebSockets, Server-Sent Events, gRPC streaming — for real-time updates.

---

## 11. Practical Heuristics

- **Start with fewer layers.** A 3-tier app (UI + API+logic + DB) is plenty for most projects. Add layers only when pain forces you.
- **Test each layer in isolation.** Business logic should have unit tests with no DB. API endpoints should have integration tests with a real DB.
- **Don't leak the database into the API.** Returning raw ORM rows ties your wire format to your schema — every column rename becomes a breaking API change.
- **One layer's "primary key" is another's "implementation detail."** The user doesn't care about your `users.id` UUID; expose what's meaningful.
- **Write code that doesn't import what it doesn't need.** If your business logic file imports `express`, the layers are leaking.

---

## 12. Quick Reference: Picking Tools by Layer

| Layer | Safe default (2026) | When to deviate |
|-------|---------------------|-----------------|
| Presentation | React + TypeScript + Vite + Tailwind | Use Svelte/Solid for perf-critical apps; Next.js when SEO + SSR matter |
| API | Fastify (Node) or FastAPI (Python) | gRPC for service-to-service; tRPC for full-stack TS |
| Business | Plain functions/classes in app language | DDD/Clean Arch when domain complexity warrants it |
| Data Access | Prisma or Drizzle (TS), SQLAlchemy (Py) | Raw SQL + sqlc when perf matters most |
| Database | PostgreSQL | DynamoDB for serverless scale; ClickHouse for analytics |
| Infra | A managed PaaS (Render, Fly.io, Railway) | Kubernetes when you have ops headcount |
| Observability | OpenTelemetry + Grafana | Datadog/Sentry when you'd rather pay than operate |
