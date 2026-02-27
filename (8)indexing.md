
---
# 📑 Indexing in System Design

## 🧭 Overview

**Indexing** is a database optimization technique used to improve the speed of data retrieval operations.

Instead of scanning the entire dataset, an index allows the database to quickly locate the required data by maintaining a structured reference to its location.

Indexing is essential for building high-performance and scalable data-driven applications.

---

## 📘 What is Indexing?

Indexing works like a **shortcut mechanism** that helps databases find data efficiently.

Without indexing:

* The database performs a full table scan.
* Every record must be checked sequentially.

With indexing:

* The database directly jumps to the required data location.
* Query execution becomes significantly faster.

---

## 🏪 Real-World Analogy

### Bookstore Example

Imagine searching for a book in a large bookstore:

* Without an index → you check every shelf.
* With an index → you look at the category guide and go directly to the correct aisle.

The index reduces search time and improves efficiency.

---

### 📚 Book Index Example

At the end of a book, the index lists topics with page numbers.

Instead of flipping through all pages, you:

1. Search the term in the index.
2. Navigate directly to the correct page.

Databases use indexing in a similar way.

---

Your notes are **already good conceptually**, but the flow can be made clearer by improving **structure, spacing, and visual hierarchy** — not changing your wording.
I will **keep your wording**, only organize it professionally and add tiny clarifying connectors where needed.

Here is the **same content, structured for readability (GitHub-ready)** 👇

---

## 🗂️ Types of Database Indexes

---

### 📘 Imagine This Database Table

**`users` table**

| id | name | email                                   | city   |
| -- | ---- | --------------------------------------- | ------ |
| 1  | Ali  | [ali@gmail.com](mailto:ali@gmail.com)   | Delhi  |
| 2  | John | [john@gmail.com](mailto:john@gmail.com) | Mumbai |
| 3  | Sara | [sara@gmail.com](mailto:sara@gmail.com) | Delhi  |

---

### ⚙️ Why Indexing Exists

When a column is searched frequently, the developer creates an index on that column to make queries faster.
The database then reads all existing data once and creates a shortcut list that helps it directly find rows instead of checking every record during searches.

---

# 🔹 1. Primary Index (Primary Key Index)

A **Primary Key** is a column that uniquely identifies each row inside one table.

👉 Every row must be uniquely identified.
The database automatically creates an index on the primary key.

**Here:**

```
id = Primary Key
```

---

### What database creates

```
1 → Row of Ali
2 → Row of John
3 → Row of Sara
```

---

### When you search

```
Find user where id = 2
```

**Database:**

```
Check index → see 2 → go directly to John
```

✅ Fast because every ID is unique.

---

### Real-World Example

In a hospital system, searching a patient using their unique Patient ID directly opens the correct record because the database uses a primary index built on that ID.

---

# 🔹 2. Secondary Index

Email can be unique, but a primary key must stay permanent and efficient.
Databases use an ID as the primary key because it never changes (`id → always John`), while an email can change
(`john@gmail.com → john12@gmail.com`), so email is kept as a secondary index for searching.

---

### What it means

A **Secondary Index** is an additional index created on a column other than the primary key to improve search performance.

While the primary key uniquely identifies each record, secondary indexes allow fast searching using commonly queried fields such as email, username, or name.

---

### Example → email

**Database builds**

```
ali@gmail.com   → Ali
john@gmail.com  → John
sara@gmail.com  → Sara
```

---

### When you search

```
Find user where email='john@gmail.com'
```

**Database:**

```
Check email index → jump directly to John
```

✅ Faster search without scanning table.

---

### Simple meaning

```
Primary index   = search by ID
Secondary index = search by other fields (email, name, city)
```

---

## ✅ Why This Version Is Better (What Changed — Not Content)

* Clear learning flow: **Table → Why → Primary → Secondary**
* Visual separation of concepts
* Easier scanning for readers worldwide
* GitHub-friendly formatting
* No wording changes — only structure

---

## 🔹 3. Composite Index

What it means
Shortcut built using multiple columns together.

You search:
Find user where name = 'Sara' AND city = 'Delhi'

Database must do:

Check row 1 → not Sara ❌
Check row 2 → not Sara ❌
Check row 3 → Sara + Delhi ✅

It checks rows one by one.
Slow for millions of rows.

Composite Index Idea

We create an index using both columns together:
CREATE INDEX name_city_index ON users(name, city);
Database builds shortcut like:

(name, city) INDEX

(Ali, Delhi)   → row 1
(John, Mumbai) → row 2
(Sara, Delhi)  → row 3

Instead of searching each column separately or scanning rows one by one, a composite index creates a single shortcut using multiple columns together, so the database can directly find data when queries use those columns at the same time.

Example query:

Find user where name='Sara' AND city='Delhi'
Database builds shortcut like:
(Ali, Delhi) → Row
(John, Mumbai) → Row
(Sara, Delhi) → Row
Search happens

Database directly finds:
(Sara, Delhi) → correct row

✅ Faster when query uses multiple conditions.

Simple meaning
Composite index = shortcut using combination of columns.

---

## 🔹 4. Hash Index
What it means

Database converts value into a special code (hash).

Example internally
john@gmail.com → CODE_X12 → John's row
When you search
Find user where email='john@gmail.com'

Database:

Convert email → CODE_X12
Jump directly to row

✅ Extremely fast exact match.

❌ Cannot handle ranges like:

city > 'Delhi'
Simple meaning

Hash index = super-fast shortcut for exact value search.

---

An index stores column values in an organized structure so the database can search efficiently.
It maps each value to the location of the corresponding row in the table.
Using this structure, the database quickly navigates to the data instead of scanning all records.
When data is inserted, updated, or deleted, the index must also be updated so the mappings remain accurate and do not point to invalid data locations.

                           OR

Indexing is like creating a tree structure that organizes data into branches based on indexed values, allowing the database to quickly navigate to the required record.
Each branch helps narrow down where the data exists instead of searching everything.
When data is added or deleted, the tree must also be updated by adding or removing branches so the structure continues to point to the correct records.

---

## ✅ Advantages of Indexing

* Faster query performance
* Improved application responsiveness
* Efficient searching and filtering
* Reduced database workload during reads

---

Challenges of Indexing

Indexing improves query performance, but it also introduces operational costs.
Understanding these trade-offs is important when designing scalable database systems.

🔸 1. Extra Write Operations
✅ What it means

Indexes must stay synchronized with the table data. Whenever a record changes, related indexes must also be updated.
Index = value → location mapping for fast data access.

⚙️ How it happens

When a new record is inserted or updated:
Database writes data to the main table.
Database updates every index connected to that column.
Each index structure is reorganized internally.

❓ Why it happens

Indexes act as shortcut structures. If the shortcut is not updated, searches would return incorrect results.

📘 Example

Adding a new user:
INSERT INTO users VALUES (4, 'Rahul', 'rahul@gmail.com');

Database updates:
Users table
Primary index (id)
Secondary index (email)

🎯 Impact

Faster read operations ✅
Slower write operations ❌

🔸 2. Unused Indexes

✅ What it means
Indexes that are never used by queries still consume storage and processing resources.

⚙️ How it happens

Even if no query uses an index, the database still:
Stores the index structure
Updates it whenever data changes

❓ Why it happens

The database cannot assume an index is unnecessary because it was explicitly created by the developer.

📘 Example

An index created on city:
CREATE INDEX city_index ON users(city);
If the application never searches by city, the index still gets updated for every insert or update.

🎯 Impact

Wasted storage space
Additional processing overhead

🔸 3. Management Complexity
✅ What it means

Too many indexes increase system complexity and reduce overall performance.

⚙️ How it happens

Each data modification must update multiple indexes simultaneously.
Example indexes:

email
username
city
phone
status

Updating one record triggers updates across all indexes.

## 🧠 When to Use Indexing

Index columns that are:

* Frequently searched
* Used in filtering (`WHERE`)
* Used in sorting (`ORDER BY`)
* Used in joins

Avoid indexing columns that change frequently.

---

## 🎯 Interview-Ready Definition

> Indexing is a database optimization technique that creates a structured reference to data locations, enabling faster query execution by avoiding full table scans.

---
