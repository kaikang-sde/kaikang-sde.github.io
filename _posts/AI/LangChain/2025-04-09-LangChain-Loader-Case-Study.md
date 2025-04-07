---
title: LangChain Loader Case Study
date: 2025-04-07
order: 3
categories: [AI, LangChain]
tags: [LangChain, LLM, Loader, TextLoader, CSVLoader, JSONLoader, PyPDFLoader, Docx2txtLoader, WebBaseLoader]
author: kai
---


# 🚀 LangChain Loader Case Study
LangChain provides a powerful set of **document loaders** that unify the process of ingesting data from various sources (files, web, databases) into a structured format for downstream LLM applications (RAG, chatbots, search agents).


## 📋 TextLoader for Plain Text Files
**Scenario:** Load Local `.txt` Documents
- In many enterprise or research settings, raw data often starts as plain text files. LangChain’s `TextLoader` simplifies converting `.txt` files into structured `Document` objects.

**Common Parameters:**
- encoding: File encoding format (default is "utf-8")
- autodetect_encoding: Automatically detect encoding (helpful for Chinese or special characters)

```python
from langchain_community.document_loaders import TextLoader

# 1. Initialize loader
loader = TextLoader("data/test.txt", autodetect_encoding=True)

# 2. Load documents
documents = loader.load()

# 3. Explore results
print(documents)                                # Full document object(s)
# Output
# [Document(metadata={'source': 'data/test.txt'}, page_content='Case 1: Intelligent Customer Service System\nTraditional #Challenge:\nThe customer service knowledge base is frequently updated, making it difficult for the model to stay in sync in #real time.\nUser Query:\n“Is this KK Classroom mobile phone suitable for elderly people?”\nSystem Retrieval:\nKnowledge #entries related to target user groups for the product.\nGenerated Response:\n“This product is suitable for adults aged 18 and #above, including middle-aged and elderly users.”\n')]

print(len(documents))                           # Number of documents -> Output: 1
print(documents[0].page_content[:100])          # Preview first 100 characters -> Output: CCase 1: Intelligent Customer Service System   Traditional Challenge:  The customer service knowledge ba
print(documents[0].metadata)                    # Output: {'source': 'data/test.txt'}
```



## 📊 CSVLoader for CSV Files
**Scenario:** Load Local CSV Data with `CSVLoader`
- `CSVLoader` is a built-in document loader from `langchain_community` designed to read `.csv` files and convert **each row into a structured `Document` object**.
- This is especially useful for feeding tabular data (such as product info, logs, survey results) into LLM pipelines.

```python
from langchain_community.document_loaders import CSVLoader

# Load CSV with comma as delimiter
loader = CSVLoader("data/test.csv", csv_args={"delimiter": ","})
documents = loader.load()

# Convert each line into a Document object, and include the line number in the metadata, 
print(len(documents))  # Output: 8

# Inspect the first document's metadata and content
print(documents[0].metadata)       # {'source': 'data/test.csv', 'row': 0}
print(documents[0].page_content)   # "Name: Alice, Age: 25, Role: Engineer"


# Iterate over all documents
for doc in documents:
    print(doc.metadata)
    print(doc.page_content)
    print("**" * 10)
```

### Load CSV with Custom Column Names
Sometimes CSV files **do not include headers**, or you want to **override existing headers** for consistency or downstream processing.
LangChain’s `CSVLoader` supports **custom column name definition** via the `csv_args` parameter.

```python
from langchain_community.document_loaders import CSVLoader

# Manually define column names for the CSV rows
loader = CSVLoader(
    "data/test.csv",
    csv_args={"fieldnames": ["Product Name", "Quantity Sold", "Customer Name"]}
)

# Load and convert each row into a Document object
documents = loader.load()

# Total number of documents
print(len(documents))

# View metadata and content of a sample row
print(documents[1].metadata)     # e.g., {'source': 'data/test.csv', 'row': 1}
print(documents[1].page_content) # Output: Product Name: Widget, Quantity Sold: 50, Customer Name: Alice
```


## 📦 JSONLoader for JSON Files
**Scenario:** Load Local JSON Files with `JSONLoader`
- LangChain provides `JSONLoader` for structured data ingestion from JSON files. This is especially useful when dealing with nested data, APIs, or log files in `.json` format.

**Core Parameters:**

| Parameter        | Type       | Required ✅ | Description                                                                 |
|------------------|------------|-------------|-----------------------------------------------------------------------------|
| `file_path`      | `str`      | ✅          | Path to the JSON file.                                                     |
| `jq_schema`      | `str`      | ✅          | **jq-style query** to define what data to extract.                         |
| `content_key`    | `str`      | ❌          | If extracting an object, defines which field to use as `page_content`.     |
| `metadata_func`  | `Callable` | ❌          | Custom function to generate metadata.                                      |
| `text_content`   | `bool`     | ❌          | If `True` (default), forces the extracted value into a string.             |

### `jq_schema` Is Required
Unlike flat CSV or TXT, JSON often includes **deeply nested** structures. The `jq_schema` field leverages [jq-style syntax](https://stedolan.github.io/jq/manual/) to:
- Extract specific nodes
- Filter or transform objects
- Combine multiple fields
- `pip install jq`

### Common `jq_schema` Examples

| Scenario             | `jq_schema`                          | Description                                  |
|----------------------|--------------------------------------|----------------------------------------------|
| Extract root array   | `.[]`                                | For flat array JSON                          |
| Nested field         | `.data.posts[].content`              | Pull nested content from `data > posts`      |
| With filter          | `.users[] | select(.age > 18)`       | Get users where `age > 18`                   |
| Merge multiple keys  | `{name: .username, email: .contact}` | Return a combined object                     |


```python
from langchain_community.document_loaders import JSONLoader
loader = JSONLoader(
    file_path="data/test.json",
    jq_schema=".articles[]",  # Extract each element in the articles array via jq_schema
    content_key="content"     # Extract content as text
)

docs = loader.load() # Returns a list of documents
print(len(docs))
print(docs[0])

# Output:
# page_content='This is the first article...' metadata={'source': '/Users/kai/MyProjects/ai-langchain/data/test.json', 'seq_num': 1}
```

## 📑 PyPDFLoader for PDF Document
**Scenario:** Load Local PDF Data with `PyPDFLoader`
- `PyPDFLoader` is a specialized document loader in **LangChain** designed to parse and extract text from **PDF files**.
    - Automatically splits PDF into **per-page** `Document` objects.
    - Includes useful metadata (e.g., page number, source file).
    - Ideal for **multi-page** report processing, contracts, or scanned documentation.
    - `pip install pypdf`


```python
from langchain_community.document_loaders import PyPDFLoader

# Step 1: Load the PDF file
loader = PyPDFLoader("data/test.pdf")

# Step 2: Parse and split into pages
pages = loader.load()  # Returns a list of Document objects

# Step 3: Inspect results
print(f"Total pages: {len(pages)}") # Total pages: 68

# Preview first page
page_content = pages[0].page_content # Get the content of the first page
metadata = pages[0].metadata  # Get the metadata of the first page

print(f"First Page Content:\n{page_content[:200]}...")  # Show first 200 characters
print(f"Metadata: {metadata}")  # Metadata: {'source': 'data/test1.pdf', 'page': 0}
```

### Load Specific Page Ranges
To optimize performance or target specific sections, you can specify page indices when calling `load()`.

> ⚠️ Note: Page indexing starts from **0**, so `pages[1]` refers to **Page 2** in the PDF.

```python
from langchain_community.document_loaders import PyPDFLoader

# Load selected pages (Page 2 to Page 4)
loader = PyPDFLoader("data/test.pdf")
pages = loader.load([1, 2, 3])
print(f"Loaded {len(pages)} pages.")
```

### Merge All Page Content into a Single String

```python
# Merge text content from all loaded pages
full_text = "\n\n".join([page.page_content for page in pages])
print(f"Merged full text length: {len(full_text)} characters") 
# Output: Merged full text length: 84542 characters
```

### Common Issues and Solutions with PDF Loading
When working with `PyPDFLoader`, you may encounter some common pitfalls. Here are troubleshooting tips and fixes:

#### Issue 1: PDF Cannot Be Loaded or Text Is Empty
- The PDF may be **scanned images** (non-selectable text).
- The PDF is **encrypted/password-protected**.

**Solutions:**
- Use **OCR tools** (Optical Character Recognition) to extract text:
  - Libraries: `pytesseract` + `pdf2image`
  - Example: Convert pages to images and apply OCR.


#### Issue 2: Text Splitting Is Suboptimal
- Splits mid-sentence or cuts context in half.

**Solutions:**
Use a **customized text splitting strategy** via RecursiveCharacterTextSplitter.

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    separators=["\n\n", "\n", "."],  # prioritize paragraphs, then lines
    chunk_size=500,
    chunk_overlap=100
)
```

### Batch Processing Multiple PDFs
When working with a large collection of PDF files (e.g., document repositories, contract archives, academic papers), it's often necessary to **load and parse multiple PDFs** at once.

LangChain makes this easy with a simple directory traversal and `PyPDFLoader`.

```python
import os
from langchain_community.document_loaders import PyPDFLoader

pdf_folder = "docs/"  # Folder path
all_pages = []  # Store all Document objects

for filename in os.listdir(pdf_folder):
    if filename.endswith(".pdf"):
        filepath = os.path.join(pdf_folder, filename)
        loader = PyPDFLoader(filepath)
        pages = loader.load()  # Load data into Document objects. Return: list[Document]
        all_pages.extend(pages)
```

### Extracting Text from Images in PDF Files
While `PyPDFLoader` in LangChain is great for **text extraction**, it **does not support OCR** by default. This means if your PDF contains **scanned images**, **charts**, or **embedded tables**, the textual content might be missing.

To solve this, we need to integrate **OCR (Optical Character Recognition)** tools like `RapidOCR-ONNXRuntime`.

### Limitation of `PyPDFLoader`

- ✅ Extracts **text-based PDFs**.
- ❌ Cannot handle **scanned images**, tables, or charts without additional OCR support.
- ⚙️ Requires `extract_images=True` and **third-party OCR integration** for full content extraction.

### Recommended OCR Tool: `RapidOCR-ONNXRuntime`
**RapidOCR-ONNXRuntime** is a **lightweight OCR (Optical Character Recognition) toolkit** built on top of the **ONNX Runtime inference engine**. It is designed for **efficient, cross-platform deployment**, and optimized for speed and low resource consumption.

- **Cross-platform**: Windows, Linux, macOS, even mobile & embedded devices.
- **Multilingual OCR**: Supports Chinese, English, Japanese, Korean, etc.
- **Lightweight**: Model size ~few MBs.
- **Integrated Pre/Post-Processing**:
  - Pre: binarization, skew correction, cropping.
  - Post: text cleanup, formatting.
- `pip install rapidocr-onnxruntime`

| Tool                      | Engine         | Speed  | Accuracy | Language Support | Dependency | Use Case                         |
|---------------------------|----------------|--------|----------|------------------|------------|----------------------------------|
| **RapidOCR-ONNXRuntime**  | ONNX Runtime   | ⭐⭐⭐⭐   | ⭐⭐⭐     | Multilingual     | Light      | Cross-platform, lightweight OCR |
| Tesseract                 | Custom engine  | ⭐⭐     | ⭐⭐      | Multilingual     | Heavy      | Legacy projects                 |
| EasyOCR                   | PyTorch        | ⭐⭐     | ⭐⭐⭐     | Multilingual     | Medium     | Rapid prototyping               |
| Microsoft Read API        | Cloud service  | ⭐⭐⭐⭐   | ⭐⭐⭐⭐    | Multilingual     | None       | Enterprise cloud-scale use      |

```python
from langchain_community.document_loaders import PyPDFLoader

loader = PyPDFLoader("data/pdf-img.pdf", extract_images=True)
pages = loader.load()
print(pages[0].page_content)
```


## 📂 Docx2txtLoader for Word Files
**Scenario:** Load Local `.docx` Documents
- **`Docx2txtLoader`** is a document loader in the LangChain ecosystem designed to extract **raw textual content** from Microsoft Word `.docx` files.
- It is ideal for processing **Word** reports, project documents, or any `.docx` files that need to be converted into structured `Document` objects for downstream tasks like retrieval, embedding, or LLM-based generation.
- **Only `.docx`** format is supported. Legacy `.doc` files must be converted first.
- `pip install docx2txt`


| Feature               | Description                                             |
|------------------------|---------------------------------------------------------|
| 📄 Format Supported     | `.docx` (Microsoft Word Open XML Format)               |
| 🔍 Extracts             | Paragraphs, bullet points, table text                  |
| ❌ Ignores              | Styling details (fonts, colors, alignment, etc.)       |
| 📦 Output              | `Document` objects (with `page_content` and `metadata`)|


```python
from langchain_community.document_loaders import Docx2txtLoader

# Load a .docx file
loader = Docx2txtLoader("data/test1.docx")

# Parse the document into LangChain Document objects
documents = loader.load()

# Inspect the results
print(documents[0].page_content[:300])  # Preview first 300 characters
print(documents[0].metadata)            # {'source': 'data/test1.docx'}
```

### Batch Loading `.docx` Files 
When working with multiple Word documents in a folder, you can use `Docx2txtLoader` to **automatically load all `.docx` files**, extract their text content, and convert them into LangChain-compatible `Document` objects for downstream processing.

```python
from langchain_community.document_loaders import Docx2txtLoader
import os

# Define the folder path containing .docx files
folder_path = "data/"
all_docs = []

# Iterate through all .docx files in the folder
for filename in os.listdir(folder_path):
    if filename.endswith(".docx"):
        file_path = os.path.join(folder_path, filename)
        loader = Docx2txtLoader(file_path)
        # Load and merge documents into one list
        all_docs.extend(loader.load())
        print(f"Loaded file: {filename}")

# Inspect results
print(f"Total documents loaded: {len(all_docs)}")
print(all_docs[0].page_content[:300])  # Preview first 300 characters
print(all_docs[0].metadata)            # Metadata includes file path
```


### Common Issues & Solutions When Using Docx2txtLoader

#### Problem 1: `.doc` File Loading Error

**Cause:**  
`docx2txt` only supports the modern `.docx` file format. Legacy `.doc` files are not compatible.

**Solution:**  
Use Microsoft Word or a compatible tool (e.g., LibreOffice) to **convert `.doc` files to `.docx`** format by using "Save As".

#### Problem 2: Images or Charts Not Extracted

**Cause:**  
`Docx2txtLoader` is designed to **extract only plain text content**. Images, charts, and embedded media are ignored.

**Solutions:**

- For **image extraction**, consider using [`python-docx`](https://python-docx.readthedocs.io/) to manually extract embedded images.
- For **OCR on embedded visuals**, integrate with tools like `Tesseract OCR` or `RapidOCR` to process document images.
- For tables and complex layouts, use more advanced layout-aware tools like `Unstructured` or `pdfplumber` (if converted to PDF).


## 🌐 WebBaseLoader for Web Page
- `WebBaseLoader` is a built-in document loader in **LangChain** designed to fetch and parse **static web page content**.
- It performs HTTP GET requests to retrieve the **HTML**, and then uses `BeautifulSoup` to extract **cleaned text** (strips out tags, scripts, styles, etc.).
- Converts webpage text into LangChain `Document` objects.
- Automatically includes metadata (e.g., source URL).
- Ideal for **news**, **blogs**, **product pages**, or **public documentation**.
- `pip install beautifulsoup4`          
- `pip install requests`          
- See more details: [`LangChain - WebBaseLoader`](https://python.langchain.com/docs/integrations/document_loaders/web_base/)


### Common Use Cases

| Scenario             | Description                                          |
|----------------------|------------------------------------------------------|
| Knowledge Retrieval  | Crawl public docs/blogs into an enterprise chatbot  |
| Sentiment Monitoring | Extract real-time news, reviews, or social content  |
| Competitive Research | Track competitor product descriptions/prices        |
| SEO Aggregation      | Scrape static content for analysis                  |

### Target Web Page Requirements:
1. **No JavaScript Rendering**
   - `WebBaseLoader` is designed for **static HTML pages only**.
   - If the content is rendered dynamically via JavaScript (e.g., SPAs or React/Vue pages), it **won’t be captured**.
   - For such cases, `SeleniumURLLoader` can be used, but it's considered **heavyweight and less stable**. Use it **only when necessary**.

2. **No Anti-Crawling Mechanisms**
   - Some websites may block bots or automated requests using:
     - IP rate limiting
     - CAPTCHA
     - Missing or suspicious `User-Agent`
   - **Solution**: Add appropriate **request headers** or configure **proxy servers** to simulate real browsers.

3. **Dynamic Content Extraction Still Requires Custom Logic**
    - Even when using headless browsers (like Selenium), different websites structure their content differently.
	- You will often need to write custom parsers or selectors to extract the actual text/data you need.

```python
import os

# Set USER_AGENT to avoid request rejections or missing headers
# ⚠️ IMPORTANT: Set this BEFORE importing WebBaseLoader
os.environ['USER_AGENT'] = (
    'Mozilla/5.0 (Windows NT 14.0; Win64; x64) '
    'AppleWebKit/567.36 (KHTML, like Gecko) '
    'Chrome/58.0.444.11 Safari/337.3'
)

from langchain_community.document_loaders import WebBaseLoader

# Initialize loader with target URLs (can be a single URL or a list)
urls = ["https://kaikang-sde.github.io/posts/Introduction-to-LLMs/"]
loader = WebBaseLoader(urls)

# Load documents (returns a list of Document objects)
docs = loader.load()

# Inspect the result
print(f"Extracted text length: {len(docs[0].page_content)} characters")
print(f"Preview of first 200 characters:\n{docs[0].page_content[:200]}...")
print(f"Metadata: {docs[0].metadata}")
```

### Batch Load Multiple Web Pages
```python
import os

# Set USER_AGENT before importing WebBaseLoader to avoid header errors
os.environ['USER_AGENT'] = (
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) '
    'AppleWebKit/537.36 (KHTML, like Gecko) '
    'Chrome/58.0.3029.110 Safari/537.3'
)

from langchain_community.document_loaders import WebBaseLoader

# Define multiple target URLs
urls = [
    "https://kaikang-sde.github.io/posts/Introduction-to-LLMs/",              # Baidu News
    "https://kaikang-sde.github.io/posts/Python-List-and-Common-Operations/"    # Baidu Tieba (Forum)
]

# Initialize the loader
loader = WebBaseLoader(urls)

# Load all documents
docs = loader.load()

# Display results
print(f"Total documents loaded: {len(docs)}") # Total documents loaded: 2
print("Document sources:")
for doc in docs:
    print(f"- {doc.metadata['source']}")

# Document sources:
# - https://kaikang-sde.github.io/posts/Introduction-to-LLMs/
# - https://kaikang-sde.github.io/posts/Python-List-and-Common-Operations/
```



<br>



---

🚀 Stay tuned for more insights into LangChain!



