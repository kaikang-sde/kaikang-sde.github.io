---
title: LangChain Retrievers
date: 2025-04-27
order: 1
categories: [AI, LangChain]
tags: [LangChain, Retriever, Vector Store, RAG]
permalink: /LangChain-Retrievers/
author: kai
---

# 🚀 LangChain Retrievers

A **Retriever** is a standardized component in modern LLM pipelines that enables models to access external knowledge beyond their training data. It abstracts the retrieval process and provides a clean, consistent interface regardless of the underlying data source.

[RAG Data Pipeline Overview]({{ site.baseurl }}/LangChain/RAG-System-Pipeline-Document-Loading/#-rag-data-pipeline-overview)


## 🔌 Unified Interface

One of the key strengths of a Retriever is its **unified interface**. No matter whether the data comes from a local vector store, a relational database, or an online search engine, the Retriever **normalizes** the retrieval process and **returns a list of `Document` objects**.

This standardization makes it easier to integrate retrieval into LLM applications without worrying about the data source’s format or structure.


## 🌐 Multi-Source Hybrid Retrieval

To enhance recall and completeness, a Retriever can support **hybrid retrieval strategies**, such as:

- **Vector databases** (e.g., FAISS, Chroma)
- **Traditional SQL/NoSQL databases**
- **External search engines** (e.g., Bing, Google, ElasticSearch)

By combining structured and unstructured search techniques, hybrid retrieval **increases the chances of retrieving relevant and comprehensive context**, which is especially important in enterprise use cases or long-context generation.


## 🔁 Relationship with VectorStore

A Retriever **does not directly manage or store data**. Instead, it depends on a **VectorStore** to handle the heavy lifting of:

- Vectorizing the input query and stored documents
- Performing similarity search (e.g., using cosine similarity, HNSW, etc.)

The Retriever acts as a **bridge** between the LLM and the VectorStore, ensuring that relevant context is fetched when needed.


## 🔍 Role in RAG Systems

In RAG architectures, the Retriever is **the entry point for external data**. Its primary role is to:

1. Accept a natural language query or prompt from the user
2. Fetch relevant documents from external sources
3. Return these documents as context to the generator (e.g., a language model like GPT-4 or Claude)

This enables the LLM to **generate responses grounded in real-time, external information**, improving accuracy and reliability.


## 🧩 Multiple Retriever Implementations

LangChain provides several out-of-the-box Retriever implementations tailored for different use cases:

- **`VectorStoreRetriever`**: The most commonly used Retriever that wraps a vector database for similarity-based search.
- **`MultiQueryRetriever`**: Executes multiple rewritten queries simultaneously to capture diverse aspects of a user’s question, improving **recall**.
- **`SelfQueryRetriever`**: Uses a language model to generate structured search queries (e.g., SQL or metadata filters) from a natural language prompt.


## ⚙️ Key Features

`from langchain_core.retrievers import BaseRetriever`

**Modular Design:**

Retrievers follow a **plugin-style modular architecture**, allowing easy customization and extension.
- Plug in **custom retrieval algorithms**
- Combine **hybrid strategies** (e.g., vector + keyword search)
- Apply **reranking techniques** for better relevance

**Async Support for High Concurrency:**

LangChain supports asynchronous retrieval with the `async_get_relevant_documents` method. This is especially useful in **high-concurrency** production scenarios such as:
Real-time Q&A bots
- Multi-user enterprise agents
- Retrieval pipelines over large corpora

**Seamless Integration with LangChain Components:**

Retrievers are natively interoperable with other LangChain modules, such as:
- **Text Splitters** – Break documents into chunks before embedding
- **Memory** – Store past interactions and retrieve based on context
- **Prompt Templates** – Dynamically incorporate retrieved content into prompts

This allows you to create powerful **chain-based** applications where retrieval, memory, and generation are tightly coupled in a single workflow.


## 📦 Common Retriever Types

[Multiple Retriever Implementations]({{ site.baseurl }}/LangChain-Retrievers/#-multiple-retriever-implementations)


### VectorStoreRetriever Example

```python
from langchain_community.vectorstores import FAISS

# Step 1: Embed your documents into vectors
retriever = FAISS.from_documents(docs, embeddings).as_retriever(
    search_type="mmr",  # Use Maximal Marginal Relevance for diversification
    search_kwargs={
        "k": 5,  # Retrieve top 5 most relevant documents
        "filter": {"category": "news"}  # Optional metadata filter
    }
)
```


#### as_retriever()

The `as_retriever()` method is a convenient utility in LangChain that allows you to **convert a vector store instance into a retriever object**. This enables seamless integration with LangChain chains, such as `RetrievalQA`, `ConversationalRetrievalChain`, and others.

```python
def as_retriever(self, **kwargs: Any) -> VectorStoreRetriever:
    """Return VectorStoreRetriever initialized from this VectorStore.

    Args:
        **kwargs: Keyword arguments to pass to the search function.
            Can include:
            search_type (Optional[str]): Defines the type of search that
                the Retriever should perform.
                Can be "similarity" (default), "mmr", or
                "similarity_score_threshold".
            search_kwargs (Optional[Dict]): Keyword arguments to pass to the
                search function. Can include things like:
                    k: Amount of documents to return (Default: 4)
                    score_threshold: Minimum relevance threshold
                        for similarity_score_threshold
                    fetch_k: Amount of documents to pass to MMR algorithm
                        (Default: 20)
                    lambda_mult: Diversity of results returned by MMR;
                        1 for minimum diversity and 0 for maximum. (Default: 0.5)
                    filter: Filter by document metadata

    Returns:
        VectorStoreRetriever: Retriever class for VectorStore.
     """
    tags = kwargs.pop("tags", None) or [] + self._get_retriever_tags()
    return VectorStoreRetriever(vectorstore=self, tags=tags, **kwargs)
```
#### Key Parameters 

| `search_type`   | Description          | Best For     | Underlying Operation (e.g., Milvus)    |
|---------------|--------------------|------------------|-----------------------------------------|
| `"similarity"`  | Standard similarity-based retrieval      | General-purpose retrieval      | `search()`      |
| `"mmr"`    | Maximal Marginal Relevance: promotes diversity in results     | Diversified QA, redundancy reduction | `max_marginal_relevance_search()`   |
| `"similarity_score_threshold"` | Retrieves results **only above a score threshold**            | High-precision retrieval (filtering noise)  | `search()` + post-filter by `score_threshold`   |

#### MMR Example

```python
mmr_retriever = vector_store.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 3,
        "fetch_k": 20,
        "lambda_mult": 0.5
    }
)
```

#### Common `search_kwargs` Parameters

| Parameter        | Type        | Default | Description                                                                 |
|------------------|-------------|---------|-----------------------------------------------------------------------------|
| `k`              | `int`       | `4`     | The number of top documents to return                                       |
| `filter` / `expr`| `dict` / `str` | `None`  | Metadata filtering conditions; used to narrow search scope                  |
| `param`          | `dict`      | `None`  | Vector DB-specific parameters (e.g., Milvus index config, metric type, etc) |


## 👀 Integration Example

### Setup

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
    collection_name="retriever_test",
    connection_args={
        "uri": "http://54.163.61.180:19530",
        "db_name": "my_database"
    }
)
```

### retriever - defauly is similarity search

```python
# Without Retriever
# query = "How to integrate database?"
# results = vector_store.similarity_search(query, k=3)


# With Retriever
retriever = vector_store.as_retriever(search_kwargs={"k": 3})

results = retriever.invoke("How to integrate database?")

for result in results:
    print(f"content: {result.page_content}\nmetadata: {result.metadata}\n")

# Output:
# content: LangChain support multiple database integrations. I am learning AI and LangChain.
# metadata: {'pk': 457482682126599167, 'source': 'kai.com/doc1'}

# content: I will learn architecture course
# metadata: {'pk': 457482682126599169, 'source': 'kai.com/doc3'}

# content: Milvus is good at vector search, the wesite of Kai is good
# metadata: {'pk': 457482682126599168, 'source': 'kai.com/doc2'}
```

### retriever - change to MMR

```python
retriever = vector_store.as_retriever(search_type="mmr",search_kwargs={"k": 3})
```



## 📈 Recall in Information Retrieval
**Recall** is a critical metric in both **information retrieval** and **machine learning**, used to evaluate how effectively a system finds *all relevant results*.

> **Recall** = (Number of Relevant Documents Retrieved) / (Total Number of Relevant Documents)

It measures the system’s ability to **find all the relevant information**, even if some irrelevant results are also included.

### Example
Imagine you're building a document retriever, and there are **100 truly relevant documents** in the dataset.

- If your system retrieves **80 of them**, then:

  ```text
  Recall = 80 / 100 = 80%
  ```

### Trade-off: Recall vs. Precision

- A **high recall** score means your system **misses fewer relevant documents**.
- However, it may **include more irrelevant documents**, which can **lower precision**.

| Metric | Foucs | Risk |
|--------|-------|------|
| Recall | Don't miss any relevant results | Might include noisy data |
| Precision | Only return relevant results | Might miss some valid entries |


A well-balanced system **optimizes both recall and precision**, depending on the use case.


### How to Improve Recall and Precision in Large Language Model Retrieval
In real-world RAG (Retrieval-Augmented Generation) systems, ensuring high **recall** (retrieving all relevant information) and **precision** (retrieving only relevant information) is crucial for generating accurate and complete answers. However, several challenges can make this difficult—especially when user queries are vague or documents use varied terminology.


#### Background: Why Retrieval Fails

- **Unclear User Queries**  
  Users may provide ambiguous or underspecified queries (e.g., "data policy" instead of "GDPR compliance").

- **Terminology Gap**  
  Documents in the corpus may use different expressions to describe the same concept (e.g., "carbon footprint" vs. "emission score").

- **Query-Expression Mismatch**  
  The user’s phrasing might not align with how the knowledge is written in the documents, leading to low overlap in embeddings or keyword matching.


#### Solution: MultiQueryRetriever
`MultiQueryRetriever` is a powerful retriever type in LangChain that improves both **recall** and **precision** by generating and executing **multiple semantically related queries** derived from the original user question.

**How It Works：**

> **Original Query** → **LLM Generates Query Variants** → **Parallel Search** → **Deduplication & Reranking** → **Final Output**

1. **Query Expansion via LLM**  
   Uses a language model (e.g., GPT-4) to generate **N reformulations** of the original query.
   - Techniques: paraphrasing, elaboration, translation, synonym expansion

2. **Multi-Query Execution**  
   Each query variant is **executed in parallel** using the underlying retriever (e.g., FAISS or Chroma).

3. **Result Merging & Deduplication**  
   Merges all returned documents, **removes duplicates**, and **reranks** for relevance.

4. **Final Document List**  
   The top-k deduplicated, reranked documents are returned to the generation model.


**Dual Enhancement Example: Boosting Both Recall & Precision**

| Metric       | Improvement |
|--------------|-------------|
| **Recall**   | ↑ +25%      |
| **Precision**| ↑ +18%      |

By reformulating the question from **multiple semantic perspectives**, `MultiQueryRetriever` ensures more **comprehensive** and **accurate** coverage of the relevant information.

#### Code Example

```python
retriever = MultiQueryRetriever.from_llm(
    retriever=base_retriever,
    llm=ChatOpenAI()
)
```

#### Typical Scenarios Where MultiQueryRetriever Excels

1. **Terminology Mismatch:**
Users and documents often describe the same concept using **different terms**.
- User Query: `"How to renew an SSL certificate?"`  
- Document Text: `"TLS certificate renewal instructions"`

2. **Ambiguous or Vague Queries:**
Short or generic user questions lack enough context for accurate retrieval.
- Query: `"How to backup a database?"`  
- Actual Document: `"Steps for implementing a disaster recovery plan for MySQL"`

3. **Multilingual or Code-Switched Queries:**
Mixed-language input is common in **technical searches**, especially in bilingual environments.
- Query: `"如何配置Nginx的SSL证书 with Let's Encrypt?"`  
- Document: `"How to install SSL on Nginx using Let's Encrypt"`

4. **Domain-Specific Semantic Variation:**
In fields like **medicine**, the same symptom may be described in diverse ways.
- Query: `What to do if you have stomach pain?`
- Document: `How to deal with upper abdominal discomfort?`

#### AI Doctor Code Example

```python
import os
from langchain_milvus import Milvus
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_community.document_loaders import TextLoader
from langchain.retrievers.multi_query import MultiQueryRetriever
from langchain_text_splitters import RecursiveCharacterTextSplitter

import logging

# Set the basic configuration for logging
logging.basicConfig()
# Set the logging level for the MultiQueryRetriever
logging.getLogger("langchain.retrievers.multi_query").setLevel(logging.INFO)

# replace with your actual key
os.environ["OPENAI_API_KEY"] = "*"


# Step 1: Load source data
# Load text data from local file via TextLoader
loader = TextLoader("data/qa.txt")
data = loader.load()  # Load data into a variable

# Step 2: transform text into smaller chunks
# Initialize the text splitter, splitting the text into smaller chunks
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=100, chunk_overlap=10)
# Split the text into smaller chunks
splits = text_splitter.split_documents(data)

# Step 3: Embed - Initialize the embedding model
openai_embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small",
    max_retries=3,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

#  Step 4: Store - Initialize the vector database with embedding model
vector_store = Milvus.from_documents(
    documents=splits,
    embedding=openai_embeddings,
    collection_name="mulit_retriever",
    connection_args={
        "uri": "http://54.163.61.180:19530",
        "db_name": "my_database"
    }
)


# Step 5: Initialize the MultiQueryRetriever from LLM Model
llm = ChatOpenAI(  # Initialize the LLM model
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

retriever_from_llm = MultiQueryRetriever.from_llm( # Initialize the MultiQueryRetriever
    retriever=vector_store.as_retriever(),
    llm=llm
)

# Step 6: Invoke the LLM based on the question
question = "I don't know why Kai got cramps."

# Generate multiple questions based on original question
results = retriever_from_llm.invoke(question)

len(results)

for result in results:
    print(f"content: {result.page_content}\nmetadata: {result.metadata}\n")


# INFO:langchain.retrievers.multi_query:
# Generated queries: 
# ['What could be the possible reasons for Kai experiencing cramps?  ', 
# 'Can you explain the common causes of cramps that Kai might be facing?  ', 
# 'What factors might contribute to the cramps that Kai is having?']

# content: 93.
# Patient: How to prevent leg cramps during pregnancy?
# AI Doctor: Suggestions:Take calcium supplements (1000mg/day), Soak feet in warm water before bed, Elevate legs while sleeping, Avoid standing or sitting too long.
# metadata: {'pk': 457482682126599242, 'source': 'data/qa.txt'}
# 
# More...
```




#### MultiQueryRetriever.from_llm()

```python
def from_llm(
    cls,
    retriever: BaseRetriever,
    llm: BaseLanguageModel,
    prompt: BasePromptTemplate = DEFAULT_QUERY_PROMPT,
    parser_key: Optional[str] = None,
    include_original: bool = False,
) -> "MultiQueryRetriever":
    """Initialize from llm using default template.

    Args:
        retriever: retriever to query documents from
        llm: llm for query generation using DEFAULT_QUERY_PROMPT
        prompt: The prompt which aims to generate several different versions
            of the given user query
        include_original: Whether to include the original query in the list of
            generated queries.

    Returns:
        MultiQueryRetriever
    """

# Default prompt
DEFAULT_QUERY_PROMPT = PromptTemplate(
    input_variables=["question"],
    template="""You are an AI language model assistant. Your task is 
    to generate 3 different versions of the given user 
    question to retrieve relevant documents from a vector  database. 
    By generating multiple perspectives on the user question, 
    your goal is to help the user overcome some of the limitations 
    of distance-based similarity search. Provide these alternative 
    questions separated by newlines. Original question: {question}""",
)
```



<br>




---

🚀 Stay tuned for more insights into LangChain and Vector Database!



