---
title: Understanding Chains in LangChain
date: 2025-03-31
categories: [AI, LangChain]
tags: [LangChain, Chains, LLMChain, LCEL, Workflow, Prompt Engineering]
author: kai
---

# 🚀 Understanding Chains in LangChain
Chains are the **core architectural unit** in LangChain — enabling developers to **combine multiple components** (models, prompts, parsers, memory, tools, etc.) into **reusable and well-defined workflows**.

Think of them as **middleware pipelines** for large language models (LLMs), similar to how middleware chains work in web frameworks or rule engines.

## 🧠 Concept: What Exactly is a Chain?
A **Chain** is a **linked sequence of functional units** that pass data along a predefined path. Each unit (model, parser, etc.) does its job and hands off to the next.

> Input → [Prompt Template] → [LLM] → [Parser] → Output


## ⚙️ LLMChain: The Foundational Chain
`LLMChain` is the **most basic and essential chain** type in LangChain, designed specifically for:

- Prompt templating
- LLM invocation
- Optional output parsing (Optional)

> It is the earliest implementation of Chains in LangChain, and while still usable, **LCEL (LangChain Expression Language)** is now the recommended alternative.

### Core Responsibilities of `LLMChain`

| Component        | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| Prompt Template  | Dynamically injects user input into pre-defined templates                   |
| LLM Execution    | Sends the formatted prompt to the model (e.g., OpenAI, Qwen, Claude, etc.) |
| Output Parsing   | Optionally parses the raw output (e.g., extract JSON, text, structured data) |

```python
from langchain_openai import ChatOpenAI
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
import os


os.environ["OPENAI_API_KEY"] = "*"  # replace with your actual key

# 1. Create prompt template
prompt_template = PromptTemplate(
    input_variables=["product"],
    template="You are a copywriter, list three words for selling points for {product}",
)

# 2. Create LLM instance
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# 3. Create LLMChain instance
chain = LLMChain(llm=llm, prompt=prompt_template)

# 4. Run LLMChain
response = chain.run(product="a smartphone")
print(response)

# Output:
# response = chain.run(product="a smartphone")
# 1. **Innovative**  
# 2. **Seamless**  
# 3. **Versatile**  
```

>  LangChainDeprecationWarning: The class `LLMChain` was deprecated in LangChain 0.1.17 and will be removed in 1.0. Use :meth:"~RunnableSequence, e.g., `prompt | llm`" instead.


## 💡 LCEL - LangChain Expression Language
LangChain Expression Language (LCEL) is a **declarative programming style** introduced in **LangChain v0.3+**, designed to make building and composing AI workflows **simpler, faster, and more powerful**.

**Core Idea:** LCEL uses the **pipe operator `|`** to connect components — much like Unix pipes or functional stream programming.

### Why LCEL?
- **A Clean Syntax:** Example: `chain = prompt | llm | parser`
- **Runnable Standard:** All components follow the Runnable protocol
- **I/O Standardization:** All inputs/outputs are dict - no need for custom data conversions
- **Async + Streaming:** Native support for async and streaming output (like ChatGPT typing)
- **Debug-Friendly:** Built-in tools for tracing, retrying, and instrumentation

###  LCEL vs Legacy Chain Architecture

| Feature | v0.2 (Classic Chain) | v0.3+ (LCEL) |
|---------|----------------------|--------------|
| **Composition Style** | Class Inheritance (Chain subclass) | `|` |
| **Interfaces** | Chain/LLMChain subclasses | Runnable Protocol |
| **Data Passing** | Varies (string, object) | Always Dict in/out |
| **Async Support** | Limited | First-class async and stream() |
| **Debugging Tools** | Manual | Built-in tracing, retry, inspect |
| **Module Location** | langchain.chains | langchain_core.runnables |

### LCEL Core Syntax
LCEL introduces the **pipe operator (`|`)** to connect components. The output from one component becomes the input to the next — similar to Unix pipelines or functional composition.

`chain = component_a | component_b | component_c`

### Example: Build a Simple Q&A Chain

```python
from langchain_core.prompts import ChatPromptTemplate  
from langchain_openai import ChatOpenAI  
from langchain_core.output_parsers import StrOutputParser  
import os


os.environ["OPENAI_API_KEY"] = "*"  # replace with your actual key

# 1. Create prompt template
prompt = ChatPromptTemplate.from_template("Your are a prfessional AI teacher. Please answer this question: {question}")  

# 2. Create LLM instance
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# 3. Define an Output Parser
output_parser = StrOutputParser()

# 4. Compose the LCEL Chain
chain = prompt | llm | output_parser  

response = chain.invoke({"question": "How to learn LLM?"})
print(response)

# Output:
# Learning about Large Language Models (LLMs) involves understanding their architecture, training processes, applications, and ethical considerations. Here’s a structured approach to guide your learning:...More
```





<br>



---

🚀 Stay tuned for more insights into LangChain!



