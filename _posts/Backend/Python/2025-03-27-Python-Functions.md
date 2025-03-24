---
title: Python Functions
date: 2025-03-24
order: 4
categories: [Backend, Python]
tags: [Python, Functions]
author: kai
---

# 🚀 Python Functions
Python functions are **reusable blocks of code** that can take inputs (parameters), execute code, and optionally return a result. They are a core part of writing modular and maintainable Python programs.<br>

## 🔒 Basic Syntax
```python
def function_name(parameters):
    """Docstring (optional): describes what the function does"""
    # Code block
    return result  # Optional
```

### Components Explained
- **def:** Keyword to define a function
- **function_name:** Name of the function (follows naming rules)
- **parameters:** Optional input values (can be zero or more)
- **return:** Optional keyword to send back a result
- **docstring:** Optional multi-line string to describe the function purpose

### Example
```python
def add(a, b):
    """
    return the sum of a and b
    """
    return a + b

print(add(2, 3))  # Output: 5
print(add(3, 4))  # Output: 7
```

## 📥 Parameter Passing
In Python, arguments are passed by **object reference**, but the behavior differs between **mutable** and **immutable** objects.

### Immutable Types
For **immutable objects**, such as integers, strings, and tuples — **any modification inside a function creates a new object**. The original data remains unchanged.

```python
def modify_num(n):
    n += 10
    print("Inside function, n:", n)  # Inside function, n: 20

num = 10
modify_num(num)
print("Outside function, num:", num)  # Outside function, num: 10 (unchanged)
```

- n += 10 creates a new integer object and rebinds n to it.
- The variable num outside the function still refers to the original value 10.
- This demonstrates that **immutable types are passed by value (copy of reference)** — meaning, the reference is passed, but the object cannot be modified in place.

### Mutable Types
For **mutable objects**, such as lists and dictionaries — **modifying the object inside the function will affect the original object** outside the function.


```python
def modify_list(lst):
    lst.append(4)
    print("Inside function, lst:", lst)  # [1, 2, 3, 4]

my_list = [1, 2, 3]
modify_list(my_list)
print("Outside function, my_list:", my_list)  # [1, 2, 3, 4]
```

- Lists are mutable, so append() changes the content in-place.
- The function works with a reference to the same object in memory.
- This means **mutable types are passed by reference**, and in-place operations will persist after the function call.


##  🔁 Return Values
In Python, the `return` statement is used to **exit a function** and optionally **send a value back** to the caller.

### Return a Single Value

```python
def square(n):
    return n ** 2

print(square(4))  # Output: 16
```

### Return Multiple Values (Tuple Unpacking)
You can return multiple values from a function, which are automatically packed into a tuple and can be unpacked on the receiving end.

```python
def calculate(a, b):
    return a + b, a - b, a * b

sum_result, sub_result, mul_result = calculate(5, 3)
print(sum_result)  # Output: 8

# This is what happens internally:
result = calculate(5, 3)
print(result)  # Output: (8, 2, 15)  → It's a tuple
```

### No Return Value (Returns None)
If a function does not explicitly return a value using return, it returns None by default.

```python
def greet(name):
    print(f"Hello, {name}!")

result = greet("Alice")  # Output: Hello, Alice!
print(result)            # Output: None
```

## 🧷  None in Python
`None` is a **unique built-in constant** in Python used to signify the **absence of a value** or a **null placeholder**.

- It’s the **only value of the `NoneType`** type.
- Often used as the default value for function arguments or as an empty return value.

```python
a = None
print(type(a))  # <class 'NoneType'>
```

### Comparison with Other “Empty” Values

| Type | Description| Example |
|------|------------|---------|
| None | Absolute null (non-instantiable) | `x = None` |
| False | Boolean false | `if not x:` |
| 0, "" | Numeric or string empty state | `count = 0 / s = ""` |
| [], {} | Empty data containers | `lst = [] / dct = {}` |

### Explicit Initialization with None
In Python, **variables must be explicitly assigned** before they can be used. You **cannot declare** a variable without assigning a value — doing so will result in a syntax error.

- None is often used to represent a placeholder for a future value.
- It is useful for declaring empty slots, especially in data structures like lists.


```python
# 1. Incorrect: SyntaxError due to missing value
x =    # SyntaxError: invalid syntax

# 2. Correct: Use None as a placeholder value
x = None

# 3. Create a list with 5 empty slots initialized to None
y = [None] * 5
print(y)  # Output: [None, None, None, None, None]


# 4. Using None as a Default Parameter
def greet(name=None):
    print(f"Hello, {name or 'Guest'}!") # The expression name or 'Guest' works because None is considered a falsy value. If name is None, 'Guest' is used instead.

greet()        # Output: Hello, Guest!
greet("Kai")   # Output: Hello, Kai
```

✅ Best Practice Tip
- Always use None (not mutable types like [] or {}) as a default parameter value to avoid unintended side effects.


## 🧠 Advanced Parameter Types
Python provides flexible ways to define and call functions using various types of parameters.

| Parameter Type       | Description                              | Example                            |
|----------------------|------------------------------------------|------------------------------------|
| **Positional**        | Arguments are matched to parameters **in the order** they're defined. | `add(3, 5)` |
| **Keyword**           | Arguments can be passed by **explicitly specifying parameter names**.| `add(a=3, b=5)`  |
| **Default**           | Use default values to make some arguments **optional**.<br> Default parameters must come **after** positional parameters.   | `def greet(name="Guest")` |
| **Variable Positional** | Use `*args` to accept **any number of positional arguments**. They are stored as a **tuple**. | `*args`   |
| **Variable Keyword** | Use `**kwargs` to accept **any number of keyword arguments**. They are stored as a **dictionary**. | `**kwargs`  |

### The Standard Parameter Order 
The full rule is:
- `Positional-only (Python 3.8+) → Positional or keyword → Default parameters → *args → Keyword-only → **kwargs`

But for most practical use cases:
- `Positional arguments → Default arguments → *args → **kwargs`
- `def func(positional, default=42, *args, **kwargs):`

```python
def greet(name, greeting="Hello"):
    print(f"{greeting}, {name}")  

def greet(greeting="Hello", name):  # ❌ SyntaxError, Invalid order
    pass
```

### Examples

```python
# 1. Variable Positional Parameters: *args
def total(*numbers):
    return sum(numbers)

print(total(1, 2, 3))     # Output: 6
print(total(5, 10, 15))   # Output: 30

# 2. List/Tuple Unpacking with `*`
def add(a, b, c):
    return a + b + c

numbers = [1, 2, 3]
print(add(*numbers))  # 6 -> equivalent to add(1, 2, 3)

# 3. Variable Keyword Parameters: **kwargs
def display_info(**info):
    for key, value in info.items():
        print(f"{key}: {value}")

display_info(name="Kai", age=28, language="Python")
# Output:
# name: Kai
# age: 28
# language: Python

# 4. Dictionary Unpacking with **
def greet(name, message):
    print(f"{message}, {name}!")

params = {"name": "Alice", "message": "Hello"}
greet(**params)  # Hello, Alice! -> equivalent to greet(name="Alice", message="Hello")

# 5. Mixed Parameter Types in Python Functions
def complex_func(a, b, c=0, *args, **kwargs):
    print("a:", a)
    print("b:", b)
    print("c:", c)
    print("args:", args)
    print("kwargs:", kwargs)

complex_func(1, 2, 3, 4, 5, name="Alice", age=25)
# Output:
# a: 1
# b: 2
# c: 3
# args: (4,5)
# kwargs: {'name': 'Alice', 'age':25}

# 6. dynamic SQL generation 
def build_sql(table, **conditions):
    query = f"SELECT * FROM {table}"
    if conditions:
        where_clause = " AND ".join([f"{key} = '{value}'" for key, value in conditions.items()])
        query += f" WHERE {where_clause}"
    return query

print(build_sql("users", name="Alice", age=25))
# SELECT * FROM users WHERE name = 'Alice' AND age = '25'
```

<br>


---

🚀 Stay tuned for more insights into Python!