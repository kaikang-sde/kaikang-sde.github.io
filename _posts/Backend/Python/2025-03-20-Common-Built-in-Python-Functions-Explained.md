---
title: Common Built-in Python Functions Explained
date: 2025-03-20
categories: [Backend, Python]
tags: [Python, Built-in Functions]
author: kai
---

# 🚀 Common Built-in Python Functions
Python comes with many **built-in functions**, master the most commonly used ones for efficient coding.

## 🤝 Data Type Conversion
Python provides built-in functions for **type conversion** (also known as **type casting**) to change data from one type to another.


| Function  | Purpose | Example | Output |
|-----------|----------------------|----------------|--------|
| `int("X")`  | Converts string to integer | `int("123")` | `123` |
| `float("X")` | Converts string to floating point | `float("9.9")` | `9.9` |
| `str(x)`  | Converts to string | `str(100)` | `"100"` |
| `bool(x)` | Converts to boolean | `bool(0)`, `bool(1)` | `False`, `True` |


## 🍿 Container Type Conversion
Python provides built-in functions to **create and convert container types**, including `list()`, `tuple()`, `dict()`, and `set()`.

| Method  | Description | Example | Output |
|---------|------------|---------|--------|
| `list(string)` | Converts a string into a list of characters | `list("Python")` | `['P', 'y', 't', 'h', 'o', 'n']` |
| `list(tuple)` | Converts a tuple to a list | `list((1, 2, 3))` | `[1, 2, 3]` |
| `list()` | Creates an empty list | `list()` | `[]` |
| `tuple(list)` | Converts a list to a tuple | `tuple([10, 20, 30])` | `(10, 20, 30)` |
| `tuple(dict)` | Converts a dictionary to a tuple (keys only) | `tuple({'a': 1, 'b': 2})` | `('a', 'b')` |
| `tuple()` | Creates an empty tuple | `tuple()` | `()` |
| `dict(list of pairs)` | Converts a list of key-value pairs into a dictionary | `dict([('a', 1), ('b', 2)])` | `{'a': 1, 'b': 2}` |
| `dict(keyword arguments)` | Creates a dictionary using keyword arguments | `dict(name="Alice", age=25)` | `{'name': 'Alice', 'age': 25}` |
| `dict()` | Creates an empty dictionary | `dict()` | `{}` |
| `set(list)` | Removes duplicates from a list | `set([1, 2, 2, 3])` | `{1, 2, 3}` |
| `set(string)` | Converts a string into a set of unique characters | `set("apple")` | `{'a', 'p', 'l', 'e'}` |
| `set()` | Creates an empty set<br> Note: {} creates an empty dictionary | `set()` | `set()` |


## 🥳 Math and Sequences

### range()
The `range()` function generates **integer sequences**, `<class 'range'>`, commonly used for loops and iteration.<br>
**parameters:**
- **`start`**: The starting number (inclusive). Default is `0`.
- **`stop`**: The end number (exclusive). Required.
- **`step`**: The difference between each number in the sequence. Default is `1`.

| Syntax | Description | Example|
|--------|-------------|--------|
| `range(stop)` | Generates a sequence from `0` to `stop - 1` (default step = `1`) | `r1 = list(range(5))` -> **output** [0, 1, 2, 3, 4]|
| `range(start, stop)` | Generates a sequence from `start` to `stop - 1` (step = `1`) | `r2 = list(range(2, 5))` -> **output** [2, 3, 4]|
| `range(start, stop, step)` | Generates a sequence with a custom step size | `r3 = list(range(2, 10, 2))` -> **output** [2 4 6 8]|
| `for i in range(start, stop, step)` | Using range() in a Loop | `for i in range(2, 10, 2)` -> **output** 2 4 6 8|

### len()
The `len()` function **returns the number of elements** in a container (string, list, tuple, dictionary, etc.).

| Data Type  | Example | Output |
|------------|-------------|--------|
| **String** | `len("Python")` | `6` |
| **List** | `len([1, 2, 3, 4])` | `4` |
| **Tuple** | `len((10, 20, 30))` | `3` |
| **Dictionary** | `len({'a': 1, 'b': 2})` | `2` |
| **Set** | `len({1, 2, 3})` | `3` |

### sum() / max() / min()
Python provides built-in functions to **perform common mathematical operations** on sequences like lists, tuples, and sets.

| Function  | Description | Example | Output |
|-----------|-------------|------------|--------|
| `sum(iterable)` | Returns the sum of all elements in an iterable | `sum([1, 2, 3])` | `6` |
| `sum(iterable, start)` | Adds a starting value to the sum | `sum([1, 2, 3], 10)` | `16` |
| `max(iterable)` | Returns the maximum value in an iterable | `max([1, 2, 3])` | `3` |
| `max(arg1, arg2, ...)` | Returns the largest of multiple values | `max(5, 10, 2)` | `10` |
| `min(iterable)` | Returns the minimum value in an iterable | `min([1, 2, 3])` | `1` |
| `min(arg1, arg2, ...)` | Returns the smallest of multiple values | `min(5, 10, 2)` | `2` |


### time Module
The `time` module in Python provides functions to work with **timestamps, delays, and formatted time strings**. <br>
The `time.time()` function returns the **current timestamp** as the number of **seconds** since **January 1, 1970 (Unix Epoch time)**.

| Function  | Description | Example |
|-----------|------------|---------|
| `time.time()` | Returns the current timestamp in seconds | `1698753641.50238` |
| `time.sleep(seconds))` | Pause execution for x seconds   | `start = time.time()`<br>`time.sleep(2)  # Pause execution for 2 seconds`<br>`end = time.time()` |
| `time.strftime()` | formats the current time into a human-readable string.  | `now = time.strftime("%Y-%m-%d %H:%M:%S")` -> **output:** "2030-10-01 14:30:00"|

<br>

---

🚀 Stay tuned for more insights into Python!