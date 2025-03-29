---
title: Python Lambda (Anonymous) Functions
date: 2025-03-28
categories: [Backend, Python]
tags: [Python, lambda, anonymous-function]
author: kai
---

# 🚀 Python Lambda (Anonymous) Functions
Python's `lambda` functions — also known as **anonymous functions** — provide a concise way to write small, throwaway functions without needing a full `def` block.

## 🔒 Basic Syntax
```python
lambda parameters: expression
```

- **No def** keyword
- **No function name**
- Must consist of a **single expression**
- Can accept **multiple parameters**, separated by commas
- Lambda functions are often used as **inline callbacks or function arguments**.

### Examples
```python
# 1: Add Two Numbers
add = lambda a, b: a + b
print(add(3, 5))  # Output: 8

# 2. Square a Number
square - lambda x: x**2
print(square(4))  # Output: 16

# 3. Using Lambda as a Function Argument
def operate(func, a, b):
    return func(a, b)

result = operate(lambda x, y: x * y, 3, 4)
print(result)  # Output: 12 -> func = lambda x, y: x * y
```

## ⚠️ Limitations of Lambda Functions
- Single expression only — no statements or multiline blocks
    - A `lambda` can only contain **a single expression**. You **cannot** include statements like `if`, `for`, or `try`.
    - Assignment (=), Loops (for, while), Condition blocks (if, elif, else), Exception handling (try, except)  **are not allowed** inside lambda.
- No return statement — the result of the expression is implicitly returned
- Best suited for simple, short-term logic only
- Avoid using lambda for complex logic. Use def instead for clarity and readability.

```python
invalid = lambda x: if x > 0: x else -x # ❌ SyntaxError

valid = lambda x: x if x > 0 else -x # ✅ Valid: Use inline conditional expressions

complex_lambda = lambda x: (lambda y: x + y)(10) # ❌ Complex, hard-to-read lambda
```


## 📊 Feature Comparison: Lambda vs. def

| Feature         | `lambda` Function                    | Regular Function (`def`)              |
|------------------|--------------------------------------|----------------------------------------|
| **Name**         | Anonymous (no name by default)       | Must be named                          |
| **Code Length**  | Single-line only                     | Supports multi-line code blocks        |
| **Use Case**     | Simple logic (e.g., expression eval) | Complex logic (loops, conditionals)    |
| **Return Value** | Implicit (expression result)         | Explicit with `return` keyword         |


## 🆚 Lambda + map/filter vs. List Comprehensions
Python offers multiple ways to perform **data transformation and filtering** — most notably:

- `lambda` functions combined with `map()` or `filter()`
- List comprehensions

| Scenario                       | `lambda` + `map`/`filter`         | List Comprehension                   |
|-------------------------------|-----------------------------------|--------------------------------------|
| **Simple transformation**     | Less readable                     | More readable and intuitive          |
| **Simple filtering**          | Less expressive                   | Clean and concise                    |
| **Function as parameter**     | Required (`lambda` or named func) | Not directly supported               |
| **Performance**               | Comparable                        | Comparable (generator saves memory)  |

### Examples
```python
# 1. Simple Transformation 
# 1.1 `lambda` + `map`
nums = [1, 2, 3, 4, 5]
res = list(map(lambda x: x**2, filter(lambda x: x % 2 == 0, nums)))
print(res)  # Output: [4, 16]

# 1.2 List Comprehension
res = [num**2 for num in nums if num % 2 == 0]
print(res) # [4, 16]
```




<br>


---

🚀 Stay tuned for more insights into Python!