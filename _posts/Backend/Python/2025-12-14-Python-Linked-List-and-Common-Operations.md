---
title: Python Linked List and Common Operations
date: 2025-12-14
categories: [Backend, Python]
tags: [Python, Data Structures, Linked List]
author: kai
permalink: /posts/backend/python/python-linked-list-and-common-operations/
---

# 🚀  Python Linked List and Common Operations

## 🔗 Linked List in Python
A linked list is:

- A **linear data structure**
- Composed of **nodes**
- Each node stores:
  - **data (value)**
  - a reference to the **next node**

Unlike arrays (Python lists):

- Elements are **not stored contiguously in memory**
- Access is sequential, not random


## 🫨 Node Definition (`ListNode`)

Each node contains a **value** and a **pointer** to the next node.
- val → stores the data
- next → links the current node to the next node
- next = None means this is the last node

```python
class ListNode:
    def __init__(self, val):
        self.val = val      # data stored in the node
        self.next = None    # reference to the next node
```

### Understanding next and None
**next**
- Connects one node to the next
- Forms the chain structure

**None**
- A special, global singleton object in Python
- Represents “no value” or “no reference”
- Used as a placeholder when there is no next node

### Head Node Represents the Entire Linked List
If you can access the head, you can access the entire list by following **.next**.

```python
def build_linked_list(nums):
    node_1 = ListNode(1)
    node_2 = ListNode(3)
    node_3 = ListNode(5)
    node_4 = ListNode(7)
    node_1.next = node_2
    node_2.next = node_3
    node_3.next = node_4
    return node_1  
```

![List Node Creation](/assets/img/posts/Backend/Python/LinkedListCreation.png)


## 📊 Basic Linked List Operations

```python
class LinkedList:
    def __init__(self):
        self.head = None

    def get(self, index):
        cur = self.head
        for i in range(index):
            cur = cur.next
        return cur

    def add(self, index, value):
        if index > 0:
            pre = self.head
            for i in range(index - 1):
                pre = pre.next
            new_node = ListNode(value)
            new_node.next = pre.next
            pre.next = new_node
        elif index == 0:
            new_node = ListNode(value)
            new_node.next = self.head
            self.head = new_node

    def set(self, index, value):
        cur = self.head
        for i in range(index):
            cur = cur.next
        cur.val = value
    
    def remove(self, index):
        if index > 0:
            pre = self.head
            for i in range(index - 1):
                pre = pre.next
            pre.next = pre.next.next

        elif index == 0:
            self.head = self.head.next
    
    def traverse(self):
        cur = self.head
        while cur is not None: # better than while cur
            print(cur.val)
            cur = cur.next

    def is_empty(self):
        return self.head is None
```


## ⚠️ List vs. Linked List Time Explexity

|--|--|--|
| Operation | List | Linked List |
| --- | --- | --- |
| add(0, v) | O(n) | O(1) |
| add(last, v) | O(1) | O(n) |
| add / remove | O(n) | O(n) |
| get | O(1) | O(n) |
| set | O(1) | O(n) |

### Why Are List Get / Set Operations O(1) in Python?
A Python list is implemented as **a dynamic array.**

That means:
- All elements are stored **contiguously** in memory
- Python knows:
    - the starting address of the array
    - the size of each element reference

```python
Index:   0     1     2     3     4     5
Memory: [ * ] [ * ] [ * ] [ * ] [ * ] [ * ]  # (Each * is a reference to an object)
Address: A   A+1   A+2   A+3   A+4   A+5

nums[2] # address = base_address + i * element_size = A + 2 * element_size -> O(1)
```

- element_size: In a Python list, element_size is **the size of a reference (pointer)**, not the size of the object itself.





<br>
---

🚀 Stay tuned for more insights into Python!