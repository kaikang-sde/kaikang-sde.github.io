---
title: Embedding in LangChain
date: 2025-04-15
categories: [AI, LangChain]
tags: [LangChain, LLM, Vector, Text Embedding]
author: kai
---

# 🚀 Embedding in LangChain: A Universal Interface for Vector Representations
LangChain provides a **standardized interface** for integrating various **text embedding models**, allowing developers to easily plug in models from OpenAI, HuggingFace, Cohere, and others — all through a unified API.

## 🧠 What Does It Do?

**Embedding** is the process of converting text into a list of numerical values (vectors) that capture the semantic meaning of the text.  
In LangChain, embedding models serve two main roles:

- **Document Embedding**: Converts a list of texts (e.g., documents, paragraphs) into a list of vectors
- **Query Embedding**: Converts a single query text into a vector for similarity comparison

> **Analogy**:  
> Think of embedding models like **different phone chargers**.  
> LangChain acts as a **universal adapter** that lets you use any brand — OpenAI, Cohere, HuggingFace — seamlessly.


## ⚙️ Interface Definition (Abstract Base Class)
LangChain’s Embeddings class is defined as an **abstract base class** (ABC), and embedding model implementations must override the following methods:

```python
from abc import ABC, abstractmethod

class Embeddings(ABC):

    @abstractmethod
    def embed_documents(self, texts: list[str]) -> list[list[float]]:
        """
        Embed a list of input documents into dense vectors.

        Args:
            texts: List of strings to be embedded.

        Returns:
            A list of embedding vectors, one for each document.
        """

    @abstractmethod
    def embed_query(self, text: str) -> list[float]:
        """
        Embed a single query into a dense vector.

        Args:
            text: The query string.

        Returns:
            A single embedding vector representing the query.
        """
```

### Core API Methods & Properties

| Method               | Description                                     | Example Use Case                    |
|----------------------|--------------------------------------------------|-------------------------------------|
| `embed_query(text)`  | Converts a single text input into an embedding  | Real-time user query embedding      |
| `embed_documents(texts)` | Converts a batch of texts into embeddings       | Embedding documents for RAG/QA      |


### Additional Runtime Properties

| Property           | Description   | Use Case Example                            |
|--------------------|---------------|---------------------------------------------|
| `max_retries`      | Number of times to retry a failed API request     | Handle unstable cloud APIs                |
| `request_timeout`  | Maximum allowed time (in seconds) for each API call  | Control latency in processing long queries  |


## 👍🏻 Supported Embedding Model Types in LangChain

Different embedding models offer different trade-offs in terms of **dimension**, **performance**, and **accuracy**. LangChain supports a variety of embedding sources to suit your needs.

| Model Type            | Representative Models  | Key Characteristics    |
|-----------------------|------------------------|------------------------|
| Cloud API   | OpenAI, Cohere, HuggingFace Hub | No local resources required, pay-as-you-go       |
| Local Open Source  | Sentence-Transformers, FastText | High data privacy, can run fully offline         |
| Custom Fine-Tuned   | User-trained models   | High domain adaptability, tailored performance   |

> **Tip**: Choose cloud-based models for convenience and fast prototyping. Use local models for privacy or cost control. Fine-tune your own model if you need domain-specific precision (e.g., legal, medical, finance).


## ☁️ Cloud API Example: OpenAI 

```python
import os
from langchain_openai import OpenAIEmbeddings


os.environ["OPENAI_API_KEY"] = "*"  # replace with your actual key


openai_embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small",
    max_retries=3,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)


comments = [
    "The quality of the product is good, but the shipping is too slow",
    "Reasonable price, will buy again!",
    "Size is too small, recommend buying the big one"
]

# Return: List of embeddings, one for each text.
embeddings = openai_embeddings.embed_documents(comments)
print(embeddings) 
print(len(embeddings)) # 3 -> 3 elements in comments
print(len(embeddings[0])) # 1536

# 1536 -> turn the first element in comment to vector, which has 1536 elements
# Actually, each comment has a vector of 1536


embeddings = openai_embeddings.embed_query("This is a test comment")    
print(len(embeddings)) # 1536
```

## 🔐 Self-Hosting Embedding Models for Private Deployment
Deploying text embedding models locally is becoming an essential choice for enterprises seeking **data privacy, cost efficiency, and domain customization**. While cloud-based APIs are convenient, they may not be suitable for sensitive or high-volume use cases.

| Requirement Type | Typical Use Case | Example |
|------------------|------------------|---------|
| **Data Security** | Government, Finance, Healthcare | Medical record embeddings must comply with national Data Security Law |
| **Customization** | Domain-specific terminology | Legal documents require accurate parsing of professional legal terms |
| **Cost Control** | High-frequency, long-term usage | E-commerce platforms embedding millions of product reviews daily |
| **Network Restrictions** | Air-gapped environments | Internal knowledge bases for defense and aerospace R&D |


### Local Embedding Workflow (Closed-Loop)

The complete data pipeline in a self-hosted environment looks like this:

```text
User Data 
   ↓
Intranet Server (Isolated)
   ↓
Local Embedding Model (e.g., BGE, Sentence-BERT)
   ↓
Vectorized Output
   ↓
Internal Database Storage (e.g., FAISS, Qdrant)
```

> ✅ No data ever leaves the organization’s firewall. This ensures compliance with legal regulations and builds trust with users.

[RAG Data Pipeline Overview]({{ site.baseurl }}/LangChain/RAG-System-Pipeline-Document-Loading/#-rag-data-pipeline-overview)

### Local Deployment: Running Embedding Models with Ollama
Deploying embedding models locally can be fast and developer-friendly using [Ollama](https://ollama.com/search?c=embed), a tool for managing and serving LLMs locally.


**Step 1: Download an Embedding Model**
- Ollama provides access to many open-source embedding models, including domain-specific ones. Visit the embedding model hub: [https://ollama.com/search?c=embed](https://ollama.com/search?c=embed)
- Example: `ollama run mofanke/acge_text_embedding`
- This command pulls and prepares the model locally.

**Step 2: Start Ollama in the Background**
- To run the model as a background service on the default port 11434:
- `ollama serve &`

**Step 3: List Running Models**
- Verify the currently available models with: `ollama ps`

**Step 4: Test Embedding API with cURL**
- Once the server is running, test the embedding API using a simple curl command:  
- `curl http://localhost:11434/api/embeddings -d '{"model": "mofanke/acge_text_embedding", "prompt": "Who are you"}'`


### Using Ollama Embedding Models in LangChain (Custom Integration)
LangChain v0.3.x does **not yet natively support Ollama embedding models**, but we can easily implement a custom embedding class to bridge this gap using a REST API call.

- **Integrate an Ollama-hosted embedding model with LangChain via a custom wrapper.**
- Prerequisites: `pip install requests langchain`
- Ensure the Ollama embedding model is already running on your local machine

```python
from typing import List
from langchain.embeddings.base import Embeddings
import requests

class OllamaEmbeddings(Embeddings):
    def __init__(self, model: str = "llama2", base_url: str = "http://localhost:11434"):
        self.model = model
        self.base_url = base_url

    def _embed(self, text: str) -> List[float]:
        try:
            response = requests.post(
                f"{self.base_url}/api/embeddings",
                json={
                    "model": self.model,
                    "prompt": text  # NOTE: some models may require `"text"` instead of `"prompt"`
                }
            )
            response.raise_for_status()
            return response.json().get("embedding", [])
        except Exception as e:
            raise ValueError(f"Ollama embedding error: {str(e)}")

    def embed_query(self, text: str) -> List[float]:
        return self._embed(text)

    def embed_documents(self, texts: List[str]) -> List[List[float]]:
        return [self._embed(text) for text in texts]
    

# Testing...
embedder = OllamaEmbeddings(model="mofanke/acge_text_embedding")

query_vector = embedder.embed_query("What is LangChain?")
print(query_vector[:5])  
# Preview first 5 dimensions -> [0.04526716470718384, 0.1308051496744156, -0.31260570883750916, 0.3838874101638794, 0.2710624933242798]

docs = ["LangChain powers LLM apps", "Ollama runs local models"]
doc_vectors = embedder.embed_documents(docs) 
print(f"Encoded {len(doc_vectors)} documents.") # Encoded 2 documents.
```





<br>




---

🚀 Stay tuned for more insights into LangChain and RAG!



