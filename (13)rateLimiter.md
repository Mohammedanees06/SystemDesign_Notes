

# 🚦 Rate Limiter in System Design

A **Rate Limiter** controls how many requests a user or system can make within a specific time window to protect services from overload, abuse, or unfair usage — ensuring **system stability, fairness, and security**.

---

## 📘 What is Rate Limiting?

Think of a rate limiter like a **bouncer at a club door** — it lets people in up to a limit, then stops new ones until there's space again.

In technical terms, rate limiting restricts the number of requests allowed from a client within a defined period.

**Examples:**
- Max **100 requests per minute per user**
- Max **5 login attempts per minute**

```
User sends 100 requests   → ✅ All allowed
User sends 101st request  → ❌ Blocked (429 Too Many Requests)
```

> **Rate Limiter** = A traffic controller that prevents too many requests from overwhelming a system.

---

## 🎯 Why Do We Need Rate Limiting?

| Problem              | How Rate Limiting Helps            |
| -------------------- | ---------------------------------- |
| Server overload      | Caps maximum incoming requests     |
| DDoS attacks         | Blocks flood of malicious traffic  |
| Brute-force login    | Limits repeated login attempts     |
| Unfair usage         | Ensures equal access for all users |
| API cost control     | Prevents runaway usage bills       |
| System performance   | Keeps response times stable        |

---

## ❓ Interview Requirement Gathering

Before designing, clarify requirements with the interviewer.

### 1️⃣ Functional Requirements

- **What should be limited?** — Per user? Per IP? Per API key?
- **What action when limit exceeds?** — Block? Delay? Queue?
- **Is limit global or endpoint-specific?**

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

## ⚠️ Why Rate Limiting Must Be SERVER-SIDE

### ❌ Client-Side Rate Limiting (Wrong Approach)

Client-side checks run inside the browser, mobile app, or frontend code. The problem: **users have full control over the client.** They can modify JavaScript, use tools like Postman or curl, write bots, or simply disable frontend checks.

```
Website allows 5 login attempts.
Frontend shows error after 5 tries.

Attacker ignores the UI and sends requests directly:

curl -X POST /login   ← 6th attempt  ✅ reaches server
curl -X POST /login   ← 7th attempt  ✅ reaches server
curl -X POST /login   ← 100th attempt ✅ reaches server

❌ Client protection completely bypassed.
```

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

| Placement              | Examples                         |
| ---------------------- | -------------------------------- |
| API Gateway            | AWS API Gateway, Kong            |
| Reverse Proxy          | Nginx, HAProxy                   |
| Middleware Layer        | Inside the application itself    |
| CDN Edge Layer         | Closest to user, fastest blocking|

---

## ⚙️ Rate Limiting Algorithms

### 🔹 1. Fixed Window Counter

Divides time into fixed buckets (e.g., every 60 seconds) and counts requests in each bucket.

> **Real-life analogy:** A restaurant allows only 10 customers per hour, resetting the count every hour on the clock — at 1:00, 2:00, 3:00 exactly.
>
> ```
> 1:00 PM → 1:59 PM   → 10 customers entered ✅
> 2:00 PM              → counter resets to 0
> 2:00 PM → 2:59 PM   → 10 more customers ✅
> ```
>
> **The burst problem:** A clever group exploits the reset boundary:
> ```
> 1:58 PM → 10 customers rush in  ✅ (last 2 mins of window 1)
> 2:00 PM → 10 customers rush in  ✅ (first second of window 2)
>
> = 20 customers in 2 minutes 💥 overwhelmed
> ```
> The rule said 10/hour, but they got 20 in 2 minutes by exploiting the reset moment.

- ✅ Simple and fast
- ❌ **Burst problem** at window boundaries

---

### 🔹 2. Sliding Window Log

Stores the **exact timestamp** of every request. When a new request arrives, it counts how many requests happened in the last N seconds.

> **Real-life analogy:** Same restaurant, but now the rule is: *"No more than 10 customers who entered in the last 60 minutes at any point."*
>
> The staff keeps a diary — they write down the exact time every customer entered.
>
> ```
> Customer arrives at 2:45 PM
> Staff opens diary → counts everyone who entered after 1:45 PM
> Diary says: 7 people → ✅ let them in
>
> 10th customer at 2:50 PM → count = 10 → ✅ allowed
> 11th customer at 2:51 PM → count = 10 → ❌ blocked
>
> At 3:00 PM → the 1:45 PM entries are older than 60 mins
> → they fall out of the window → count drops → new customers allowed
> ```
>
> **The problem:** The diary gets huge. Every single entry must be stored and checked. At scale, storing every timestamp costs a lot of memory.

- ✅ No burst problem — perfectly accurate
- ❌ High memory usage at scale

---

### 🔹 3. Sliding Window Counter (Optimized)

A hybrid approach — combines fixed window simplicity with sliding window accuracy using weighted approximation.

> **Real-life analogy:** The staff doesn't write every customer's exact time. Instead, they keep two numbers: how many came *this hour* and how many came *last hour*.
>
> ```
> Last hour: 8 customers total
> This hour: 20 mins have passed (33% of the hour)
> Remaining from last hour that still count: 8 × 67% = ~5
>
> Effective count = 5 (leftover) + current hour count
> ```
>
> It's an approximation — not 100% perfect, but close enough and uses almost no memory.
> **This is what Cloudflare actually uses in production for billions of requests.**

- ✅ Low memory, high accuracy
- ⚠️ Approximate (not exact)

---

### 🔹 4. Token Bucket ⭐ (Most Popular)

A bucket holds tokens. Each request consumes one token. Tokens refill at a steady rate.

> **Real-life analogy:** A piggy bank holds a max of 10 coins. Every API request costs 1 coin. The bank auto-adds 2 coins every second.
>
> ```
> Start with 10 coins (full)       🪙🪙🪙🪙🪙🪙🪙🪙🪙🪙
>
> Send 5 requests → 5 coins spent  🪙🪙🪙🪙🪙
>
> 1 second passes → 2 coins added  🪙🪙🪙🪙🪙🪙🪙
>
> Send 3 more → 3 coins spent      🪙🪙🪙🪙
>
> Bank hits 0 → next request       ❌ rejected
> Wait 1 sec → 2 coins added → can send 2 requests again
> ```
>
> If idle for a while, you accumulate coins (up to max). So you can burst when needed, but can't abuse it forever because the bucket has a maximum size. Think of it like mobile data — you get 1GB/day, unused data rolls over up to a limit.

- ✅ Allows short bursts (uses saved-up tokens)
- ✅ Controls average rate over time
- ✅ Simple to implement with Redis

---

### 🔹 5. Leaky Bucket

Requests go into a queue (the bucket) and are processed at a **constant rate** — like water leaking out of a hole.

> **Real-life analogy:** A bucket with a small hole at the bottom. Water (requests) pours in from the top at any speed, but drips out at a fixed, constant rate.
>
> ```
>               requests pour in (any speed)
>                       ↓↓↓↓↓↓
>                 ┌──────────────┐
>                 │   waiting    │
>                 │   in line    │
>                 └──────┬───────┘
>                        │  drip drip drip
>                        ↓  (1 request/sec, always steady)
>                   processed output
> ```
>
> If too many pour in, the bucket overflows — extra requests are dropped.
>
> Think of a call center with 5 agents. Calls go into a hold queue. Agents answer at their own pace. If the queue is full, new callers get *"all agents busy, please call later."*

- ✅ Smooth, constant output rate
- ❌ No burst capability

---

### 📊 Algorithm Comparison

| Algorithm              | Burst Allowed   | Memory | Accuracy | Best For             |
| ---------------------- | --------------- | ------ | -------- | -------------------- |
| Fixed Window           | ⚠️ Edge burst   | Low    | Medium   | Simple APIs          |
| Sliding Window Log     | ✅ No burst     | High   | High     | Strict limits        |
| Sliding Window Counter | ✅ Controlled   | Low    | High     | Production systems   |
| Token Bucket           | ✅ Yes          | Low    | High     | General APIs         |
| Leaky Bucket           | ❌ No           | Medium | High     | Traffic smoothing    |

### 🎯 Algorithm Selection Guide

| Situation / Use Case                       | Best Algorithm           |
| ------------------------------------------ | ------------------------ |
| Simple internal API                        | Fixed Window             |
| Need exact accuracy, small scale           | Sliding Window Log       |
| High traffic, need accuracy + efficiency   | Sliding Window Counter   |
| Public API with bursty user behavior       | Token Bucket             |
| Output must be smooth and constant         | Leaky Bucket             |
| **Interview default answer**               | **Token Bucket ⭐**      |

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

## 🧱 Core Components

### 🔹 Counter Store (Redis)

Stores how many requests each user has made.

**Why Redis?**
- Runs in memory → extremely fast
- Supports atomic increment (`INCR` command)
- Built-in key expiry (auto-resets counters)

```
Redis key:   rate_limit:user123:/login
Redis value: 47   (requests made so far this window)
Redis TTL:   13s  (time until window resets)
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

| Platform        | Rate Limit           | Purpose             |
| --------------- | -------------------- | ------------------- |
| Instagram       | 200 actions/hour     | Prevent spam bots   |
| GitHub API      | 5000 requests/hour   | Fair API usage      |
| Banking Apps    | 5 login attempts/min | Stop brute-force    |
| Google Maps API | Quota per day        | Cost control        |
| Twitter/X API   | Varies by tier       | Monetization        |

---

## ⚠️ Challenges in Distributed Systems

### 🔸 Shared Counter Problem

Multiple servers with local counters allow users to bypass limits by hitting different servers.

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

> Rate limiting protects backend systems by controlling request volume at the **server level**, ensuring fairness, stability, and security.
>
> 👉 **Rate Limiter = Traffic control system for APIs.**