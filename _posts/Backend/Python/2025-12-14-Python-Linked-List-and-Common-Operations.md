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

### 1️⃣ Traversal
Visit each node from head to tail.
- Time complexity: O(n)

```python
def traverse_linked_list(head):
    current = head
    while current is not None: # better than while current:
        print(current.val)
        current = current.next
```






<br>
---

🚀 Stay tuned for more insights into Python!