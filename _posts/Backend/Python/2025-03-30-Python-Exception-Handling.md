---
title: Python Exception Handling
date: 2025-03-25
order: 5
categories: [Backend, Python]
tags: [Python, exception, try-except]
author: kai
---

# 🚀 Python Exception Handling
In real-world Python applications, things can — and will — go wrong: a file might not exist, an API call may time out, or user input might be invalid.  
That’s where **exception handling** becomes essential.

## ❓ Why Use Exception Handling

| Purpose              | Benefit                                                             |
|----------------------|---------------------------------------------------------------------|
| Prevent crashes       | Catch runtime errors and prevent program from stopping             |
| Friendly messages     | Replace cryptic errors with helpful prompts                        |
| Safe resource cleanup | Ensure files, network connections, etc., are properly released     |

## 🔧 Basic `try-except` Syntax
- **try:** Code that might raise exceptions
- **except:** Catch and handle specific exceptions
- **else:** Code that runs **only if no exception occurs**
- **finally:** Cleanup code (always runs, even after return/break)

```python
try:
    risky_operation()  # Code that might raise an exception
except SomeExceptionType1:
    handle_type1() # Handler for Exception Type 1
except SomeExceptionType2 as e:
    print(f"Error occurred: {e}") # Handler for Exception Type 2 with access to the exception object
else:
    # Executes ONLY if no exception was raised
    print("Everything went well!")
finally:
    # Always runs (whether exception occurred or not)
    print("Cleanup completed.")
```

## ✅ Best Practices
- **Be specific:** Catch specific exceptions (ValueError, etc.)
- **Avoid bare except:** Use except Exception instead, if needed
- **Use finally for cleanup:** Always close files, DBs, or sockets
- **Log or display useful messages:** Don't silently ignore errors

```python
# ❌ Bare except: — Avoid this unless absolutely necessary
try:
    risky_operation()
except:
    print("Something went wrong.")  # Catches *everything*, even system-level exits

# ✅ Use except Exception: — Safer and clearer
try:
    risky_operation()
except Exception as e:
    print("Handled runtime error:", e)
```


## 🛠️ Common Exception Types in Python
Here’s a quick-reference table with examples to help you recognize frequent exception types.

| Exception Type      | Trigger Scenario                   | Example Code                              |
|---------------------|-------------------------------------|--------------------------------------------|
| `ZeroDivisionError` | Division by zero                   | `10 / 0`                                   |
| `FileNotFoundError` | File does not exist                | `open("missing.txt")`                      |
| `ValueError`        | Invalid type conversion            | `int("abc")`                               |
| `IndexError`        | List index out of range            | `lst = [1]; lst[2]`                        |
| `KeyError`          | Dict key not found                 | `d = {"a":1}; d["b"]`                      |
| `TypeError`         | Operation on wrong data type       | `"a" + 1`                                  |


## 🧱 Custom Exceptions
Built-in exceptions like `ValueError` or `FileNotFoundError` cover many common issues. But what if your application needs to **raise errors specific** to your **business logic**?

- Inherits from Exception (or a subclass)
- Initializes with a message using `super().__init__()`
- You can add any extra attributes or methods if needed

**Steps:**
1. Define custom class: `class MyError(Exception): ...`
2. Raise your exception: `raise MyError("Something's wrong")`
3. Catch the exception: `except MyError as e:`


| Reason                       | Benefit                                         |
|------------------------------|-------------------------------------------------|
| Domain-specific logic        | Handle app-specific rules clearly              |
| Improved readability         | Communicates intent more precisely             |
| Better control in `except`   | Catch only relevant, meaningful errors         |

### raise and except

| Keyword | Purpose | Direction | Use Case |
|---------|---------|-----------|----------|
| raise | Trigger an exception | Throws error | When your code detects a problem |
| except | Handle an exception | Catches error| When your code responds to errors |


## 🤖 Examples
**1. Basic Exception Handling with User Input**

```python
try:
    num = int(input("Please enter a number："))
    result = 100 / num
    print(result)
except ValueError:
    print("Please enter a NUMBER!")
except ZeroDivisionError:
    print("Error: Division by zero.")
except Exception as e:
    print("Unknow Error", e)
else:
    print("No Exception！")
finally:
    print("Will execute regardless of whether an exception occurs!")
```

**2. Catching Multiple File-Related Exceptions**
- Combines multiple exceptions using a tuple
- Provides a single fallback message for similar error types

```python
try:
    with open("data.txt", "r") as f:
        content = f.read()
except (FileNotFoundError, PermissionError) as e: # Combines two common file exceptions using a tuple
    print(f"File operation failed：{e}")
```

**3. Custom Exception Definition**

```python
class InvalidAgeError(Exception):
    """Invalid Age Exception"""
    def __init__(self, age):
        self.age = age
        super().__init__(f"Invalid Age {age}，must greater than 0！")
    

def set_age(age):
    if age <= 0:
        raise InvalidAgeError(age) # Raise the Exception in Business Logic
    print("Successfully set the age：", age)


# Testing
try:
    set_age(-5)
except InvalidAgeError as e:
    print(e)  # Output: Invalid Age -5，must greater than 0!
```


<br>



---

🚀 Stay tuned for more insights into Python!