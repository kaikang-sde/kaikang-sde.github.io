---
title: Python Strings and Common Operations
date: 2025-03-19
categories: [Backend, Python, String]
tags: [Python, Basics, Formatting, Data Types, Import]
permalink: /python-strings-and-common-operations/
author: kai
---

# 🚀 Python Strings and Common Operations
In Python, **strings are immutable**, meaning once created, their contents cannot be changed. Any modification results in the creation of **a new string object**.

### 📝 Different Ways to Define Strings

```python
s1 = 'Single-quoted string'
s2 = "Double-quoted string"
s3 = '''Triple-quoted strings 
        support multi-line text'''
s4 = r"Raw string\nNo escaping"  # Output: r - Raw string, No escaping
```

#### Explanation
1. **Single and Double Quotes (`' / "`)**:
- Both can be used interchangeably for defining strings.
- Double quotes are preferred when the string contains an apostrophe (') to avoid escaping.
2. **Triple Quotes (`''' / """`)**:
- Used for multi-line strings.
- Often utilized for docstrings in functions and classes.
3. **Raw Strings (`r"..."`)**:
- Prefixing **a string with r** makes it a raw string, where escape sequences (\n, \t, etc.) are not processed.
- Useful for handling file paths and regular expressions.

```python
path = r"C:\Users\Kai\Documents\file.txt"
print(path)  # Output: C:\Users\Kai\Documents\file.txt
```

## 🍿 Python Escape Characters
Escape characters in Python allow you to include **special characters** in strings that would otherwise be difficult to type or cause syntax errors.
An **escape character** starts with a backslash (`\`), followed by a specific character.

### Common Escape Characters in Python

| Escape Sequence | Description  | Example |
|----------------|-------------|---------|
| `\\`  | Backslash | Printing a File Path (Using `\\` to Escape `\`)<br>`print("C:\\Users\\Kai\\file.txt")` → **Output:** `# Output: C:\Users\Kai\file.txt` |
| `\n`  | Newline   | `print("Hello\nWorld")` → **Output:** <br> `Hello` <br> `World` |
| `\t`  | Tab (Indentation) | `print("Name:\tAlice")` → **Output:** `Name:  Alice` |
| `\'` or `\"` | Quotes inside strings | `print('It\'s a beautiful day!')` → **Output:** `It's a beautiful day!`<br>`print("He said, \"Hello!\"")` → **Output:** `He said, "Hello!"` |

If you want to disable escape sequences, use **a raw string by prefixing it with r**.


## 🧲 String Indexing
In Python, **strings are sequences of characters**, and each character has a unique index.<br>

### Indexing Types
- **Positive Indexing** → Starts from `0`, counting **left to right**.  
- **Negative Indexing** → Starts from `-1`, counting **right to left**. 

```python
s = "Python"

print(s[0])   # Output: P (first character, index 0)
print(s[-1])  # Output: n (last character, index -1)
```

### String Slicing
Slicing allows extracting a substring from a string using the syntax: `str[start:end:step]`.
- **start:** Start index (inclusive, default 0)
- **end:** End index (exclusive, default is end of string)
- **step:** Step size (default 1, positive for forward slicing, negative for reverse slicing)

#### Examples
```python
s = "Python"

# 1️⃣ Extracting a Substring (s[start:end])
print(s[2:5])  # Output: tho (end exclusive, 2:5 means -> characters from index 2 to 4)

# 2️⃣ Omitting Start or End Index
print(s[:4])   # Output: Pyth (from start, 0 to index 3)
print(s[2:])   # Output: thon (from index 2 to end)

# 3️⃣ Using Step (s[start:end:step])
print(s[::2])  # Output: Pto (every second character)
print(s[::-1]) # Output: nohtyP (reverse string)
```

## 🎯 Summary Table of String Methods

### Case Conversion
Common built-in methods to **convert string cases** efficiently.

| Method        | Description | Example<br> (`s = "Hello, Python"`) | Output |
|--------------|-------------|---------------------------------|--------|
| `s.upper()`  | Converts **all characters** to uppercase | `s.upper()` | `"HELLO, PYTHON"` |
| `s.lower()`  | Converts **all characters** to lowercase | `s.lower()` | `"hello, python"` |
| `s.title()`  | Capitalizes the **first letter of each word** | `s.title()` | `"Hello, Python"` |
| `s.swapcase()` | **Swaps** uppercase to lowercase and vice versa | `s.swapcase()` | `"hELLO, pYTHON"` |


### Searching and Replacing

| Method        | Description | Example<br> (`s = "Hello World"`) | Output |
|--------------|-------------|-------------------------------|--------|
| `s.find(sub)`  | Returns the **first occurrence index** of `sub`,<br> or `-1` if not found | `s.find("World")` | `6` |
| `s.index(sub)` | Similar to `find()`, <br>but **raises an error** if `sub` is not found | `s.index("lo")` | `3` |
| `s.count(sub)` | Returns **the number of occurrences** of sub in the string | `s.count("l")` | `3` |
| `s.replace(old, new)` | **Replaces all occurrences** of old with new | `s.replace("World", "Python")` | `"Hello Python"` |
| `s.replace(old, new, count)` | **limit replacements** using the **optional count parameter** | `s = "apple apple apple"`<br>`s.replace("apple", "banana", 2)` | `"banana banana apple"` |
| `startswith(substring)` | Returns `True` if the string **starts with** `substring` | `"file.txt".startswith("file")` | `True` |
| `startswith(substring, start, end)` | Checks within a specified **range** | `"hello world".startswith("world", 6)` | `True` |
| `endswith(substring)` | Returns `True` if the string **ends with** `substring` | `"image.jpg".endswith(".jpg")` | `True` |
| `endswith(tuple_of_suffixes)` | Checks **multiple possible endings** | `"report.pdf".endswith((".jpg", ".pdf"))` | `True` |


### Splitting and Joining

| Method        | Description | Example | Output |
|--------------|-------------|------------------------|----------------------|
| `s.split(sep)` | Splits string into **a list** based on the separator | `"apple,banana".split(",")` | `['apple', 'banana']` |
| `s.splitlines()` | Splits **a multi-line string** into a list of lines | `"Line1\nLine2".splitlines()` | `['Line1', 'Line2']` |
| `s.join(lst)` | Joins a list of strings into **one string** | `"-".join(["2023", "10", "01"])` | `"2023-10-01"` |


### Whitespace Removal and Padding
The **original string (S) will not change**, still `" Python "`.  
Reason: strings are **immutable**, meaning that any modification to a string creates a new string instead of modifying the original one.

| Method        | Description | Example (`s = " Python "`) | Output |
|--------------|-------------|-------------------------------|--------|
| `s.strip()`  | Removes whitespace from **both sides** | `s.strip()` | `"Python"` |
| `s.lstrip()` | Removes whitespace from the **left** | `s.lstrip()` | `"Python   "` |
| `s.rstrip()` | Removes whitespace from the **right** | `s.rstrip()` | `"   Python"` |
| `s.center(width, char)` | **Centers text** within a **specified width** using **char** for padding.<br>Will not remove whitespaces.<br>width is the length of the result | `s.center(20, "*")` | `"****** Python ******"` |
| `s.zfill(width)` | **Left-pads** the string with zeros until it reaches width.<br>Will not remove whitespaces.<br>width is the length of the result| `s.zfill(10)` | `"00 Python "` |

### string formatting

[String Formatting in Python]({{ site.baseurl }}/python-basics-quick-reference-guide/#%EF%B8%8F-python-output)

<br>

---

🚀 Stay tuned for more insights into Python!