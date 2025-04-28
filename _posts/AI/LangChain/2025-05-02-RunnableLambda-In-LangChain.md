---
title: RunnableLambda in LangChain
date: 2025-04-28
order: 5
categories: [AI, LangChain]
tags: [LangChain, LLM, Runnable, RunnableLambda]
author: kai
---

# 🚀 RunnableLambda in LangChain
`RunnableLambda` is a powerful and flexible component in LangChain that allows you to **wrap any Python function** as a **Runnable-compatible object**,  
making it **seamlessly integrable** into any LCEL (LangChain Expression Language) chain.

## 🧠 Core Functionality

- **Custom Python Function** ➔ **Wrapped by RunnableLambda** ➔ **Usable in LangChain Chains**
- This lets you **inject any external logic**, **connect APIs**, or **extend functionality** without breaking the LCEL flow.

Your function behaves like any other LangChain component:  
- Supports `.invoke()`, `.ainvoke()`, `.batch()`, `.stream()`
- Can be connected with `|` pipes to prompts, models, retrievers, etc.


```python
class RunnableLambda(Runnable[Input, Output]):
    """
    Wrap any Python function or callable into a Runnable object,
    allowing custom logic to integrate with the LangChain ecosystem.
    """
```

## 🆚 Differences Between Normal Python Functions 


| Feature           | Normal Python Function          | RunnableLambda                             |
|-------------------|----------------------------------|--------------------------------------------|
| **Composable**     | Cannot connect to Chain directly | Fully chainable with LCEL `|` syntax   |
| **Type Checking**  | Dynamic, no guarantees        | Enforces input/output type structure   |
| **Async Support**  | Must manually implement `async`| Built-in support for `ainvoke()` and async pipelines |
| **Batch Processing**| Must manually loop over inputs | Supports `batch()` natively for parallel optimization |


## 🗞️ Suitable Use Cases

| Scenario                         | Purpose                                                    |
|-----------------------------------|------------------------------------------------------------|
| Insert Custom Logic              | Add logging, monitoring, feature extraction, etc.          |
| Transform Data Format             | Convert data formats (e.g., JSON parsing, schema validation) |
| Enrich Input or Output            | Dynamically modify prompt inputs based on runtime information |
| Integrate with External Systems   | Fetch APIs, call databases, inject metadata                |
| Lightweight Business Rules        | Simple condition checks, decision trees, branching hints   |


## 🛠️ API Overview

```python
from langchain_core.runnables import RunnableLambda

# Step 1: Define a simple Python function
def log_input(x):
    print(f"Input: {x}")
    return x

# Step 2: Insert it into the LCEL chain
chain = prompt | RunnableLambda(log_input) | model
```


## 📈 Data Flow
```text
User Input
   ↓
RunnableLambda (custom Python logic)
   ↓
PromptTemplate
   ↓
Model (LLM)
   ↓
(Optionally another RunnableLambda for postprocessing)
   ↓
Final Output
```


## ➕ Usage Examples
### Basic Example

```python
def clean_text(x):
    # Remove excessive whitespace
    return {"text": x["text"].strip()}

chain = RunnableLambda(clean_text) | prompt | model
```


### Text Cleaning Chain

```python
from langchain_core.runnables import RunnableLambda

# Build a simple text cleaning chain
text_clean_chain = (
    RunnableLambda(lambda x: x.strip())  # Step 1: Remove leading/trailing whitespace
    | RunnableLambda(str.lower)          # Step 2: Convert text to lowercase
)

# Invoke the chain
result = text_clean_chain.invoke("  Hello123World  ") 

print(result)  # Output: "hello123world"
```

### Printing and Filtering Sensitive Content

```python
import os
from langchain_core.runnables import RunnableLambda
from langchain_openai import ChatOpenAI

os.environ["OPENAI_API_KEY"] = "*" 

# Define the LLM
model = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Define a simple sensitive content filter, replace the word "violence" to "***"
def filter_content(text: str) -> str:
    return text.replace("violence", "***")


# Build the chain
chain = (
    RunnableLambda(lambda x: x["user_input"])  # Extract "user_input" field
    | RunnableLambda(lambda x: (print(f"[DEBUG] Raw Input: {x}"), x)[1])  # Print the intermediate input
    | RunnableLambda(filter_content)  # Filter sensitive words
    | RunnableLambda(lambda x: (print(f"[DEBUG] Filtered Input: {x}"), x)[1])  # Print after filtering
    | model
)

# Invoke the chain
result = chain.invoke({"user_input": "violence is bad! Do not do violence."})

print(result)

# Output:
# [DEBUG] Raw Input: violence is bad! Do not do violence.
# [DEBUG] Filtered Input: *** is bad! Do not do ***.
# content="It seems like you're expressing a concern about something specific. If you could provide more context or clarify what you're referring to, I'd be happy to help or discuss it further!" additional_kwargs={'refusal': None} response_metadata={'token_usage': {'completion_tokens': 35, 'prompt_tokens': 16, 'total_tokens': 51, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_0392822090', 'finish_reason': 'stop', 'logprobs': None} id='run-c3d0ff33-daf3-4e9a-9172-527240b2bd80-0' usage_metadata={'input_tokens': 16, 'output_tokens': 35, 'total_tokens': 51, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}}
```







<br>




---

🚀 Stay tuned for more insights into LangChain!



