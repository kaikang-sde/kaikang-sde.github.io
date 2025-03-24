---
title: Python Iterators and Generators
date: 2025-03-24
order: 3
categories: [Backend, Python]
tags: [Python, Iterators, Generators]
author: kai
---

# 🚀 Python Iterators and Generators
Efficient iteration is a fundamental aspect of Python programming. In this post, we’ll explore two powerful tools for iteration: **Iterators** and **Generators**. Understanding the difference between them can help you write **cleaner** and **more memory-efficient** code.

## 🔄 Iterator
- An **iterator** is an object that allows you to traverse through elements in a **collection (like a list, tuple, or string)**, one element at a time, while remembering the current position.

### Why Use Iterators?
- ✅ **Memory efficiency**: Iterators can be used to process large datasets without loading everything into memory.
- ✅ **Abstraction**: Provide a uniform way to traverse elements, regardless of the container's data type.

### Iterator vs Iterable

| Term           | Explanation                                                              |
|----------------|--------------------------------------------------------------------------|
| **Iterable**   | An object that implements the `__iter__()` method. It can return an iterator. |
| **Iterator**   | An object that implements **both** `__iter__()` and `__next__()` methods. |

- All **iterators are also iterables**, but **not all iterables are iterators**.
- The `iter()` function is used to get an iterator from an iterable.
- The `next()` function retrieves the **next element** from an iterator.

### Two Core Methods

1. `__iter__()` 
- Returns the iterator object itself.
- Triggered when `iter(obj)` is called.

2. `__next__()`  
- Returns the **next item** from the iterator.
- Raises `StopIteration` when there are no more items.
- Triggered when `next(obj)` is called.


### Common Iterable Types in Python

- Lists (`list`)
- Tuples (`tuple`)
- Strings (`str`)
- Dictionaries (`dict`)
- Sets (`set`)
- File objects
- Generators


### Examples
```python
# 1. Manual Iteration
data = [10, 20, 30]
it = iter(data)  # get an iterator - (variable) it: Iterator[int]

print(next(it))  # 10 next - retrieve the next item from an iterator
print(next(it))  # 20
print(next(it))  # 30
print(next(it))  # Raises StopIteration


# 2. Check If an Object Is Iterable
from collections.abc import Iterable

print(isinstance([1, 2, 3], Iterable))  # True (list is iterable)
print(isinstance("hello", Iterable))   # True (string is iterable)
print(isinstance(123, Iterable))       # False (integer is not iterable)
print(isinstance(10, (int, str)))   # True (10 is an int)
```

- `isinstance(object, classinfo)`: check if an object is an instance of a specific class or a subclass thereo
- `isinstance(object, (type_1, type_2))`: pass a tuple of types to check if the object is any one of them

### How Python for Loop Works Under the Hood
- for loops in Python are syntactic sugar over iter() and next().
- All Python loops over collections are built upon the iterator protocol.

```python
for item in my_list:
    print(item)

# Equivalent while Loop Using iter() and next()
my_iter = iter(my_list) # 1. returns an iterator object from the iterable my_list.

while True:
    try:
        item = next(my_iter) # 2. fetches the next item in the sequence.
        print(item)
    except StopIteration: # 3. When all items are exhausted, next() raises a StopIteration exception.
        break
```

## 🔧 Custom Iterator
In Python, you can create your **own iterator** by defining a class that implements two special methods: `__iter__()` and `__next__()`.

### The Iterator Protocol

To create a custom iterator, a class must implement:

| Method       | Description |
|--------------|-------------|
| `__iter__()` | Should **return the iterator object** itself. |
| `__next__()` | Should **return the next value**, and raise `StopIteration` when there are no more values. |

This allows the object to work with a `for` loop or any code that uses Python's iterator protocol.

### Example: A Custom Range Iterator
A class `RangeIterator` that behaves similarly to Python’s built-in `range()`.

```python
class RangeIterator:
    def __init__(self, start, end):
        self.current = start
        self.end = end

    def __iter__(self):
        # Return the iterator object itself
        return self

    def __next__(self):
        if self.current >= self.end:
            raise StopIteration
        val = self.current
        self.current += 1
        return val


range_iterator = RangeIterator(1, 5)
for num in range_iterator:
    print(num) # 1, 2, 3, 4
```

## 🧠 Generator
A **generator** is a type of **iterator** that is defined using a **function with the `yield` keyword** instead of `return`.<br>

### How it works
- Each time `yield` is encountered, the function **pauses** and **remembers its state**.
- When `next()` is called, it **resumes** execution from where it left off.

### Why Use Generators

| Feature            | Benefit                                      |
|--------------------|----------------------------------------------|
| **Lazy Evaluation** | Items are generated one at a time when needed |
| **Memory Efficient** | Does not store all items in memory             |
| **Clean Syntax**     | Easier to read than manually implementing iterators |
| **Stateful**         | Automatically maintains the internal state across iterations |

### Examples
#### Case Study 1: Generator Call Chain and Lifecycle
1. When a **generator function** uses the `yield` keyword, the function's **execution is paused** at that point, and the expression after `yield` is returned as the next value in the iteration.
2. Each time `next()` is called on the generator object, the function **resumes execution from where it left off**, continuing until it hits the next `yield` or finishes execution.
3. Once all yield expressions are exhausted, the generator raises StopIteration.<br>

✅ This behavior allows the generator to **produce values one at a time** instead of computing all results at once.<br>
✅ This lifecycle makes generators ideal for **lazy evaluation, streaming data, and memory-efficient** iteration patterns.

```python
def my_gen():
    print("Step 1: Start")
    yield "A"
    print("Step 2: Continue")
    yield "B"
    print("Step 3: Final")
    yield "C"
    print("Done")

g = my_gen()

print(next(g))  # Output: Step 1: Start → "A"
print(next(g))  # Output: Step 2: Continue → "B"
print(next(g))  # Output: Step 3: Final → "C"
print(next(g))  # Raises StopIteration
```

#### Case Study 2: Infinite Sequence with Generators
Generators can also be used to implement **infinite sequences**, such as generating natural numbers endlessly. This is possible because generators yield values **on demand** and **maintain internal state**, making them memory-efficient and ideal for infinite data streams.

- Infinite generators are lazy, meaning they don’t compute or store all values — they yield one value at a time, on demand.

```python
def natural_numbers():
    num = 1
    while True: # the while True loop keeps the generator alive indefinitely.
        yield num
        num += 1

# Create the generator object
numbers = natural_numbers()

# Print first 5 numbers
for _ in range(5):
    print(next(numbers)) # Each call to next(numbers) resumes execution from the last yield, producing the next number.
```


## 🔁 `yield` vs `return` 
Both `yield` and `return` are used in functions, but they serve **different purposes** and behave differently in terms of execution flow and memory usage.

| Feature        | `yield`                                | `return`                           |
|----------------|-----------------------------------------|------------------------------------|
| **Function State** | Preserved between calls (pauses execution) | Cleared immediately (ends function) |
| **Execution Count** | Can yield multiple times (on each `next()`) | Executes once, then exits          |
| **Return Value** | Returns a **generator object**        | Returns a **single value**         |
| **Use Case**    | Useful for **lazy iteration** and **streams** | Useful for **immediate values**    |

- **`return`**: Used in regular functions to send back a final result.
- **`yield`**: Turns the function into a **generator**, allowing it to produce a series of results lazily.

## 🔄 Iterator vs Generator
Both **iterators** and **generators** allow iteration over data, but they differ in implementation, readability, and use cases.


| Feature         | **Iterator**                         | **Generator**                            |
|------------------|--------------------------------------|-------------------------------------------|
| **Implementation** | Requires a **class** with `__iter__()` and `__next__()` methods | Defined using a **function with `yield`** |
| **Memory Usage** | Low                                 | Extremely low (lazy evaluation)          |
| **Code Complexity** | Higher (requires boilerplate code) | Lower (simple and concise syntax)         |
| **State Management** | Must be handled **manually**      | Automatically managed by Python           |
| **Best Use Case** | Complex iteration logic             | Simple to moderate iteration logic        |

- Use an **iterator** when:
  - You need **full control** over the iteration process.
  - You're implementing custom data structures with non-linear access.
  
- Use a **generator** when:
  - You want to simplify code for **linear iteration**.
  - You're working with **large or infinite datasets**.
  - You want **memory-efficient**, lazy evaluation.

> 🧠 Generators are **a special case of iterators**: all generators are iterators, but not all iterators are generators.



<br>


---

🚀 Stay tuned for more insights into Python!