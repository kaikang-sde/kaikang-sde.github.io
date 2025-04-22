---
title: LangChain VectorStore Architecture for RAG Systems
date: 2025-04-22
order: 4
categories: [AI, LangChain]
tags: [LangChain, Vector Database, Milvus, RAG, VectorStore]
author: kai
---

# 🚀 LangChain VectorStore Architecture for RAG Systems
LangChain provides a unified abstraction for vector databases through the `VectorStore` interface, enabling seamless integration with different backends such as **Milvus**, **Chroma**, **Pinecone**, and more.


## 🔧 Core RAG Design Flow in LangChain

```plaintext
       Document
          │
          ▼
     Text Splitter
          │
          ▼
     Embedding Model
          │
          ▼
  [Milvus | Chroma | Pinecone …] ←→ VectorStore        
```

- **Document**: Any raw text, PDF, HTML, etc., containing the information to be indexed.
- **Text Splitter**: Breaks long documents into manageable chunks for embedding.
- **Embedding Model**: Converts each text chunk into a dense numerical vector.
- **VectorStore**: Stores the vectors in a searchable format to support similarity search or MMR retrieval.


## 💡 VectorStore Abstraction
[Reference Documentation](https://python.langchain.com/docs/integrations/vectorstores/)

LangChain defines a generic `VectorStore` interface in `langchain_core.vectorstores.VectorStore`. Each specific vector database (e.g. Milvus, Chroma) implements this interface with methods like:

- `add_documents()`
- `similarity_search()`
- `max_marginal_relevance_search()`

This abstraction allows developers to switch between databases without changing application logic.

### Installation for Milvus Integration

To use **Milvus** as the backend vector store: `pip install langchain-milvus`


![Abstract - VectoreStore](/assets/img/posts/AI/LangChain/VectorStoreABC.png)


## 🛠️ Common Methods in `VectorStore`

| Method Name  | Description / Scenarios | Example Parameters   |
|---------------|--------------|----------------------|
| `from_documents()`| Create a vector store from a list of documents<br> - For initializing a new vector store with pre-embedded documents. | `documents`, `embedding`, `**kwargs`  |
| `add_documents()` | Append new documents to an existing vector store<br> - When incrementally updating the store with new content.  | `documents`  |
| `similarity_search()` | Perform a basic similarity search<br> - Ideal for fetching top-k semantically similar chunks. | `query`, `k=4`  |
| `similarity_search_with_score()`   | Return similarity results along with score   | `query`, `k=4`  |
| `max_marginal_relevance_search()`  | Perform MMR-based search to balance relevance & diversity<br> - Recommended for applications requiring **diversity** in responses. | `query`, `k=4`  |
| `as_retriever()`  | Wrap the store as a retriever for use in chains<br> - Enables plug-and-play integration with LangChain's chain or agent-based systems.       | `search_kwargs={}` |


## 🗜️ Initialization Method - `from_documents()`
The `from_documents()` class method is a convenient way to initialize a vector store directly from a list of LangChain `Document` objects. It automatically converts the input documents into embedding vectors and stores them in the specified vector database backend.

```python
@classmethod
def from_documents(
    cls,
    documents: List[Document],
    embedding: Embeddings,
    **kwargs
) -> VectorStore:
    """Return VectorStore initialized from documents and embeddings.

    Args:
        documents: List[Document] ->  A list of langchain_core.documents.Document objects to be embedded and stored.
        embedding: Embeddings -> An embedding model instance used to convert text into vectors.
        kwargs: dict -> Additional arguments specific to the underlying vector database (e.g., collection name, index parameters).

    Returns:
        VectorStore: Returns an instance of a VectorStore subclass that holds all embedded documents and is ready for similarity or MMR-based search.
    """
```

### `from_documents()` vs. `add_documents()`

- Use `from_documents()` when you're setting up a **brand new vector store** from scratch.
- Use `add_documents()` when you're **adding new data** to an already created vector store (e.g., weekly sync, new articles, updates).


| Feature           | `from_documents()`                             | `add_documents()`                                |
|-------------------|------------------------------------------------|--------------------------------------------------|
| **Method Type**   | Class method (static)                          | Instance method                                  |
| **Primary Use**   | Initialize a new vector store with documents   | Append new documents to an existing vector store |
| **Collection**    | Automatically creates a new collection         | Requires collection to already exist             |
| **Performance**   | Higher cost (embedding + indexing)             | Lower cost (direct insertion only)               |
| **Use Case**      | First-time document ingestion                  | Incremental updates                              |
| **Connection**    | Needs full config (DB, collection, etc.)       | Reuses existing instance settings                |


## ➕ Insert Plain Text into Vector Store - `add_texts()` 
The `add_texts()` method is used to **insert raw text data** directly into an existing vector store, converting them into vector representations and storing them along with optional metadata.

```python
def add_texts(
    self,
    texts: Iterable[str],
    metadatas: Optional[list[dict]] = None,
    *,
    ids: Optional[list[str]] = None,
    **kwargs: Any,
) -> list[str]:
    """Run more texts through the embeddings and add to the vectorstore.

    Args:
        texts:Iterable[str] -> A list or iterable of raw text strings to embed and insert into the store.
        metadatas: Optional[List[dict]] -> Optional list of metadata dictionaries matching the texts.
        ids: Optional list of IDs associated with the texts.
        **kwargs: dict -> vectorstore specific parameters, such as namespace, collection name.
            One of the kwargs should be `ids` which is a list of ids
            associated with the texts.

    Returns:
        List[str]: A list of document IDs generated or returned by the vector store for each inserted text.

    Raises:
        ValueError: If the number of metadatas does not match the number of texts.
        ValueError: If the number of ids does not match the number of texts.
    """
```

## 🔍 Perform Similarity-Based Retrieval - `similarity_search()`
The `similarity_search()` method is used to retrieve the most semantically similar documents to a given query string, based on vector embeddings and similarity metrics (e.g., cosine similarity, L2 distance).

```python
def similarity_search(
        self, query: str, k: int = 4, **kwargs: Any
    ) -> list[Document]:
        """Return docs most similar to query.

        Args:
            query: str -> Input query string to be embedded and used as search key
            k: int -> Number of top similar documents to return (default = 4).
            filter: Optional[dict] -> Metadata filtering conditions (e.g., {"source": "docs"}), optional.
            **kwargs: dic -> Additional backend-specific search parameters (e.g., nprobe, radius).

        Returns:
            List[Document]: A list of LangChain Document objects, ranked by similarity to the input query.
        """
```

## 🎯 Maximal Marginal Relevance (MMR) Search - `max_marginal_relevance_search()` 
The `max_marginal_relevance_search()` method enhances standard vector similarity search by incorporating **diversity**. It aims to return results that are both **highly relevant** to the query and **mutually diverse**, reducing redundancy in the output.

```python
def max_marginal_relevance_search(
    self,
    query: str,
    k: int = 4,
    fetch_k: int = 20,
    lambda_mult: float = 0.5,
    **kwargs: Any,
    ) -> list[Document]:
    """Return docs selected using the maximal marginal relevance.

    Maximal marginal relevance optimizes for similarity to query AND diversity
    among selected documents.

    Args:
        query: str -> The input query text to search for.
        k: int -> Number of Documents to return. Defaults to 4.
        fetch_k: int -> Number of Documents to fetch to pass to MMR algorithm. Default is 20.
        lambda_mult: float -> Number between 0 and 1 that determines the degree
            of diversity among the results with 0 corresponding
            to maximum diversity and 1 to minimum diversity.
            Defaults to 0.5. (Relevance-diversity tradeoff parameter)
        **kwargs: Arguments to pass to the search method.

    Returns:
        List[Document]: A list of Document objects selected to maximize both relevance and diversity.
        """
```

## 🆚 Feature Comparison of Popular Vector Databases

Different vector databases offer varying levels of support for critical features.

| Feature                  | Milvus | FAISS | Pinecone | Chroma |
|--------------------------|:------:|:-----:|:--------:|:------:|
| Distributed Support      | ✅     | ❌    | ✅       | ❌     |
| Metadata Filtering       | ✅     | ❌    | ✅       | ✅     |
| Automatic Index Handling | ✅     | ❌    | ✅       | ✅     |
| Local Deployment         | ✅     | ✅    | ❌       | ✅     |
| Similarity Algorithms    | 8      | 4     | 3        | 2      |

- **Distributed Support**: Whether the database can scale horizontally across multiple nodes.
- **Metadata Filtering**: Support for combining vector search with structured attribute filters (e.g., `category == 'AI'`).
- **Automatic Index Handling**: Automatically creates and manages vector indexes for performance optimization.
- **Local Deployment**: Ability to run the database on a local machine without requiring cloud infrastructure.
- **Similarity Algorithms**: Number of distance metrics supported, such as cosine similarity, Euclidean distance (L2), inner product, etc.


### Selection Tips:

- Use **Milvus** for large-scale production with metadata filtering and high scalability.
- Choose **FAISS** for lightweight, local GPU-accelerated search.
- Opt for **Pinecone** if you want a serverless, fully-managed experience.
- Pick **Chroma** for fast prototyping and local development.


## ❄️ Vector Database in Knowledge Base Cold Start

When launching a new AI-powered knowledge base, there's often no historical query log or prebuilt vector store. This is known as the **cold start problem**, and it typically includes:

- A large collection of documents but no precomputed embeddings.
- No cached indexes.
- No prior user query data.
- Immediate need for semantic retrieval.


### Challenges

| Challenge | Description |
|----------|-------------|
| Embedding Latency | Generating vector embeddings for thousands of documents in real-time is costly and slow. |
| Irrelevant Results | Without proper chunking, early retrieval results may be imprecise or noisy. |
| Lack of Feedback | No historical user behavior to refine retrieval quality. |
| API Cost | Embedding every document through an external service (e.g., OpenAI) can be expensive. |


### Best Practices
1. Pre-split Documents Before Embedding
    - Use semantic-aware chunking to reduce embedding noise and improve retrieval
2. Use Cached Embeddings
    - Avoid redundant computation with CacheBackedEmbeddings
3. Batch Insert into Vector Store
    - Don’t insert one by one. Use add_documents() or from_documents() for efficiency.
4. Pre-build Indexes (if supported)
    For vector stores like Milvus, Pinecone, or FAISS, create indexes before querying.
5. Load to Memory for Fast Query
    - Milvus or similar databases require explicit memory loading before search



<br>




---

🚀 Stay tuned for more insights into LangChain and Vector Database!



