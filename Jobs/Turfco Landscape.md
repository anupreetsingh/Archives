# Turfco — (Full Stack Developer)

Turfco Landscape App System Design

A customer-facing web app for a landscaping company: a visitor describes their property and the service they want, gets an **instant price estimate** from a rule-based engine, creates an account, books the job, and pays a deposit online. Staff use an admin view to manage pricing rules and bookings.

## Key terms (defined before use)

- **MERN** — MongoDB (document database), Express (Node.js HTTP framework), React (frontend SPA), Node.js (JS runtime). One language across the stack.
- **SPA (Single-Page App)** — the React bundle loads once; navigation and rendering happen client-side, and the page talks to the backend only through JSON API calls.
- **JWT (JSON Web Token)** — a signed token the server issues at login; the client sends it on every request so the server can verify identity **without a session lookup** (stateless auth).
- **Rule-based cost estimation engine** — pricing logic expressed as **data** (rule documents in MongoDB: conditions + rate adjustments) instead of hardcoded `if/else`, evaluated server-side against the customer's inputs.
- **Stripe Checkout / webhook** — Stripe hosts the card-entry page (so card data never touches our servers → minimal PCI scope); a **webhook** is Stripe calling *our* API afterward to tell us, server-to-server, that the payment actually succeeded.
- **Reverse proxy (Nginx)** — the single public entry point; terminates TLS, serves the static React build, and forwards `/api/*` to the Node process.

## High-level architecture (request flow, start → end)

```
Browser (React SPA)
   │  HTTPS
   ▼
Nginx (TLS termination, static files, /api reverse proxy, rate limiting)
   │
   ▼
Node.js / Express API  ──►  MongoDB Atlas (users, rules, estimates, bookings, payments)
   │
   ├──►  Stripe (create Checkout session)          [outbound]
   ◄──┤  Stripe webhook → /api/webhooks/stripe     [inbound, signed]
   │
   └──►  Email service (booking confirmations)     [outbound]
```

Every user action follows the same thread: **React SPA → Nginx → Express route → middleware (auth, validation) → controller → MongoDB → JSON response → React re-render.** The three feature flows below are specializations of that thread.

## Flow 1 — Authentication

1. **Register:** `POST /api/auth/register` → validate input → hash password with **bcrypt** (never store plaintext) → insert `users` document with `role: "customer"`.
2. **Login:** `POST /api/auth/login` → bcrypt-compare → issue a short-lived **access JWT** (~15 min, kept in memory on the client) and a long-lived **refresh token** (httpOnly, Secure cookie — invisible to JS, so XSS can't steal it).
3. **Authenticated request:** client sends `Authorization: Bearer <JWT>` → an Express **auth middleware** verifies the signature and expiry, attaches `req.user`, and a **role middleware** gates admin routes (`role: "admin"` for rule editing, booking management).
4. **Refresh:** when the access token expires, `POST /api/auth/refresh` reads the cookie and issues a new access token — the user stays logged in without re-entering credentials.

Why JWT over server sessions here: the API stays stateless (any process restart or second instance needs no shared session store), which is the right trade at this traffic level.

## Flow 2 — Cost estimation (the rule engine)

**The core idea: pricing is data, not code.** Turfco's owners change prices seasonally; that must not require a redeploy.

1. Customer fills the estimate form: service type (mowing, aeration, fertilizing, cleanup…), lot size, frequency (one-time / weekly / biweekly), property details (slope, fenced, obstacles).
2. `POST /api/estimates` → validation → the engine loads the **active rule set** from the `rules` collection (cached in memory for 5 min — rules change rarely, so this cuts a DB round trip from the hottest endpoint).
3. Evaluation is a **deterministic pipeline**: start from the service's base rate → apply matching *quantity rules* (e.g. `$ per 1000 sq ft` tiers) → apply *multiplier rules* (slope +15%, weekly frequency ×0.9 discount) → apply *flat surcharges* (travel fee by postal-code zone) → clamp to a configured minimum job price.
4. Each rule document looks like:

   ```json
   {
     "service": "mowing",
     "type": "multiplier",
     "condition": { "field": "frequency", "op": "eq", "value": "weekly" },
     "factor": 0.9,
     "priority": 20,
     "active": true
   }
   ```

   Rules are sorted by `priority`, and the engine records **which rules fired** into the saved estimate — so every quoted price is auditable and reproducible.
5. The estimate document (inputs + line items + total + rule snapshot) is saved and returned; the SPA renders the price breakdown. Admins CRUD rules through the admin UI; a rule edit takes effect within the cache window, no deploy.

Why rule-engine over hardcoded pricing: (a) non-developers change prices via the admin UI, (b) the audit trail explains any quote, (c) testing is table-driven — feed inputs, assert totals.

## Flow 3 — Booking & payment

1. From an estimate, `POST /api/bookings` creates a booking with `status: "pending_payment"` and the **estimate total snapshotted** (a later rule change must not reprice an existing booking).
2. The API creates a **Stripe Checkout session** for the deposit (e.g. 25%) with the booking id in metadata, and returns the redirect URL; the customer pays on Stripe's hosted page.
3. **The webhook is the source of truth, not the browser redirect** (a user can close the tab after paying): Stripe calls `POST /api/webhooks/stripe`; the handler verifies the **webhook signature**, and on `checkout.session.completed` marks the payment `paid` and the booking `confirmed`, then sends the confirmation email.
4. The handler is **idempotent** — it records processed Stripe event ids, so a redelivered webhook can't double-confirm — and unhandled event types are acked with 200 so Stripe stops retrying.

## Data model (MongoDB collections)

| Collection  | Holds | Key indexes |
|---|---|---|
| `users` | email, bcrypt hash, role, profile | unique `email` |
| `rules` | pricing rules as documents (above) | `service + active + priority` |
| `estimates` | inputs, line items, total, fired-rules snapshot | `userId + createdAt` |
| `bookings` | estimate ref, snapshotted price, schedule, status | `userId`, `status + date` |
| `payments` | Stripe session/event ids, amount, status | unique `stripeEventId` (idempotency) |

Documents fit MongoDB well here: an estimate is a naturally nested object (inputs + line items), read and written as a unit — no joins needed on the hot path.

## Capacity & scaling honesty (1000 hits/day)

- 1000 hits/day ≈ **~0.01 req/s average**, maybe ~1–2 req/s at peak. A single Node process on one small VM (see Deployment below) with MongoDB Atlas free/shared tier handles this with huge headroom — Node's event loop comfortably serves thousands of req/s of I/O-bound work.
- So the design intentionally avoids: microservices, queues, Redis, Kubernetes, horizontal autoscaling. **The engineering judgment is knowing not to build them** — the complexity would cost more than it buys.
- What the scale *does* justify: Nginx rate limiting on `/api/auth/*` (brute-force protection), the in-memory rule cache, MongoDB indexes above, Atlas automated backups, PM2 for process restart on crash, and basic monitoring (uptime check + error logging with pino → log file / free tier of a log service).
- **Growth path** (if traffic 100×'d): PM2 cluster mode across cores → move rule cache to Redis → read replicas → split the estimator into its own service. Each step is incremental; nothing in the current design blocks it.

## Deployment

**The single instance = an AWS EC2 (or Lightsail) instance, with MongoDB on Atlas.** This is the specific answer to "where did it run":

- **One AWS EC2 t3.small (or Lightsail) instance** running Ubuntu — administered over **SSH**. Nginx and PM2 configured by hand; deploys via `git pull` + `pm2 reload` (near-zero downtime for a single process).
- **MongoDB Atlas** (shared tier) for the database — managed backups, no self-hosted DB to babysit on the same box.
- **Nginx** on the instance: TLS (Let's Encrypt/certbot auto-renewal), serves the built React bundle, proxies `/api/*` to Node, rate-limits `/api/auth/*`.
- **PM2**: process supervision (restart on crash, boot on reboot), log capture.

**Why a rented cloud VM and not an on-prem office machine:** Stripe webhooks require a publicly reachable, always-up HTTPS endpoint — an office box behind a consumer ISP means non-static IP, router port-forwarding, and outages that silently drop payment confirmations (a paid-on-Stripe / unconfirmed-in-app mismatch). Add no backups, no physical security on a machine touching payment flows, and the economics ($6–15/month for a VPS with snapshots) and the VM wins on every axis that matters. Naming **AWS + Atlas** on a resume also carries the highest keyword recognition.

Interview one-liner: *"A single small EC2 instance behind Nginx, MongoDB on Atlas — 1K hits/day doesn't justify more, and the growth path (PM2 cluster → Redis cache → read replicas) was known but deliberately not built."*

## Security checklist

- bcrypt password hashing; JWTs signed with a strong secret; refresh token in httpOnly+Secure cookie
- Input validation on every route (e.g. Joi/zod) — the estimator especially, since its inputs drive pricing
- Stripe webhook **signature verification**; card data never touches the server (hosted Checkout)
- CORS locked to the app origin; Helmet security headers; rate limiting on auth endpoints
- Role-based access middleware: customers can only read their own estimates/bookings; only admins mutate rules

## Resume bullets (Action–Method–Impact)

Chosen two — one for the estimator (the differentiator), one for auth + payments (the production-hardening story):

1. **Engineered a rule-based cost estimation engine** (Node.js/Express, MongoDB) that priced jobs from admin-configurable pricing rules with a full audit trail of applied rules, **replacing manual quoting with instant self-serve estimates and cutting quote turnaround from ~2 days to under a minute**.
2. **Secured and automated the booking pipeline** by implementing JWT authentication with refresh-token rotation and role-based access control, and integrating Stripe Checkout with signature-verified, idempotent webhooks — **enabling reliable online deposit collection across ~30K requests/month with zero duplicate charges**.

Alternates (swap in if a posting emphasizes different skills):

- **Designed and deployed the full MERN stack** behind an Nginx reverse proxy with TLS, rate limiting, and indexed MongoDB queries, **sustaining ~1K daily requests at sub-100 ms median API latency on a single low-cost instance**.
- **Reduced pricing-change lead time from a code deploy to a same-day admin edit** by modeling pricing logic as versioned rule documents in MongoDB with an in-memory cache, **letting non-technical staff run seasonal price updates independently**.

> **Before using:** verify/replace the numbers (2 days, 30K/month, sub-100 ms) with real ones — interviewers probe quantified claims. "~30K requests/month" is just 1000/day × 30; keep whichever framing sounds bigger honestly.

## What happened to the app

For the requirements at the time — self-serve quotes and online deposit collection — the system was well designed and right-sized. But I was the only technical person on the team, and after I left there was nobody to manage it: even a well-behaved custom app needs *someone* who can SSH into the server, renew certificates, update dependencies, or investigate a failed webhook.

For a non-technical team, "the website is doing something weird and nobody here can look at it" is a real operational risk. So they retired it and consolidated to a simple builder-style marketing site — which is what runs at turfcolandscape.com today, a pure lead-gen site with a "Start my project" CTA and no login, booking, payments, or quote tool. Phone calls and in-person quoting absorbed what the app used to do; honestly, the correct call once the maintainer was gone.

This is one of the most common arcs in small-business software: commission a custom app, later realize the ambition outgrew the need, and retire it. Knowing when custom software stops being worth its upkeep is the same judgment as deliberately not building microservices for 1K hits/day (see Capacity & scaling above).

## Likely interview follow-ups

- *Why MongoDB over Postgres?* → estimates/rules are nested documents read as a unit; no cross-entity transactions needed on the hot path; one JS type system end to end. (Concede: Postgres would also work fine — the honest answer is fit + team velocity, not a hard constraint.)
- *Why is the webhook the source of truth instead of the redirect?* → redirect is client-controlled and skippable; webhook is signed, server-to-server, retried by Stripe.
- *What happens if two rule edits conflict?* → rules apply by `priority`; estimates snapshot fired rules, so past quotes are immune to edits.
- *How would you scale it 100×?* → see growth path above — and lead with "measure first."
