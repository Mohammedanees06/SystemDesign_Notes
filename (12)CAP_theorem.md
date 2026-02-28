# ⚖️ CAP Theorem in System Design

The CAP Theorem explains the fundamental trade-offs that distributed systems must make between **Consistency, Availability, and Partition Tolerance**.

It states that a distributed system can guarantee **only two out of the three properties at the same time**.

---

## 📘 What is CAP Theorem?

In a distributed system, it is impossible to simultaneously achieve Consistency, Availability, and Partition Tolerance during network failures.

When a network partition occurs, the system must choose between consistency and availability.

A network issue (network partition) occurs when servers in a distributed system cannot communicate with each other due to connection failures such as network outages, latency problems, or broken links.

* The servers are still running, but data exchange between them temporarily stops.
* This causes each server (node) to operate independently until the connection is restored.

---

## 🧩 The Three Properties

* **C — Consistency** → all servers show same latest data  
* **A — Availability** → every request gets a response  
* **P — Partition Tolerance** → system keeps working even if network breaks  

---

### 🔹 Consistency (C)

All servers return the same and latest data at any given time.

* Every read receives the most recent write.
* No stale or outdated data is returned.

**Example**

After updating your profile, every user immediately sees the updated information.

---

### 🔹 Availability (A)

Every request receives a response, even if some servers are failing.

* System always responds.
* Response may not contain the latest data.

**Example**

Website always loads, even during server issues.

---

### 🔹 Partition Tolerance (P)

The system continues operating even when communication between servers is broken.

* Network failures are tolerated.
* Nodes may not communicate temporarily.

**Example**

Servers in different regions lose connection but continue functioning independently.

---

## 🔄 Why Partition Tolerance is Mandatory

In real distributed systems, network failures are unavoidable.

Therefore:

**Modern distributed systems must always tolerate partitions (P).**

Partition Tolerance means the system continues running even when servers cannot communicate due to a network failure.

* It keeps servers operational so the system can choose Consistency (reject requests) or Availability (serve requests).
* If partition tolerance is not supported, a network failure would cause the entire system to shut down because nodes cannot coordinate safely.
* It is mandatory in distributed systems since network failures are unavoidable in real-world environments.

---

### Real Trade-off


Consistency vs Availability


---

## ⚖️ CAP Combinations

### ✅ CP (Consistency + Partition Tolerance)

* System prioritizes correct data over availability.
* Some requests may fail or wait.
* Ensures strong data accuracy.

**Examples**

* Banking systems
* Distributed databases requiring strict correctness

---

### ✅ AP (Availability + Partition Tolerance)

* System prioritizes availability over immediate consistency.
* Always responds.
* Data may temporarily differ across nodes.

**Examples**

* Social media feeds
* DNS systems

---

### ❌ CA (Consistency + Availability)

Possible only when no network partition exists.

Not realistic for distributed systems.

---

## 📱 Real-World Examples

| System Type      | CAP Choice | Reason                         |
|------------------|------------|--------------------------------|
| Banking Systems  | CP         | Data accuracy is critical      |
| Social Media     | AP         | Always available experience    |
| Single Server DB | CA         | No network partition           |

---

## 🧠 What Partition Tolerance REALLY Means

Partition tolerance does NOT mean servers can communicate.

It means the system keeps running even when communication is broken.

### Network Partition = network split


Server A ❌ network ❌ Server B


Servers are alive, but connection between them failed.

Partition tolerance means:

* ✅ servers don’t crash
* ✅ system keeps operating in some way

---

### Property Behavior During Partition

* **Partition Tolerance (P)** → system stays alive during network failure.
* **Consistency choice (CP)** → system rejects requests to keep data correct.
* **Availability choice (AP)** → system accepts requests even if data differs temporarily.

---

## 🌍 Banking Example (Step-by-Step)

Two bank servers:

* Bangalore Server
* Mumbai Server

Balance = ₹10,000

---

### 🚨 Network Failure


Bangalore ❌ Mumbai


They cannot sync.

Now you try withdrawing money in Mumbai.

---

### ⚖️ Bank’s Decision (CP System)

Bank says:

> “We must NEVER show wrong balance.”

So what do they do?

They STOP availability temporarily.

Mumbai server responds:


Transaction unavailable. Try later.


---

### ✅ How Consistency is Maintained

Important insight:

👉 Consistency is preserved by REFUSING operations, not by syncing.

They do NOT allow risky updates.

So:

* No new withdrawals allowed
* No conflicting data created
* Data stays correct everywhere

Consistency achieved by blocking writes.

---

## ⚠️ Important Insight

CAP applies only during network partitions.

When the network is healthy, systems can achieve all three properties.

---

## 🎯 Key Takeaway

During network failures, distributed systems must choose between:

* returning correct data (**Consistency**), or
* always responding (**Availability**).

---

## ⭐ Simple Memory Line

👉 **CAP = Choose any two during failure: Consistency, Availability, Partition Tolerance.