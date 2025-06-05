---
title: Python List and Common Operations
date: 2025-03-21
categories: [Backend, Python]
tags: [Python, Data Structures, List]
author: kai
permalink: /posts/backend/python/python-list-and-common-operations/
---

# 🚀 Python List and Common Operations
A **list** is a built-in data structure in Python that allows storing **multiple values** in an ordered, **mutable** sequence.

- **Ordered** → Elements maintain insertion order.  
- **Mutable** → Supports adding, removing, and modifying elements. Memory address remains unchanged.
- **Allows Duplicates** → The same values can appear multiple times.  

## 📌 Defining a List

| Definition Method | Example | Output |
|------------------|---------|--------|
| **Direct Assignment** | `list1 = [1, 1, 2, 3]` | `[1, 1, 2, 3]` |
| **From Iterable** | `list2 = list("abc")` | `['a', 'b', 'c']` |
| **Empty List** | `list3 = []` | `[]` |
| **Mixed Data Types** | `list4 = [1, "hello", True, [2, 3]]` | `[1, 'hello', True, [2, 3]]` |

## 👀 List Indexing and Slicing
Python lists support **indexing** (accessing elements) and **slicing** (extracting sublists).<br>
**Slicing Syntax:**`list[start:end:step]`
- **start:** Starting index (inclusive, default 0).
- **end:** Ending index (exclusive, default is end of list).
- **step:** Step size (default 1 for normal slicing, -1 for reverse order).

### 1️⃣ Accessing Elements (Indexing)
```python
lst = ["a", "b", "c", "d", "e"]

print(lst[0])    # Output: 'a' (first element)
print(lst[-1])   # Output: 'e' (last element)
```

### 2️⃣ Extracting a Sublist (Slicing)
```python
lst = ["a", "b", "c", "d", "e"]

print(lst[1:3])  # Output: ['b', 'c']  (elements from index 1 to 2)
print(lst[::2])  # Output: ['a', 'c', 'e'] (every second element)
```

## 🤡 Adding and Removing Elements
Python provides several built-in methods to **add, modify, and remove elements** from a list.

| Method | Description | Example `lst = [1, 2, 3]`| Output |
|--------|-------------|---------|--------|
| `append(obj)` | Adds an element to the **end of the list** | `lst.append(4)` | `[1, 2, 3, 4]` |
| `insert(index, obj)` | Inserts an element **at a specific index** | `lst.insert(1, "x")` | `[1, 'x', 2, 3]` |
| `extend(iterable)` | **Merges an iterable** into the list | `lst.extend([4, 5])` | `[1, 2, 3, 4, 5]` |
| `remove(obj)` | Removes the **first occurrence** of obj | `lst.remove(2)` | `[1, 3]` |
| `pop(index=-1)` | **Removes and returns** the element **at index** (default is last) | `lst.pop(1)` | `2 -> [1, 3]` |
| `clear()` | **Removes all elements** from the list | `lst.clear()` | `[]` |


## 🙀 Query and Statistics Methods
Python lists provide **built-in methods to search for elements, count occurrences, and measure list size**.

| Method | Description | Example `lst = [1, 'b', 2, 2, 3]`| Output |
|--------|-------------|---------|--------|
| `index(obj)` | Returns the first occurrence index of `obj` (raises error if not found) | `lst.index("b")` | `1` |
| `count(obj)` | Returns the number of times obj appears in the list | `lst.count(2)` | `2` |
| `len(lst)` | Returns the number of elements in the list | `len([1, 'b', 2, 2, 3])` | `5` |


## 🤖 Sorting and Reversing Methods
Python provides **built-in methods to sort and reverse lists efficiently**, allowing for both in-place modifications and new sorted lists.

| Method | Description |
|--------|-------------|
| `sort(key=None, reverse=False)` | Sorts the list **in-place** (modifies original list, no return value) | 
| `sorted(iterable)` | Returns a **new sorted list** (original list remains unchanged) | 
| `reverse()` | Reverses the list **in-place** (modifies original list, no return value) |

### Examples
```python
------------sort(): in ascending order----------------------
lst = [3, 1, 4, 2]
lst.sort()  
print(lst)  # Output: [1, 2, 3, 4]

------------sort(reverse=True): in descending order---------
lst = [3, 1, 4, 2]
lst.sort(reverse=True) 
print(lst)  # Output: [4, 3, 2, 1]

------------sort(): No return value-------------------------
lst = [3, 1, 4, 2]
print(lst.sort()) # Output: None

------------sorted(): Return a new sorted list---------------
lst = [3, 1, 4, 2]
print(sorted(lst)) # Output: [1, 2, 3, 4]
print(lst)  # Output: [3, 1, 4, 2]

------------reverse(): Reverses the list---------------------
lst = [3, 1, 4, 2]
print(lst.reverse()) # Output: None
print(lst) # Output: [2, 4, 1, 3]
```
<br>

---

🚀 Stay tuned for more insights into Python!