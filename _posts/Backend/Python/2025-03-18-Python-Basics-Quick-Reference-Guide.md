---
title: Python Basics - Quick Reference Guide
date: 2025-03-18
categories: [Backend, Python]
tags: [Python, Basics, Formatting, Data Types, Import]
author: kai
---

# 🚀 Python Basics: Quick Reference Guide

## 🖥️ Python Output
Python provides multiple ways to format string output, supporting complex expressions.

### 1️⃣ Basic `print()` Syntax

```python
print(value1, value2, ..., sep=' ', end='\n', file=sys.stdout, flush=False)
```

#### Parameters
- values: Values to be printed, can be multiple, separated by commas.
- sep: Separator, default is a space.
- end: Ending character, default is a newline `\n`.
- file: Output destination (e.g., file object), default is console.
- flush: Whether to forcibly flush the output (default False).

#### Example

```python
# Output multiple values with default newline
print("Hello", "Python", 2030)  # Hello Python 2030

# Modify separator and ending character
print("Kai", "Kang", "666", sep="---", end="!!!")  # Kai---Kang---666!!!
```


### 2️⃣ str.format() (Recommended)

```python
name = "kai"
age = 30
print("Name: {}, Age: {}".format(name, age))  # Fill by order -> Name: kai, Age: 30
print("Name: {name}, Age: {age}".format(name="Lao Kai", age=30))  # Fill by keyword -> Name: Lao Kai, Age: 30
```

### 3️⃣ f-strings (Python 3.6+)
- The f-string format **starts with f** followed by a string.
- Inside {}, expressions are evaluated and replaced with their values.
- In older versions, % formatting was used, but f-strings simplify the process.

#### Example

```python
name = "Kai"
age = 18
print(f"Name: {name}, Age: {age}")  # Output: Name: Kai, Age: 18
```

### 4️⃣ Using Expressions in f-strings

```python
# Mathematical operations
print(f"Result: {3 * 4 + 5}")  # Output: Result: 17

# Ternary operator
score = 85
print(f"Grade: {'Excellent' if score >= 90 else 'Good'}") # Output: Grade: Good
```

### 🚫 Common Errors
1. **Missing f Prefix** - No error but outputs raw text.
2. **Invalid Expressions** - Syntax error when missing required parentheses.

### 📝 Notes
- The default newline character is `\n`, use `end=""` to prevent a newline.
- f-strings do not support escape characters like `\n`. Use variables instead.
- Performance Comparison (Execution time for 1,000,000 iterations):
    - f-string: 0.12秒
    - format(): 0.25秒
    - % format: 0.18秒


## ⌨️ Python Input
Python provides the input() function to get user input from the console. The return value is always a string.

### 1️⃣ Basic input() Syntax

```python
user_input = input("Prompt message: ")
```

### 📝 Notes
#### 1. Type Conversion
Since `input()` always returns a string, it must be converted when handling numbers.

```python
age = int(input("Please enter your age: ")) # User enters: 11
print(f"Next year, you will be {age + 1} years old.") # Output: Next year, you will be 12 years old.
```

#### 2. Handling Multiple Inputs
Use `.split()` to separate inputs and `map()` to convert them.
- The map() function applies a function (func) to each element of a given iterable (lst).

```python
values = input("Enter two numbers (separated by space): ").split()  # User enters: 1 2
a, b = map(int, values)  # Convert to integers
print(f"The sum of the two numbers is: {a + b}")  # Output: 3
```


## 📦 Python Import Statements
In Python, modules can be imported using the `import` statement or `from ... import` to selectively import objects.

| Feature           | `import module` | `from module import obj` |
|------------------|----------------|--------------------------|
| **Namespace**    | Preserves full module namespace | Imports obj directly into scope |
| **Memory Usage** | Higher         | Lower                    |
| **Code Readability** | Explicit module source | Potential naming conflicts |
| **Modification Impact** | Requires full reference updates | Local modifications do not affect original module |
| **Use Case** | Multiple objects needed | Few frequently used objects |

### 1️⃣ Importing an Entire Module
To import an entire module, use the `import` statement:

```python
import math

print(math.sqrt(16))  # Output: 4.0
```

### 2️⃣ Importing Specific Functions
To import specific functions or variables from a module:

```python
from math import sqrt, pi

print(sqrt(16))  # Output: 4.0
print(pi)        # Output: 3.141592653589793
```

### 3️⃣ Relative Imports (For Package Usage)
Relative imports are used within packages to import modules from the same package or parent directories.
- Absolute imports are preferred for better code clarity.
- Relative imports are useful only within well-structured packages.

```python
# Inside mypackage/sub/mod.py
from .. import utils  # Import from the parent directory
from . import helper  # Import from the same directory
```

### 4️⃣ Custom Modules
If you have a custom module named my_module.py containing:

```python
# my_module.py
def greet():
    print("Hello!")

---------- import --------------
import my_module

my_module.greet()  # Output: Hello!

------ from ... import----------
from my_module import greet

greet() # Output: Hello!

```

### ✅ Best Practices for Imports
Follow Import Order:
1.	Standard Library Modules (e.g., math, os)
2.	Third-party Libraries (e.g., numpy, requests)
3.	Custom Modules (user-defined scripts)

#### Example
```python
# Recommended
import numpy as np
from django.db import models

# Avoid this (may cause conflicts)
from module import *
```


## 🏷️ Python Variables
Python variables do **not** require explicit declaration. A variable is created when assigned a value.

### Dynamic Typing
Python uses **dynamic typing**, meaning a variable’s type is determined by its value.

```python
a = 10      # Integer
a = "Hello" # Dynamically changes type (actually binds to a new object)
```

### Variable Naming Conventions

| Rule Type  | Correct Examples| Incorrect Examples |
|------------------|----------------|--------------------------|
| **Valid Characters**    | var_1, 变量_中文 | 2var, var-name |
| **Case Sensitivitye** | age != Age         | -                    |
| **Avoid Reserved Words** | list_ = [1,2,3] | list = [1,2,3] |
| **PEP8 Recommendation** | user_name | userName (PascalCase) |

PEP8: Python Enhancement Proposal #8, a guideline for writing clean Python code.

## 📊 Python Data Types
In Python, **everything is an object**, including primitive data types. You can use the built-in `type()` function to check the type of any variable.

| Category  | Data Types | Examples |
|------------------|----------------|--------------------------|
| **Numbers**    | 1. Integer (int): Whole numbers<br>2. Floating-point (float): Decimal numbers<br>3. Boolean (bool): A subclass of int, where True == 1, False == 0 | a = 10  #int<br>b = 3.14    # float<br>c = True    # bool |
| **Sequences** | 1. String (str): Immutable text sequence<br>2. List (list): Ordered, mutable collection<br>3. Tuple (tuple): Ordered, immutable collection       | s = "hello"    # str<br>lst = [1, 2, 3]    # list<br>tpl = (4, 5, 6)   # tuple |
| **Mapping** | Dictionary (dict): Key-value pairs | d = {"name": "Alice", "age": 25}   # dict |
| **Sets** | 1. Set (set): Unordered, unique elements<br>2. Frozen Set (frozenset): Immutable version of a set | s = {1, 2, 3}    # set<br>fs = frozenset([1, 2, 3])    # frozenset |
| **Other** | None (Null type): Represents absence of a value | x = None    # NoneType |

### Data Type Comparison Table
Understanding Python's built-in data types is essential for writing efficient and structured code. The table below summarizes key properties of each type, including **mutability**, **ordering**, and **whether duplicate values are allowed**.

| Type  | Mutable? | Ordered? | Allows Duplicates? | Syntax Example |
|-------|---------|----------|--------------------|----------------|
| **Integer (`int`)** | ❌ No  | -  | -  | `a = 10` |
| **Float (`float`)** | ❌ No  | -  | -  | `b = 3.14` |
| **String (`str`)** | ❌ No  | ✅ Yes | ✅ Yes | `s = "hello"` |
| **List (`list`)** | ✅ Yes | ✅ Yes | ✅ Yes | `lst = [1, 2, 3]` |
| **Tuple (`tuple`)** | ❌ No  | ✅ Yes | ✅ Yes | `tpl = (1, 2)` |
| **Dictionary (`dict`)** | ✅ Yes | ❌ No  | ☝🏻 Unique Keys | `d = {'a': 1}` |
| **Set (`set`)** | ✅ Yes | ❌ No  | ☝🏻 No Duplicates | `s = {1, 2, 3}` |

- **Lists and dictionaries are mutable**, meaning their values can be changed after creation.  
- **Strings and tuples are immutable**, ensuring their values remain constant.  
- **Sets and dictionaries do not allow duplicate keys**, which makes them useful for unique collections.


#### Examples
##### 1️⃣ Integer (`int`)
- No size limitation (limited only by system memory).
- Supports multiple numeric bases:
  - **Binary** (`0b1010`)
  - **Octal** (`0o12`)
  - **Hexadecimal** (`0x1A`)

```python
a = 10          # Decimal
b = 0b1010      # Binary → 10
c = 1_000_000   # Underscore separator for readability (Python 3.6+)

print(a, b, c)  # Output: 10 10 1000000
```

##### 2️⃣ Floating-Point (float)
- Represents numbers with decimals.
- Supports scientific notation (e.g., 2.5e3 = 2500.0).

```python
x = 3.14
y = 2.5e-3  # Equivalent to 0.0025

print(x, y)  # Output: 3.14 0.0025
```

##### 3️⃣ Boolean (bool)
- Has two possible values: True (1) and False (0).
- bool is a subclass of int, meaning True is treated as 1 and False as 0.

```python
is_active = True
print(int(is_active))  # Output: 1
```

<br>

---

🚀 Stay tuned for more insights into Python!