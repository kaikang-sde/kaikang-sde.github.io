---
title: Summation, Dot Product, and Cosine Similarity
date: 2025-04-13
categories: [AI, LangChain]
tags: [LangChain, LLM, Vector, Summation, Dot Product, Cosine Similarity]
author: kai
---

# 🚀 Summation, Dot Product, and Cosine Similarity
Understanding how vectors work is essential in machine learning, recommendation systems, and LLM-based applications. Three fundamental concepts form the foundation of vector operations: **Summation**, **Dot Product**, and **Cosine Similarity**.


## 🔢 Summation and Σ (Sigma)
The symbol `Σ` is the **capital Greek letter Sigma**, and it stands for **summation**, or **the total sum of a sequence**.

**General Syntax:** `Σⁿᵢ(expression)` <br>
**More formally:**  `Σⁿᵢ₌₁ aᵢ = a₁ + a₂ + a₃ + … + aₙ`

- `i = 1`: Start index
- `n`: End index
- `aᵢ`: Term to be summed (can be any expression involving `i`)

### Examples
1. Example 1: Sum of first 5 natural numbers
    - `Σ⁵ᵢ₌₁ i = 1 + 2 + 3 + 4 + 5 = 15`

2. Example 2: Sum of squares
    - `Σ³ᵢ₌₁ i² = 1² + 2² + 3² = 1 + 4 + 9 = 14`


## 🧮 Dot Product
The **dot product** (also called **inner product**) is **an operation between two vectors** that **returns a single scalar(number)**.

**Given two vectors:**
- A = [a₁, a₂, …, aₙ]
- B = [b₁, b₂, …, bₙ]

The **dot product** is defined as:
- `A · B`
- `= a₁·b₁ + a₂·b₂ + … + aₙ·bₙ`
- `= Σ (aᵢ * bᵢ) for i=1 to n`
- `= Σⁿᵢ₌₁(aᵢ * bᵢ)`

Where:
- `aᵢ` and `bᵢ` are the components of vectors A and B
- `∑` means summation from `i = 1` to `n`

### 2D Vector Dot Product Example
**Let:**
- **A** = [2, 3]
- **B** = [4, -1]

**Then:**
- `A · B = (2 * 4) + (3 * -1) = 8 - 3 = 5`


### 3D Vector Dot Product Example
**Let:**
- **A** = [1, 0, -2]
- **B** = [3, 4, 1]

**Then:**
- `A · B = (1 * 3) + (0 * 4) + (-2 * 1) = 3 + 0 - 2 = 1`


### Geometric Interpretation
The dot product also equals: `A · B = ||A|| × ||B|| × cos(θ)`

Where:
- `||A||` and `||B||` are the **magnitudes** (lengths) of vectors A and B
- `θ` is the **angle** between them


## 🤷🏻 Cosine Similarity
Cosine similarity measures how **aligned** two vectors are in terms of direction — the smaller the angle between them, the more similar they are.

- **Sine (sinθ):** The ratio of the length of the opposite side to the hypotenuse in a right triangle.
- **Cosine (cosθ):** The ratio of the length of the **adjacent** side to the **hypotenuse** (i.e., the two sides that form the angle).

![Formula](/assets/img/posts/AI/LangChain/CosineSimilarity.png)


### cosine similarity formula
- `similarity(A, B) = cos(θ) = (A · B) / (||A|| ||B||)`


### Angle-Based Meaning

The value of cosine similarity directly relates to **the angle (θ) between two vectors**:

| Cosine Value | Angle | Interpretation                    |
|--------------|-------|------------------------------------|
| `1`          | 0°    | Vectors point in exactly the same direction — **maximum similarity** |
| `0`          | 90°   | Vectors are orthogonal — **no similarity** |
| `-1`         | 180°  | Vectors point in opposite directions — **completely dissimilar** |

Imagine two arrows originating from the same point:
- If they overlap perfectly, the angle is 0°, and cosine similarity is 1.
- If they form a right angle (90°), they have nothing in common.
- If they point in opposite directions (180°), cosine similarity is -1.

This angle becomes a **proxy for semantic similarity** in applications like text embeddings.


### Example: 
Suppose we have:
- A = [3, 4]
- B = [1, 2]

Then:
- `A · B = 3×1 + 4×2 = 3 + 8 = 11`
- `||A|| = sqrt(3² + 4²) = 5`
- `||B|| = sqrt(1² + 2²) = 2.236`
- `cos(θ) = (A · B) / (||A|| ||B||) = 11 / (5×2.236) = 11 / 11.18 = 0.984`
- So the two vectors form an **acute angle**.


### Why Cosine Similarity?
- 🛒 **Online Shopping**: Find the pair of pants that best matches the shirt you just bought.
- 🎧 **Music Recommendation**: Discover songs with a similar vibe to the one currently playing.
- 🍱 **Food Delivery**: Suggest new restaurants whose dishes align with your usual tastes.
- 🤖 **LLM + RAG Systems**: Given a natural language query, retrieve the most relevant documents to answer the user's intent.

What do these use cases have in common?

They all involve **measuring similarity** between two entities. In the context of vector space, this similarity is often captured by **comparing the direction** of two vectors — not their exact values or magnitudes.

Cosine similarity gives us a **mathematical way to measure this directional closeness**, making it especially useful in recommendation systems, natural language processing, and retrieval-augmented generation (RAG) workflows.

> In short: If you're building anything with LLMs, embeddings, or recommendations — cosine similarity is a must-have in your toolbox.


### Use Cases of Cosine Similarity in LLM
Cosine similarity is a key technique behind many intelligent applications powered by large models. Below are two of its most common and impactful use cases:

#### Example 1:  Semantic Search
**Problem**: A user types a natural language query — how do we retrieve the most relevant documents?

**Solution**: Encode both the query and all candidate documents into vector representations, then compute cosine similarity to rank them.

**Workflow**:
1. Convert the query into an embedding vector.
2. Convert all documents into embedding vectors (offline or on-the-fly).
3. Compute cosine similarity between the query vector and each document vector.
4. Return the Top-K documents with the highest similarity scores.

**Example**:

```text
Query: "How do I get a refund?"
Document A: "Return policy for damaged goods..."
Document B: "Company refund procedures and processing time..."
→ Document B ranks higher by semantic similarity.
```


#### Example 2:  Recommendation Systems (User-Item Matching)

**Problem**: How do we recommend the most suitable item to a specific user?

**Solution**: Represent both users and items as vectors, and compute the cosine similarity as a recommendation score.

**Example**:

```text
User Vector: [0.3, 0.5, -0.2] → Interests in Technology, Sports, Art
Item Vector: [0.4, 0.1, 0.0] → A tech-related article
Similarity: 0.3×0.4 + 0.5×0.1 = 0.17 -> Recommend
-> The article is moderately relevant — it can be recommended to the user.
```

## 🤖 Summary
```text
User Input -> Embedding Model -> Text Vector -> Vector Database -> Cosine Similarity Calculation -> Top-K Results -> LLM Generation -> Final Answer
```




<br>




---

🚀 Stay tuned for more insights into LangChain and RAG!



