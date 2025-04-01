---
title: Python Modules
date: 2025-04-01
order: 2
categories: [Backend, Python]
tags: [Python, Modules, pip, json, os, random, math, sys]
author: kai
---

# 🚀 Python Modules
**Modules** are the building blocks of Python projects. Whether you’re **managing dependencies with pip**, **handling files with os, or processing data with json**, mastering **Python’s module system and built-in libraries** is essential for clean, efficient, and scalable development.


## 🔧 Package Manager Tool - pip
When developing in Python, managing dependencies is essential — and **`pip`** is the built-in tool that does exactly that.

- `pip` is Python’s **official package management tool**
- It interfaces with **[PyPI](https://pypi.org/)**, which hosts **over 400,000+ packages**
- Included by default in:
  - **Python 2.7.9+**
  - **Python 3.4+**

###  Core pip Commands
```bash
pip install requests               # Latest version
pip install numpy==1.21.0          # specific version

pip install --upgrade requests     # Upgrade a specific package
pip uninstall pandas               # uninstall a specific package

python -m pip install --upgrade pip # Upgrade pip itself

pip list                     # View all installed packages in the current environment
pip list --outdated          # Check which package versions are outdated
pip show flask               # View detailed information about a specific package

pip freeze > requirements.txt # export -> Creates a full list of installed packages and their versions.
pip install -r requirements.txt # Reinstall from File -> Used to recreate environments (e.g., from GitHub projects or across machines).
```

## 📦 Common Python Built-in Modules
Python comes with a rich **standard library** of built-in modules that cover everything from file manipulation to mathematical computations.  

### `os`: File & Path Operations
Useful for interacting with the **operating system** — especially the **file system**.

```python
import os

# Get the current directory (current working directory) -> /Users/kai/MyProjects/ai-python
print(os.getcwd()) 

# List the files in the current directory -> ['hello.py', '__pycache__']
print(os.listdir("."))

# create the new directory
os.mkdir("demo_folder")

# Join paths (cross-platform safe) -> demo_folder/file.txt
full_path = os.path.join("demo_folder", "file.txt")
print(full_path)

# delete file.txt from demo_folder, will not delete the folder.
# FileNotFoundError if the file does not exist.
os.remove("demo_folder/file.txt")
```

### `json`: Handle JSON Data
JSON (JavaScript Object Notation) is the most common data format for **APIs**, **configuration files**, and **data exchange**.
Python’s built-in `json` module provides four core methods to handle **JSON serialization and deserialization**.


| Function           | Purpose                           | Input            | Output              |
|--------------------|------------------------------------|------------------|---------------------|
| `json.dump(obj, f)`| Serialize `obj` to a **file**          | Python object    | File (write JSON)   |
| `json.dumps(obj)`  | Serialize `obj` to a **string**        | Python object    | JSON string         |
| `json.load(f)`     | Parse JSON from file               | JSON file object | Python object       |
| `json.loads(str)`  | Parse JSON from string             | JSON string      | Python object       |

```python
import json

data = {"name": "Alice", "age": 25, "is_student": True}

# 1. Convert python data/dict to a JSON string. Disable ASCII escaping to preserve non-ASCII characters (e.g. Chinese)
json_str = json.dumps(data, ensure_ascii=False) 
print("JSON String:", json_str)  # JSON String: {"name": "Alice", "age": 25, "is_student": true}


# 2.2 JSON string to python dict
data_restored = json.loads(json_str)
print("Dict:", data_restored["name"])  # Dict: Alice


# 2.3 Read JSON from a file
# write to a file
with open("user.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=4)  # indent=4 for pretty print

# read from a file
with open("user.json", "r", encoding="utf-8") as f:
    loaded_data = json.load(f)
    print("file data:", loaded_data) # file data: {'name': 'Alice', 'age': 25, 'is_student': True}

```

### `sys`: Script-Level Controls
Use it for **command-line arguments, exiting,** and **environment info.**

**Status Code**
- 0: Success
- 1: General error
- 2: Misuse of shell builtins
- Others: Customizable error types

```python
import sys

# 1. Get command-line arguments
# read from cmd：python script.py arg1 arg2
print("Script Name:", sys.argv[0])

# 2. Forcefully exit the program with a status code
if len(sys.argv) < 2:
    print("Less paramaters！")
    sys.exit(1)  # General error

# 3. Temporarily add custom module path
sys.path.append("/my_modules")
```

### `random`: Random Data Generator
Essential for **games, sampling, or mock data generation.**

```python
import random

# 1 Generate a random integer, includes both ends
# Generate a random integer between 1 and 10
num = random.randint(1, 10)  
print("Random number:", num) # Random number: 4

# 2 Randomly select an element
fruits = ["Apple", "Banana", "Orange"]
choice = random.choice(fruits) 
print("Selected fruit:", choice) # Selected fruit: Apple

# 3 Shuffle a list in-place
cards = ["A", "2", "3", "J", "Q", "K"]
random.shuffle(cards)
print("Shuffled cards:", cards)  # Shuffled cards: ['J', '2', '3', 'K', 'Q', 'A']

# 4 Generate a random verification code (6 letters and numbers)
import string
chars = string.ascii_letters + string.digits  # get all letters and digits
code = ''.join(random.choices(chars, k=6))    # get 6 random chars
print("verification code:", code)  # verification code: fV1kDs
```

### `math`: Scientific and Geometric Calculations
Useful for **precise math, geometry, and engineering** use cases.

```python
import math

# 1. Calculate square roots and powers
a = math.sqrt(25)     # 5.0
b = math.pow(2, 3)    # 8.0
print(f"square root: {a}, power: {b}") # square root: 5.0, power: 8.0

# 2. Round up / Round down
c = math.ceil(3.2)    # 4
d = math.floor(3.8)   # 3
print(f"Round up to the nearest integer: {c}. Round down to the nearest integer: {d}.")
# Round up to the nearest integer: 4. Round down to the nearest integer: 3.

# 3. The constant π and angle conversion between degrees and radians
radius = 5
area = math.pi * radius ** 2  # Formula for the area of a circle
degrees = math.degrees(math.pi)  # 180.0
print(f"The area of a circle: {area:.2f}, Convert radians to degrees: {degrees}")
# The area of a circle: 78.54, Convert radians to degrees: 180.0

# 4. Logarithmic operations
log_value = math.log(100, 10)  # Compute base-10 logarithm of 100 → 2.0
print("Logarithmic result:", log_value)
# Logarithmic result: 2.0
```


<br>



---

🚀 Stay tuned for more insights into Python!