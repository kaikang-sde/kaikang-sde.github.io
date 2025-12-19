---
title: Python Basics
date: 2025-03-18
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

## ☢️ Integer Operations

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
Python Integer supports the following mathematical operations:

- `+`: Addition
- `-`: Subtraction
- `*`: Multiplication
- `/`: Division
- `//`: Integer Division
- `%`: Modulo
- `**`: Exponentiation

#### / vs //
- `/`  → True division, always returns a **float**
- `//` → Integer/Floor division, returns the integer floor (Floor(x) = the largest integer that is less than or equal to x -> The largest integer in here refers to the numerical value, not the data type)
    - Floor toward negative infinity
    - If either operand is a float, the result is a float
    - If both operands are integers, the result is an integer

For example:
```python
print(2 / 3)   # 0.6666666666666666
print(4 / 2)   # 2.0
print(123 / 10) # 12.3

print(3 // 2)   # 1 (1.5 -> floor(1.5) = 1 rounded down)
print(3.0 // 2)  # 1.0 (1.5 -> floor(1.5) = 1.0 rounded down)
print(123 // 10) # 12 (12.3 -> floor(12.3) = 12 rounded down)
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

## 🧮 Float Operations
Floats follow IEEE-754 double-precision format, meaning:
- They can represent very large or very small real numbers
- They may have rounding errors due to binary representation
- They are not exact when handling decimals like 0.1 or 0.2

```python
print(0.1 + 0.2)   # 0.30000000000000004
```

### Assignment Operators
Python Float supports the following assignment operators:

- `+=`: Addition assignment
- `-=`: Subtraction assignment
- `*=`: Multiplication assignment
- `/=`: Division assignment


### Mathematical operators
Python Float supports the following mathematical operations:

- `+`: Addition
- `-`: Subtraction
- `*`: Multiplication
- `/`: Division

#### Notes
- There is no separate division operator for floats — `/` always performs true division and returns a float.
    - `print(3.5 / 1.2)   # 2.916666...`
- Floats are not exact, so comparisons should avoid direct equality.
    ```python
    # Not recommended
    if x == 0.3:
        ...
    # Better: compare with a tolerance
    if abs(x - 0.3) < 1e-9:
        ...
    ```
- Python’s `math.isclose()` is even clearer:
    ```python
    import math
    if math.isclose(x, 0.3, rel_tol=1e-9):
        ...
    ```

## 🕹️ Control Flow
Control flow is a foundational concept for writing clean and correct solutions. It determines **how your program executes**, how decisions are made, and how loops iterate over data.  

### 1. Sequential Execution
Python executes statements **from top to bottom**, line by line:

```python
# This is the simplest control flow: each statement runs exactly once in order.
a = 1
b = 2
c = a + b
print(c)
```


### 2. Conditional Execution
Conditionals allow your program to branch based on logic.

#### if statement
```python
if condition:
    # do something
```

#### if-else statement
```python
if condition:
    # executed when condition is True
else:
    # executed when condition is False
```

#### if-elif-else statement
Used for multiple mutually exclusive conditions:
```python
if condition1:
    # executed when condition1 is True
elif condition2:
    # executed when condition1 is False and condition2 is True
else:
    # executed when both condition1 and condition2 are False
```

### 3. Loop Execution
Loops are essential when iterating through arrays, matrices, graphs, or trees.

#### For loop
Used when iterating over a sequence.

1. **Looping directly through values** <br>
Best when you only need the value itself:

```python
for item in iterable:
    # do something with item
```

2. **Looping with indices using range** <br>
Useful when you need the index:

```python
for i in range(len(nums)):
    print(i, nums[i])
```

3. **Looping with both index and value (enumerate)** <br>
Useful when you need both the value and its index:

```python
for i, num in enumerate(nums):
    print(i, num)
```


#### While loop
A while loop repeats as long as its condition is True. While loops are useful when:
- You don’t know the number of iterations in advance
- You want full manual control over iteration
- Implementing two-pointer algorithms

Be careful to update variables to avoid infinite loops.

```python
i = 0
while i < len(nums):
    print(nums[i])
    i += 1   # manually update loop variable
```

#### Nested Loops
Nested loops often appear in matrix problems, brute force approaches, and dynamic programming.

```python
for i in range(len(nums)):
    for j in range(i + 1, len(nums)):
        print(nums[i], nums[j])
```

### 4. Indentation Rules
Python uses indentation to define code blocks.
- Code with the same indentation level belongs to the same block.
- Standard indentation is 4 spaces (not tabs).
- Good indentation directly affects correctness — especially in loops and nested conditionals.


### 5. Understanding range()
Range is **inclusive at the start** and **exclusive at the end**.

```python
range(n)           # 0, 1, ..., n-1
range(m, n)        # m, m+1, ..., n-1
range(m, n, k)     # m, m+k, m+2k, ..., last < n
range(n, m, -1)    # n, n-1, ..., m+1
```


### 6. Loop Control: break and continue
#### break — exit the loop immediately

```python
for i in range(10):
    if i == 5:
        break
    print(i)
# Output: 0 1 2 3 4
```

#### continue — skip the rest of this iteration and continue to next loop

```python
for i in range(10):
    if i == 5:
        continue
    print(i)
# Output: 0 1 2 3 4 6 7 8 9
```


## 🥋 Functions
A **function** is a block of code that performs a specific task.

Benefits:

- **Code reuse** — write once, use many times  
- **Readability** — logical grouping of behavior  
- **Maintainability** — easier to test and debug  

Python provides many **built-in functions** (e.g., `print()`, `len()`, `sorted()`), but you can also define your own — known as **user-defined functions**.

### General Structure of a Function
In a broad sense, every function follows this pattern: input → function → output
- Python uses the `def` keyword to define a function.
- Function names are normally started with a verb (e.g., get_max, find_index, compute_sum)
- Parameters define the inputs
- return defines the output (if omitted, the function returns None)

A function call:
- Executes the code inside the function
- Produces an output value that you can use like any other variable

```python
# Define a function
def add(a, b):
    return a + b

# Call the function
result = add(3, 5)
print(result)   # 8
```

### Function Parameters
Python uses **pass-by-value** (of the reference).
More clearly:
- For **immutable types** (int, float, str, tuple), Python copies the reference, but since values cannot change, it behaves like **copying the value.**

```python
def increment(x):
    x += 1
    return x

a = 5
print(increment(a))   # 6
print(a)              # 5 (unchanged)
```

- For **mutable types** (list, dict, set), Python copies the reference, but since values can change, it behaves like **passing the reference.**

```python
def add_one(nums):
    nums.append(1)

lst = [0]
add_one(lst)
print(lst)            # [0, 1]
```

## 🎁 Object-Oriented Programming
OOP is both:

- A **way of thinking**: everything in the world can be seen as an object  
- A **way of designing programs**: represent real-world entities using code

In Python, almost **everything is an object** — integers, strings, lists, functions, even classes themselves.


### 1. Object

An **object** represents something with:

- **Attributes (properties)** → what it *has*  
- **Behaviors (methods)** → what it *does*

**Attributes** <br>
- Dog → four legs
- Car → wheels, steering wheel
- Computer → screen, keyboard

**Behaviors** <br>
- Dog → bark
- Car → accelerate, turn
- Computer → run programs, play videos

Objects combine **data + logic**, making them excellent for representing real-world entities.


### 2. Class
A **class** is a blueprint for creating objects.

- It defines what **attributes** and **behaviors** its objects will have
- It does *not* create anything until instantiated
- In Python, a class is the abstraction of some real-world concept

Syntax rules:

- Use the `class` keyword
- Class names use **UpperCamelCase**
- Define attributes and methods inside the class
Example:

```python
class Dog:
    def __init__(self, name, age):
        self.name = name     # attribute
        self.age = age       # attribute

    def bark(self):          # method (behavior)
        print(f"{self.name} is barking!")
```

### 3. `__init__`(Constructor)
`__init__ `is the constructor — Python automatically calls it when creating an object. It initializes object attributes. 

```python
def __init__(self, name, age):
    self.name = name
    self.age = age
```

### 4. self
**self** is a reference to **the current object (instance)**.
- Required when defining methods
    - Inside a class, methods need a way to access:
        - Attributes that belong to the object
        - Other methods of the object

    ```python
    def accelerate(self, amount):
        self.speed += amount
    # Without self, Python would have no idea which object’s speed you’re trying to update.
    ```
- Hidden when calling methods (Python passes it automatically)
    ```python
    c = Car("Tesla") 
    # Python creates an object in memory and then calls: Car.__init__(c, "Tesla")
    # Here, c is passed as the first argument — that argument is what we name self.
    # So, inside the class: self.brand = brand
    # Means: “Assign this object’s brand attribute to the value passed in.”
    ```
- Every object gets its own self.


### 5. Creating an Object (instantiation)
A class is only a definition. An instance (or object) is created from that class.

When we instantiate:
- Python allocates memory for the object
- Calls `__init__`
- Returns a usable object

### 6. Member Variables (Attributes)
Also called fields.

- Attributes describe the object’s state. 
- Naming rules follow standard variable naming conventions.
- Attributes belong to objects, not classes.

```python
self.name
self.age
self.score
```

### 7. Member Functions (Methods)
Also called methods.

- Describe what an object can do. 
- Naming rules follow normal function naming conventions.
- Methods always take self as the first parameter.

```python
def bark(self):
    print("Woof!")
```

### 8. Access Control for Member Variables & Methods
In Python, all attributes (member variables) and methods (member functions) are **public by default**. This means they can be accessed:
- Inside the class  
- Outside the class  
- Even from other modules  

However, in real-world software design, we often need **access control** to protect internal state, avoid accidental misuse, and provide controlled interfaces for modification.  
Python provides *convention-based* access control using naming rules.

#### Public Members (Default in Python)

Anything defined without a leading underscore is **public**.

```python
class User:
    def __init__(self):
        self.name = "Alice"      # public attribute

    def greet(self):             # public method
        print("Hello!")

# You can access them anywhere:
u = User()
print(u.name)
u.greet()    
```

#### Protected Members (_attribute)
Anything defined with a single leading underscore is **protected**.

Meaning:
- Intended for internal use within the **class or subclasses**
- Not meant to be accessed from outside
- Python does NOT enforce this — it’s a convention, not a restriction
- IDEs and linters will warn you


```python
class User:
    def __init__(self):
        self._token = "abc123"   # protected attribute
```

#### Private Members (__attribute)
A double leading underscore triggers name mangling. Private members:
- Can only be accessed inside the class
- Cannot be accessed directly from outside
- Cannot be used by subclasses
- Python rewrites the attribute name to _ClassName__attribute internally

```python
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance   # private attribute

    def deposit(self, amount):
        self.__balance += amount

acct = BankAccount(100) 
print(acct.__balance)  # AttributeError
print(acct._BankAccount__balance)  # Works (but not recommended)
```

### 9. Getter and Setter Methods
Because private variables cannot be accessed directly, Python classes often use **getter and setter** methods to provide controlled access.

```python
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance

    # Getter
    def get_balance(self):
        return self.__balance

    # Setter with validation
    def set_balance(self, value):
        if value < 0:
            raise ValueError("Balance cannot be negative")
        self.__balance = value

acct = BankAccount(100)
print(acct.get_balance())     # 100
acct.set_balance(150)         # updates safely
```


### 10. Complete Example
To clearly demonstrate **class definition**, **attributes**, **methods**, **access control**, **constructor (`__init__`)**, and **instantiation**,  
here is a complete, real-world example using a `Job` class.

This example models a job posting with:

- Public attributes  
- Protected and private attributes  
- Getter/Setter methods  
- Behavior (methods)  
- Object instantiation 

```python
class Job:
    def __init__(self, title, company, salary):
        # Public attribute
        self.title = title

        # Protected attribute (convention: internal use)
        self._company = company

        # Private attribute (name-mangled)
        self.__salary = salary

    # Public method
    def describe(self):
        print(f"Job Title: {self.title}")
        print(f"Company: {self._company}")
        print(f"Salary: ${self.__salary}")

    # Getter for private salary
    def get_salary(self):
        return self.__salary

    # Setter for private salary with validation
    def set_salary(self, value):
        if value <= 0:
            raise ValueError("Salary must be positive.")
        self.__salary = value

    # Additional behavior method
    def promote(self, increase):
        print(f"Promoting job '{self.title}' with salary increase of {increase}")
        self.__salary += increase


# Create instance (object) of Job class
job1 = Job("Software Engineer", "Google", 150000)
job2 = Job("Data Scientist", "Amazon", 170000)
print(job1.title)   # Software Engineer (public)
print(job1._company)  # Google (protected)
print(job1.__salary)  # AttributeError (private)
print(job1.get_salary())     # Access via getter (private)

# Access attributes and methods
job1.describe()         # Job Title: Software Engineer
                        # Company: Google
                        # Salary: $150000

job2.promote(20000)     # Promoting job 'Data Scientist' with salary increase of 20000
job2.describe()         # Job Title: Data Scientist
                        # Company: Amazon
                        # Salary: $190000
```


<br>

---

🚀 Stay tuned for more insights into Python!