---
title: Recursion Basic
date: 2025-05-20
categories: [Algorithm, Recursion]
tags: [Recursion, Function Design, Data Structures]
author: kai
---

## 🔄 What Is Recursion?
Recursion is not an algorithm in itself—it's a **programming technique** where a function calls itself to solve a subproblem. Whether it's used in backtracking, divide-and-conquer, or tree traversal, understanding recursion is key to mastering algorithm design.

> **Recursion** = Going deeper (breaking a problem down), then returning (combining results)

It’s used to solve problems that can be broken into **smaller, similar subproblems**.

### Recursive Data Structures

Recursion is not just in functions—**data structures** themselves can be recursive:

- **Trees**: Each node points to subtrees.
- **Linked Lists**: Each node points to another list (next node).

These structures **contain smaller versions of themselves**, which is the essence of recursion.


### Recursion ≠ Algorithm

Recursion is a **style of implementation**, not a specific algorithm.

- Brute force can be written using recursion.
- Divide-and-conquer often uses recursion.
- Backtracking and DFS are typically recursive.


### The 3 Pillars of Recursion

Every recursive function must clearly define the following:

#### 1. Definition

- What parameters does the function take?
- What does it return?
- What does it mean logically?

**Example**:  
The function receives an integer n and returns the product of all numbers from 1 to n.

```python
def factorial(n: int) -> int:
    ...
```

#### 2. Decomposition (Recursive Case)
How do you reduce the problem into smaller pieces?

**Example:** `factorial(n) = n * factorial(n - 1)`

#### 3. Exit (Base Case)
When does the function stop calling itself?
- `if n == 1: return 1`

- Without a base case, the recursion will never terminate and likely lead to a stack overflow.

### Visual Example: Factorial
- Each function call is **pushed onto the call stack**, and once the base case is hit, the results are **returned and unwound** step by step.

```python
def factorial(n):
    if n == 1:
        return 1
    return n * factorial(n - 1)

factorial(3)
= 3 * factorial(2)
= 3 * 2 * factorial(1)
= 3 * 2 * 1 = 6
```

### Space Overhead of Recursion
Recursion consumes stack memory due to repeated function calls:
- For example, n in the first call is different from n in the deeper recursive levels.

| **Component**     | **Description**                                                  |
|-------------------|------------------------------------------------------------------|
| **Parameters**     | Each recursive function call keeps its own copy of parameters.   |
| **Return Value**   | Space is reserved on the call stack to hold the return value.    |
| **Local Variables**| Each level of recursion has its own local variable scope.        |


#### Recursion and the Stack
Every recursive function call creates a new "stack frame," which stores that specific invocation's **local variables**, **parameters**, and the **return address**.

**Example:**

```python
def factorial(n: int) -> int:
    if n == 1: 
        return 1
    
    return n * factorial(n - 1)
```

- `factorial(4)` is called → `n=4`'s context is pushed onto the stack.
- Inside `factorial(4)`, it calls `factorial(3)` → `n=3`'s frame is pushed.
- This continues until `factorial(1)` is called.

```text
Top of Stack
┌─────────────┐
│ factorial(1)│
│ factorial(2)│
│ factorial(3)│
│ factorial(4)│
└─────────────┘
Bottom of Stack
```

**Unwinding the Stack:**

Once the base case is hit (`factorial(1)`), the recursion **starts returning**:

1. `factorial(1)` returns `1`
2. `factorial(2)` receives result and returns `2 × 1`
3. `factorial(3)` returns `3 × 2 × 1`
4. `factorial(4)` returns `4 × 3 × 2 × 1 = 24`

At each return step, the **top stack frame is popped**, and control goes back to the previous call.


#### What is a Stack Frame?

A **stack frame** is a memory block storing:
- Parameters passed to the function
- Local variables within the function
- The return address to jump back to after completion

Each function call creates a new stack frame.


#### Key Takeaways

- **Recursion = Stack operations** in disguise.
- The **call stack** enables recursion by isolating each function call.
- Improper recursion (e.g. missing base case) can cause a **stack overflow**.


## 🧠 What is Divide and Conquer?
Divide and Conquer is an **algorithmic design pattern** that breaks a problem into smaller subproblems, **solves them recursively**, and then **merges the results**.

1. **Divide** the problem into smaller subproblems of the same type.
2. **Conquer** each subproblem recursively (solve it).
3. **Combine** the solutions of the subproblems to form the solution to the original problem.

This approach simplifies complex problems by breaking them down into manageable chunks, solving each chunk, and then combining their results.


### Suitable Data Structures

Divide and Conquer is ideal for problems based on:

- **Binary Trees**  
  A tree where each subtree is itself a smaller binary tree. Most binary tree problems can be approached this way.  
  > _Tip: Think about how the result of the entire tree relates to its left and right children._

- **Arrays**  
  Large arrays can be divided into smaller, non-overlapping subarrays. Examples include:
  - Merge Sort
  - Quick Sort


### Divide and Conquer vs Traversal

| Approach     | Analogy                                     | Code Pattern                          |
|--------------|---------------------------------------------|---------------------------------------|
| Traversal    | A single person walks all nodes, taking notes | Uses shared/global variables         |
| Divide & Conquer | Delegates work to "minions", then aggregates | Uses return values from subcalls     |

Technically, divide-and-conquer on trees often resembles **post-order traversal** — solve left and right, then combine at the root.

#### Real Examples

- **Merge Sort** (Array-based)
  - Divide array into two halves
  - Sort each half
  - Merge the sorted halves

- **Binary Tree Maximum Path Sum**
  - Calculate left and right subtree contributions
  - Combine them to form the maximum path



## 🆚 Recursion vs Divide and Conquer
- You can write recursive algorithms without using Divide and Conquer.
- But Divide and Conquer almost always requires recursion to solve and combine subproblems.

| **Feature**        | **Recursion**                            | **Divide and Conquer**                      |
|--------------------|-------------------------------------------|---------------------------------------------|
| **Type**           | Programming technique                     | Algorithm design strategy                   |
| **Structure**      | Function calling itself                   | Divide → Conquer → Combine                  |
| **Use Case**       | Any problem reducible to subproblems      | Problems solvable via independent parts     |
| **Combination Step** | Not required                            | Always required                             |
| **Example**        | Factorial, Fibonacci                      | Merge Sort, Quick Sort, Binary Search       |







---

Stay tuned for more insights into algorithm fundamentals!