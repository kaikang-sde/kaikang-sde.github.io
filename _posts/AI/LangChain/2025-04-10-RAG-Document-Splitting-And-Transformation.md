---
title: RAG Document Splitting & Transformation
date: 2025-04-10
categories: [AI, LangChain]
tags: [LangChain, LLM, RAG, Splitting, Transformation, CharacterTextSplitter, RecursiveCharacterTextSplitter]
author: kai
permalink: /posts/ai/langchain/rag-document-splitting-and-transformation/
---


# 🚀 RAG Document Splitting & Transformation

In Retrieval-Augmented Generation (RAG) systems, **transforming raw documents into structured, semantically meaningful chunks is essential**. This process ensures downstream components like **vector databases** and **language models can process and understand content** efficiently.

[RAG Data Pipeline Overview]({{ site.baseurl }}/posts/ai/langchain/rag-system-pipeline-document-loading/#-rag-data-pipeline-overview)


## 🚨 Why Do We Need Document Transformers?

### 1. Model Input Limitations
Large Language Models (LLMs) have strict token limits:
- **GPT-4** supports up to 32K tokens
- **Claude 3** allows up to 200K tokens

Naively feeding large documents directly into LLMs will cause truncation, token overflow, or API errors.

### 2. Uneven Information Density
Key insights may be scattered throughout lengthy documents. Without splitting, important context could be lost or diluted.

### 3. Format Incompatibility
Documents come in diverse formats—PDF, HTML, Markdown, code files—each with different structural characteristics. These must be normalized before being used.


## 🧠 Document Transformers

**Document Transformers** are core components in LangChain’s document pipeline. They **take raw content** and **apply transformations** to make it **suitable for downstream tasks** like vectorization and RAG.

### Core Responsibilities:
- **Text Splitting**: Break content into semantically coherent chunks that fit model constraints
- **Noise Removal**: Strip special characters, navigation links, or boilerplate
- **Metadata Enrichment**: Add useful context like file source, page numbers, timestamps

### Key Operations

| Operation         | Description                                                                 |
|-------------------|-----------------------------------------------------------------------------|
| `Chunking`        | Split text by length or semantic boundaries to avoid splitting sentences   |
| `Cleaning`        | Remove ads, navigation bars, special symbols, or malformed text            |
| `Metadata Tagging`| Add source info, creation time, authorship, etc., for traceability         |


### Benefits

- **Preserve Semantic Integrity**: Prevent logical breaks in sentence/paragraph structure
- **Token-Aware Segmentation**: Keep chunk sizes within LLM-compatible token limits
- **Improve Vector Quality**: Semantic chunking leads to better embeddings and retrieval accuracy


### Common Issues & How Transformation Solves Them

| Issue Type           | Raw Document Example      | Pre-Transformation Problem                     | Post-Transformation Effect                              |
|----------------------|---------------------------|--------------------------------------------------|----------------------------------------------------------|
| Oversized Content    | 500-page legal contract   | Causes LLM input overflow, API failures         | Split into context-aware paragraphs                      |
| Fragmented Information | Product manuals (PDF)    | Scattered technical specs across pages          | Reorganized by feature or function                       |
| Noisy Content         | Web-scraped HTML          | Contains ads, menus, scripts                    | Extracted clean main body text                           |
| Mixed Format Content  | Code repository docs      | Markdown + code snippets intermixed             | Separated code blocks and explanatory text               |


## 🔧 Core Class: TextSplitter
The `TextSplitter` class in LangChain serves as a **base interface** and does **not implement `split_text()` directly(Abstract method)**. Instead, concrete splitter implementations (like `RecursiveCharacterTextSplitter`, `CharacterTextSplitter`, `MarkdownHeaderTextSplitter`, etc.) define their own splitting strategies.


```python
from langchain_text_splitters import TextSplitter

class TextSplitter(BaseDocumentTransformer, ABC):
    """Interface for splitting text into chunks."""

    def __init__(
        self,
        chunk_size: int = 4000,
        chunk_overlap: int = 200,
        length_function: Callable[[str], int] = len,
        keep_separator: Union[bool, Literal["start", "end"]] = False,
        add_start_index: bool = False,
        strip_whitespace: bool = True,
    ) -> None:
        ...
```

### Method Flow Overview
The typical execution chain for document splitting is:<br>
`split_documents() ➝ create_documents() ➝ split_text()`

| Method             | Purpose                                                                 |
|--------------------|-------------------------------------------------------------------------|
| `split_text()`     | Core method to split a single raw string into a list of text chunks.     |
| `create_documents()` | Wraps `split_text()` and binds **user-provided metadata** to each chunk. |
| `split_documents()`  | Operates on a list of `Document` objects, automatically **inherits and propagates** metadata during splitting. |

### Comparison of TextSplitter Methods

| Method              | Input Type            | Output Type        | Metadata Handling         | Typical Use Case                                              |
|---------------------|-----------------------|---------------------|----------------------------|----------------------------------------------------------------|
| `split_text()`      | Single `str`          | `List[str]`         | ❌ No metadata             | Basic string splitting without metadata binding               |
| `create_documents()`| `List[str]`           | `List[Document]`    | ✅ Manual metadata required| Convert raw text into `Document` objects with custom metadata |
| `split_documents()` | `List[Document]`      | `List[Document]`    | ✅ Metadata auto-preserved| Split preloaded documents (e.g. from PDF, Word, etc.)         |

> 🧠 Tip: Use `split_documents()` when working with outputs from LangChain loaders, as it simplifies chunking while preserving useful metadata like file source, page number, etc.


### chunk_size
In Text Splitters, `chunk_size` defines the **maximum length of each text chunk**—either in characters or tokens—depending on the splitter's configuration.

**Purpose:**

- Prevents chunks from exceeding the token limits of LLMs (e.g., GPT-4, Claude).
- Controls granularity of retrieval:  
  - **Smaller chunks** → better precision but may lack context.  
  - **Larger chunks** → better context retention but risk missing specific details.

#### Example

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

# Set chunk size to 100 characters, with a 20-character overlap between chunks
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=100,
    chunk_overlap=20
)

# Example input text
text = "Python is an interpreted language suitable for rapid development. It supports object-oriented programming and has a clean syntax."

# Resulting chunks (pseudo-output)
[
  "Python is an interpreted language suitable for rapid development.",
  "development. It supports object-oriented programming and has a clean syntax."
]
```

### chunk_overlap
`chunk_overlap` specifies the **number of overlapping characters (or tokens)** between adjacent chunks. It helps preserve context across chunks when splitting text.

**Purpose:**
Avoids cutting important information mid-sentence or mid-paragraph, ensuring smoother semantic continuity and improved downstream performance in RAG or summarization tasks.
> This ensures that critical context from the end of one chunk is preserved at the start of the next.


```python
from langchain_text_splitters import CharacterTextSplitter

# Config with 50-character chunks and 10-character overlap
text_splitter = CharacterTextSplitter(chunk_size=50, chunk_overlap=10)

text = "Deep learning requires large datasets and computing power. CNNs perform well in image processing."

# Hypothetical output chunks:
[
  "Deep learning requires large datasets and computing pow",
  "computing power. CNNs perform well in image processing."
]
```

#### Token-Based vs. Character-Based Overlap
When using LangChain's `TextSplitter`, it's important to understand whether you're performing **token-based** or **character-based** chunking. This affects how your text is split and how overlapping content is handled between chunks.

| Feature | Token-Based Overlap | Character-Based Overlap |
|--------|---------------------|--------------------------|
| **Unit of Measurement** | Tokens (used by LLMs like GPT-4, Claude) | Raw characters (letters, symbols, etc.) |
| **Length Function** | Uses a tokenizer (e.g., tiktoken) | Uses Python’s built-in `len()` |
| **Typical Classes** | `TokenTextSplitter`, `RecursiveCharacterTextSplitter` with tokenizer | `CharacterTextSplitter`, or default `RecursiveCharacterTextSplitter` |
| **Use Case** | When adhering to LLM token limits | Lightweight, general-purpose splitting |
| **Customization** | Requires defining `length_function` using a tokenizer | Uses `length_function=len` by default |


#### Token-Based Example (using tiktoken)
If `chunk_size = 1024` and `chunk_overlap = 128`, and the total text is 2560 tokens, the chunks will be split as follows:

- **Chunk 1**: tokens `1 – 1024`
- **Chunk 2**: tokens `897 – 1920` (128-token overlap with chunk 1)
- **Chunk 3**: tokens `1793 – 2560` (128-token overlap with chunk 2)


```python
from langchain_text_splitters import RecursiveCharacterTextSplitter
from tiktoken import encoding_for_model

# Define token-based length function
encoding = encoding_for_model("gpt-3.5-turbo")
def tiktoken_len(text):
    return len(encoding.encode(text))

# Token-aware splitter
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    length_function=tiktoken_len
)
```

#### Character-Based Example (default behavior)
```python
from langchain_text_splitters import CharacterTextSplitter

# Simple character splitter
splitter = CharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    length_function=len  # Optional (default)
)
```


## 💡 Two Common Text Splitters
Text splitting is a key preprocessing step in Retrieval-Augmented Generation (RAG). LangChain provides multiple splitters for different scenarios. Two of the most commonly used are:

- `RecursiveCharacterTextSplitter`
- `CharacterTextSplitter`

### Common Parameters

| Parameter          | Type  | Default Value | Description      |
|--------------------|-------|--------|------------------|
| `chunk_size`       | `int` | 4000 |  Maximum size of chunks to return  |
| `chunk_overlap`    | `int` | 200 | Overlap in characters between chunks |
| `length_function`  | `Callable[[str], int]` | len |  Function that measures the length of given chunks |
| `keep_separator`  | `Union[bool, Literal["start", "end"]]` | False | Whether to keep the separator and where to place it in each corresponding chunk (True='start') | 
| `add_start_index` | `bool` | False | If `True`, includes chunk's start index in metadata | 
| `strip_whitespace` | `bool` | True | If `True`, strips whitespace from the start and end of every document  |



### 1️⃣ CharacterTextSplitter
`CharacterTextSplitter` is the **most basic and efficient** text splitter in LangChain. It uses a **fixed-length character-based splitting strategy**, making it ideal for processing well-structured and uniformly formatted documents.

#### Core Features

- **Deterministic character-based splitting**: splits text strictly based on character counts.
- **Format-friendly**: works well with logs, code, or pre-structured documents.
- **No semantic analysis**: fast and predictable, but may cut through sentences or tokens.


#### Parameters Specific to `CharacterTextSplitter`

| Parameter            | Type    | Default   |Description                             |
|----------------------|---------|-----------|--------------------------------------------|
| `separator`          | `str`   | `"\n\n"`  | Delimiter used to split the text (e.g., newline, space, comma). |
| `is_separator_regex` | `bool`   | `False`    | If `True`, treats separators as regular expressions.  |

✅ This splitter is **simple and fast**, ideal for fixed-format or predictable text structures like logs or CSVs.


#### Example: Long Text Processing

```python
from langchain.text_splitter import CharacterTextSplitter

text = """
    This is the first part of a long document that needs to be split........, Each text block has a maximum length(characters or tokens)
    Document loaders are designed to load document objects.
    LangChain has hundreds of integrations with various data sources to load data from:
    Slack, Notion, Google Drive
"""

# Create a character-based splitter
splitter = CharacterTextSplitter(
    separator=" ",        # Use space as the separator
    chunk_size=50,        # Each chunk has a max length of 50 characters
    chunk_overlap=10      # Each chunk will overlap the previous one by 10 characters
)

# Perform the splitting
chunks = splitter.split_text(text)

# Print the number of chunks and the content
print(len(chunks)) # Output: 8
for chunk in chunks:
    print(chunk)

# Output:  Each chunk will have a maximum length of 50 characters each. If the text is far shorter than 50 characters, it will not be split into multiple chunks even it has spaces.
# This is the first part of a long document that
# that needs to be split........, Each text block
# text block has a maximum length(characters or
# or tokens)
# Document loaders are designed to load
# to load document objects.
# LangChain has hundreds
# hundreds of integrations with various data sources
# sources to load data from:
# Slack, Notion, Google
# Google Drive
```


#### Example: Log Files Processing
When dealing with system logs or monitoring data, it's crucial to split them properly for downstream analysis, especially when feeding them into language models or logging dashboards. 

```python
from langchain.text_splitter import CharacterTextSplitter

# Simulated log entries
log_data = """
[ERROR] 2026-03-15 14:22:35 - Database connection failed
[INFO] 2026-03-15 14:23:10 - Retrying connection...
[WARNING] 2026-03-15 14:23:45 - High memory usage detected
"""

# Initialize a line-based splitter with overlap
splitter = CharacterTextSplitter(
    separator="\n",       # Use newline as separator
    chunk_size=60,        # Each chunk has a max length of 60 characters
    chunk_overlap=20      # Overlap 20 characters between chunks for context
)

# Perform the split
log_chunks = splitter.split_text(log_data)

# Output chunks
for chunk in log_chunks:
    print(chunk)

# Output:
# [ERROR] 2026-03-15 14:22:35 - Database connection failed
# [INFO] 2026-03-15 14:23:10 - Retrying connection...
# [WARNING] 2026-03-15 14:23:45 - High memory usage detected
```

### 2️⃣ RecursiveCharacterTextSplitter
`RecursiveCharacterTextSplitter` is designed to handle complex or nested documents by recursively applying a **priority list of separators** to break large text blocks into manageable chunks.


#### Core Features

- **Recursive Splitting Logic**: Applies a sequence of separators in descending order of granularity.
- **Semantic-Aware**: Prioritizes splitting at meaningful natural language boundaries (e.g., paragraph, sentence).
- **Configurable**: Customizable chunk size, overlap, and separator strategy.
- **Token/Character Friendly**: Helps conform to LLM input size limits while maintaining context.


#### Parameters Specific to `RecursiveCharacterTextSplitter`

| Parameter | Type  | Default    | Description       |
|-----------|-------|------------|-------------------|
| `separators` | `List[str]`| `["\n\n", "\n", " ", ""]` | List of separators to split text by priority (semantic-first). |
| `keep_separator` | `bool` or `"start"/"end"` | `False` | Whether to keep the separator at the start or end of each chunk. |
| `is_separator_regex` | `bool`   | `False`    | while is_separator_regex is technically present, as of the current LangChain implementation, it is **not actively** used inside the logic of RecursiveCharacterTextSplitter. |

✅ This splitter is **semantic-aware** — it attempts to preserve natural boundaries (paragraphs, sentences) as much as possible.


#### separators
`separators` is a prioritized list of delimiter strings used to **recursively split long text** into semantically meaningful chunks.

**Purpose:**

The goal of `separators` is to **preserve semantic integrity** by preferring natural boundaries like paragraphs, sentences, and phrases during chunking. If the text is too long, the splitter will:

1. Attempt to split by the **first separator** (e.g., paragraph).
2. If any chunk is still too long, split by the **next separator** (e.g., sentence).
3. Continue recursively until chunks meet the `chunk_size` requirement.

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=100,
    chunk_overlap=20,
    separators=["\n\n", ".", ",", " "]  # Custom separator list
)

text = "First Paragraph\n\nSeconde Paragraph.Third Paragraph, Fourth Paragraph"

chunks = text_splitter.split_text(text)
print(chunks) # Output: ['First Paragraph\n\nSeconde Paragraph.Third Paragraph, Fourth Paragraph']
```

**Example Splitting Process:**

1.	Try `\n\n` → Split into:
- "First Paragraph"
- "Seconde Paragraph.Third Paragraph, Fourth Paragraph"
2.	Still too long? Try `.` →
- "Seconde Paragraph"
- "Third Paragraph, Fourth Paragraph"
3.	Continue with `,` or `space` if necessary


#### Example: RecursiveCharacterTextSplitter
```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

# Initialize a recursive text splitter
splitter = RecursiveCharacterTextSplitter(
    # The order of separators from high-level to low-level
    separators=["\n\n", "\n", " ", ""],
    chunk_size=20,     # Maximum number of characters in each chunk
    chunk_overlap=4    # Overlapping characters between adjacent chunks
)

# Input text
text = "I Love English hello world, how about you? If you're looking to get started with chat models, vector stores, or other LangChain components from a specific provider, check out our supported integrations"

# Split the text into overlapping chunks
chunks = splitter.split_text(text)  # 13

# Output the results
print(len(chunks))
for chunk in chunks:
    print(chunk)

# Output:
# I Love English hello
# world, how about
# you? If you're
# looking to get
# get started with
# chat models, vector
# stores, or other
# LangChain
# components from a
# a specific
# provider, check out
# out our supported
# integrations
```


#### Pitfull: Unreasonable Separator Order
When using `RecursiveCharacterTextSplitter`, **separator ordering matters a lot**. An incorrect order can severely impact the semantic integrity of your chunks.

**Bad Example:**
Using a space `" "` as the first separator causes the `text to be split too early`, potentially cutting off complete sentences or breaking semantic meaning prematurely.

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

# This puts space " " before newline "\n"
bad_splitter = RecursiveCharacterTextSplitter(
    separators=[" ", "\n"],  # ❗ Too aggressive, splits at every space first
    chunk_size=500
)
```

### CharacterTextSplitter vs. RecursiveCharacterTextSplitter

| Feature / Capability                | `CharacterTextSplitter`                                  | `RecursiveCharacterTextSplitter`                                        |
|-------------------------------------|-----------------------------------------------------------|--------------------------------------------------------------------------|
| **Splitting Strategy**             | Fixed character length with optional overlap              | Recursive splitting by prioritized separators                           |
| **Semantic Awareness**             | ❌ No — may cut through sentences                          | ✅ Yes — prefers semantically meaningful breaks                          |
| **Performance**                    | ⚡️ Fastest (O(n))                                          | 🚀 Fast but slightly slower due to recursion                            |
| **Custom Separators Support**      | ✅ Yes — uses `separator`                                  | ✅ Yes — uses `separators` (list of priorities)                         |
| **Best for Structured Data**       | ✅ Log files, code, CSVs                                   | ✅ Natural language, knowledge docs                                     |
| **Best for Natural Language**      | ❌ May lose context                                        | ✅ Preserves sentence/paragraph integrity                               |
| **Token/Character Count Accuracy** | ✅ Precise control via `chunk_size`                        | ✅ Balances accuracy with context preservation                          |
| **Overlap Support**               | ✅ `chunk_overlap` supported                               | ✅ `chunk_overlap` supported                                            |
| **Whitespace Handling**            | ✅ via `strip_whitespace`                                  | ✅ via `strip_whitespace`                                               |
| **Regular Expression Support**     | ✅ via `is_separator_regex`                                | ❌ (No regex-based separator; uses hardcoded logic internally)          |
| **Multilingual Support**           | ✅ Basic support                                           | ✅ Stronger support due to recursive handling of multilingual syntax     |

---

### Recommended Use Cases

| Scenario                              | Recommended Splitter                  |
|---------------------------------------|---------------------------------------|
| Structured logs                       | `CharacterTextSplitter`              |
| Code files or formatted text blocks   | `CharacterTextSplitter`              |
| Preprocessed clean data               | `CharacterTextSplitter`              |
| Technical documentation (PDF, DOCX)   | `RecursiveCharacterTextSplitter`     |
| Multi-language corpora                | `RecursiveCharacterTextSplitter`     |
| Chat transcripts or FAQ datasets      | `RecursiveCharacterTextSplitter`     |


### Key Differences in Parameters

| Parameter           | `CharacterTextSplitter`         | `RecursiveCharacterTextSplitter`                |
|---------------------|----------------------------------|--------------------------------------------------|
| `separator`         | Single string or regex          | List of prioritized separators                  |
| `chunk_size`        | Max chunk size (chars)          | Max chunk size (chars/tokens)                   |
| `chunk_overlap`     | Overlap between chunks          | Same                                             |
| `keep_separator`    | ✅ Optional                      | ✅ Optional                                       |
| `length_function`   | ✅ Customizable                  | ✅ Customizable                                   |
| `is_separator_regex`| ✅ Supported                     | ❌ Not applicable                                 |


### Practical Notes

- **`separator`**: Use logical separators to avoid semantic breaks. For multilingual texts, use custom lists like `["\n\n", ".", "!", "？", "。"]`, different punctuation marks **across languages**
- **`chunk_size`**: Should be tuned based on your target LLM's context window (e.g., GPT-4 = 32k tokens, Claude 3 = 200k).
- **`chunk_overlap`**: Crucial for preserving continuity between chunks in long documents.
- **`strip_whitespace`**: Set to `False` if whitespace is meaningful (e.g., in code blocks).
- **`is_separator_regex`**: Enables powerful custom splitting (e.g., `separator=r"\d+\."` to split by numbered lists).





<br>





---

🚀 Stay tuned for more insights into LangChain and RAG!



