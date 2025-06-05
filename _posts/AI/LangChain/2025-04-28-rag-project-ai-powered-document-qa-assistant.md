---
title: RAG project - AI-powered Document Q&A Assistant
date: 2025-04-28
categories: [AI, LangChain]
tags: [LangChain, RAG, Milvus, LLM]
author: kai
permalink: /posts/ai/langchain/rag-project-ai-powered-document-qa-assistant/
---

# 🚀 RAG project - AI-powered Document Q&A Assistant
This is a hands-on RAG (Retrieval-Augmented Generation) project, will build an **AI-powered Document Q&A Assistant** that can answer questions about online documentation and API manuals. The system leverages **LangChain** for orchestration and **Milvus** for high-performance vector storage.


## 🎯 Project Objective

Create an AI document assistant that:

- Loads and chunks online documents
- Generates vector embeddings for document segments
- Stores them in a vector database (Milvus)
- Answers user queries using a RAG pipeline with LLM support

## 🧰 Technology Stack

| Component          | Technology                |
|--------------------|----------------------------|
| Framework          | [LangChain](https://www.langchain.com/) |
| Vector Database    | [Milvus](https://milvus.io) |
| Embedding Model    | OpenAIEmbeddings `text-embedding-3-small` |
| LLM Model          | ChatOpenAI (`gpt-4o-mini`)   |


## 🧱 Functional Overview

### 1. Document Loading & Splitting

- **WebBaseLoader**: Loads documents from specified URLs.
- **RecursiveCharacterTextSplitter**: Splits loaded text into chunks of a defined size with overlapping context to preserve semantic continuity.

### 2. Vector Embedding Generation

- **OpenAIEmbeddings**: Generates vector embeddings using text-embedding-3-small.
- Includes robust retry handling (up to 3 times).

###  3. Vector Storage and Retrieval

- **Milvus**: Acts as the vector store backend.
- A collection named `doc_qa_db` is created to hold document vectors.
- Supports **similarity search** for retrieving relevant content.


### 4. Retrieval-Augmented Q&A (RAG)
- **ChatOpenAI (gpt-4o-mini)**: LLM used to generate answers.
- **PromptTemplate**: Constructs the LLM prompt by combining user query and retrieved content.
- **RAG Chain**: Integrates retriever and LLM into a complete pipeline.


## 🥷🏻 Code Example

### Step 1: 
- Document Loading and Splitting
- Vector Embedding Generation
- Load Embedding into Vector Storage 
- Test via Milvus similarity search

```python
import os
from langchain_milvus import Milvus
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import WebBaseLoader
from langchain.prompts import PromptTemplate
from langchain.schema.runnable import RunnablePassthrough


os.environ["OPENAI_API_KEY"] = "*" 
COLLECTION_NAME = 'doc_qa_db'

# Step 1: Load source data from 3 websites
loader = WebBaseLoader([
    'https://milvus.io/docs/overview.md',
    'https://milvus.io/docs/release_notes.md',
    'https://milvus.io/docs/quickstart.md'
])


# Load all documents
docs = loader.load()

# Step 2: transform documents into smaller chunks via LangChain's RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1024, chunk_overlap=20)
all_splits = text_splitter.split_documents(docs)


# Step 3: Initialize the embedding model, vector store, LLM Model
openai_embeddings = OpenAIEmbeddings(  # Initialize the embedding model
    model="text-embedding-3-small",
    max_retries=3,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

vector_store = Milvus.from_documents(  # Initialize the vector store 
    documents=all_splits,
    embedding=openai_embeddings,
    collection_name=COLLECTION_NAME,
    connection_args={
        "uri": "http://54.163.61.180:19530",
        "db_name": "my_database"
    }
)

# Testing via Milvus API
query = "What are the main components of Milvus?"
docs = vector_store.similarity_search(query)
print(len(docs))
```


### Step 2
- The embedding already in vectore db, need not to load data into db again
- Retrieval-Augmented Q&A (RAG)
- Test via LLM 

```python
# Initialize the vector store
vector_store = Milvus(
    openai_embeddings,
    collection_name=COLLECTION_NAME,
    connection_args={"uri": "http://54.163.61.180:19530",
                     "db_name": "my_database"}
)

llm = ChatOpenAI(  # Initialize the LLM model
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)


# the results of the similarity search are used as retriever, and the question input is passed to the LLM after the retrieval, and the enhanced answer is obtained.
retriever = vector_store.as_retriever()


# Step 4: Define the PromptTemplate, which is used to construct the input to the LLM
template = """
You are a AI document assistant. Using the following context to answer the last question.
If you don't know the answer, just say you don't know, don't try to make up an answer.
Answer the question within 10 sentences, and try to make the answer as simple as possible.
Always end your answer with "Thanks for your question."”.
{context}
Question: {question}
"""
rag_prompt = PromptTemplate.from_template(template)

rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | rag_prompt
    | llm
)

question = "What are the main components of Milvus?"
print(rag_chain.invoke(question))

# content='Milvus consists of several main components that work together to provide efficient vector database functionality. 
# These components include:\n\n1. **Milvus Lite**: A lightweight Python library for quick prototyping and running on devices with limited resources.\n2. 
# **Milvus Standalone**: A single-machine server deployment that bundles all components into a single Docker image for easy setup.\n3. 
# **Milvus Distributed**: A cloud-native architecture designed for large-scale deployments on Kubernetes clusters, ensuring redundancy and scalability.\n4. \
# **APIs and SDKs**: Milvus offers various APIs and software development kits (SDKs) for different programming languages, including Python, Go, Java, Node.js, and C#.\n5. 
# **Data Types**: It supports advanced data types like sparse vectors, binary vectors, JSON, and arrays, along with various distance metrics for similarity searches.\n\n
# These components make Milvus a powerful tool for handling unstructured data and performing advanced searches. 
# Thanks for your question.' 
# additional_kwargs={'refusal': None} 
# response_metadata={'token_usage': {'completion_tokens': 197, 'prompt_tokens': 1118, 'total_tokens': 1315, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_0392822090', 'finish_reason': 'stop', 'logprobs': None} id='run-770d1ebe-c6a1-4e13-b00f-f1cb292bf1e9-0' usage_metadata={'input_tokens': 1118, 'output_tokens': 197, 'total_tokens': 1315, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}}
```


## 🔧 Building a RAG Chain in LangChain

**RAG Chain Definition**:

```python
rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | rag_prompt
    | llm
)
```

This RAG chain consists of three main stages:

1. **Input Dictionary Construction** - `{"context": retriever, "question": RunnablePassthrough()}`
- **context:** The retriever is called with the user’s question and returns relevant document chunks from a vector store.
- **question:** Passed through as-is using `RunnablePassthrough()`. This ensures that the **raw input question** is preserved and forwarded to the next stage.
- Both context and question will be passed to the next component in the pipeline as a dictionary


2. **Prompt Construction** - `| rag_prompt`
- rag_prompt is a PromptTemplate that formats the context and question into a well-structured prompt string.
- It defines how the language model should interpret the documents and user query.


3. **Language Model Generation** - `| llm`
- The final prompt string from the PromptTemplate is passed into the LLM (e.g., GPT-4, Qwen, Claude).
- The LLM uses the context-enhanced prompt to generate a response.


### Summary of Flow

```text
User Question
     ↓
→ Passed to both:
     - Retriever → returns context documents
     - Passthrough → keeps original question
     ↓
→ Combined into {"context": ..., "question": ...}
     ↓
→ Passed into rag_prompt to build the final prompt
     ↓
→ Final prompt goes into LLM for answer generation
     ↓
→ Response returned to the user
```


## 🌍 Use Cases and Future Directions
This RAG-based Document Q&A Assistant is highly adaptable and can be applied across a wide range of domains beyond just developer documentation.

### Use Cases
#### Education

- Summarize lecture notes and course materials
- Assist teachers in lesson planning and Q&A preparation
- Enable students to interact with learning resources through natural language queries

#### Enterprise Knowledge Base

- Build intelligent Q&A systems over internal documentation (e.g., HR policies, SOPs, training manuals)
- Improve onboarding and support by surfacing relevant answers from unstructured documents
- Enable cross-department knowledge sharing through searchable context-aware assistants

#### Technical Support

- Provide AI-powered helpdesk functionality by retrieving answers from product manuals, SDK documentation, and FAQs
- Support developers and customers with intelligent troubleshooting and API lookup



### Future Directions

To further enhance the system’s capability and flexibility, several improvement paths are planned:

#### Support More Document Loaders

- Extend support for file formats such as **PDF**, **Word (DOCX)**, and **Markdown**
- Enable batch ingestion from local folders, cloud storage, or document management systems

#### Multilingual Support

- Integrate multilingual embedding models and translation pipelines
- Enable cross-language document retrieval and answer generation (e.g., English ↔ Chinese)

#### Efficiency Optimizations

- Improve embedding generation speed through batching and parallelization
- Optimize retrieval performance using indexing strategies and hybrid search
- Implement caching for frequently asked queries to reduce response latency




<br>




---

🚀 Stay tuned for more insights into LangChain and RAG!



