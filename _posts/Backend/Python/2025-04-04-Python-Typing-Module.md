---
title: Python Typing Module
date: 2025-04-01
order: 4
categories: [Backend, Python]
tags: [Python, typing, type-hints, static-checking]
author: kai
---

# 🚀 Python Typing Module
Since **Python 3.5**, the introduction of **PEP 484** added **type hints** to the language — enabling static type checking, better editor support, and improved code readability.

While Python is still **dynamically typed**, the `typing` module provides tools for optional type annotations without affecting runtime behavior.

```python
# Traditional dynamic type code example
def calculate(a, b):
    return a + b  # cannot see the types

result1 = calculate(3, 5)    # No error
result2 = calculate("3", 5)  # Raise error until runtime
```

## 🎯 Why Use Typing?

| Benefit                | Description                                                                |
|------------------------|----------------------------------------------------------------------------|
| **Improved Readability** | Clarifies expected input/output types for functions and variables     |
| **Smarter IDEs**         | Enables autocomplete, inline type hints, and refactoring tools        |
| **Static Checking**      | Tools like `mypy` detect bugs before runtime                          |
| **Better Documentation** | Auto-generates more informative API docs with types                   |


## 🤷🏻 Before Types
- If you are not using mypy, pyright, or Pylance in strict mode
- Type hints don’t affect runtime behavior, so Python still executes the code
- IDEs, like VSCode, might just treat the incorrect hint as a generic object and ignore it

If you enable mypy or set "python.analysis.typeCheckingMode": "strict" in VSCode:

## ⚙️ Primitive Types

Type annotations can be added to **variables, function parameters**, and **return values** using Python's built-in types.

```python
age: int = 25                    # Integer
name: str = "Alice"              # String
price: float = 9.99              # Floating-point
is_valid: bool = True            # Boolean
data: bytes = b"binary data"     # Byte data

# Python type hints accept subtypes, meaning:
value: int = True  # Valid, because bool is a subtype of int
```
> ⚠️ Note: Type hints do not affect runtime execution — they are only enforced by static checkers like mypy.

## 📦 Container Types
In modern Python, especially since **PEP 484**, the `typing` module provides powerful ways to annotate **container types** like `List`, `Dict`, `Tuple`, and `Set`.


| Use Case                         | Typing Syntax Example               |
|----------------------------------|-------------------------------------|
| A list of integers               | `List[int]`                         |
| A dictionary with string keys and integer values | `Dict[str, int]`     |
| A set of float values            | `Set[float]`                        |
| A tuple with fixed structure     | `Tuple[str, int, bool]`             |
| A list of lists (matrix)         | `List[List[float]]`                 |

> 💡 All types must be **imported from `typing`** (Python 3.8 and earlier).<br>
🆕 From Python 3.9+: use built-in generics like list[int], dict[str, float], etc.<br>
🆕 From Python 3.10+: use X | Y instead of Union[X, Y].

### List: Homogeneous Sequence
Use `List[X]` when you're working with a list of elements of the same type.

```python
from typing import List

scores: List[int] = [90, 85, 95]  # A list of integers

# Nested list (e.g., 2D matrix)
matrix: List[List[float]] = [[1.1, 2.2], [3.3]]

# Python 3.9+ 
scores:list[int] = [90, 85, 95]
```

### Dict: Key-Value Hints
use `Dict[KeyType, ValueType]` when you're working with key-value mappings. 

```python
from typing import Dict, Union

# Union: each value can be an int or a str.
config: Dict[str, Union[int, str]] = {
    "timeout": 30,     #int
    "mode": "fast"     #str
}

# Python 3.10+
new_config: dict[str, str | int] = {"timeout": 30,  "mode": "slow"}
```

### Tuples: Fixed and Variable-Length Sequences
Using the `Tuple` type when you're working with **fixed-structure tuples** and **variable-length tuples**.
- `Tuple[T1, T2, ..., Tn]`: a tuple with known length and element types(can be different types)
- `Tuple[T, ...]`: a tuple with any number of elements of the same type


```python
from typing import Tuple

point: Tuple[float, float] = (1.5, 2.0)               # fixed length and same type
record: Tuple[int, str, bool] = (101, "Kai", True)    # fixed length and different types
scores: Tuple[int, ...] = (90, 85, 88, 92)            # Any-length tuple of ints, same type
empty: Tuple[()] = ()                                 # Empty tuple

# Python 3.9+
point: tuple[float, float] = (1.5, 2.0) 
```

### Set: Unordered Collections with Unique Elements
Use `Set[T]` when working with **unordered collections of unique values**. All elements follow **a specific type** or **union of types**.

```python
from typing import Set, Union

# A set of integers
unique_ids: Set[int] = {1, 2, 3, 4}

# A set of strings or integers (mixed types)
tags: Set[Union[str, int]] = {"urgent", 1001}

# Python 3.10+
tags: set[str | int] = {"urgent", 1001} 
```


### Any: Flexibility with Caution
In Python's typing system, `Any` is the ultimate wildcard.  
It tells the type checker to **skip all type checks** for the annotated value — meaning it can be **anything**.

While useful for dynamic behavior or legacy code, `Any` should be used **sparingly** to avoid losing the benefits of static typing.

```python
from typing import Any

def debug_log(obj: Any) -> None:
    print(repr(obj))

debug_log("test")      # 'test'
debug_log([1, 2, 3])   # [1, 2, 3]
```


### Function Type Annotations 
Function annotations in Python allow you to **explicitly define the types of parameters and return values**, making your code more self-documenting, testable, and IDE-friendly.

- **param**: type → Defines parameter type
- **->** type → Defines return type

```python
def greet(name: str) -> str: # param -> name -> str, return value -> str
    return f"Hello, {name}"

def calculate(a: int, b: int) -> int: # param -> a -> int, param -> b -> int, return value -> int
    return a * b

# None: no return value
def show_info(info: str) -> None: # param -> info -> str, no return value 
    print(info)
```
> 🧪 Type annotations are ignored at runtime and only used by static checkers like mypy.


### Literal: Enforcing Value-Level Constraints
In Python’s type hinting system, `Literal` is used to **restrict a variable or function argument to a set of specific constant values** — essentially mimicking **type-safe enums**.

Introduced in **PEP 586**, `Literal` helps prevent bugs caused by invalid "magic strings" and improves code safety and editor support.

```python
from typing import Literal

HttpMethod = Literal["GET", "POST", "PUT", "DELETE"]

def send_request(method: HttpMethod, url: str) -> None:
    print(f"Sending {method} request to {url}")

send_request("POST", "/api")  
send_request("PATCH", "/api") # mypy error: Argument 1 to "send_request" has incompatible type "Literal['PATCH']"; expected "Literal['GET', 'POST', 'PUT', 'DELETE']"  [arg-type]
```

### Union: Accepting Multiple Input Types Safely
`Union[X, Y]` — which allows variables or parameters to be **one of several possible types**.
When using Union, you usually need to **check the type at runtime** before operating on the value.

```python
from typing import Union

def safe_length(x: Union[str, list]) -> int:
    if isinstance(x, str):
        return len(x)
    elif isinstance(x, list):
        return len(x)
    return 0

# Python 3.10+
def process(value: int | str) -> None:
    ...
```


## Optional: Safely Handling Nullable Values
`Union[X, None]` allows a value to be either **type X or None**, but the argument is still **required** when calling the function. To make the argument optional (i.e., callable without passing anything), you must also provide **a default value like = None.**. —> you **cannot omit the argument** entirely.

- Function parameters with default values
- Return values that might be absent or missing
- Attributes that are initialized later
- Representing nullable fields from APIs or databases



```python
from typing import Optional

def greet1(name: Optional[str] = None) -> str:
    if name:
        return f"Hello, {name}!"
    else:
        return "Hello, world!"

def greet2(name: Optional[str]) -> str:
    if name:
        return f"Hello, {name}!"
    else:
        return "Hello, world!"

print(greet1())         # Hello, world!
print(greet1("Kai"))    # Hello, Kai!

print(greet2())         # mypy: TypeError: greet2() missing 1 required positional argument: 'name'
print(greet2(None))     # Hello, world! 
print(greet2("Jojo"))   # Hello, Jojo!
```


## Type Alias: Creating Semantic Names for Complex Types
A **type alias** is simply a descriptive name assigned to an existing type or combination of types.  
It helps clarify the **semantic meaning** of values, especially in domain-specific contexts.

```python
from typing import List, Tuple

UserId = int # UserId as an alias for int
Point = Tuple[float, float] # Point as a 2D coordinate (x, y) using Tuple[float, float]

def get_user(id: UserId) -> str:
    return f"User{id}"

def plot(points: List[Point]) -> None: # without Type Alias: def plot(points: List[Tuple[float, float]]) -> None:
    for x, y in points:
        print(f"({x}, {y})")


print(get_user(1))     # User1
plot([(1, 2), (3, 4)]) # (1, 2)
                       # (3, 4) 
```


## NewType: Stronger Type Safety for Domain-Specific Values
`NewType` creates a **new distinct type** based on an existing one. This allows type checkers like `mypy` to **treat it as a separate type**, even though it behaves the same at runtime.

```python
from typing import NewType

UserId = NewType('UserId', int)

def print_id(user_id: UserId) -> None:
    print(user_id)

print_id(UserId(1001))  # 1001
print_id(1001)      # mypy error: Argument 1 to "print_id" has incompatible type "int"; expected "UserId"  [arg-type]. 
```


## TypeVar: Building Generic Functions and Classes
`TypeVar` allows you to define a **placeholder type** that can be filled with any actual type when the function or class is used.

```python
from typing import Sequence, TypeVar

T = TypeVar('T')  # A generic type, no constraints

# 1. Generic Functions
def first(items: Sequence[T]) -> T:
    return items[0]

first([1, 2, 3])         # Inferred as int
first(["a", "b", "c"])   # Inferred as str  

# 2. Type Constraints
from typing import TypeVar, Sequence

Num = TypeVar('Num', int, float)  # Only int and float allowed

def total(values: Sequence[Num]) -> Num:
    return sum(values)

print(total([1, 2, 3.1])) # 6.1

# 3. Don’t forget to apply the TypeVar to both parameter and return value for it to be useful:
def identity(x: T) -> T:  # Correct
    return x

def identity_wrong(x: T) -> Any:  # Pointless — return loses type info
    return x
```



<br>



---

🚀 Stay tuned for more insights into Python!