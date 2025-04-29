---
title: RunnablePassthrough in LangChain
date: 2025-04-29
order: 3
categories: [AI, LangChain]
tags: [LangChain, LLM, Runnable, RunnablePassthrough]
author: kai
---

# 🚀 RunnablePassthrough in LangChain
`RunnablePassthrough` is a simple yet powerful component in LangChain's Runnable ecosystem.  
Its main purpose is to **pass input data through the chain without any modification**—or **optionally enhance it** by attaching additional fields.


## 🧠 Core Functionality

- **Direct Passthrough**:  
  By default, `RunnablePassthrough` simply forwards the input to the next component **unchanged**.
  
- **Context Enhancement via `.assign()`**:  
  You can use `.assign()` to **attach new fields** to the input before passing it on, allowing dynamic context building for downstream Runnables.


```plaintext
Original Input 
    ↓
RunnablePassthrough
    ↓
Decision: use assign?  
    ↓
    ├── YES -> Add new fields → Output enhanced dictionary
    └──  No → Output original input unchanged
  ```


## 🛠️ Application Scenarios
- **Preserve raw user input:** Ensure the original question or data is always available downstream
- **Dynamically augment context:** Attach additional fields like retrieval results, metadata, or computed features
- **Combine multiple information sources:** Merge LLM input and external data without rewriting prior chain logic


## ➕ Usage Examples

### Basic Usage Example
The input "Hello" will be passed directly to the model without any modification.

```python
from langchain_core.runnables import RunnablePassthrough

# Create a passthrough chain
chain = RunnablePassthrough() | model

# Invoke the chain
output = chain.invoke("Hello")
```


### Adding a Processed Field

```python
from langchain_core.runnables import RunnablePassthrough

# Use assign() to add a new field
# The assign method accepts keyword arguments, where the value is a function
# that defines how to compute the new field based on the input
chain = RunnablePassthrough.assign(
    processed=lambda x: x["num"] * 2
)

# Now when invoking the chain, the input will be extended
output = chain.invoke({"num": 3})

print(output)  # {'num': 3, 'processed': 6}
```

Output:
- Original input dictionary is preserved
- New field "processed" is attached


### enriching the context with retrieved documents

```python
# Define a chain that adds retrieval context dynamically
chain = (
    RunnablePassthrough.assign( # Dynamically adds a new field context using the user question
        context=lambda x: retrieve_documents(x["question"]) # Fetches relevant documents based on the question
    )
    | prompt
    | llm
)

# Input structure
input_data = {"question": "What is LangChain?"}

# Invoke the chain
response = chain.invoke(input_data)
```

**Data flow:**
```text
Original input: {"question": "What is LangChain?"}
        ↓
RunnablePassthrough.assign(context=retrieve_documents(question))
        ↓
Enriched input: {"question": "What is LangChain?", "context": [retrieved_docs]}
        ↓
prompt.format(context, question)
        ↓
llm.generate(prompt)
        ↓
Final Answer
```

### Using RunnablePassthrough to Build a Contextual LLM Chain in LangChain
The goal is to pass both the **retrieved context** and the **original question** into the LLM, building a powerful Retrieval-Augmented Generation (RAG) system.

- The user input should be passed **unchanged** to the prompt.
- At the same time, the **retriever** fetches relevant documents.
- Both **context** and **question** are fed into a structured prompt for the LLM to generate an informed answer.

```python
import os
from langchain_milvus import Milvus
from langchain_core.documents import Document
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI, OpenAIEmbeddings


os.environ["OPENAI_API_KEY"] = "*" 

# Initialize embedding model
openai_embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small",
    max_retries=3,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Prepare example documents
document_1 = Document(
    page_content="LangChain supports multiple database integrations, featured in Kai's Classroom's AI course.",
    metadata={"source": "kai.com/doc1"},
)
document_2 = Document(
    page_content="Milvus excels at vector search tasks, highly recommended by Jojo's courses.",
    metadata={"source": "kai.com/doc2"},
)

documents = [document_1, document_2]

# Create Milvus vector store based on documents
vector_store = Milvus.from_documents(
    documents=documents,
    embedding=openai_embeddings,
    collection_name="runnable_test",
    connection_args={"uri": "http://54.163.61.180:19530",
                     "db_name": "my_database"}
)

# Create retriever from the vector store
retriever = vector_store.as_retriever(search_kwargs={"k": 2})

# Define a prompt template 
# Context is filled by retriever, Question is directly passed through (user input)
prompt = ChatPromptTemplate.from_template(
    "Answer the question based on the following context:\n\n{context}\n\nQuestion: {question}")

# Define the LLM
model = ChatOpenAI(  # Initialize the LLM model
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Build the chain
chain = {
    "context": retriever,               # context is filled by retriever
    "question": RunnablePassthrough()   # question is directly passed through
} | prompt | model

# Run the chain with user input
result = chain.invoke("Does LangChain support multiple database integrations?")
print(result)

# Output
# content='Yes, LangChain supports multiple database integrations.' 
# additional_kwargs={'refusal': None} 
# response_metadata={'token_usage': {'completion_tokens': 10, 'prompt_tokens': 110, 'total_tokens': 120, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_0392822090', 'finish_reason': 'stop', 'logprobs': None} id='run-c67a8ed1-55c6-4920-adf2-ada4d4c8f09f-0' usage_metadata={'input_tokens': 110, 'output_tokens': 10, 'total_tokens': 120, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}}


# if change the documents 1 to NOT SUPPORT
document_1 = Document(
    page_content="LangChain doesn not support multiple database integrations, featured in Kai's Classroom's AI course.",
    metadata={"source": "kai.com/doc1"},
)

# the result will be 
# content='No, LangChain does not support multiple database integrations.' ...
```

#### Data Flow

```text
User Input: "Does LangChain support multiple database integrations?"
    ↓
RunnablePassthrough → forwards "question"
Retriever → retrieves top-2 relevant documents → fills "context"
    ↓
PromptTemplate → combines "context" + "question" → constructs prompt
    ↓
LLM → generates contextual answer
    ↓
Output: Answer based on retrieved content
```

> This structure guarantees that the LLM receives both the user’s original question and relevant knowledge context, leading to higher answer accuracy.


#### Explanation of the Chain Construction 
```python
chain = {
    "context": retriever,               
    "question": RunnablePassthrough()
} | prompt | model
```

`{ "context": retriever, "question": RunnablePassthrough() }`
- This defines a dictionary of two **parallel Runnables**: one retrieves documents, one passes the question as-is.
- **context: retriever**: The retriever is responsible for fetching related documents based on the user's question.
- **question: RunnablePassthrough()**: The RunnablePassthrough simply forwards the original question without any changes.

**At runtime:**
1.	User input: e.g., "Does LangChain support multiple database integrations?"
2.	The chain branches the input into two flows:
    - context: The input goes into the retriever, which returns a list of relevant documents.
	- question: The same input is directly passed through using RunnablePassthrough().
3.	The two results (context and question) are merged into a dictionary:
```python
{
    "context": ["LangChain supports multiple database integrations, featured in Kai's Classroom's AI course.", ... ],
    "question": "Does LangChain support multiple database integrations?"
}
```
4. This merged dictionary is fed into the prompt template: `"Answer the question based on the following context:\n\n{context}\n\nQuestion: {question}"`

    ```text
    "Answer the question based on the following context:

    ["LangChain supports multiple database integrations, featured in Kai's Classroom's AI course.", ... ]

    Question: Does LangChain support multiple database integrations?"
    ```

5. The LLM (model) receives the final prompt and generates the answer.


**Data Flow:**
```text
User Input
    ↓
    ├── "context" → retriever → retrieved documents
    └── "question" → RunnablePassthrough → original question
        ↓
Combined dictionary {context, question}
        ↓
Formatted into prompt (ChatPromptTemplate)
        ↓
Model (LLM) generates answer
        ↓
Final Output
```  








<br>




---

🚀 Stay tuned for more insights into LangChain!



