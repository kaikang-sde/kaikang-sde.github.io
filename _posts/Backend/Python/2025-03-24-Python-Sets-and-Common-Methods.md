---
title: Python Sets and Common Methods
date: 2025-03-24
categories: [Backend, Python]
tags: [Python, Data Structures, Set]
author: kai
---

# 🚀 Python Sets (set) and Common Methods
A **set** is an **unordered collection of unique elements** in Python. Sets are useful for **membership testing**(checking whether an element exists in a collection), **removing duplicates**, and performing **mathematical set operations** like union, intersection, and difference.

- **Uniqueness:** Sets automatically remove duplicate values.
- **Unordered:** The order of elements is not guaranteed and may vary each time.
- **Fast membership testing:** Checking for element existence is O(1) on average.
- **No Index Access:** sets are unordered, elements cannot be accessed by index. (TypeError: 'set' object is not subscriptable)

## 🎩 Defining a Tuple

| Method | Example | Output |
|--------|---------|--------|
| **an empty set** | `empty_set = set()` | `set()` |
| **Direct definition** | `s1 = {1, 2, 3}` | `{1, 2, 3}` |
| **From an iterable** | `s2 = set([1, 2, 2, 3])` | `{1, 2, 3}` |

```python
s = {} # This creates an empty dictionary, NOT a set
print(type(s)) # Output: <class 'dict'>

s1 = set()
print(type(s1))  # Output: <class 'set'>
```

## 🧢 Frozenset
A frozenset is an **immutable** version of a set:
-  Cannot add or remove elements
-  Can be used as a dictionary key or stored in other sets (because it’s hashable)

```python
fs = frozenset([1, 2, 3])
```

## 🎓 Common CRUD Operations
Python provides several built-in methods to modify sets efficiently. Since sets are **mutable**, you can dynamically add or remove elements after creation.

| Method | Description | Example<br> `s = {1, 2, 3}`| Output |
|--------|-------------|---------|--------|
| `s.add(element)` | Adds a **single element** to the set.| `s.add(4)` | `{1, 2, 3, 4}` |
| `s.update(iterable)` | Adds **multiple elements** from an iterable.<br>e.g., list, tuple, another set | `s.update([4, 5])` | `{1, 2, 3, 4, 5}` |
| `s.remove(element)` | Removes the specified element.<br>If the element is not present, it raises a **KeyError**. | `s.remove(3)`<br>`s.remove(99)` | `{1, 2}`<br>`KeyError: 99` |
| `s.discard(element)` | Removes the element if it exists.<br>If the element is not present, **no error** is raised. | `s.discard(3)`<br>`s.discard(99)` | `{1, 2}`<br>`{1, 2, 3}` |
| `s.pop()` | Removes and returns a random element from the set.<br>Raises **KeyError** if the **set is empty**. | `value = s.pop()` | Random value (e.g., 1) |
| `s.clear()` | Removes all elements, making the set empty. | `s.clear()` | `set()` |


## 🪖 Union, Intersection, Difference, Symmetric Difference
Python provides powerful set operations that allow you to perform **mathematical-like comparisons** between sets. These operations return **new sets** without modifying the originals.

| Method  | Operator    | Description| Example<br>`s1 = {1, 2, 3, 4}`<br>`s2 = {3, 4, 5, 6}` | Output |
|--------|-------------|------------|---------|--------|
|`union(s2)` | `|` | Combines elements from both sets, removing duplicates. | `s1 | s2`<br>`s1.union(s2)` | `{1, 2, 3, 4, 5, 6}` |
|`intersection(s2)` | `&` | Returns elements that are common to both sets. | `s1 & s2`<br>`s1.intersection(s2)` | `{3, 4}` |
|`difference(s2)` | `-` | Returns elements that are in s1 but not in s2. | `s1 - s2`<br>`s1.difference(s2)` | `{1, 2}` |
|`symmetric_difference(s2)` | `^` | Returns elements that are in either set, but not in both. | `s1 ^ s2`<br>`s1.symmetric_difference(s2)` | `{1, 2, 5, 6}` |


## ⛑️ Set Relationship Checks
In addition to set operations, Python provides built-in methods to evaluate **logical relationships** between sets, such as **subset**, **superset**, and **disjoint**.

| Method | Description | Example<br>`s1 = {1, 2}`<br>`s2 = {1, 2, 3}`<br>`s3 = {4, 5}`| Output |
|--------|-------------|---------|--------|
| `issubset()` | Checks whether all elements of s1 are also in s2. | `s1.issubset(s2)`<br>`s2.issubset(s1)`| `True`<br>`False` |
| `issuperset()` | Checks whether s1 contains all elements of s2 | `s2.issuperset(s1)`<br>`s1.issuperset(s2)`| `True`<br>`False` |
| `isdisjoint()` | Checks if two sets have no elements in common. | `s1.isdisjoint(s3)`<br>`s1.isdisjoint(s2)`| `True`<br>`False` |

## 🎓 Examples
```python
# 1. If you want to remove duplicates from a list, converting it to a `set` is the quickest way.
lst = [1, 2, 2, 3, 3, 3]
unique = set(lst)              # {1, 2, 3}
new_lst = list(unique)         # [1, 2, 3] (order is not guaranteed)

# 2. Finding Common Elements Between Lists
list1 = [1, 2, 3]
list2 = [2, 3, 4]

common = set(list1) & set(list2)
print(common)  # Output: {2, 3}

# 3. Permission Check System
user_permissions = {"read", "write"}
required_permissions = {"write", "execute"}

if required_permissions.issubset(user_permissions):
    print("Access is sufficient")
else:
    print("Insufficient permissions：", required_permissions - user_permissions)
    # Output: Insufficient permissions:  {'execute'}
```


<br>



---

🚀 Stay tuned for more insights into Python!