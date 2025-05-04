---
title: LLM Execution Trace and LangSmith
date: 2025-05-04
order: 7
categories: [AI, LangChain]
tags: [LangChain, LLM, Agent, LangSmith, Debugging]
author: kai
---

# 🚀 LLM Execution Trace & LangSmith: Debugging, Testing, and Monitoring Intelligent Agents
Building intelligent agents based on large language models (LLMs) is powerful—but often painful. Developers typically face the following **critical challenges**:
1. Debugging is Painful
2. Testing is Tedious
3. Production Monitoring is Missing

## 📉 LangSmith: A Unified Solution for Tracing, Testing, and Monitoring
**LangSmith** is a **debugging, tracing, and monitoring platform** officially developed by the LangChain team, designed to accelerate and stabilize LLM-based application development.

📍 **Installation:** `pip install langsmith==0.3.19`

📍 **Official site:** [https://smith.langchain.com](https://smith.langchain.com)

LangSmith addresses three major pain points in the lifecycle of LLM apps:

- **Hard to debug**: It's difficult to trace how an agent or chain made decisions step by step.
- **No result tracking**: You can't easily measure prompt effectiveness or compare versions.
- **No monitoring in production**: You have no visibility into latency, failures, or usage trends.

It plays a similar role to:

- `Spring Boot Actuator` in the Java ecosystem (for app health monitoring)
- The `ELK Stack` (for log analysis and observability)


## 🔧 Core Capabilities of LangSmith

LangSmith equips developers with essential features to debug, optimize, and monitor LLM applications:

### 1. **Execution Tracing & Debugging**

- Tracks the **entire execution path** of LLMs and agents, including:
  - Prompt templates
  - Tool invocations
  - Intermediate reasoning steps
- Visualizes **inputs, outputs, and duration** at each step
- Captures **success/failure rates**, latency, and execution metadata

### 2. **Production Monitoring**

- Monitors API errors and timeouts in **real time**
- Identifies **frequently asked queries**, failure spikes, and latency trends
- Helps ensure production-grade reliability for LLM-powered systems


## 🧭 Feature Matrix

| Module              | Description                                               | Use Case Example                              |
|---------------------|-----------------------------------------------------------|------------------------------------------------|
| **Execution Tracing** | Full chain/agent call logging and visualization          | Debugging complex LLM workflows               |
| **Prompt Analysis** | Compare prompt variations and observe output differences  | Prompt tuning and engineering optimization    |
| **Version Comparison** | Evaluate model or prompt changes across different versions | Model upgrades and regression testing         |
| **Production Monitoring** | Monitor API latency, error rate, and cost in real time | Operational health tracking                   |
| **Team Collaboration** | Share traces, dashboards, and results across teams      | Multi-developer LLM application development   |


## 🏛️ LangSmith System Architecture
LangSmith is engineered to **capture, store, and visualize** the entire lifecycle of interactions between your application, LLMs, tools, and user inputs.

Its architecture ensures traceability and observability without compromising performance or security.

```text
Your Application
   ↓
LangChain SDK  ➝ LangSmith API
     ↓               ↓           
LLM Providers  ➝ Trace Storage
     ↓               ↓ 
External Tools ➝ Analytics Engine
```


### Key Components

| **Component**         | **Description**                                                                 |
|------------------------|---------------------------------------------------------------------------------|
| **LangChain SDK**      | Automatically instruments LangChain apps to capture prompts, inputs, and outputs |
| **LangSmith API**      | Secure channel to stream logs, traces, and tool usage data to the LangSmith backend |
| **LLM Providers**      | Supports any OpenAI-compatible models, including Anthropic, Azure OpenAI, and more |
| **Trace Storage**      | Efficient, encrypted storage for all chain, agent, and tool execution metadata |
| **Analytics Engine**   | Powers dashboards with latency charts, token usage, error rates, and performance trends |
| **External Tools**     | Seamlessly integrates with third-party tools invoked by agents (e.g., search APIs, calculator, databases) |


## 🔍 LangChain Trace Integration with LangSmith
> Self-Hosting Option (Optional): If you’re deploying LangSmith privately (e.g., using Langfuse), you can replace the default endpoint: `os.environ["LANGSMITH_ENDPOINT"] = "http://localhost:3000"  # Example local server`


### Basic Example
```python
import os

# Enable tracing
os.environ["LANGCHAIN_TRACING_V2"] = "true"

# Set your API key from LangSmith dashboard
os.environ["LANGCHAIN_API_KEY"] = "*"

# Set the LangSmith backend endpoint
os.environ["LANGSMITH_ENDPOINT"] = "https://api.smith.langchain.com"

# Set your custom project name (used for grouping traces)
os.environ["LANGSMITH_PROJECT"] = "ai_test"

from langchain_openai import ChatOpenAI
import logging
logging.basicConfig(level=logging.DEBUG)

os.environ["OPENAI_API_KEY"] = "*"  


llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

response = llm.invoke("What is the LLM Agent?")
print(response.content)
```

![LangSmith Integration Example](/assets/img/posts/AI/LangChain/LangSmithBasicExp.png)



### Complex Example

```python
import os
from langchain_milvus import Milvus
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import WebBaseLoader
from langchain.prompts import PromptTemplate
from langchain.schema.runnable import RunnablePassthrough

# LangSmith Config
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "*"
os.environ["LANGSMITH_ENDPOINT"] = "https://api.smith.langchain.com"
os.environ["LANGSMITH_PROJECT"] = "ai_rag_test"

os.environ["OPENAI_API_KEY"] = "*" 

COLLECTION_NAME = 'doc_qa_db'

# Step 1: Load source data
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
```

![LangSmith RAG Example](/assets/img/posts/AI/LangChain/LangSmithRAGExp.png)










<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



