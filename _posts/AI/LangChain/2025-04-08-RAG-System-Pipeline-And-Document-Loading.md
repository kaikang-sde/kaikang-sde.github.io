---
title: RAG System Pipeline & Document Loading (Loaders)
date: 2025-04-08
order: 1
categories: [AI, LangChain]
tags: [LangChain, LLM, RAG, Retrieval-Augmented Generation, Loader]
author: kai
---


# 🚀 RAG System Pipeline & Document Loading (Loaders)
Retrieval-Augmented Generation (RAG) is an advanced framework that **combines traditional information retrieval techniques with the power of Large Language Models (LLMs)** to generate more **accurate and context-aware responses**.

**RAG–LLM Interaction Architecture Diagram:**

![RAG–LLM Interaction Architecture Diagram](/assets/img/posts/AI/LangChain/RAG-LLMInteractionArchDiagram.png)

## 🔁 RAG Data Pipeline Overview

Retrieval-Augmented Generation (RAG) enhances language model output by grounding it with external, up-to-date, and domain-specific knowledge. This is made possible by **a well-defined data flow pipeline that integrates document processing and vector-based retrieval**.

```text
Raw Data → Data Loader → Preprocessing → Vectorization → Vector Storage → RAG
               ↗              ↗              ↗
            PDF           Cleaning     Embedding Model
         Databases         Chunks
         Websites     Metadata tagging
```

![Data Connection](/assets/img/posts/AI/LangChain/DataConnection.jpg)


## 📄 Document Loaders in RAG Systems
**Document Loaders** are the **first stage** in a Retrieval-Augmented Generation (RAG) pipeline.  
They are responsible for **extracting content** from various raw data sources—both local and online—and converting it into a **structured and unified document format** that can be processed downstream.

**Core Responsibilities:**

| Task                     | Description                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| **Data Source Adapter**  | Supports integration with file systems, URLs, APIs, and cloud platforms     |
| **Format Normalization** | Unifies text content and metadata into a common document object             |
| **Metadata Tagging**     | Automatically adds source, page number, or section headers as metadata      |
| **Compatibility Bridge** | Converts data into formats compatible with chunking, embedding, and retrieval |


## 📚 Document Loaders in LangChain
LangChain provides a rich set of **document loaders** that support various file types and data sources. These loaders act as the **entry point** for converting raw data into structured `Document` objects used in RAG pipelines.

🔗 [LangChain Document Loaders Integration](https://python.langchain.com/docs/integrations/document_loaders/)  


### Commonly Used Loaders

LangChain offers various loader classes under `langchain_community.document_loaders`. 

| Loader Class               | Supported Format       | Description / Use Case                                     |
|---------------------------|------------------------|------------------------------------------------------------|
| `TextLoader`              | `.txt`, Markdown       | Load plain text or Markdown documents                      |
| `PyPDFLoader`             | `.pdf`                 | Extract text from PDF files (e.g., manuals, reports)       |
| `Docx2txtLoader`          | `.docx`                | Load Microsoft Word files                                  |
| `UnstructuredHTMLLoader`  | `.html`                | Extract content from static HTML pages                     |
| `CSVLoader`               | `.csv`                 | Load tabular data from CSV files                           |
| `JSONLoader`              | `.json`                | Parse structured data from JSON documents                  |
| `SeleniumURLLoader`       | Dynamic Web URLs       | Use Selenium to load JavaScript-rendered web pages         |
| `WebBaseLoader`           | Static Web URLs        | Load and parse raw content from basic HTML pages           |


### LangChain `BaseLoader` Interface 
In LangChain, document loaders are standardized through an abstract interface called **`BaseLoader`**. It ensures all custom and built-in loaders follow a consistent structure and support **memory-efficient lazy loading**.

```python
from abc import ABC
from langchain_core.documents import Document

class BaseLoader(ABC):
    """Interface for Document Loader.

    Implementations should implement the lazy-loading method using generators
    to avoid loading all Documents into memory at once.

    `load` is provided just for user convenience and should not be overridden.
    """

    def load(self) -> list[Document]:
        """Load data into Document objects."""
        return list(self.lazy_load())
```


### LangChain `Document` Object 
In LangChain, raw data from various sources—such as files, APIs, databases, or websites—must be **converted into a standard document format** for further processing, especially in Retrieval-Augmented Generation (RAG) pipelines.


LangChain provides built-in and custom **document loaders**, which convert raw data into a list of structured `Document` objects.

Each loader class implements a `.load()` method that returns a `List[Document]`, each `Document` include:
- `page_content`
- `metadata`

```python
class Document(BaseMedia):
    """Class for storing a piece of text and associated metadata.

    Example:

        .. code-block:: python

            from langchain_core.documents import Document

            document = Document(
                page_content="Hello, world!",
                metadata={"source": "https://example.com"}
            )
    """

    page_content: str
    """String text."""
    type: Literal["Document"] = "Document"
```

## 📂 LangChain Document Loaders: Categories & Common Types
LangChain offers a wide range of **document loaders** to support data ingestion from diverse sources. These loaders transform raw data (files, web content, databases, etc.) into structured `Document` objects used throughout the LangChain pipeline.

### File Loaders

| Loader Type           | Description                                                  |
|-----------------------|--------------------------------------------------------------|
| `TextLoader`          | Loads plain text files (`.txt`)                              |
| `CSVLoader`           | Parses `.csv` files row-by-row into `Document` objects       |
| `PyPDFLoader`         | Extracts content and metadata from PDF files (via PyPDF2)    |
| `Docx2txtLoader`      | Reads Microsoft Word `.docx` documents                       |
| `UnstructuredFileLoader` | General-purpose parser for various unstructured file types  |


### Web Loaders

| Loader Type           | Description                                                    |
|-----------------------|----------------------------------------------------------------|
| `WebBaseLoader`       | Fetches and extracts readable text from static web pages       |
| `SeleniumLoader`      | Supports dynamic pages that require JavaScript rendering       |


### Database Loaders

| Loader Type           | Description                                           |
|-----------------------|-------------------------------------------------------|
| `SQLDatabaseLoader`   | Executes SQL queries and loads the result into documents |
| `MongoDBLoader`       | Loads documents from MongoDB collections               |


### Custom or Other Loaders
LangChain is **extensible** and allows developers to create their own loaders by inheriting from the base class.<br>
`from langchain_core.document_loaders import BaseLoader`





<br>



---

🚀 Stay tuned for more insights into LangChain!



