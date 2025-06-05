---
title: LangChain PromptTemplate
date: 2025-03-29
categories: [AI, LangChain]
tags: [LangChain, PromptTemplate, LLM]
author: kai
permalink: /posts/ai/langchain/langchain-prompttemplate/
---

# 🚀 LangChain PromptTemplate: Dynamic Prompt Design in Action
In modern LLM workflows, **flexibility and reusability** are key.  
Hardcoding prompts leads to duplicated logic and brittle interfaces.  
LangChain solves this by introducing the `PromptTemplate` — a core utility for **dynamically assembling prompts** in a structured, parameterized way.

## 📌 What Is `PromptTemplate`?

`PromptTemplate` is a component in LangChain that allows you to create **dynamic, reusable** text prompts with variables.


### Benefits

- **Avoid** prompt **hardcoding**
- Assemble prompts with **variable inputs**
- Support **pre-filling variables** for modular design
- Lay the foundation for **advanced templates** (`ChatPromptTemplate`, `FewShotPromptTemplate`, etc.)

### PromptTemplate Workflow
```text
User Inputs → PromptTemplate → Formatted Prompt → LLM → Structured Output
```

## 🧩 Key Parameters and Methods

### 1. `template` — Define Prompt Structure
The actual text template with **placeholder variables**, enclosed in curly braces `{}`.
- Acts like a formatted string
- Placeholders must match input_variables

### 2. input_variables — Declare Required Placeholders
**A list of strings** representing the **dynamic inputs** the template expects at runtime.
- Provide all values manually when calling .format()
- Gives maximum flexibility, but less reusability if some values stay constant across many prompts.
- Ensures validation and completeness

### 3. partial_variables — Pre-Fill Fixed Variables (Optional)
Used to **inject constant values** that remain the same across multiple uses.
- Reduces redundancy when you have shared system instructions or global roles.
- great for defaulting or fixing certain variables (e.g., target audience, tone, region, etc.) without hardcoding them into the prompt string.
- When using partial_variables, LangChain expects you **not to include that variable in input_variables** — because it’s already fixed.

### 4. .format() — Format Prompt with Actual Values
**Main method** for injecting values into the template.
- All input_variables must be supplied unless pre-filled via partial_variables
- Returns a fully constructed string ready to be passed to the LLM


### Example

```python
from langchain.prompts import PromptTemplate

# Define a multilingual domain-specific template
template = """  
You are a professional {domain} consultant. Explain to {audience} using simple terms. Please use {language} to answer:  
Question: {question}  
Answer:  
"""   

# Create PromptTemplate instance
prompt = PromptTemplate(  
    input_variables=["domain", "language", "question"],  # You must provide all values manually when calling .format().
    partial_variables={"audience": "high school students"}, # pre-filled audience once, and you don’t have to (and must not) include it in .format() again.
    template=template
)  

# Print the template object (shows structure & variables)
print(prompt)

# Format the prompt with actual input values
filled_prompt = prompt.format(  
    domain="Cybersecurity",  
    language="English",  
    question="How to prevent phishing attacks?"  
)  

print(filled_prompt)
# Output:
# You are a professional Cybersecurity consultant. Explain to high school students using simple terms. Please use English to answer:  
# Question: How to prevent phishing attacks?  
#Answer:
```

### ⚙️ from_template(): Auto-Detecting Input Variables in LangChain
The `from_template()` class method allows you to **skip explicitly listing variables**.  
It **parses the template string**, identifies placeholders (`{}`), and infers the required input variables.

## 🧪 Code Example

```python
from langchain.prompts import PromptTemplate  

# Define the prompt without manually declaring input_variables
template = "Please translate the following text into {target_language}：{text}"
prompt = PromptTemplate.from_template(template)

# Output inferred variables
print(prompt.input_variables)   # LangChain automatically detects: ['target_language', 'text']
```

## 🆚 .invoke() and .format() 
LangChain’s `PromptTemplate` provides two important methods to use with variables:

| Method | Result Type | Purpose | Example Output |
|--------|-------------|---------|----------------|
| `.format()` | str | Pure string formatting (like f-string) | "Write a blog post title for the product: TechReflections by Kai" |
| `.invoke()` | str or LangChain RunnableOutput | Used in LangChain workflows (Runnable-compliant) | Same string, but wrapped in execution-friendly object |

> ✅ Both return the same text output, but .invoke() is future-proof for LangChain chains.
⚠️**LangChainDeprecationWarning**: The method BaseChatModel.__call__ was deprecated in langchain-core 0.1.7 and will be removed in 1.0.
**Use :meth:~invoke instead**.

- When use `.format()`: 
    - You just want the raw formatted string
    - Debugging or visualizing the prompt
- When use `.invoke()`: 
    - You're using the prompt in a chain / LLM workflow
    - Combining with LLM, tools, or agents


## 👀 LangChain Output Parsing: Extracting and Structuring LLM Responses
In LangChain, **output parsing** is a critical step that **transforms raw LLM output into structured**, usable formats. Whether you're building a chatbot, data extractor, or code generator — having control over output parsing helps you convert model responses into predictable formats.

Large Language Models (LLMs) usually return plain text, which:
- ❌ May be verbose or inconsistent
- ❌ May include formatting artifacts
- ❌ Is hard to process directly in applications

LangChain provides **output parsers** that help:
- ✅ Clean up text  
- ✅ Convert it to Python-native structures (e.g. `dict`, `list`)  
- ✅ Enforce schema or formats like JSON, YAML, or Markdown  

### Core Parser Types

| Parser                  | Description                                        | Output Format |
|-------------------------|----------------------------------------------------|---------------|
| `StrOutputParser`       | Returns raw string (default for basic use)         | `str`         |
| `JsonOutputParser`      | Parses output into Python `dict` via `json.loads()`| `dict`        |
| `PydanticOutputParser`  | Validates output against a Pydantic schema         | `BaseModel`   |
| `CommaSeparatedListOutputParser` | Parses comma-separated items into list     | `List[str]`   |
| `MarkdownListParser`    | Extracts bullet lists from markdown-formatted text | `List[str]`   |


### Examples

```python
# 1. `StrOutputParser` — Clean Text Output
from langchain_core.output_parsers import StrOutputParser

parser = StrOutputParser()
result = parser.invoke("  Hello world!  ")  # 'Hello world!'

# 2. JsonOutputParser — Parse Structured Output
from langchain_core.output_parsers import StrOutputParser

parser = JsonOutputParser()
output = parser.invoke('{"name": "Kai", "role": "Engineer"}')
print(output["role"])  # Engineer

# Use in prompt
# Return the result in the following JSON format:
# {"name": ..., "role": ...}

# 3. PydanticOutputParser — Schema Validation
from pydantic import BaseModel
from langchain_core.output_parsers import StrOutputParser

class User(BaseModel):
    name: str
    email: str

parser = PydanticOutputParser(pydantic_object=User)

output = parser.invoke('{"name": "Alice", "email": "a@example.com"}')
print(output.name)  # Alice

# 4. Common Use Pattern in LangChain
from langchain.prompts import PromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser

prompt = PromptTemplate.from_template("What's a good name for a {product}?")
llm = ChatOpenAI()
parser = StrOutputParser()

chain = prompt | llm | parser

response = chain.invoke({"product": "blog generator"})
print(response)
```

`chain = prompt | llm | parser` -> **This one-liner creates a modular chain that**:
1.	Formats the prompt
2.	Sends it to the LLM
3.	Parses the LLM output

## ✍🏻 PromptTemplate in Action: Building with OpenAI LLM

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser
import os

os.environ["OPENAI_API_KEY"] = "*"  # replace with your actual key

# 1. Create prompt template
prompt = PromptTemplate.from_template("Write 3 catchy advertising slogans for {product}, targeting a young audience.")

# 2. Create the LLM instance
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# 3. Create the output parser
out_parser = StrOutputParser()

# 4. Chain
chain = prompt | llm | out_parser

# 5. Invoke
answer = chain.invoke({"product": "Nike SB shoes"})
print(answer)
# Output Example:
# 1. "Unleash Your Style, Own the Street – Lace Up with Nike SB!"
# 2. "Ride the Board, Rock the Look – Nike SB: Where Skate Meets Style!"
# 3. "Step Up Your Game – Elevate Every Trick with Nike SB!"
```



<br>



---

🚀 Stay tuned for more insights into LangChain!



