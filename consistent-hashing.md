# Consistent Hashing

Consistent hashing is a technique used in distributed systems to **minimize the number of keys that need to be remapped** when nodes (servers) are added or removed.

It is commonly used in **distributed caches, databases, load balancers, and sharded systems**.

---

## Why Consistent Hashing Is Needed

### Traditional Hashing Problem

In a traditional hashing approach:

```text
server = hash(key) % N
```

Where:

* `N` = number of servers

**Problem:**

* If `N` changes (server added/removed)
* Almost **all keys get remapped**

This causes:

* Cache misses
* Heavy data movement
* Performance degradation

---

## What Is Consistent Hashing

**Definition:**

> Consistent hashing minimizes key redistribution when nodes are added or removed by mapping both keys and servers onto a hash ring and assigning each key to the next clockwise node.

**Key Benefit:**

* Only a **small subset of keys** move when the cluster changes

---

## Core Idea: The Hash Ring

Instead of mapping keys using modulo, consistent hashing maps everything onto a **circular hash space (ring)**.

Typical hash space:

```text
0 ----------------------------> 2^32 - 1
|                                |
|________________________________|
```

* The end wraps back to the beginning
* This forms a logical **ring**

---

## Step-by-Step: How Consistent Hashing Works

### Step 1: Hash Servers onto the Ring

Each server is hashed using a hash function (for example, MD5, SHA-1, MurmurHash).

Example servers:

```text
Server A -> hash(A) = 10
Server B -> hash(B) = 40
Server C -> hash(C) = 80
```

Placed on the ring:

```text
0 ---- A(10) ---- B(40) ---- C(80) ---- 100
```

---

### Step 2: Hash Keys onto the Same Ring

Each key is also hashed using the **same hash function**.

Example keys:

```text
key1 -> hash(key1) = 15
key2 -> hash(key2) = 45
key3 -> hash(key3) = 90
```

---

### Step 3: Assign Key to the Next Clockwise Server

**Rule:**

> A key is assigned to the **first server encountered when moving clockwise** on the ring.

Assignments:

```text
key1 (15) -> Server B (40)
key2 (45) -> Server C (80)
key3 (90) -> Server A (10)  // wraps around
```

---

## What Happens When a Server Is Removed

### Example: Remove Server B

Before:

```text
A(10) ---- B(40) ---- C(80)
```

After removal:

```text
A(10) ------------ C(80)
```

**Impact:**

* Only keys that were mapped to **Server B** are reassigned
* Those keys now go to **Server C**
* Keys on A and C remain unchanged

✅ Minimal key movement

---

## What Happens When a Server Is Added

### Example: Add Server D at position 60

Before:

```text
A(10) ---- B(40) ---- C(80)
```

After:

```text
A(10) ---- B(40) ---- D(60) ---- C(80)
```

**Impact:**

* Only keys between `B(40)` and `D(60)` move to D
* All other keys stay on the same servers

✅ Minimal redistribution

---

## Virtual Nodes (VNodes)

### Why Virtual Nodes Are Needed

If each server appears only once on the ring:

* Load distribution can be uneven
* One server may get more keys than others

---

### Solution: Virtual Nodes

Each physical server is represented by **multiple virtual nodes**.

Example:

```text
Server A -> A1, A2, A3
Server B -> B1, B2, B3
Server C -> C1, C2, C3
```

These virtual nodes are spread across the ring:

```text
A1 -> B1 -> C1 -> A2 -> B2 -> C2 -> A3 -> B3 -> C3
```

**Benefits:**

* Better load balancing
* Smoother scaling
* Used in real systems like Redis, Cassandra, DynamoDB

---

## Comparison: Traditional Hashing vs Consistent Hashing

| Feature                 | Traditional Hashing | Consistent Hashing |
| ----------------------- | ------------------- | ------------------ |
| Server addition/removal | Rehash all keys     | Rehash few keys    |
| Scalability             | Poor                | Excellent          |
| Cache stability         | Low                 | High               |
| Data movement           | High                | Minimal            |

---

## Real-World Use Cases

* Distributed caching (Redis, Memcached)
* Distributed databases (Cassandra, DynamoDB)
* Load balancers
* Sharded storage systems
* Message queues

---

## Interview-Friendly Summary

> Consistent hashing places both keys and servers on a hash ring and assigns each key to the next clockwise server, ensuring that only a small number of keys are remapped when servers are added or removed.

---

## Key Takeaway

Consistent hashing is all about **stability and scalability** in distributed systems.
It allows systems to grow and shrink **without massive reorganization of data**.
