---
title: Python Basics
date: 2025-12-03
categories: [Backend, Python]
tags: [Python, Basics]
author: kai
permalink: /posts/backend/python/python-basics/
---

# 🚀 Python Basics

## 🏷️ Python Variables
Variables are one of the most fundamental concepts in imperative programming.  
A variable can be thought of as a **container that stores data** — a name that points to a value in memory.

However, Python handles variables very differently from many traditional languages like C, C++, or Java.

### Dynamic Typing
In Python:
- **Variables themselves do NOT have types**
- **Values have types**
- A variable is simply a *reference* (a label) that points to a value in memory

```python
# The type of x changes depending on what value you assign to it.
# Python allows this flexibility because the type is checked at runtime, not at compile time.
x = 10       # x refers to an integer
print(type(x))  # <class 'int'>
x = "hello"  # now x refers to a string
x = 3.14     # now x refers to a float
```

### Variable Naming Conventions

| Rule Type  | Correct Examples| Incorrect Examples |
|------------------|----------------|--------------------------|
| **Valid Characters**    | var_1, 变量_中文 | 2var, var-name |
| **Case Sensitivitye** | age != Age         | -                    |
| **Avoid Reserved Words** | list_ = [1,2,3] | list = [1,2,3] |
| **PEP8 Recommendation** | user_name | userName (PascalCase) |

### Guide References
[PEP 8 Style Guide](https://peps.python.org/pep-0008/) <br>
[Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)

## 📊 Boolean logic
Python has **only two boolean values**: `True` and `False`. They usually come from conditions or comparisons:

```python
x > 5
target in nums
len(arr) == 0
left <= right
is_valid = True
```

### Boolean Operators (and, or, not)
Python supports three logical operators: `and`, `or`, `not`.

- `and`: Returns True if both operands are True
- `or`: Returns True if at least one operand is True  
- `not`: Returns the opposite boolean value

### Operator Precedence
In Python, the precedence of logical operators is: `not` > `and` > `or`. To avoid confusion → always use parentheses in complex conditions

For example:
```python
result = not True and False or True
# Evaluation order: not True -> False, then False and False -> False, then False or True -> True

not A and B or C # ((not A) and B) or C
```

## 🧮 Integer Operations

### Assignment Operators
Python supports the following assignment operators:

- `=`: Simple assignment
- `+=`: Addition assignment
- `-=`: Subtraction assignment
- `*=`: Multiplication assignment
- `/=`: Division assignment
- `//=`: Integer division assignment
- `%=`: Modulo assignment
- `**=`: Exponentiation assignment

### Mathematical operators
Python supports the following mathematical operations:

- `+`: Addition
- `-`: Subtraction
- `*`: Multiplication
- `/`: Division
- `//`: Integer Division
- `%`: Modulo
- `**`: Exponentiation

#### / vs //
- `/`  → True division, always returns a **float**
- `//` → Integer/Floor division, returns the integer floor (the largest integer less than the result)
    - floor toward negative infinity

For example:
```python
print(2 / 3)   # 0.6666666666666666
print(4 / 2)   # 2.0
print(123 / 10) # 12.3

print(3 // 2)   # 1 (1.5 -> floor(1.5) = 1 rounded down)
print(123 // 10) # (12.3 -> floor(12.3) = 12 rounded down)
print(-7 // 4)  # -2 (-1.75 -> floor(-1.75) = -2 , because -2 < -1.75)
```

#### Getting Integer Part of a Division
Use `//` because it’s faster and avoids floating-point issues.

```python
int(3 / 2)   # 1
3 // 2       # 1   (recommended)
```

#### Remainder %
`a % b`: 
- a = dividend, b = divisor
- Rule 1: When the divisor is positive, Python remainder is always non-negative.
- Rule 2: Python remainder follows this identity: `a == (a // b) * b + (a % b)`
    - a = 17, b = 5 -> a // b = 3, a % b = 2 -> a == (17 // 5) * 5 + (17 % 5)
    - a = -6, b = 5 -> -6 // 5 = -2, -2 * 5 = -10， -6 = -2 * 5 + X -> X = 4 -> a % b
- Rule 3: Python % matches mathematical modulo


```python
print(6 % 5)    # 1
print(-6 % 5)   # 4   
```


#### Exponentiation **
Python 3 has no integer overflow — integers grow automatically to arbitrary size. This makes Python great for math-heavy problems.

```python
print(2 ** 3)   # 8
print(2 ** -3)  # 0.125
print(3 ** 100) # A very large integer
```

#### Extracting Digits

|---|---|
| Digit| Expression|
|---|---|
| Ones| num % 10|
| Tens| num // 10 % 10|
| Hundreds| num // 100 % 10|

```python
num = 123
print(num % 10)    # 3 (123 % 10 = 3)
print(num // 10 % 10) # 2 (123 // 10 = 12, 12 % 10 = 2)
print(num // 100 % 10) # 1 (123 // 100 = 1, 1 % 10 = 1)
```

<br>

---

🚀 Stay tuned for more insights into Python!