# ⚡ Caching in System Design

## 1. Overview

**Caching** is a performance optimization technique in which frequently accessed data is stored in a temporary storage layer called a **cache**.

Since cache storage is faster than the primary data source (such as a database), caching helps:

* Reduce data retrieval time
* Improve system performance
* Decrease server load
* Enhance user experience

Caching is widely used in distributed systems to optimize response time and scalability.

---

## 2. Real-World Analogy: Bookstore

Consider a bookstore that places its most popular books in a **“Bestseller” section** near the entrance.

Instead of searching through the entire store:

* Customers can directly access popular books quickly
* This reduces search time
* Improves customer satisfaction

Similarly, in software systems, frequently requested data is stored in cache for faster access.

---

## 3. Role in Web Applications

In web applications, caching is implemented by introducing a **cache layer** between the server and the database.

### Request Flow:

1. Client sends request for data.
2. Server checks if data exists in cache.
3. If found (**Cache Hit**) → Data is returned immediately.
4. If not found (**Cache Miss**) →

   * Data is fetched from the database
   * Stored in cache for future use
   * Returned to the client

This reduces repeated database queries.

---

## 4. Types of Cache

### 🔹 Memory Cache

**Definition:**
Cache stored in RAM for faster data retrieval.

**Examples:**

* Redis
* Memcached

**Use Cases:**

* Session data
* Authentication tokens
* Frequently accessed database queries

---

### 🔹 Disk Cache

**Definition:**
Cache stored on disk storage.

**Examples:**

* Browser cache
* Operating system disk cache

**Use Cases:**

* Web pages
* Images
* Large media files

---

## 5. Caching in Real World – Instagram Example

When you open Instagram, a lot of data loads:
Profile pictures,Posts,Reels,Comments,Followers count
If Instagram fetched this from the database every time → it would be very slow.

So caching is used.

🔹 1. Memory Cache (Server-Side RAM)

Suppose:

You open a popular celebrity’s profile.
Millions of users are requesting:

Followers count
Profile info
Recent posts

Instead of:

App → Database → Send data


Instagram does this:
App → Server → Check Memory Cache (Redis)

If data exists:

✔ It is returned directly from server RAM
✔ No database call
✔ Faster response

So here:

👉 Data is stored in Instagram’s server RAM,
❌ NOT on your phone’s RAM.

🔹 2. Disk Cache (Client-Side Storage)

Now suppose:
You open a reel or someone’s profile picture.

Instagram:
Downloads the image/video once
Stores it on your phone storage
Next time:
You open same profile again


Instead of downloading again from server:

✔ Instagram loads it from
Your phone’s disk storage (cache)

That’s why:

Profile pictures load instantly
Previously viewed reels open faster

RAM is used to store frequently accessed data because it is fast but has limited storage, while disk storage is used for large data like images and videos since it has more space but is slower.

## 6. Cache Hit and Cache Miss

### Cache Hit

Occurs when requested data is found in cache.

* Faster response time
* Reduced database load

---

### Cache Miss

Occurs when requested data is not found in cache.

* Data must be fetched from database
* Slower response time

---

### Cache Hit/Miss Ratio

* A **high cache hit ratio** indicates efficient caching.
* A **low cache hit ratio** suggests ineffective cache utilization.

Efficient caching improves system performance and reduces backend workload.

---

## 7. Cache Eviction Policies

Since cache has limited capacity, older or less useful data must be removed when full.
This process is known as **Cache Eviction**.

---

### 🔹 Random Eviction

Removes items randomly.

* **Advantage:** Simple
* **Limitation:** Important data may be removed

---

### 🔹 FIFO (First-In-First-Out)

Removes oldest stored data.

* **Advantage:** Easy implementation
* **Limitation:** Frequently accessed data may be removed

---

### 🔹 LFU (Least Frequently Used)

Removes least accessed data.

* **Advantage:** Retains frequently used data
* **Limitation:** Requires tracking usage frequency

---

### 🔹 LRU (Least Recently Used)

Removes data not accessed recently.

* **Advantage:** Commonly used and effective
* **Limitation:** Requires tracking access order

---

## 8. Challenges of Caching

### 🔸 Staleness

Cached data may become outdated if the original data changes.
Sometimes cached data becomes outdated when the original data is updated.
Users may see old or incorrect information from the cache.
Cache needs to be updated regularly to avoid this.

Example
Database followers count = 501
Cache still shows = 500

---

### 🔸 Overhead

Caching requires additional memory and processing power.
Caching requires extra memory and processing power.
The system has to manage both cache and database.
This increases resource usage and system complexity.

---

### 🔸 Consistency

Cached data must remain synchronized with the primary data source.
Cached data should always match the database data.
If not updated properly, users may see different values.
Keeping cache and database data the same is important.

Example:
Database = 501
Server 1 cache = 501
Server 2 cache = 500
Different users see different values → this is Consistency issue

---

Staleness: Cache is outdated
Consistency: Data is different across system components

## 9. Summary

Caching:

* Reduces database load
* Improves response time
* Enhances scalability
* Optimizes system performance

However, it must be managed carefully to maintain data consistency and prevent outdated information.

---

