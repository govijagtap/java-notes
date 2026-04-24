# HashMap Internals – Consolidated Notes

## 1. Basic Structure

* HashMap stores data as key-value pairs.
* Internally uses an array:

  ```java
  Node<K,V>[] table;
  ```
* Each index is called a **bucket**.

---

## 2. Lazy Initialization

* Table is **not created during constructor**.
* It is created during first `put()`.

---

## 3. Node Structure

```java
class Node<K,V> {
    int hash;
    K key;
    V value;
    Node<K,V> next;
}
```

* `next` pointer creates a linked list inside a bucket.

---

## 4. Put Operation Flow

1. Calculate index:

   ```java
   index = (n - 1) & hash;
   ```
2. If bucket empty → insert node
3. If collision → traverse linked list
4. If key exists → update value
5. Else → append node

---

## 5. Collision Handling

* Before Java 8 → Linked List
* Java 8+ → Linked List → Tree (Red-Black Tree if size ≥ 8)

---

## 6. Tree Conversion

* Happens when:

  * Bucket size ≥ 8
  * Table size ≥ 64
* Converts Nodes → TreeNodes
* Uses Red-Black Tree rules:

  * New node is RED
  * Fix using recoloring and rotation

### Special Case (Very Important 🔥)

* If:

  * Bucket size ≥ 8
  * BUT table size < 64

👉 Then **tree conversion does NOT happen**.

👉 Instead:

```text
HashMap triggers RESIZE
```

👉 Reason:

* Increasing table size reduces collisions
* Better distribution avoids need for tree

---

## 7. Red-Black Tree Basics

* Balanced binary search tree
* Maintains:

  * No two RED nodes adjacent
  * Equal black height
* Fix operations:

  * Recolor
  * Left rotation
  * Right rotation

---

## 8. Resize Mechanism

* Trigger:

  ```text
  size > capacity * loadFactor
  ```
* Default:

  * capacity = 16
  * loadFactor = 0.75
* On resize:

  * capacity doubles
  * new table created

---

## 9. Rehash Optimization

* No full hash recomputation
* Uses:

  ```text
  new index = old index OR old index + oldCapacity
  ```

---

## 10. Why Power of 2 Capacity

* Ensures:

  ```text
  (n - 1) = 111...111 (bit mask)
  ```
* Enables efficient index calculation using `&`
* Ensures uniform distribution

---

## 11. Size Handling

* HashMap maintains a separate field:

  ```java
  int size;
  ```
* Increments on new key insertion
* Independent of bucket count

---

## 12. Time Complexity

| Operation | Average | Worst    |
| --------- | ------- | -------- |
| put()     | O(1)    | O(log n) |
| get()     | O(1)    | O(log n) |

---

## 13. Key Concepts

* Uses `hashCode()` for bucket location
* Uses `equals()` for key comparison
* Allows:

  * 1 null key
  * multiple null values

---

## 14. Important Insights

* Array stores references, not values directly
* Linked list formed using `next` pointer
* Tree improves performance under heavy collision
* Resize is expensive (O(n))

---

## 15. Summary

* HashMap = Array + LinkedList + Tree
* Optimized for fast access
* Uses hashing + bit operations for performance
