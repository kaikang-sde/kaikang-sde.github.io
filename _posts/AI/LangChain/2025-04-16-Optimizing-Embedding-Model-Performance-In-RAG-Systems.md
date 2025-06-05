---
title: Optimizing Embedding Model Performance in RAG Systems
date: 2025-04-16
categories: [AI, LangChain]
tags: [LangChain, LLM, Embedding Caching, CacheBackedEmbeddings]
author: kai
permalink: /posts/ai/langchain/optimizing-embedding-model-performance-in-rag-systems/
---

# 🚀 Optimizing Embedding Model Performance in RAG Systems
When building a Retrieval-Augmented Generation (RAG) system, **embedding generation** becomes a critical and expensive component. Embeddings are used to convert text into vector representations, which are later compared for semantic similarity.

However, there are **performance and cost challenges**.

## ⚠️ Pain Points in Embedding Computation

| Issue | Description |
|-------|-------------|
| **High computation cost** | Each embedding call requires compute (especially for large models or remote APIs). |
| **Redundant computation** | Repeated embedding of the same text wastes resources. |
| **API usage limits** | Commercial APIs often have rate limits or charge per call. |
| **Latency bottleneck** | Embedding generation can slow down real-time applications. |


## ✅ The Solution: Caching

**Caching embeddings** is an effective and practical solution to address the above issues.

- **Reduces computational cost**  
  → Embeddings are computed only once per unique input.

- **Improves response speed**  
  → Cache lookups are 10–100x faster than generating embeddings.

- **Bypasses API limits**  
  → Cached embeddings are stored locally and don't consume API quotas.

- **Supports offline scenarios**  
  → Useful when operating in environments without internet access.

### Embedding Cache Architecture


```text
[Application] → Check Cache → Hit Cache → Return Cached Embedding
                     ↓
                Miss Cache -> Call Embedding Model -> Store Result in Cache -> Return New Embedding
```


## 🎁 CacheBackedEmbeddings
`CacheBackedEmbeddings` is a LangChain utility that **wraps an embedding model** and automatically checks/stores embeddings in a persistent cache. 
- The system hashes each text input and uses that as a unique key to store or retrieve the embedding vector.

[LangChain CacheBackedEmbeddings API Reference](https://python.langchain.com/api_reference/langchain/embeddings/langchain.embeddings.cache.CacheBackedEmbeddings.html#langchain.embeddings.cache.CacheBackedEmbeddings)


### Core Syntax & Parameters

```python
from langchain.storage import LocalFileStore
from langchain.embeddings import CacheBackedEmbeddings

# 1. Define your underlying embedding model (e.g. OpenAI, Ollama, HuggingFace)
embedding_model = YourEmbeddingModel()

# 2. Choose a storage backend (e.g. Local file system)
storage = LocalFileStore("./embedding_cache")

# 3. Wrap the embedding model with cache
cache = CacheBackedEmbeddings(
    underlying_embeddings=embedding_model,  # Your original embedding model
    document_embedding_store=storage,       # The caching backend
    namespace="custom_namespace"            # (Optional) Namespace to isolate projects
)
```

| Parameter                | Type          | Description                                                                 |
|--------------------------|---------------|-----------------------------------------------------------------------------|
| `underlying_embeddings` | `Embeddings`  | The original embedding model (e.g., `OpenAIEmbeddings`, `OllamaEmbeddings`) |
| `document_embedding_store` | `BaseStore` | The storage backend for caching (e.g., `LocalFileStore`, `RedisStore`)     |
| `namespace`              | `str`         | Optional namespace to avoid key collisions between different projects       |


### Storage Backends Supported in `langchain.storage`
LangChain provides a flexible caching mechanism through the `langchain.storage` module. You can choose from several storage backends depending on your performance, persistence, and scalability needs.

| Storage Class              | Description                                                                 |
|---------------------------|-----------------------------------------------------------------------------|
| `LocalFileStore`          | Stores cached data as files on the local filesystem                         |
| `InMemoryStore`           | Simple in-memory storage for fast, non-persistent use cases                 |
| `InMemoryByteStore`       | In-memory byte-level key-value store                                        |
| `RedisStore`              | Redis-based persistent key-value storage                                    |
| `UpstashRedisStore`       | Redis store using Upstash (serverless Redis)                                |
| `UpstashRedisByteStore`   | Byte-level store for Upstash Redis                                          |
| `EncoderBackedStore`      | Wrapper for encoding keys/values before saving to a store                   |
| `InvalidKeyException`     | Custom exception raised when store key is invalid                           |
| `create_kv_docstore()`    | Helper to initialize a document-level key-value store                       |
| `create_lc_store()`       | Factory to quickly create a LangChain-compatible store from configs         |


## 🧩 Use Case: Accelerating Smart Customer Service with Embedding Caching

In a typical smart customer service system:

- The knowledge base contains over **100,000 question-answer pairs**.
- When a user asks a question, the system must **retrieve the most relevant QA match**.
- Using a traditional method, the system **calculates embeddings for every question at query time**, which becomes **computationally expensive and slow**.


### Solution: Embedding Cache for Speed & Efficiency
To address the performance bottleneck, we implement a **caching mechanism for embeddings** using LangChain's `CacheBackedEmbeddings`.

| Action | Traditional | With Cache |
|--------|-------------|-------------|
| First-time embedding | Computed for each query | Pre-computed and cached |
| Repeated queries     | Recomputed repeatedly   | Directly loaded from cache |
| New questions        | Full re-embedding       | Dynamically update cache |


### Implementation Strategy

1. **Initial Preprocessing**
   - Upon system startup or first-time indexing, pre-compute all QA embeddings.
   - Store them using `LocalFileStore`, `RedisStore`, or other persistent cache.

2. **Real-time Query**
   - Convert user question to embedding.
   - Compare it against **cached QA vectors** for fast similarity search (e.g., cosine similarity).

3. **Dynamic Updates**
   - When new QA pairs are added, automatically generate embeddings and add to the cache.


### Basic Implementation (No Caching)

```python
from langchain.embeddings import OpenAIEmbeddings

# Initialize the embedding model with your API key
embedder = OpenAIEmbeddings(openai_api_key="sk-xxx")

# Generate an embedding for a sample text
vector = embedder.embed_documents(["How do I reset my password?"])

# Output the dimensionality of the generated vector
print(f"Embedding dimension: {len(vector[0])}")
```

### Implementation with LocalFileStore

```python
from langchain.storage import LocalFileStore
from langchain.embeddings import CacheBackedEmbeddings

# Step 1: Create a local file-based cache store
fs = LocalFileStore("./embedding_cache/")

# Step 2: Wrap your embedding model with a cache decorator
cached_embedder = CacheBackedEmbeddings.from_bytes_store(
    underlying_embeddings=embedder,
    document_embedding_store=fs,
    namespace="openai-embeddings"  # optional: separates different model versions
)

# Step 3: Generate embedding (writes to cache if not present)
vector1 = cached_embedder.embed_documents("How do I reset my password?")

# Step 4: Reuse the same input (reads from cache instantly)
vector2 = cached_embedder.embed_documents("How do I reset my password?")

print(f"Cache consistency check: {vector1 == vector2}")  # Expected: True
```

### Advanced Configuration: Distributed Embedding Cache with Redis

```python
from redis import Redis
from langchain.storage import RedisStore
from langchain.embeddings import CacheBackedEmbeddings

# Step 1: Connect to Redis server
redis_client = Redis(host="localhost", port=6379)

# Step 2: Create Redis-based store with TTL (24h)
redis_store = RedisStore(redis_client, ttl=86400)  # 86400 seconds = 24 hours

# Step 3: Wrap the embedding model with Redis-backed cache
cached_embedder = CacheBackedEmbeddings.from_bytes_store(
    underlying_embeddings=embedder,
    document_embedding_store=redis_store,
    namespace="openai-v3"  # optional: isolate model/version
)
```


## 🔁 Embedding Caching in Practice: Performance Comparison
When building RAG systems at scale, caching can significantly reduce the cost and latency of embedding generation. LangChain's `CacheBackedEmbeddings` provides a plug-and-play caching mechanism for your embedding models.

### Key API Design Differences

LangChain separates use cases via two main methods:

| Method             | Description                                  | Default Caching Behavior |
|--------------------|----------------------------------------------|------------------|
| `embed_documents`  | Batch embedding for document preprocessing   | ✅ **Cached**     |
| `embed_query`      | Real-time embedding for user queries         | ❌ **Not cached** |

> set the`cache_query` parameter to `True` when initializing the embedder to enable Caching.


### Design Considerations

| Aspect         | `embed_documents`                              | `embed_query`                                 |
|----------------|--------------------------------------------------|------------------------------------------------|
| Use Case       | Preprocessing large document corpora            | Real-time user interaction                    |
| Data Repetition| High (legal clauses, product FAQs)              | Low (diverse user queries)                    |
| Performance    | Batch I/O amortizes cache overhead               | One-off cache checks may add latency          |
| Storage Impact | Caching improves retrieval & cost-efficiency    | Caching may be wasteful for low-hit scenarios |


```python
import os
import time
from langchain_openai import OpenAIEmbeddings
from langchain.storage import LocalFileStore
from langchain.embeddings import CacheBackedEmbeddings


os.environ["OPENAI_API_KEY"] = "*"  # replace with your actual key


# Step 1: Initialize the embedding model
openai_embedding_model = OpenAIEmbeddings(
    model="text-embedding-3-small",
    max_retries=3,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Step 2: Initialize local file-based storage
storage = LocalFileStore("./embedding_cache/")


# Step 3: Wrap the model with CacheBackedEmbeddings
cached_embeddings = CacheBackedEmbeddings.from_bytes_store(
    openai_embedding_model,
    storage,
    namespace="openai_emb"  # Use a namespace to isolate different embedding models
)

# Step 4: Embed a batch of texts (intentionally duplicated/similar)
texts = ["LangChain AI LLM Dev Course", "How to use LangChain AI LLM Dev Course to learn AI"]

# First-time embedding (computes and caches)
start_time = time.time()
emb1 = cached_embeddings.embed_documents(texts)
print(f"Embedding dimension: {len(emb1[0])}")

first_call_duration = time.time() - start_time
print(f"First call duration: {first_call_duration:.4f} seconds")

# Second-time embedding (retrieves from cache)
start_time = time.time()
emb2 = cached_embeddings.embed_documents(texts)
print(f"Cache hit equality check: {emb1 == emb2}")
second_call_duration = time.time() - start_time
print(f"Second call duration: {second_call_duration:.4f} seconds")


# First Time Output:
# Embedding dimension: 1536
# First call duration: 1.3351 seconds
# Cache hit equality check: True
# Second call duration: 0.0007 seconds
```

### from_bytes_store()
In LangChain, `CacheBackedEmbeddings.from_bytes_store(...)` is a convenience constructor that wraps a **byte-based storage** backend (like LocalFileStore or RedisStore) into a format that CacheBackedEmbeddings can use directly. Most storage systems (local files, Redis, Upstash, etc.) store data as binary (bytes) instead of structured objects.

- Adapts byte-based stores into a format LangChain’s embedding cache system understands.
- Automatically wraps the byte store with an internal serializer/deserializer to handle vectors.


## Best Practices for Using CacheBackedEmbeddings in LangChain

Use `CacheBackedEmbeddings` when you're dealing with:

- **Highly repetitive text corpora**, such as:
  - Product descriptions
  - Legal clauses
  - Support FAQs
- **High API cost sensitivity**: Avoid redundant API calls.
- **On-premise or offline models**: Speed up inference on repeated inputs.

### Storage Backend Selection Guide

| Storage Type      | Pros                                 | Cons                          | Recommended For            |
|-------------------|--------------------------------------|-------------------------------|----------------------------|
| `LocalFileStore`  | Zero setup, easy to debug            | Not suitable for distributed  | Local development / testing|
| `RedisStore`      | Fast, distributed, supports TTL      | Requires Redis setup          | Production environments    |
| `InMemoryStore`   | Fastest, no I/O latency              | Volatile: data lost on restart| Temporary experiments      |

> Use **namespaces** (e.g., `"product-vectors"`) to isolate cache entries across different embedding models or projects.












<br>




---

🚀 Stay tuned for more insights into LangChain and RAG!



