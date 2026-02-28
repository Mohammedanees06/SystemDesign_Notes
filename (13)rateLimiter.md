# 🚦 Rate Limiter in System Design

A **Rate Limiter** controls how many requests a user or system can make within a specific time window to protect services from overload, abuse, or unfair usage.

It ensures **system stability, fairness, and security**.

---

## 📘 What is Rate Limiting?

Think of a rate limiter like a **bouncer at a club door** — it lets people in up to a limit, then stops new ones until there's space again.

In technical terms, rate limiting restricts the number of requests allowed from a client within a defined period.

**Examples:**

- Max **100 requests per minute per user**
- Max **5 login attempts per minute**

If the limit is exceeded → the request is **rejected**.

```
User sends 100 requests → ✅ All allowed
User sends 101st request → ❌ Blocked (429 Too Many Requests)
```

---

## 🧠 Simple Definition

> Rate Limiter = A traffic controller that prevents too many requests from overwhelming a system.

---

## 🎯 Why Do We Need Rate Limiting?

| Problem | How Rate Limiting Helps |
| ------- | ----------------------- |
| Server overload | Caps maximum incoming requests |
| DDoS attacks | Blocks flood of malicious traffic |
| Brute-force login | Limits repeated login attempts |
| Unfair usage | Ensures equal access for all users |
| API cost control | Prevents runaway usage bills |
| System performance | Keeps response times stable |

---

## ❓ Interview Requirement Gathering (VERY IMPORTANT)

Before designing, clarify requirements with the interviewer.

### 1️⃣ Functional Requirements

Ask:

- What should be limited?
  - Per user?
  - Per IP?
  - Per API key?
- What action when limit exceeds?
  - Block request?
  - Delay request?
  - Queue request?
- Is limit global or endpoint-specific?

### 2️⃣ Non-Functional Requirements

- High availability
- Low latency
- Distributed support
- Accuracy vs performance trade-off
- Horizontal scalability

### 3️⃣ Traffic Questions

- Expected requests per second?
- Number of users?
- Burst traffic allowed?
- Global or regional limits?

---

## ⚠️ Why Rate Limiting is Done on SERVER SIDE (Important Interview Question)

This is one of the most common interview questions. Here's the clearest way to understand it.

### ❌ Client-Side Rate Limiting (Wrong Approach)

Client-side checks run inside the browser, mobile app, or frontend code.

**The problem:** Users have full control over the client. They can:

- Modify JavaScript in the browser
- Use tools like Postman or curl
- Write bots or scripts
- Simply disable frontend checks

**Example — what goes wrong:**

```
Website allows 5 login attempts.
Frontend shows error after 5 tries.

Attacker ignores the UI and sends requests directly:

curl -X POST /login   ← 6th attempt  ✅ reaches server
curl -X POST /login   ← 7th attempt  ✅ reaches server
curl -X POST /login   ← 100th attempt ✅ reaches server

❌ Client protection completely bypassed.
```

---

### ✅ Server-Side Rate Limiting (Correct Approach)

The server checks the request count **before** doing any processing.

```
Client → Server → Rate Limiter Check → API Logic
                       ↓
               Limit exceeded?
               Yes → ❌ Return 429
               No  → ✅ Continue
```

Even if an attacker bypasses the UI entirely, the server still blocks the request.

**Real Example — Instagram login:**

```
Attacker sends 100 login requests via curl
       ↓
Frontend warning → ignored by attacker
       ↓
Server rate limiter → ❌ blocks after limit
       ↓
Attacker is stopped regardless of UI
```

> Only the server has the authority to enforce limits. The client is just a suggestion.

---

## 🧩 Where Rate Limiter Lives in Architecture

```
Client (Browser / Mobile App)
          ↓
    Load Balancer
          ↓
  API Gateway / Middleware  ← ⭐ Rate Limiter sits here
          ↓
   Application Servers
          ↓
       Database
```

**Common placements:**

- **API Gateway** — most common (e.g. AWS API Gateway, Kong)
- **Reverse proxy** — Nginx, HAProxy
- **Middleware layer** — inside the app itself
- **CDN edge layer** — closest to the user, fastest blocking

---

## ⚙️ Types of Rate Limiting Algorithms

---

### 🔹 1. Fixed Window Counter

Divides time into fixed buckets (e.g. every 60 seconds) and counts requests in each bucket.

```
Minute 1: [■■■■■■■■■■] 10 requests → ✅ allowed
Minute 2: [■■■■■■■■■■■■■■■] 15 requests → ❌ 5 blocked
Minute 3: [■] counter resets → ✅ allowed again
```

- ✅ Simple and fast
- ❌ **Burst problem:** A user can send 100 requests at 0:59 and 100 more at 1:01, effectively sending 200 requests in 2 seconds

---

### 🔹 2. Sliding Window Log

Stores the **exact timestamp** of every request. When a new request arrives, it counts how many requests happened in the last N seconds.

```
Request arrives at 1:05:30
Look back 60 seconds → count requests from 1:04:30 to 1:05:30
If count < limit → ✅ allow
If count ≥ limit → ❌ reject
```

- ✅ Very accurate — no burst problem
- ❌ High memory usage (must store every timestamp)

---

### 🔹 3. Sliding Window Counter (Optimized)

A smarter version that combines two fixed windows mathematically to approximate a sliding window — without storing every timestamp.

```
Current window count + (Previous window count × overlap %) = Effective count
```

- ✅ Accurate enough for production
- ✅ Memory efficient
- ✅ Used widely in real systems (e.g. Cloudflare)

---

### 🔹 4. Token Bucket ⭐ (Most Popular)

Imagine a bucket that holds tokens. Each request uses one token. Tokens refill at a steady rate.

```
Bucket capacity = 10 tokens
Refill rate     = 2 tokens/sec

Timeline:
t=0  → Bucket: [■■■■■■■■■■] 10 tokens
t=0  → 5 requests → Bucket: [■■■■■] 5 tokens left
t=1  → +2 refill  → Bucket: [■■■■■■■] 7 tokens
t=1  → 3 requests → Bucket: [■■■■] 4 tokens left

User sends 11th request when bucket is empty → ❌ rejected
```

- ✅ Allows short bursts (uses saved-up tokens)
- ✅ Controls average rate over time
- ✅ Simple to implement with Redis

---

### 🔹 5. Leaky Bucket

Requests go into a queue (the bucket) and are processed at a **constant rate** — like water leaking out of a hole.

```
Requests IN  →  [■■■■■■■■]  →  Processed at fixed rate OUT
                   Queue
               (overflow = dropped)
```

- ✅ Produces smooth, consistent output traffic
- ❌ Queued requests may be delayed
- **Use case:** Traffic smoothing (e.g. video streaming, billing systems)

---

### 📊 Algorithm Comparison

| Algorithm | Burst Allowed | Memory | Accuracy | Best For |
| --------- | ------------- | ------ | -------- | -------- |
| Fixed Window | ⚠️ Edge burst | Low | Medium | Simple APIs |
| Sliding Window Log | ✅ No burst | High | High | Strict limits |
| Sliding Window Counter | ✅ Controlled | Low | High | Production systems |
| Token Bucket | ✅ Yes | Low | High | General APIs |
| Leaky Bucket | ❌ No | Medium | High | Traffic smoothing |

---

## 🏗️ High-Level Design

```
Incoming Request
       ↓
Rate Limiter Service
       ↓
Generate Key (userId + endpoint)
       ↓
Check Counter in Redis
       ↓
  Limit exceeded?
  ├── No  → ✅ Increment counter → Forward to API Logic
  └── Yes → ❌ Return HTTP 429 (Too Many Requests)
```

---

## 🧱 Components Needed

### 🔹 Counter Store

Stores how many requests each user has made.

**Why Redis?**

- Runs in memory → extremely fast
- Supports atomic increment (`INCR` command)
- Built-in key expiry (auto-resets counters)

```
Redis key:   rate_limit:user123:/login
Redis value: 47  (requests made so far this window)
Redis TTL:   13s (time until window resets)
```

### 🔹 Identifier Key

Used to track limits per user, IP, or endpoint:

```
rate_limit:{userId}:{endpoint}
rate_limit:{ipAddress}
rate_limit:{apiKey}:{endpoint}
```

### 🔹 Time Tracking

Used to reset counters or calculate sliding windows. Redis TTL handles this automatically for fixed windows.

---

## 📦 Complete Request Flow (Redis-Based)

```
1. Request arrives → GET /api/feed (user: john)

2. Generate key → rate_limit:john:/api/feed

3. Redis INCR rate_limit:john:/api/feed
   └── If key is new → also set TTL = 60s

4. Redis returns count = 47

5. Check: 47 < 100 (limit)?
   └── Yes → ✅ forward request to app server

6. Next request → count = 101
   └── 101 > 100 → ❌ return 429
       Add header: Retry-After: 13s
```

---

## 📱 Real-World Examples

| Platform | Rate Limit | Purpose |
| -------- | ---------- | ------- |
| Instagram | 200 actions/hour | Prevent spam bots |
| GitHub API | 5000 requests/hour | Fair API usage |
| Banking Apps | 5 login attempts/min | Stop brute-force |
| Google Maps API | Quota per day | Cost control |
| Twitter/X API | Varies by tier | Monetization |

---

## ⚠️ Challenges in Rate Limiting

### 🔸 Distributed Systems

Multiple servers each have their own memory — if they don't share counters, a user can hit each server independently and exceed the real limit.

```
❌ Without shared store:
User → Server 1 → local count: 5  ← thinks limit not hit
User → Server 2 → local count: 5  ← thinks limit not hit
User → Server 3 → local count: 5  ← thinks limit not hit
Total real requests: 15 ← limit bypassed!

✅ With centralized Redis:
All servers → Redis → shared count: 15 → ❌ blocked
```

### 🔸 Clock Synchronization

Time-based algorithms require all servers to agree on the current time. Even a few milliseconds of drift can cause inconsistencies at window boundaries.

### 🔸 Hot Keys

A very popular user or endpoint causes millions of reads/writes to the same Redis key — creating a bottleneck.

**Solution:** Use Redis Cluster to shard keys across nodes.

### 🔸 False Blocking

Over-aggressive limits hurt legitimate users. Always monitor metrics and tune limits based on real traffic patterns.

---

## ✅ Best Practices

- Use **Redis** or an in-memory datastore for counters
- Apply limits **per endpoint**, not just globally
- Always return **HTTP 429 Too Many Requests** when blocked
- Add a **`Retry-After`** header so clients know when to retry
- **Log and monitor** rate-limit hits to detect attacks and tune limits
- Use **Token Bucket** for general APIs — it handles bursts naturally
- Never rely on client-side checks alone

---

## 🎯 Key Takeaway

Rate limiting protects backend systems by controlling request volume at the **server level**, ensuring fairness, stability, and security.

---

## ⭐ Simple Memory Line

👉 Rate Limiter = Traffic control system for APIs.