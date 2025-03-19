---
title: Python Dictionary and Common Methods
date: 2025-03-22
categories: [Backend, Python]
tags: [Python, Dictionary, Data Structures]
author: kai
---

# 🚀 Python Dictionary (dict) and Common Methods
A **dictionary** is a **collection of key-value pairs**, where **keys are unique and immutable** (e.g., strings, numbers, tuples), and **values can be of any data type**.

- Unordered (Before Python 3.7) → Dictionary items were stored without a guaranteed order.
- Ordered (Python 3.7+) → Dictionaries retain insertion order, but relying on this is discouraged.
- Dynamic and Mutable → You can add, remove, or modify key-value pairs.
- Fast Lookup → Dictionary lookups via keys have an average time complexity of O(1).

## 📌 Creating a Dictionary

| Method | Example | Output |
|--------|---------|--------|
| **Empty Dictionary** | `dict1 = {}` | `{}` |
| **Key-Value Pairs** | `dict2 = {"name": "Alice", "age": 25}` | `{'name': 'Alice', 'age': 25}` |
| **Using `dict()` with Keyword Arguments** | `dict3 = dict(name="Bob", age=30)` | `{'name': 'Bob', 'age': 30}` |
| **Using `dict()` with Iterable** | `dict4 = dict([("id", 1001), ("city", "Beijing")])` | `{'id': 1001, 'city': 'Beijing'}` |

## 🤲🏻 Dictionary Key Rules
A dictionary key **must be immutable**, meaning it cannot be changed after creation.  
✅ Allowed: **int, str, tuple (if all elements are immutable)**  
❌ Not Allowed: **list, set, dictionary**
☝🏻 Keys Must Be Unique: Duplicate keys are not allowed. If a key is reassigned, it overwrites the existing value.

| Data Type | Valid as Key? | Example |
|-----------|--------------|---------|
| `int` | ✅ Yes | `{1: "one"}` |
| `str` | ✅ Yes | `{"name": "Alice"}` |
| `tuple` | ✅ Yes (if all elements are immutable) | `{("x", "y"): 100}` |
| `list` | ❌ No | `{[1, 2]: "list_key"}` → `TypeError` |
| `set` | ❌ No | `{ {1, 2}: "set_key"}` → `TypeError` |
| `dict` | ❌ No | `{ {"key": "value"}: "dict_key"}` → `TypeError` |
| Duplicate Keys | ❌ No | `{"a": 1, "a": 3}` -> `{"a": 3}`|


## Common Operations (CRUD)
Dictionaries in Python allow efficient **adding, modifying, deleting, and retrieving values** using keys.

| Method | Description | Example | Output |
|--------|-------------|---------|--------|
| `dict[key]` | Retrieves the value of `key` (raises error if key does not exist) | `student["name"]` | `"Alice"` |
| `dict.get(key, default)` | Retrieves the value of `key`, returns `default` if key does not exist | `student.get("age", 18)` | `20` |
| `dict[key] = value` | Adds a new key-value pair if the key does not exist, otherwise updates it | `student["gender"] = "Female"` | `Adds "gender": "Female"` |
| `dict[key] = new_value` | Modifies the existing keyâ€™s value | `student["age"] = 21` | `Updates "age" to 21` |
| `del dict[key]` | Deletes the key-value pair (raises error if key does not exist) | `del student["gender"]` | `Removes "gender"` |
| `dict.pop(key, default)` | Deletes and returns the value of key (returns default if key does not exist) | `student.pop("age")` | `Returns 21 and removes "age"` |
| `dict.clear()` | Removes all key-value pairs | `student.clear()` | `{}` |


## Common Methods
Dictionaries in Python come with **built-in methods** to **retrieve keys and values, merge dictionaries, and remove elements dynamically**.

| Method | Description | Example | Output |
|--------|------------|---------|--------|
| `keys()` | Returns all dictionary keys as a view object | `student.keys()` | `dict_keys(['name'])` |
| `values()` | Returns all dictionary values as a view object | `student.values()` | `dict_values(['Alice'])` |
| `items()` | Returns all key-value pairs as a view object | `student.items()` | `dict_items([('name', 'Alice')])` |
| `update(dict2)` | Merges dict2 into the original dictionary (overwrites existing keys) | `student.update({"age": 22, "city": "Shanghai"})` | `Updates "age" to 22, adds "city"` |
| `setdefault(key, default)` | Returns the value of key if it exists, otherwise inserts key with default | `student.setdefault("name", "Bob")` | `"Alice"` |
| `popitem()` | Removes and returns the last inserted key-value pair (LIFO order) | `student.popitem()` | `('city', 'Shanghai')` |


## Key Constraints & Important Considerations
Python dictionaries require **immutable and unique keys**, ensuring fast lookups and efficient data storage.

```python
invalid_dict = {["id"]: 1001}  # TypeError: unhashable type: 'list'
```


<br>

---

🚀 Stay tuned for more insights into Python!