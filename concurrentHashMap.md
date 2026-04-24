# SynchronizedMap vs ConcurrentHashMap (Deep Understanding)

---

## 1. Introduction

In Java, thread-safe map implementations are required when multiple threads access shared data.

Two common approaches:

* `Collections.synchronizedMap()`
* `ConcurrentHashMap`

---

## 2. SynchronizedMap

### Creation

```java
Map<K,V> map = Collections.synchronizedMap(new HashMap<>());
```

### What is Mutex?

👉 Mutex = **Mutual Exclusion lock**

* A lock that allows **only one thread at a time** to execute a critical section
* In `synchronizedMap`, the mutex is the **wrapper object itself** (or an internal lock object)

Example (conceptually):

```java
final Object mutex = this; // internal lock

public V get(Object key) {
    synchronized (mutex) {
        return map.get(key);
    }
}
```

Meaning:

* If one thread holds the mutex → others must wait
* Applies to **both read and write** operations

### Internal Working

* Wraps HashMap
* Uses a **single global lock (mutex)**

```java
synchronized (mutex) {
    // get / put / remove
}
```

---

### Behavior

#### Read (get)

* Requires lock ❌
* Blocks if another thread is writing

#### Write (put)

* Requires lock ❌

---

### Key Characteristics

* Only **one thread at a time**
* Simple implementation
* Poor scalability

---

## 3. ConcurrentHashMap

### Node Structure (Java 8+)

```java
static class Node<K,V> {
    final int hash;
    final K key;
    volatile V val;          // visibility across threads
    volatile Node<K,V> next; // for linked list chaining
}
```

Notes:

* `hash` and `key` are final → set once, never change
* `val` is volatile → readers see latest updates
* `next` is volatile → safe traversal during concurrent reads

---

### Design Goal

```text
High concurrency + Thread safety
```

---

## 4. Internal Structure (Java 8+)

* Uses `Node[] table`
* Similar to HashMap
* Supports linked list + tree

---

## 5. Write Operation (put)

### Step 1: Find bucket

### Case 1: Bucket empty

```text
CAS (Compare-And-Swap)
```

* Lock-free
* Fast

---

### Case 2: Bucket not empty

```text
synchronized (first node)
```

* Locks only that bucket

---

## 6. Read Operation (get)

```text
Completely lock-free
```

* No synchronization
* Uses `volatile` for visibility

---

## 7. CAS (Compare-And-Swap)

### Concept

```text
If current == expected → update
Else → retry
```

### Usage

* Only when bucket is empty
* Prevents race condition without locking

---

## 8. Volatile Usage

Used for:

* Node value
* next pointer
* table reference

### Guarantees

* Visibility
* Ordering

---

## 9. Happens-Before Guarantee

```text
Write → happens-before → Read
```

Ensures:

* No corrupted data
* Safe reads without locking

---

## 10. Concurrency Behavior 🔥

### Scenario: Writer + Reader on same bucket

* Writer locks bucket
* Reader does NOT wait

Reader may see:

* Old value
* New value

Reader will NEVER see:

* Partial data ❌
* Broken structure ❌

---

## 11. Comparison Table

| Feature     | SynchronizedMap | ConcurrentHashMap |
| ----------- | --------------- | ----------------- |
| Locking     | Global lock ❌   | Bucket-level ✅    |
| Read        | Blocking ❌      | Non-blocking ✅    |
| Write       | Blocking ❌      | Partial lock ✅    |
| Concurrency | Low ❌           | High ✅            |
| Performance | Poor ❌          | High ✅            |

---

## 12. Why ConcurrentHashMap is Better

* Lock-free reads
* Minimal locking
* Better CPU utilization
* Scales with threads

---

## 13. When to Use

### Use SynchronizedMap

* Low concurrency
* Simpler use cases

### Use ConcurrentHashMap

* High concurrency
* Backend systems
* Caching
* Shared state

---

## 14. Key Insights 🔥

```text
SynchronizedMap = Thread safety via blocking
ConcurrentHashMap = Thread safety via smart concurrency
```

---

## 15. First put() Call with Multiple Threads 🔥

### Scenario

Multiple threads call `put()` at the same time on an empty ConcurrentHashMap.

---

### Step-by-Step Flow

#### Step 1: Table is null

* Initially, `table` is not initialized
* First thread triggers initialization

```text
Thread A → initializes table
Thread B → waits or retries
```

👉 Table initialization is controlled to avoid multiple creations

---

#### Step 2: Bucket is empty

All threads try to insert into same bucket (for simplicity)

```text
Thread A → tries CAS (null → new Node)
Thread B → tries CAS (null → new Node)
```

---

#### Step 3: CAS decides winner

```text
Thread A → CAS succeeds ✅
Thread B → CAS fails ❌
```

👉 Only one thread inserts node

---

#### Step 4: Failed thread retries

```text
Thread B → sees bucket not empty now
→ goes to synchronized path
```

---

#### Step 5: Bucket locking (if needed)

```text
synchronized (first node)
```

* Thread B safely updates or appends

---

### Final Outcome

```text
- Only one node inserted initially
- Other threads retry safely
- No data loss
- No corruption
```

---

### Key Insight 🔥

```text
CAS handles first insert (fast path)
Lock handles collision (safe path)
```

---

## 16. Interview Questions

* Why ConcurrentHashMap is faster?
* Why get() is lock-free?
* Why null is not allowed?
* When does it use CAS?
* What happens during collision?
