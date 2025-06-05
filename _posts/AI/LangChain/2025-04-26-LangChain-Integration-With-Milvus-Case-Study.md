---
title: LangChain Integration with Milvus - Case Study
date: 2025-04-26
categories: [AI, LangChain]
tags: [LangChain, Vector Database, Milvus, RAG]
author: kai
permalink: /posts/ai/langchain/langchain-integration-with-milvus-case-study/
---

# 🚀 LangChain Integration with Milvus - Case Study
LangChain provides a unified interface for various vector databases, including **Milvus** — a scalable and high-performance vector DB. This post walks through the **practical usage of insert and delete operations** using LangChain's Milvus integration.

[LangChain Milvus Integration Docs](https://python.langchain.com/docs/integrations/vectorstores/milvus/)


## 1️⃣ Use Case: Insert and Delete Operations

- **Vectorize and insert** new documents into Milvus.
- **Delete** documents using metadata filters or IDs.

### Setup: Embedding model, vector_store, mocking data

```python
import os
# from langchain.vectorstores import Milvus # old version
from langchain_milvus import Milvus
from langchain_core.documents import Document
from uuid import uuid4
from langchain_openai import OpenAIEmbeddings


# replace with your actual key
os.environ["OPENAI_API_KEY"] = "*"

# Initialize the embedding model
openai_embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small",
    max_retries=3,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)


# Connect to Milvus Vector Store
vector_store = Milvus(
    openai_embeddings,
    connection_args={
        "uri": "http://54.163.61.180:19530",
        "db_name": "my_database"
    },
    collection_name="langchain_example"
)


# Prepare Your Documents
document_1 = Document(
    page_content="I had chocolate chip pancakes and scrambled eggs for breakfast this morning.",
    metadata={"source": "tweet"},
)
document_2 = Document(
    page_content="The weather forecast for tomorrow is cloudy and overcast, with a high of 62 degrees.",
    metadata={"source": "news"},
)
document_3 = Document(
    page_content="Building an exciting new project with LangChain - come check it out!",
    metadata={"source": "tweet"},
)
document_4 = Document(
    page_content="Robbers broke into the city bank and stole $1 million in cash.",
    metadata={"source": "news"},
)
document_5 = Document(
    page_content="Wow! That was an amazing movie. I can't wait to see it again.",
    metadata={"source": "tweet"},
)
document_6 = Document(
    page_content="Is the new iPhone worth the price? Read this review to find out.",
    metadata={"source": "website"},
)
document_7 = Document(
    page_content="The top 10 soccer players in the world right now.",
    metadata={"source": "website"},
)
document_8 = Document(
    page_content="LangGraph is the best framework for building stateful, agentic applications!",
    metadata={"source": "tweet"},
)
document_9 = Document(
    page_content="The stock market is down 500 points today due to fears of a recession.",
    metadata={"source": "news"},
)
document_10 = Document(
    page_content="I have a bad feeling I am going to get deleted :(",
    metadata={"source": "tweet"},
)

documents = [
    document_1, document_2, document_3, document_4, document_5, document_6,
    document_7, document_8, document_9, document_10,
]
```

### Insert 
```python
# Insert Documents into Milvus with Custom IDs
ids = [str(i + 1) for i in range(len(documents))]
print("Generated IDs:", ids) # Generated IDs: ['1', '2', '3', '4', '5', '6', '7', '8', '9', '10']

result = vector_store.add_documents(documents=documents, ids=ids)
print(result) # ['1', '2', '3', '4', '5', '6', '7', '8', '9', '10']
```


### Delete Documents from Milvus by ID

```python
# Delete the document with ID "1"
result = vector_store.delete(ids=["1"])
print(result) 
# (insert count: 0, delete count: 1, upsert count: 0, timestamp: 457484477161799681, success count: 0, err count: 0
```


## 2️⃣ Use Case: Similarity Search vs. MMR

- Connect LangChain with Milvus vector database
- Showcase 1: Similarity Search
- Showcase 2: MMR search to return diverse, relevant results
- Use repeated vector insertions to test how MMR handles redundancy

> Milvus does **not deduplicate** vector entries, making it ideal for demonstrating how MMR reduces repetitive search results.

### Test Scenario
Will insert multiple identical documents into the Milvus vector store and run both **similarity search** and **MMR search** to observe:

- Whether duplicate documents appear in results.
- Whether MMR introduces more diversity.

### Setup: Embedding model, vector_store, mocking data

```python
import os
from langchain_milvus import Milvus
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings


# replace with your actual key
os.environ["OPENAI_API_KEY"] = "*"

# Initialize the embedding model
openai_embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small",
    max_retries=3,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Define repeated documents to test MMR diversity
document_1 = Document(
    page_content="LangChain support multiple database integrations. I am learning AI and LangChain.",
    metadata={"source": "kai.com/doc1"},
)
document_2 = Document(
    page_content="Milvus is good at vector search, the wesite of Kai is good",
    metadata={"source": "kai.com/doc2"},
)
document_3 = Document(
    page_content="I will learn architecture course",
    metadata={"source": "kai.com/doc3"},
)
document_4 = Document(
    page_content="Today's weather is good, Jojo and Kai went to hiking",
    metadata={"source": "kai.com/doc4"},
)

documents = [
    document_1, document_2, document_3, document_4
]

# Connect to vector store
vector_store = Milvus.from_documents(
    documents=documents,
    embedding=openai_embeddings,
    collection_name="searching_test",
    connection_args={
        "uri": "http://54.163.61.180:19530",
        "db_name": "my_database"
    }
)
```

### Similarity Search

```python
query = "How to integrate database?"
results = vector_store.similarity_search(query, k=3)
for doc in results:
    print(f"content: {doc.page_content}\nmetadata: {doc.metadata}\n")


# Output:
# content: LangChain support multiple database integrations. I am learning AI and LangChain.
# metadata: {'source': 'kai.com/doc1', 'pk': 457482682126599157}

# content: LangChain support multiple database integrations. I am learning AI and LangChain.
# metadata: {'source': 'kai.com/doc1', 'pk': 457482682126599162}

# content: I will learn architecture course
# metadata: {'source': 'kai.com/doc3', 'pk': 457482682126599159}
```

- **Returns duplicate documents** (same content and metadata).
- No attempt to reduce redundancy.

### Similarity Search : Mix search (combined with metadata filtering)

```python
query = "How to integrate database?"
# Mix search (combined with metadata filtering)
results = vector_store.similarity_search_with_score(
    query,
    k=3,
    expr='source == "kai.com/doc1"'
)

print(results)

# Output: Only 2 data are returned
# [(Document(metadata={'pk': 457482682126599157, 'source': 'kai.com/doc1'}, page_content='LangChain support multiple database integrations. I am learning AI and LangChain.'), 1.0892812013626099), 
#  (Document(metadata={'pk': 457482682126599162, 'source': 'kai.com/doc1'}, page_content='LangChain support multiple database integrations. I am learning AI and LangChain.'), 1.0892812013626099)]
```

### MMR (Maximal Marginal Relevance)

```python
diverse_results = vector_store.max_marginal_relevance_search(
    query="How to integrate databases?",
    k=3,
    fetch_k=10,
    lambda_mult=0.4
)

for result in diverse_results:
    print(f"content: {result.page_content}\nmetadata: {result.metadata}\n")


# Output: No duplicates
# content: LangChain support multiple database integrations. I am learning AI and LangChain.
# metadata: {'pk': 457482682126599157, 'source': 'kai.com/doc1'}

# content: Today's weather is good, Jojo and Kai went to hiking
# metadata: {'pk': 457482682126599160, 'source': 'kai.com/doc4'}

# content: I will learn architecture course
# metadata: {'pk': 457482682126599159, 'source': 'kai.com/doc3'}
```








<br>




---

🚀 Stay tuned for more insights into LangChain and Vector Database!



