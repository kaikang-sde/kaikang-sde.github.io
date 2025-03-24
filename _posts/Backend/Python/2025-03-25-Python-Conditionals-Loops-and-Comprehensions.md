---
title: Python Conditionals, Loops, and Comprehensions
date: 2025-03-24
order: 2
categories: [Backend, Python]
tags: [Python, Loops, Conditionals, Comprehensions]
author: kai
---

# 🚀 Python Conditionals, Loops, and Comprehensions
Controlling the flow of your program is fundamental in Python. This post covers **conditional logic**, **comparison operators**, and **looping structures** to help you write dynamic and intelligent code.

## 🧦 Python Comparison Operators
Python provides a set of **comparison operators** that return boolean values (`True` or `False`) based on how values relate to each other. These are commonly used in conditional statements like `if`, `while`, and in logical expressions.

| Operator | Meaning         | Example         | Result   |
|----------|------------------|------------------|----------|
| `==`     | Equal to         | `5 == 5`         | `True`   |
| `!=`     | Not equal to     | `3 != 5`         | `True`   |
| `>`      | Greater than     | `10 > 5`         | `True`   |
| `<`      | Less than        | `3 < 2`          | `False`  |
| `>=`     | Greater or equal | `5 >= 5`         | `True`   |
| `<=`     | Less or equal    | `4 <= 3`         | `False`  |

**⚠️ Syntax Tip: Don't Confuse `=` and `==`**
- `=` is the **assignment operator**, used to assign values to variables.
- `==` is the **equality comparison operator**, used to check if two values are equal.

### 🔗 Chained Comparisons
Chained comparisons help make your conditions shorter and more readable.

```python
x = 3
print(1 < x < 5)      # True (equivalent to: 1 < x and x < 5)
print(x == 3 == 3.0)  # True (works across types)
```


## 🧤 Python Logical Operators
In Python, **logical operators** are used to combine multiple conditions or invert them. They play a crucial role in conditional statements and control flow.

| Operator | Description                | Example                  | Result     |
|----------|----------------------------|--------------------------|------------|
| `and`    | Logical AND (all must be True) | `(5 > 3) and (2 < 1)`    | `False`    |
| `or`     | Logical OR (at least one True) | `(5 > 3) or (2 < 1)`     | `True`     |
| `not`    | Logical NOT (negates result) | `not (5 > 3)`            | `False`    |

### 🔺 Operator Precedence
Python follows this **logical operator precedence**: `not > and > or`<br>
To make precedence explicit and your logic clear, use parentheses:
```python
x = True
y = False
print(not x or y)         # Equivalent to: (not x) or y → False
print(not (x or y))       # Not of the whole expression → False
```

### 💥 Short-Circuit Evaluation
Python uses **short-circuiting** to improve performance and prevent unnecessary evaluations.

1. **and** Short-Circuit Rule
- If the **left-hand side is False**, Python **will not** evaluate the right-hand side:

2. **or** Short-Circuit Rule
- If the **left-hand side is True**, Python **will not** evaluate the right-hand side:

```python
print(False and 1/0)  # Output: False (no ZeroDivisionError!)

print(True or 1/0)   # Output: True (no ZeroDivisionError!)
```

### 💫 Ternary Operator
In Python, a **conditional expression**, also known as the **ternary operator**, allows you to write simple `if-else` logic in a **single line**.<br>
**Syntax:** `value_if_true if condition else value_if_false`

It evaluates the condition, and returns:
- value_if_true if the condition is True
- value_if_false if the condition is False

```python
a, b = 5, 10
max_value = a if a > b else b
print(max_value)  # Output: 10

----------Equivalent to:------------
if a > b:
    max_value = a
else:
    max_value = b
```


### ⚖️ Conditional Statements
Conditional statements allow your program to **make decisions and branch execution paths** based on conditions.<br>

**Basic Syntax**
```python
if condition1:
    block1
elif condition2:
    block2
else:
    block3
```

**Execution Order**
- Python evaluates conditions from **top to bottom**:
    - If condition1 is True, block1 is executed.
    - If condition1 is False but condition2 is True, block2 is executed.
    - If none of the conditions are True, block3 is executed (the else block).
- Once a condition matches, the rest are skipped.


## 🧣 Basic Loops 
Loops allow you to **execute code repeatedly**, making them essential for processing data, automating tasks, and controlling flow in Python.

### for loop
In Python, the `for` loop is used to **iterate over iterable objects** such as lists, tuples, strings, and ranges.<br>

**Syntax:** 
```python 
for variable in iterable:
    # loop body
```

- variable: Each item in the iterable will be assigned to this variable during each loop iteration.
- iterable: Any sequence type (like list, tuple, str, range, etc.).

### while loop
The `while` loop in Python is used to **execute a block of code repeatedly** as long as a specified condition is `True`.

```python
while condition:
    # loop body
```
- condition: A boolean expression. If True, the loop continues; if False, the loop ends.
- loop body: The indented block of code that runs on each iteration.

### Loop Control
Python provides special statements to control the flow inside loops. Two of the most common are `break` and `continue`.

1. **break** - Exit the Loop Immediately
- The `break` statement **terminates the loop** as soon as it's executed. 

2. **continue** - Skip Current Iteration
- The `continue` statement **skips the rest of the code in the loop body** for the current iteration and moves to the next one.

### Examples
```python
----------------break------------------
for num in range(10):
    if num == 5:
        break
    print(num)     

# Output:
0
1
2
3
4
# Once num == 5, the loop exits — no further iterations are performed.

----------------continue------------------
for num in range(10):
    if num % 2 == 0:
        continue
    print(num)

# Output: All even numbers are skipped due to the continue statement.
1
3
5
7
9
# All even numbers are skipped due to the continue statement.
```


## 👟 Advanced for Loops
Python provides a powerful feature called **comprehension syntax**, which allows concise and expressive data transformations. 

### What is a Comprehension?
- Constructing new sequences (list, set, dict, etc.)
- Based on an existing iterable
- Using a single-line expression that includes optional filtering

> ✨ It replaces many verbose `for` loop patterns with **one-liner expressions** that are both powerful and clean — but must be used responsibly to maintain readability.

### 1️⃣ List Comprehensions
To quickly **build a new list** from an existing iterable, with optional filtering or transformation.<br>

**Syntax:** 
1. **basic:** `[expression for item in iterable]`
2. **With Conditional Logic:** `[expression for item in iterable if condition]`
3. **With condition and expression:** `[expression1 if condition else expression2 for item in iterable]`

**Parameters:**
1. **iterable:** 
    - An existing collection or sequence that can be looped over.
    - Examples: list, tuple, string, range(), etc.
2. **item:** 
    - Represents each element in the iterable.
	- If no condition is specified, every item is processed.
	- If a condition is specified, only the matching items are processed.
3. **condition (optional):**
    - A filtering condition that determines which items are included.
	- If the condition is True, the item proceeds to the expression step.
4. **expression:**
    - Defines how to transform each item.
	- Can be any valid expression using item.

#### Examples
```python
nums = [1, 2, 3, 4, 5]

# 1. Without condition
squares = [x**2 for x in nums]  
# Output → [1, 4, 9, 16, 25]


# 2. With condition
evens = [x for x in nums if x % 2 == 0]  
# Output → [2, 4]


# 3. With condition and expression
labels = ["even" if x % 2 == 0 else "odd" for x in nums]  
# Output → ['odd', 'even', 'odd', 'even', 'odd']

# 4. Exam Score Conversion to Letter Grades.
scores = [78, 92, 65, 88, 54]
grades = ["A" if grade > 80 else "B" for grade in scores]
print(grades) # Output → ['B', 'A', 'B', 'A', 'B']
```


### 2️⃣ Dictionary Comprehension
A **dictionary comprehension** is a concise way to construct a dictionary by transforming or filtering an iterable in a single line of code.<br>

**Syntax:**<br>
`{ key_expression: value_expression for variable in iterable [if condition] }`

**Parameters:**
1. **key_expression:** 
    - Expression that defines the key.
2. **value_expression:** 
    -  Expression that defines the value.
3. **variable in iterable:**
    - Loop over any iterable object (e.g. list, range, tuple).
4. **if condition (optional):**
    -  Filter condition.


#### Examples
```python
# 1. Square numbers in a dictionary
squares = {x: x**2 for x in range(1, 6)}
print(squares)  
# Output: {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}


# 2. Filter even keys
even_squares = {x: x**2 for x in range(1, 10) if x % 2 == 0}
print(even_squares)
# Output: {2: 4, 4: 16, 6: 36, 8: 64}


# 3. Convert two lists to a dictionary
keys = ['name', 'age', 'language']
values = ['Kai', 28, 'Python']
# loops over each tuple and unpacks the elements into variables k and v, then builds key-value pairs:
info = {k: v for k, v in zip(keys, values)} # {k: v for k, v in [('name', 'Kai'), ('age', 28), ('language', 'Python')]}
print(info)
# Output: {'name': 'Kai', 'age': 28, 'language': 'Python'}


# 4. flipping the keys and values of a dictionary.
original = {'a': 1, 'b': 2, 'c': 3}
# dict_items([('a', 1), ('b', 2), ('c', 3)]) -> Each k is a key ('a', 'b', 'c'). Each v is a value (1, 2, 3).
reversed_dict = {v: k for k, v in original.items()}   
print(reversed_dict) 
# Output: {1: 'a', 2: 'b', 3: 'c'}


# 5. Extract Math Scores by Student Name
students = [
    {'name': 'Alice', 'math': 85, 'english': 90},
    {'name': 'Bob', 'math': 78, 'english': 82},
    {'name': 'Charlie', 'math': 92, 'english': 88}
]
math_scores = {stu['name']: stu['math'] for stu in students}
print(math_scores)
# Output: {'Alice': 85, 'Bob': 78, 'Charlie': 92}
```


### 3️⃣ Set Comprehension
A **Set comprehension** is a concise and readable way to generate sets from iterable data, often used to filter or transform values on the fly.<br>
**Syntax:**<br>
`{ expression for variable in iterable [if condition] }`

**Parameters:**
- **expression:** The value to include in the final set.
- **variable in iterable:** Iterates through each element in the input.
- **if condition (optional):** Filters the iterable elements before applying the expression.

#### Examples
```python
# 1. Union: Combine All Unique Elements
A = {1, 2, 3}
B = {3, 4, 5}
union = {x for x in A | B}
print(union) # Output: {1, 2, 3, 4, 5}


# 2.  Intersection (Common Elements Only)
intersection = {x for x in A if x in B}
print(intersection) # Output: {3}


# 3. Data Deduplication
words = ['apple', 'banana', 'apple', 'orange', 'banana']
unqiue_words = {word for word in words}
print(unqiue_words) # Output: {'orange', 'banana', 'apple'}

unqiue_words = {word.upper() for word in words}
print(unqiue_words) # Output: {'ORANGE', 'APPLE', 'BANANA'}


# 4. Feature Extraction (Vowels in Text)
text = "The quick brown fox jumps over the dog"
vowels = {'a', 'e', 'i', 'o', 'u'}
found_vowels = {char.lower() for char in text if char.lower() in vowels}
print(found_vowels) # Output: {'u', 'e', 'o', 'i'}
```


### 4️⃣ Tuple Comprehension
Many developers refer to **tuple comprehension**, but in reality, Python does **not** have a dedicated comprehension syntax that directly creates a tuple.  Instead, when you use **parentheses** `()` with a comprehension-like structure, Python creates a **generator expression**.<br>
**Syntax:**<br>
`(expression for variable in iterable [if condition])`

```python
# Generator expression (note: this returns a generator object)
a = (x for x in range(1, 10))

# Convert the generator to a tuple
result = tuple(a)

print(result) # Output: (1, 2, 3, 4, 5, 6, 7, 8, 9)

```


### Comprehension vs. Generator Expression

| Feature | Generator Expression (Tuple-Style) | List Comprehension |
|--------|------------|---------|
| Syntax | (expression for ...) | [expression for ...] |
| Output | Generator object | List |
| Evaluation | Lazy (on-demand)<br>values are computed only when needed.  | Eager (all at once) |
| Memory usage | Efficient | Higher for large data |
| Reusability | Â One-time iteration | Can reuse |

#### Why Use Generator Expressions?
- Lazy evaluation: values are computed only when needed.
- Efficient for processing large data streams.
- Low memory footprint compared to full list creation.


#### Important Notes
- Generator expressions cannot be indexed or sliced.
- You can only iterate once — afterward, it’s exhausted.
```python
gen = (x for x in range(3))
print(list(gen))  # [0, 1, 2]
print(list(gen))  # [] — already consumed
```


<br>


---

🚀 Stay tuned for more insights into Python!