---
title: Python File I/O Operations
date: 2025-03-31
order: 2
categories: [Backend, Python]
tags: [Python, file-io, read-write, open, best-practices, with-statement]
author: kai
---

# 🚀 Python File I/O Operations
**File I/O (Input/Output)** operations are essential in Python for **interacting with local data**.  
Whether you're reading logs, saving configurations, or writing output reports — mastering file handling is a must.


## 📚 File Operation Steps

Working with files in Python generally involves three steps:

1. **Open the file**: Connect your program to a file using `open()`
2. **Read or write**: Perform your desired operation
3. **Close the file**: Free up system resources using `.close()` or `with` block


## 🧾 File Opening Modes

| Mode  | Description                        | File Exists | File Doesn't Exist |
|-------|------------------------------------|-------------|---------------------|
| `r`   | Read-only (text, default)          | ✅ Open     | ❌ Error           |
| `w`   | Write-only, overwrite if exists    | ✅ Clear    | ✅ Create          |
| `a`   | Append to end                      | ✅ Append   | ✅ Create          |
| `r+`  | Read and write (start at beginning)| ✅ Open     | ❌ Error           |
| `b`   | Binary mode (used with other modes)| N/A         | N/A                |

You can combine modes like `rb`, `wb`, `a+b` for binary files (e.g., images, PDFs).


## 🧪 Example 1: Reading a File

```python
# 1. try-finally
file = open("my_file.txt", "w")
try: 
    file.write("Hello, World!")
finally:
    file.close()


# 2. Using with ensures the file is automatically closed.
with open("example.txt", "r", encoding="utf-8") as file:
    content = file.read()
```


## 🧱 Syntax Structure
1. **file_path:** Required. The path to the file (absolute or relative). -> `"data.txt", "/tmp/file.csv"`
2. **mode:** Not Required. File opening mode ('r', 'w', 'a', etc.) -> `'r', 'w', 'rb', 'a+'`
3. **encoding:** Not Required. For text files: preferred to set to "utf-8". -> `'utf-8', 'gbk'`
4. **file_object:** Required. The variable used to interact with the file content. -> file, f, log, etc.


```python
with open(file_path, mode='r', encoding=None, ...) as file_object:
    # Perform file operations inside this block
    # The file is automatically closed after the block
```

### Basic File Read/Write in Python with UTF-8 Encoding
Using `encoding="utf-8"` ensures proper handling of Unicode characters and avoids potential `UnicodeDecodeError` or messy output.

## 📄 Writing to a File (Overwrite Mode)
1. `.write()` **does not add a newline** by default — you must include \n manually
2. `.read()` reads the **entire content** as a single string

```python
# Overwrites the file if it exists
with open("test.txt", "w", encoding="utf-8") as f:
    f.write("Hello, World!\n")
    f.write("The secnond line")

# Reading from a File (Full Content)
with open("test.txt", "r", encoding="utf-8") as f:
    content = f.read()
    print(content)
# Output:
# Hello, World!
# The secnond line
```

## 📝 Writing Multiple Lines to a File
When handling files in Python, working **line-by-line** is not only clean — it’s also **memory-efficient**, especially with large files.

1. `.writelines()` writes **a list of strings** to the file. You must manually include \n for new lines
2. `.strip()` removes trailing \n for clean printing

```python
lines = ["Python\n", "Java\n", "C++\n"]

# Writing Multiple Lines
with open("languages.txt", "w", encoding="utf-8") as f:
    f.writelines(lines)

# Reading line by line
with open("languages.txt", "r", encoding="utf-8") as f:
    for line in f: # Best for large files, reads line by line
        print(line.strip())

# Output:
# Python
# Java
# C++
```


### ✏️ Appending with `"a"` Mode
In Python, appending to a file allows you to **add new content without overwriting** existing data. This is especially useful for use cases like logging, data accumulation, and incremental writing.
1. If the file exists, new content is added to the end
2. If the file doesn’t exist, Python will create it automatically

```python
with open("test.txt", "a", encoding="utf-8") as f:
    f.write("\nappending content")
```

### ✍🏻 Read First Line. Read First N Lines
Sometimes you don’t want to read an entire file at once — you may only need the **first line**, or want to process **the rest separately**.  
Python provides simple and flexible tools for this using `readline()` and `readlines()`.

1. `.readline()`: Reads a single line at a time. Return a string. Will not read the entire file. 
2. `.readlines()`: Reads all remaining lines as a list. Return a list[str]. Will read the entire file. 

---

```python
# Read First Line, Then the Rest
with open("test.txt", "r", encoding="utf-8") as f:
    first_line = f.readline()    # Read First Line
    next_lines = f.readlines()   # Then the Rest, return a List

print("The first line：", first_line.strip())
print("The rest：", [line.strip() for line in next_lines])

# Output:
# The first line： Hello, World!
# The rest： ['The second line', 'The third line']

# Read First N Lines
with open("test.txt", "r", encoding="utf-8") as f:
    for i in range(5):  # first 5 lines
        print(f.readline().strip())
```

### Copying an Image File
When dealing with **non-text files** like images, PDFs, or audio, you must use **binary mode** in Python's file operations.  
This ensures the data is read and written exactly as-is — without any decoding or newline conversion.

1. Always use `"rb" and "wb"` for binary files — never just "r" or "w"
2. `.read()`: Load **entire binary content** (small files) or Reads **entire file** into one string.
3. `.read(n)`: Read **specific byte** size (e.g., 4096 B)
3. **`:= `operator**: **assign a value to a variable as part of an expression**, typically used inside conditions or loops.

```python
with open("input.jpg", "rb") as src, open("output.jpg", "wb") as dst:
    dst.write(src.read())  # Copying an Image File

# For large files, consider copying in chunks 
# open source, open destination
with open("input.jpg", "rb") as src, open("output.jpg", "wb") as dst:
    while chunk := src.read(4096):  # 4KB chunks
        dst.write(chunk)
```




<br>



---

🚀 Stay tuned for more insights into Python!