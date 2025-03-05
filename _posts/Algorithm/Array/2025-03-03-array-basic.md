---
title: Arrays in Java
date: 2025-03-03
categories: [Algorithm, Array]
tags: [Array, Java]
author: kai
---

# Arrays in Jave

## What 
An **array** is a data structure that stores multiple values of the same type in a contiguous block of memory. It provides a way to group and manage a collection of elements using a single variable name.

- Arrays in Java are **fixed in size**, meaning their length is determined when they are created and cannot be changed.
- Elements in an array are accessed using an **index**, starting from `0`.
- Arrays can store **primitive data types** (`int`, `double`, `char`, etc.) or **objects** (`String`, `Integer`, `CustomClass`, etc.).


## When
Use arrays when:
- You need to store multiple elements of the same type.
- The number of elements is **fixed** or known in advance.
- You want **fast access** to elements using an index (`O(1)` time complexity).
- You need to perform operations like sorting, searching, and iteration over a collection of values.

#### Example Use Cases
- **Managing student grades:** `int[] grades = {85, 90, 78, 92};`
- **Storing a list of temperatures:** `double[] temperatures = new double[7];`
- **Representing a game board (e.g., chess, tic-tac-toe).**
- **Processing large datasets where all elements are of the same type.**


## Why
Arrays are useful because they:
- Provide **fast access** to elements (`O(1)` time complexity).
- Store multiple values under a **single variable name**, making code cleaner.
- Allow **efficient memory management** as they store elements in contiguous memory locations.
- Are **more performant** than `ArrayList` for fixed-size collections, as they avoid dynamic resizing overhead.

However, arrays also have **limitations**:
- **Fixed size:** Once an array is created, its size cannot be changed.
- **Inefficient insertions and deletions:** Adding or removing elements requires shifting, making it `O(n)` complexity.

---

## How - Code Example

#### Different Ways to Declare and Initialize Arrays
```java
// 1. Declare and allocate memory separately
int[] arr1;
arr1 = new int[5]; // Allocates memory for 5 elements

// 2. Declare and initialize in one step
int[] arr2 = new int[] {1, 2, 3, 4, 5};

// 3. Shorthand initialization
int[] arr3 = {10, 20, 30, 40, 50};
```

#### Iterating Through an Array
```java
// Using a for loop
for (int i = 0; i < numbers.length; i++) {
    System.out.println(numbers[i]);
}

// Using an enhanced for loop (for-each loop)
for (int num : numbers) {
    System.out.println(num);
}
```

#### Multidimensional Arrays
```java
// 2D Array Declaration
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

// Accessing elements
System.out.println(matrix[1][2]); // Output: 6
```

#### Sorting an Array
```java
import java.util.Arrays;

int[] values = {5, 1, 4, 2, 8};
Arrays.sort(values); // Sorts in ascending order

System.out.println(Arrays.toString(values)); // Output: [1, 2, 4, 5, 8]
```

#### Copying an Array
```java
// Method 1: Using Arrays.copyOf()
int[] copy = Arrays.copyOf(numbers, numbers.length);

// Method 2: Using System.arraycopy() - System.arraycopy(sourceArray, sourcePosition, destinationArray, destinationPosition, length);
int[] newArr = new int[5];
System.arraycopy(numbers, 0, newArr, 0, numbers.length);
```

#### Finding Maximum and Minimum in an Array
```java
int max = Integer.MIN_VALUE;
int min = Integer.MAX_VALUE;

for (int num : numbers) {
    if (num > max) max = num;
    if (num < min) min = num;
}

System.out.println("Max: " + max);
System.out.println("Min: " + min);
```

#### Using Arrays with Methods
```java
// Passing an array to a method
public static void printArray(int[] arr) {
    for (int num : arr) {
        System.out.print(num + " ");
    }
}

// Calling the method
printArray(numbers); // Output: 10 25 30 40 50
```

---

## Summary

| Feature              | Details                                      |
|----------------------|----------------------------------------------|
| **Type**            | Fixed-size, homogeneous collection           |
| **Access Time**     | `O(1)` for reading/writing                   |
| **Insertion/Deletion** | `O(n)` (requires shifting elements)        |
| **Memory Efficiency** | Stored in contiguous memory locations      |
| **Use Cases**       | Storing fixed-size collections, fast access needs |

🔹 **Use Arrays when** you need a simple, fixed-size data structure.  
🔹 **Consider alternatives like `ArrayList`** if you need dynamic resizing.


---

Stay tuned for more insights into algorithm fundamentals!