---
title:  Similarity Search vs. MMR (Maximal Marginal Relevance)
date: 2025-04-24
categories: [AI, LangChain]
tags: [LangChain, Vector Database, Milvus, CRUD]
author: kai
permalink: /posts/ai/langchain/similarity-search-vs-mmr/
---

# 🚀 Similarity Search vs. MMR (Maximal Marginal Relevance)
In vector-based search and recommendation systems, understanding the difference between **similarity search** and **MMR (Maximal Marginal Relevance)** is key to building effective retrieval and ranking strategies.

## 🔍 Similarity Search 
**Goal:** Find results *most similar* to the query.

### Example

**E-commerce Recommendation:** `"User just viewed a gaming laptop → Recommend similar gaming laptops."`

**Top-3 similar items returned:**

1. `"Alienware M15"` (cos_sim = 0.92)  
2. `"ASUS ROG Strix"` (cos_sim = 0.89)  
3. `"Lenovo Legion 5"` (cos_sim = 0.85)


### Principle
Similarity Search works by computing **vector distances** (e.g., cosine similarity or Euclidean distance) to return the **top-k most similar results**.

**Flow:**

`Query vector →  Compute distance with all vectors →  Sort by similarity →  Return Top-K results`

### Core Characteristics

- **Vector-Driven Only:**  
  Purely based on geometric distance in the vector space (e.g., Cosine, L2)

- **Homogeneous Results:**  
  Tends to return clustered, highly similar results

- **High Performance:**  
  Time complexity: `O(n + k log k)` (fast even at scale)


### Example: LangChain Retriever Configuration

```python
retriever = vector_store.as_retriever(
    search_type="similarity",
    search_kwargs={
        "k": 5,  # Return Top-5 similar documents
        "score_threshold": 0.65,  # Filter by minimum similarity score
        "filter": "category == 'AI'",  # Metadata-based filtering
        "param": {
            "nprobe": 32,  # Milvus-specific: number of clusters to probe, 
            "radius": 0.8   # Milvus-specific: limit search radius
        }
    }
)
```
- `nprobe` is a parameter in Milvus that controls the number of clusters to probe during a search.  
    - A higher `nprobe` value leads to **more accurate results**, but also **increases query latency**.

- `radius` is used in range search to specify the search radius.  
    - It can be combined with `score_threshold` to **limit the search results within a specific similarity range**.


### Typical Use Cases:
- **Precise Semantic Matching:**
    - For example: Patent retrieval, academic plagiarism detection, legal clause alignment.
- **Content-Based Recommendations:**
    - Show “more like this” items such as similar products, articles, or media based on user behavior.
- **Sensitive Content Filtering:**
    - Use high similarity thresholds to detect near-duplicate or flagged content.



## 🧮 MMR (Maximal Marginal Relevance)

**Goal:** Strike a balance between **relevance** to the user query and **diversity** in the returned results.  

Traditional similarity search (e.g., cosine similarity) often returns highly redundant results.  
MMR helps avoid this by ensuring that returned items are **not only relevant** to the query,  
but also **diverse from each other** — especially important for multi-perspective coverage or diverse recommendations.


### Example

**Query:** "User just bought a gaming laptop → Instead of showing 10 more laptops, recommend related but distinct products like a gaming mouse, mechanical keyboard, or ergonomic chair."

**Candidate Pool:**  
- Item A: Gaming Laptop (cos_sim = 0.95)  
- Item B: Gaming Mouse (cos_sim = 0.85)  
- Item C: Mechanical Keyboard (cos_sim = 0.80)  
- Item D: Another Gaming Laptop (cos_sim = 0.94)

**MMR Top-3 (λ = 0.7):**
1. Gaming Laptop (A) – high relevance  
2. Gaming Mouse – moderate relevance, high diversity  
3. Mechanical Keyboard – expands diversity further


### How It Works

**Flow:**

`Query vector → Fetch candidate pool → Rank by similarity → Apply MMR for diversity → Return Top-K`

### Conceptual Illustration

```text
Initial Candidate Pool (fetch_k = 20)
    ↓
Similarity Ranking
    → [1, 2, 3, ..., 20]
    ↓
Diversity Selection (λ = 0.5)
    ↓
Final MMR Result (k = 5)
    → [1, 5, 12, 3, 18]  # Balanced relevance and diversity
```

### LangChain Code Example: MMR Search

```python
mmr_retriever = vector_store.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 3,                # Final number of results
        "fetch_k": 20,         # Candidate pool size
        "lambda_mult": 0.6,    # Controls the trade-off between relevance and diversity in MMR, [0, 1]. 0 → more diverse, 1 → more similar
        "param": {
            "nprobe": 64,      # For IVF-based Milvus: search clusters
            "ef": 128          # For HNSW-based Milvus: search depth
        }
    }
)
```

- **`nprobe`** — Controls the number of clusters (inverted lists) to search in IVF-based indexes in Milvus.
    - **Higher `nprobe` value** → Search more clusters → **Higher recall**, but **slower performance**
    - **Lower `nprobe` value** → Search fewer clusters → **Faster**, but may miss relevant results

- **`ef`** — The size of the dynamic candidate list during HNSW (Hierarchical Navigable Small World) search in Milvus.
    - **Higher `ef`** → Search considers more nodes → **Higher recall**, but **slower**
    - **Lower `ef`** → Faster search, but may **miss relevant results**


### Typical Use Cases:
- **Diverse Recommendations:**
    - Recommend items from different categories, e.g., after viewing a gaming laptop, suggest accessories like chairs, monitors, etc. 
- **Knowledge Discovery:**
    - Explore a wide range of related but distinct research papers for academic or technical innovation.
- **Content Generation:**
    - Generate multiple creative outputs or article titles that differ in tone, topic, or structure.

> **MMR helps deliver not just the most relevant results — but the most *usefully different* ones.**


## 🔧 Key Parameter Comparison: Similarity Search vs. MMR Search

| Parameter        | Similarity Search                 | MMR Search                           | Effect on Results                                          |
|------------------|-----------------------------------|--------------------------------------|-------------------------------------------------------------|
| `k`              | Controls number of results        | Controls number of final results     | Higher `k` returns more results but may reduce relevance    |
| `lambda_mult`    | N/A                               | Float between 0 and 1                | Higher → relevance-focused; Lower → more diverse results    |
| `score_threshold`| Filters out low-quality matches   | Rarely used                          | Threshold depends on the embedding model’s similarity scale |
| `filter`         | Metadata filtering supported      | Also supported                       | Useful for applying business-level logic or field-level constraints |

> 📌 Tip:
> - For **MMR**, fine-tune `lambda_mult` (e.g., 0.6 for balance, 0.2 for diversity)
> - For **Similarity**, use `score_threshold` (e.g., ≥ 0.7) to ensure quality


## 🧾 Decision Matrix: Similarity Search vs. MMR Search

| Dimension         | Similarity Search                         | MMR (Maximal Marginal Relevance) Search               |
|-------------------|--------------------------------------------|-------------------------------------------------------|
| Result Quality | High similarity, may contain duplicates    | Better diversity, avoids redundancy                   |
| Response Time  | ~120ms (faster)                            | ~200–300ms (slightly slower due to reranking)         |
| Memory Usage   | Low (stores only top-K results)            | Higher (requires caching `fetch_k` candidates)        |
| Best Fit       | Exact matching, deduplication              | Recommendations, knowledge exploration                |
| Interpretability | Intuitive similarity-based ranking         | Composite scoring, requires extra explanation         |

> **Use Similarity Search** when precision and performance are key.  
> **Use MMR** when diversity and novelty matter more than strict relevance.


## 🏛️ Enterprise Recommendation System Architecture Example

![Enterprise Recommendation System Architecture](/assets/img/posts/AI/LangChain/EnterpriseRecommendationSystemArch.png)



<br>




---

🚀 Stay tuned for more insights into LangChain and Vector Database!



