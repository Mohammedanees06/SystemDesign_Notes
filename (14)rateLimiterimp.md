

# ❓ Interview Requirement Gathering for Rate Limiter

> **This is the FIRST thing you do in a system design interview.** Never jump into design. Always ask questions first. It shows maturity and real-world thinking.

---

## 🧠 Why Requirement Gathering Matters

Interviewers **intentionally** keep the problem vague. They want to see if you can:

- Ask the right clarifying questions
- Narrow down scope before designing
- Think about edge cases early
- Distinguish between what's needed vs what's nice-to-have

> A candidate who asks smart questions before designing **always** scores higher than one who immediately starts drawing boxes.

---

## 1️⃣ Functional Requirements

> *"What exactly should the rate limiter DO?"*

These define the **core behavior** of the system.

### 🔹 Q1: What should be limited?

Ask the interviewer: *"What entity are we tracking limits for?"*

| Option | When to Use | Example |
|--------|------------|---------|
| **Per User** | Authenticated APIs | `user:john` → max 100 req/min |
| **Per IP Address** | Public/unauthenticated APIs | `ip:192.168.1.1` → max 50 req/min |
| **Per API Key** | Third-party developer APIs | `key:abc123` → max 5000 req/hour |
| **Per Device** | Mobile apps | `device:xyz` → max 30 req/min |
| **Combination** | High-security systems | `user:john + ip:192.168.1.1` |

```
💡 Why this matters:

If you rate limit by IP only:
  → Multiple users behind the same office Wi-Fi share one IP
  → One heavy user blocks everyone else in that office ❌

If you rate limit by user only:
  → An unauthenticated attacker has no user ID
  → They bypass the limiter entirely ❌

Best practice: Use BOTH — IP for unauthenticated, User ID for authenticated
```

**What to say in interview:**
> *"I'd suggest we rate limit by user ID for authenticated endpoints and by IP address for public/unauthenticated endpoints. For third-party integrations, we'd use API keys."*

---

### 🔹 Q2: What happens when the limit is exceeded?

Ask the interviewer: *"Should we hard block, delay, or queue excess requests?"*

| Strategy | Behavior | Use Case |
|----------|----------|----------|
| **Hard Block** | Immediately reject with `429` | Most common — APIs, login |
| **Throttle/Delay** | Slow down the response | Streaming, download APIs |
| **Queue** | Hold and process later | Background jobs, webhooks |
| **Degrade** | Return partial/cached data | Search results, feeds |

```
Example — Hard Block:

Request #101 arrives (limit = 100/min)
       ↓
Rate Limiter → ❌ 429 Too Many Requests
Response headers:
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 0
  Retry-After: 23    ← seconds until window resets
```

```
Example — Throttle/Delay:

Request #101 arrives (limit = 100/min)
       ↓
Rate Limiter → ⏳ Hold for 2 seconds, then process
       ↓
Response returned (but slower)

User experience: "API is slow" instead of "API is broken"
```

```
Example — Queue:

Request #101 arrives (limit = 100/min)
       ↓
Rate Limiter → 📥 Push to message queue (Kafka/RabbitMQ)
       ↓
Worker picks it up when capacity is available
       ↓
Response sent later (async) or via webhook callback
```

**What to say in interview:**
> *"For most APIs, I'd hard block with a 429 response and include Retry-After headers. For critical operations like payment webhooks, I'd prefer queuing so we don't lose data."*

---

### 🔹 Q3: Is the limit global or endpoint-specific?

Ask the interviewer: *"Should the limit apply to ALL endpoints combined, or separate limits per endpoint?"*

| Type | How It Works | Example |
|------|-------------|---------|
| **Global** | One shared counter for all endpoints | User gets 1000 req/min total across everything |
| **Per Endpoint** | Separate counter for each endpoint | `/login` → 5/min, `/api/feed` → 100/min, `/search` → 30/min |
| **Tiered** | Different limits for different user plans | Free → 100/hr, Pro → 10,000/hr, Enterprise → unlimited |

```
Example — Why per-endpoint matters:

Global limit = 100 req/min

User loads dashboard → 80 requests (fetching feed, notifications, profile)
User tries to login → only 20 requests left
User locked out of login because they browsed too much ❌

Per-endpoint limits:
  /api/feed    → 100 req/min  ← browsing
  /login       → 5 req/min   ← security
  /api/search  → 30 req/min  ← moderate

Now login is always protected regardless of feed usage ✅
```

**What to say in interview:**
> *"I'd recommend per-endpoint limits for critical paths like login and payments, with a global fallback limit to prevent overall abuse."*

---

## 2️⃣ Non-Functional Requirements

> *"What quality attributes must the system have?"*

These define **HOW** the system should behave — not what it does, but how well it does it.

---

### 🔹 High Availability

> *"Can the rate limiter go down?"*

```
If rate limiter goes DOWN, two options:

Option A — Fail Open (allow all requests):
  Rate limiter down → all requests pass through → system might overload
  ✅ Users not affected
  ❌ No protection

Option B — Fail Closed (block all requests):
  Rate limiter down → all requests blocked → complete outage
  ✅ System protected
  ❌ Users can't access anything

Best practice: Fail OPEN with alerts
  → Better to risk some overload than block all users
  → Immediately alert the team to fix the rate limiter
```

**What to say in interview:**
> *"The rate limiter must be highly available. If it fails, I'd prefer fail-open behavior so users aren't blocked, combined with aggressive monitoring and alerts."*

---

### 🔹 Low Latency

> *"How fast should the rate limit check be?"*

```
Every single API request passes through the rate limiter.
If the check takes 100ms → every API call is 100ms slower.

Target: Rate limit check should take < 1-2ms

This is why we use Redis (in-memory):
  → INCR command: ~0.1ms
  → GET command:  ~0.1ms
  → Total check:  ~0.5ms ✅

If we used a database (MySQL/Postgres):
  → Query time: 5-50ms ❌
  → Under load: 100ms+ ❌❌
```

**What to say in interview:**
> *"The rate limiter sits in the critical path of every request, so it must add minimal latency — ideally sub-millisecond. That's why Redis is the standard choice."*

---

### 🔹 Distributed Support

> *"Will this run on a single server or multiple servers?"*

```
Single server → Easy, keep counters in local memory
Multiple servers → Need a shared counter store

Why distributed is tricky:

Without shared store:
  Server 1 sees:  count = 48   ← allows request
  Server 2 sees:  count = 48   ← allows request
  Server 3 sees:  count = 48   ← allows request
  Real total:     count = 144  ← limit was 100! 💥

With shared Redis:
  Server 1 → Redis INCR → count = 98  ← allows
  Server 2 → Redis INCR → count = 99  ← allows
  Server 3 → Redis INCR → count = 100 ← allows
  Server 1 → Redis INCR → count = 101 ← ❌ blocked ✅
```

**What to say in interview:**
> *"In a distributed environment, all servers must share the same counter store — typically a centralized Redis cluster — to ensure accurate global counting."*

---

### 🔹 Accuracy vs Performance Trade-off

> *"Do we need exact counts or are approximations okay?"*

```
Exact counting:
  → Every request is precisely tracked
  → Requires strong consistency (locks, atomic operations)
  → Slightly slower under high concurrency
  → Use for: login attempts, payment APIs

Approximate counting:
  → Off by 1-2% sometimes
  → Much faster, no locking needed
  → Use for: feed loading, search queries, general APIs

Example:
  Limit = 100 req/min
  Exact:       blocks at exactly 101st request
  Approximate: might allow 101-103 before blocking

  For a feed API → 103 vs 100 doesn't matter
  For login → those 3 extra attempts could be a security risk
```

**What to say in interview:**
> *"For security-sensitive endpoints like login, I'd prioritize accuracy. For general API traffic, a small approximation is acceptable in exchange for better performance."*

---

### 🔹 Horizontal Scalability

> *"Can we add more rate limiter instances as traffic grows?"*

```
Scaling approach:

       ┌──── Rate Limiter Instance 1 ────┐
       │                                  │
Users ─┼──── Rate Limiter Instance 2 ────┼──── Shared Redis Cluster
       │                                  │
       └──── Rate Limiter Instance 3 ────┘

Rate limiter instances are STATELESS → easy to add more
All state lives in Redis → scales via Redis Cluster sharding

Redis Cluster sharding:
  rate_limit:user_A  → Redis Node 1
  rate_limit:user_B  → Redis Node 2
  rate_limit:user_C  → Redis Node 3

  Each node handles a subset of keys → distributes load
```

**What to say in interview:**
> *"The rate limiter itself should be stateless — just a middleware or gateway layer. All state is stored in Redis, which can be horizontally scaled using Redis Cluster."*

---

## 3️⃣ Traffic & Scale Questions

> *"What kind of traffic are we dealing with?"*

These help you choose the right algorithm and infrastructure.

---

### 🔹 Expected Requests Per Second (RPS)

```
Why it matters — it decides your infrastructure:

100 RPS       → Single server + local memory is fine
10,000 RPS    → Need Redis, maybe single instance
100,000 RPS   → Need Redis Cluster + multiple rate limiter nodes
1,000,000 RPS → Need Redis Cluster + edge-level rate limiting (CDN)
```

**What to ask:**
> *"What's the expected peak RPS? Are there seasonal spikes (like Black Friday for e-commerce)?"*

---

### 🔹 Number of Users

```
Why it matters — it decides your memory needs:

Each user needs a Redis key:
  rate_limit:user123:/api/feed → ~50 bytes

  1,000 users    → ~50 KB   (trivial)
  1,000,000 users → ~50 MB  (still fine for Redis)
  100,000,000 users → ~5 GB (need Redis Cluster)

If tracking per endpoint too:
  100M users × 10 endpoints = 1 billion keys
  → Definitely need sharding
```

---

### 🔹 Burst Traffic Allowed?

```
This decides which algorithm to use:

Burst NOT allowed (strict, smooth output):
  → Use Leaky Bucket
  → Every request processed at constant rate
  → Example: payment processing, bank transactions

Burst ALLOWED (flexible, user-friendly):
  → Use Token Bucket ⭐
  → User can send 10 requests instantly if they saved up tokens
  → Example: social media feed loading, search API

No burst + exact accuracy needed:
  → Use Sliding Window Log
  → Example: login attempts, OTP verification
```

**What to ask:**
> *"Should the system allow short bursts of traffic, or do we need a strictly smooth rate?"*

---

### 🔹 Global or Regional Limits?

```
Global limit:
  User gets 100 req/min TOTAL across all regions
  All regions must sync to one counter → higher latency
  
  User in India → request → check Redis in US → 200ms latency ❌

Regional limit:
  User gets 100 req/min PER REGION
  Each region has its own Redis → low latency
  
  User in India → request → check Redis in India → 2ms ✅

  But: user could get 100/min in India AND 100/min in US = 200 total
  → Acceptable for most use cases

Hybrid:
  Regional Redis for fast checks + async sync to global counter
  → Best of both worlds but complex to implement
```

**What to ask:**
> *"Are users region-locked, or could they hit multiple regions? Do we need a single global counter or per-region limits?"*

---

## 🎯 Putting It All Together — Sample Interview Answer

> *"Before I start designing, let me clarify a few requirements:*
>
> *For functional requirements — I'll assume we're limiting per user ID for authenticated endpoints and per IP for unauthenticated ones. When limits are exceeded, we'll return a 429 with Retry-After headers. I'll set per-endpoint limits for sensitive paths like login, with a global fallback.*
>
> *For non-functional requirements — the system needs to be highly available with fail-open behavior, add sub-millisecond latency, work across distributed servers using shared Redis, and scale horizontally.*
>
> *For traffic — I'll design for approximately 100K RPS at peak, supporting 10M users, allowing short bursts using Token Bucket, with regional rate limiting and async global sync.*
>
> *Does that align with what you had in mind?"*

---

## ✅ Quick Reference Checklist

```
Before designing, confirm:

Functional:
  □ Limit by: User / IP / API Key / Combination?
  □ On exceed: Block / Delay / Queue?
  □ Scope: Global / Per-endpoint / Tiered?

Non-Functional:
  □ Availability: Fail open or fail closed?
  □ Latency: What's the budget? (aim < 1ms)
  □ Distribution: Single server or multi-region?
  □ Accuracy: Exact or approximate?
  □ Scalability: Expected growth?

Traffic:
  □ Peak RPS?
  □ Total users?
  □ Burst allowed?
  □ Global or regional?
```