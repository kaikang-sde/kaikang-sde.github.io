---
title: Python Data Structures and Common Operations
date: 2025-03-19
categories: [Backend, Python]
tags: [Python, Data Structures, List, Tuple, Set, Dictionary]
author: kai
permalink: /posts/backend/python/python-data-structures-and-common-operations/
---

# 🚀  Python Data Structures and Common Operations
A **data structure** is a way to store, organize, and operate on data efficiently. Choosing the right data structure is essential for performance and clarity.

Every data structure involves:

- **Data** — what is stored  
- **Structure** — how it is organized  
- **Operations (CRUD)** — Create, Read, Update, Delete  

## 📌 List
A `list` is an **ordered, dynamic, and highly flexible** collection in Python.

Key characteristics:

- Elements **maintain order**
- Can store **any type** (mixed types allowed)
- Supports **mutation** (items can be changed)
- Automatically resizes as needed

### Common Operations

| CRUD         | Examples                  | Description |
|--------------|---------------------------------------------------------------------------|
| **Create**   | `nums = [1, 2, 3]` <br> `empty = []` <br> `chars = list("abc")` ->  `['a', 'b', 'c']` <br> `list_1 = [1, 12.7, True, 'hi', ['a', 'b']]` <br> `list_2 = list_1[:]` -> Generate a new list with same elements   | Several ways to create lists <br> Slice operation will generate a new list(different object) | 
|              | `a = [1, 2]` <br> `b = a + [3, 4]` ->  `[1, 2, 3, 4]` <br> `c = a * 3` ->  `[1, 2, 1, 2, 1, 2]` | `+` concatenation <br> `*` repetition <br> Not in-place operations: does not modify the original list |
|              | `nums.append(10)` -> add to end <br> `nums.insert(1, 5)` -> insert at index <br> `nums.extend([7, 8])` -> add multiple items | Mutating creation, will modify the original list |
| **Read**     | `for x in nums` -> iteration <br> `nums[0]` -> indexing <br> `nums[1:3]` -> slicing <br> `2 in nums` -> membership test <br> `nums.index(2)` -> index of first occurence <br> `nums.count(2)` -> count occurrences| index(x): x must be in list <br> count(x): x must be in list |
| **Update**   | `nums[2] = 99` -> Replace a single item <br> `nums[1:3] = [4, 5]` -> Replace multiple items with slicing <br> `nums[1:3] = []` -> Remove items with slicing | Lists are mutable — elements can be replaced. |
| **Delete**   | `nums.remove(2)` -> remove first occurence <br> `nums.pop(1)` -> remove and return item at index <br> `nums.pop()` -> remove and return last item <br> `del nums[2]` -> delete item at index | Lists are mutable — elements can be removed. |
| **Other**    | `len(nums)` -> length <br> `nums.sort()` -> sort in place <br> `nums.sort(reverse=True)` -> sort in place <br> `nums.reverse()` -> reverse in place| |

### List Comprehension
List comprehension provides a concise way to create lists. 

Sometimes you need to **generate a list based on a rule**, rather than manually defining each element. 

A **list comprehension** is a concise, readable way to create lists from:
- A rule: `i`
- A loop: `for i in range(100)`
- An optional condition: `if i % 5 == 0`

For example: Generate all multiples of 5 under 100  

```python
result = [i for i in range(100) if i % 5 == 0] # Generate all multiples of 5 under 100  
result = [i ** 2 for i in range(100)] # Generate all squares under 100
result = [i ** 2 for i in range(100) if i % 5 == 0] # Generate all squares of multiples of 5 under 100
result = [0 for _ in range(100)] # Generate all 0s under 100

# Traditional loop
multiples = []
for x in range(100):
    if x % 5 == 0:
        multiples.append(x)
```

## 🗜️ Tuple
A **tuple** is another important linear data structure in Python.  It looks similar to a list, but with one major difference: **Tuples are immutable**.

Key characteristics:
- Ordered → element positions matter
- Can store mixed types → flexible
- Immutable → once created, they cannot be changed.


### Common Operations

| CRUD         | Examples                  | Description |
|--------------|---------------------------------------------------------------------------|
| **Create**   | `nums = 1, 2, 3` -> () is optional <br> `chars = tuple("abc")` -> `('a', 'b', 'c')` <br> `mixed_tuple = (1, 12.7, True, "hi", ["a", "b"])` <br> `tuple_4 = (4,)` -> must include comma for single-element tuple | Because of immutability, an empty tuple often has no meaning | 
|              | `a = (1, 2)` <br> `b = a + (3, 4)` ->  `(1, 2, 3, 4)` <br> `c = a * 3` ->  `(1, 2, 1, 2, 1, 2)` | `+` concatenation <br> `*` repetition <br> Not in-place operations: does not modify the original tuple |
| **Read**     | `for x in nums` -> iteration <br> `nums[0]` -> indexing <br> `nums[1:3]` -> slicing <br> `2 in nums` -> membership test <br> `nums.index(2)` -> index of first occurence <br> `nums.count(2)` -> count occurrences| index(x): x must be in tuple <br> count(x): x must be in tuple |


#### Unpacking
Tuple unpacking allows you to assign elements of a tuple to multiple variables in a single line. This is especially useful for clean and readable code when dealing with fixed-length sequences.

```python
student = ("Kai", 20, "Computer Science")

# Unpack the tuple into separate variables
name, age, major = student

print(f"{name}'s major is {major}")  
# Output: Kai's major is Computer Science.
```

#### Zip
The built-in **zip()** function pairs up corresponding elements from the **two lists into tuples**.

```python
keys = ['name', 'age', 'language']
values = ['Kai', 28, 'Python']
zipped = zip(keys, values)
print(zipped)  # <zip object at 0x100cc6200>
print(dict(zipped)) # {'name': 'Kai', 'age': 28, 'language': 'Python'}
print(list(zipped)) # [('name', 'Kai'), ('age', 28), ('language', 'Python')]
```

### Why Tuple?
1. Immutability = Data Safety
2. Tuples Are Faster Than Lists
3. Tuples Can Be Keys in dict and Elements in set (immutable → hashable)
4. Tuples Signal Intent (Using a tuple sends a clear message: "This data should not be modified." -> Better readability → better design.)


## 🧵 Strings
A **string** is:

- An **ordered collection of characters**
- **Fixed length** once created
- **Immutable** → cannot be changed in place
- Can be created using `'single quotes'` or `"double quotes"` (choose one style and stay consistent)

Because strings are immutable:
```python
str1 = "hello"
str1[0] = "H" # TypeError: 'str' object does not support item assignment
```

To “modify” a string, you must create a new one:
```python
s = "H" + s[1:] # "Hello"
```


### Characters
Unlike languages such as C, C++, or Java, Python does not have a char type.

A single character is simply a string of length 1:
```python
c = 'a'        # still a string
print(type(c)) # <class 'str'>
```

#### Normal Characters vs Escape Characters
Some characters cannot be typed directly, so Python uses escape sequences, each treated as one character.

Common escape characters:
- newline: `\n`
- tab: `\t`
- backslash: `\\`
- single quote: `\'`
- double quote: `\"`

#### Unicode Code Point
Internally, each character is stored as an integer, representing its Unicode value.

Python provides two useful functions:
- `ord(c)` returns the Unicode code(an integer) point of a character. That integer is displayed in decimal(base-10) by default, which often causes confusion.
- `chr(n)` returns the character corresponding to a Unicode code point

```python
ord('a') # 97
ord('A') # 65
ord('0') # 48
ord(' ') # 32

chr(97) # 'a'
chr(65) # 'A'
chr(48) # '0'
chr(32) # ' '

(ord(c) - ord('a')) # Converting characters to indices (e.g., 'a' -> 0, 'b' -> 1, etc.)
chr(ord(lower_char) - ord('a') + ord('A')) # Converting lowercase letters to uppercase
```


### Common Operations

| CRUD | Examples | Description |
|------|----------|-------------|
| **Create** | `str = "hello"` <br> `str + "world"` -> helloworld <br> `str * 3` -> hellohellohello | Create a string |
| **Read** | `for x in str` -> iteration <br> `str[0]` -> indexing <br> `str[1:3]` -> slicing <br> `"2" in str` -> membership test <br> `str.find("s")` -> find first occurence <br>  | Access a character |
| **Other** | `len(str)` -> length <br> `str.lower()` -> convert to lowercase <br> `str.upper()` -> convert to uppercase <br> `str.strip()` -> remove leading and trailing whitespace <br> `str.split(" ")` -> split into words <br> `str.join(["a", "b", "c"])` -> join with space  <br> `str[::-1]` -> reverse <br> `str(100)` -> convert to string| |
|  | `str.replace('h', 'w')` -> replace all h with w <br> `str.replace('h', '')` -> remove all h| Create a new string. Does not modify the original string. |
| | `'I am {}, and my score is {}'.format(Kai, 100)` -> I am Kai, and my score is 100. <br> `f"I am {name}, and my score is {score}"` -> I am Kai, and my score is 100. <br> `I am %s, and my score is %d % ('Kai', 100)` -> I am Kai, and my score is 100. | |
| | `strs = ["a", "b", "c"]` <br> `" ".join(strs)` -> "a b c" <br> `"".join(strs)` -> "abc" | |

- Combine strings: `+` -> `str = 'ab' + 'cd' -> 'abcd'`
- Get the length of a string: `len()` -> `len(str) -> 4`
- Get the character at index: `str[index]` -> `str[2] -> 'c'`
- Get the substring: `str[start:end]` -> `str[1:3] -> 'bc'`
- Find a substring: `str.find("substring")` -> `str.find("bc") -> 1`
- Replace a substring: `str.replace("substring", "new_substring")` -> `str.replace("bc", "xy") -> 'axyd'`
- Number to string: `str(number)` -> `str(3.14) -> '3.14'`
- String to number: `int(str)` or `float(str)` -> `int("100") -> 100` or `float("3.14") -> 3.14`








<br>
---

🚀 Stay tuned for more insights into Python!