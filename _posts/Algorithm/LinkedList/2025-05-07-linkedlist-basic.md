---
title: Linked List in Java, Python and JavaScript
date: 2025-05-07
categories: [Algorithm, LinkedList]
tags: [LinkedList, Java, Python, JavaScript]
author: kai
---

## 🔗 What is a Linked List?

A **linked list** is a fundamental linear data structure made up of individual **nodes**. Unlike arrays, linked lists do **not require contiguous memory allocation**, allowing for more flexible memory management.

Each node is like a **person in a human chain**—they hold their own value and also **know who’s next in line**.

## 🧱 Node Structure

Each **node** in a linked list typically contains two components:

- `val`: The **value/data** stored in the node.
- `next`: A **reference/pointer** to the **next node** in the list.

### Java

```java
 public class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}
```

### Python
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

### JavaScript
```javascript
function ListNode(val, next) {
    this.val = (val===undefined ? 0 : val)
    this.next = (next===undefined ? null : next)
}
```


## 🔄 Types of Linked Lists

| **Type**              | **Description**                                      | **Illustration**                 |
|-----------------------|------------------------------------------------------|----------------------------------|
| **Singly Linked List** | Each node points only to the **next** node.         | `A ➝ B ➝ C ➝ None`              |
| **Doubly Linked List** | Each node points to both the **next** and **prev**. | `None <- A ⇄ B ⇄ C -> None`      |
| **Circular Linked List** | The last node points back to the **first** node.  | `A ➝ B ➝ C ➝ A (looped)`       |


## 🧠 Why Use Linked Lists?
- ✅ Dynamic size (no need to define array capacity)
- ✅ Efficient insertions/deletions (O(1) at head or tail)
- ❌ Slower access by index (O(n))


## 👀 Add and Delete in Singly Linked List 

![Add and delete in singly linked list](/assets/img/posts/Algorithm/LinkedList/linkedlist-add-delete.png)


## 📌 Summary

| **Feature**           | **Array**                 | **Linked List**              |
|------------------------|---------------------------|-------------------------------|
| **Memory Layout**      | Contiguous                | Non-contiguous                |
| **Insert/Delete**      | Costly (`O(n)`)           | Efficient (`O(1)`) at head    |
| **Access by Index**    | Fast (`O(1)`)             | Slow (`O(n)`)                 |
| **Resizable**          | No (fixed size)           | Yes (dynamic by nature)       |

### Notes
- `O(1)` for head operations — head pointer is always accessible.
- Operations that involve traversal (e.g., insert/search/delete at index or by value) are **O(n)** in the worst case.
- There's **no direct access** by index like arrays (`O(1)` access is not supported).


## 🆚 Singly Linked List vs Array

| **Feature**            | **Array**                           | **Singly Linked List**                |
|------------------------|--------------------------------------|----------------------------------------|
| **Memory Layout**      | Contiguous memory blocks             | Non-contiguous memory (nodes scattered) |
| **Size Flexibility**   | Fixed-size (unless using dynamic resizing) | Dynamically resizable                |
| **Access Time**        | Constant time `O(1)` by index        | Linear time `O(n)` (traverse from head) |
| **Insertion (Middle)** | Costly: shift elements → `O(n)`      | Efficient: pointer change → `O(1)` (with node reference) |
| **Deletion (Middle)**  | Costly: shift elements → `O(n)`      | Efficient: pointer change → `O(1)` (with node reference) |
| **Cache Friendliness** | High (data stored contiguously)      | Low (non-contiguous, more pointer chasing) |
| **Pointer Overhead**   | None                                 | Each node stores a `next` pointer      |
