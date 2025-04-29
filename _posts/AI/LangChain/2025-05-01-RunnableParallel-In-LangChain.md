---
title: RunnableParallel in LangChain
date: 2025-04-29
order: 4
categories: [AI, LangChain]
tags: [LangChain, LLM, Runnable, RunnableParallel]
author: kai
---

# 🚀 RunnableParallel in LangChain
`RunnableParallel` is a **specialized Runnable subclass** designed to **execute multiple Runnables concurrently** and **combine their outputs into a single dictionary**.  
It provides a highly efficient way to perform **independent multi-task processing** within a LangChain pipeline.


### 🧠 Core Concept
You give it **a set of Runnables**, and for each input, it runs them in parallel and collects the **results in a dictionary**, where:
- **Keys** = names you assign to each Runnable
- **Values** = the outputs from each Runnable

```python
class RunnableParallel(Runnable[Input, Dict[str, Any]]):
    """
    A container class that executes multiple Runnables in parallel.
    Outputs a dictionary: {key1: result1, key2: result2, ...}
    """
```

## 🔥 Dictionary Auto-Conversion to `RunnableParallel` in LCEL Chains
One of the elegant features of **LCEL (LangChain Expression Language)** is that when you **use a dictionary `{}` inside a chain**, it is **automatically treated as a `RunnableParallel`** behind the scenes—without you manually wrapping it!

This greatly simplifies writing **multi-retriever** or **multi-task pipelines**.

```python
# Full Chain Definition
multi_retrieval_chain = (
    { # a dictionary {} inside a chain,
        "context1": retriever1,  # First data source
        "context2": retriever2,  # Second data source
        "question": RunnablePassthrough()  # Pass user question unchanged
    }
    | prompt_template
    | llm
    | outputParser
)

=====Equivalent to manually wrapping in RunnableParallel=====
multi_retrieval_chain = (
    RunnableParallel({ 
        "context1": retriever1,
        "context2": retriever2,
        "question": RunnablePassthrough()
    })
    | prompt_template
    | llm
    | outputParser
)
```

**The output passed into prompt_template will look like:**

```python
{
    "context1": [...retrieved docs from retriever1...],
    "context2": [...retrieved docs from retriever2...],
    "question": "user's original question"
}
```
## ⚙️ Key Features

| Feature         | Description                                 | Example                                                        |
|-----------------|---------------------------------------------|----------------------------------------------------------------|
| Parallel Execution | All child Runnables are run **simultaneously** | 3 tasks in parallel complete in ~2 seconds instead of 6 seconds |
| Type Safety     | Input and output types are **strictly validated** | Automatically checks that dictionary fields match expected types |



## 📈  Data Flow

```text
User Input
    ↓
    ├── retriever1 → "context1"
    ├── retriever2 → "context2"
    └── RunnablePassthrough → "question"
    ↓
Merge into dictionary {context1, context2, question}
    ↓
Format with prompt_template
    ↓
Send to LLM
    ↓
Parse output with outputParser
    ↓
Return final answer
```

## 📜 API Overview
**Constructor arguments: key-value pairs**
- Key: a string identifier for the branch
- Value: a Runnable object (could be a retriever, model chain, prompt chain, etc.)

✅ All Runnables inside share the same input when invoked.

```python
from langchain_core.runnables import RunnableParallel

runnable = RunnableParallel(
    key1=chain1,
    key2=chain2,
    key3=chain3,
    ...
)
```


## 🛠️ Application Scenarios
`RunnableParallel` is a powerful pattern for **multi-task**, **multi-model**, and **multi-step data processing** pipelines.  
It enables **parallel execution** while keeping the outputs **well-structured and modular**.


| Scenario                         | Description                                                  |
|----------------------------------|--------------------------------------------------------------|
| Data Parallel Processor          | Process multiple streams of data simultaneously             |
| Structured Data Assembler        | Assemble multiple analysis results into a standardized output |
| Pipeline Fork-Merge (Map Phase)  | Implement the "Map" step of a Map-Reduce-like architecture    |


### 1. Multi-Dimensional Data Analysis
Perform **sentiment analysis**, **keyword extraction**, and **entity recognition** at the same time from a text input.

```python
analysis_chain = RunnableParallel({
    "sentiment": sentiment_analyzer,
    "keywords": keyword_extractor,
    "entities": ner_recognizer
})

result = analysis_chain.invoke("LangChain is revolutionizing LLM application development.")

# Output:
{
    "sentiment": "positive",
    "keywords": ["LangChain", "LLM", "development"],
    "entities": ["LangChain"]
}
```


### 2. Multi-Model Comparison System
Run the same input through multiple LLMs to compare their responses side-by-side. Great for benchmarking, A/B testing, or ensemble systems.

```python
model_comparison = RunnableParallel({
    "gpt4": gpt4_chain,
    "claude": claude_chain,
    "gemini": gemini_chain
})
result = model_comparison.invoke("Explain quantum computing in simple terms.)

# Output:
{
    "gpt4": "Quantum computing uses quantum bits...",
    "claude": "In quantum computing, bits can be in multiple states...",
    "gemini": "Quantum computers use qubits instead of classical bits..."
}
```


### 3. Intelligent Document Processing System
Analyze a document from multiple dimensions: summary, table of contents, and statistics. Perfect for **automated PDF processing, legal document review, research summarization**, etc.

```python
from langchain_core.runnables import RunnableLambda

document_analyzer = RunnableParallel({
    "summary": summary_chain,       # Generate document summary
    "toc": toc_generator,           # Extract table of contents
    "stats": RunnableLambda(lambda doc: {
        "char_count": len(doc),
        "page_count": doc.count("PAGE_BREAK") + 1
    })
})

# Analyze a large 200-page PDF document
analysis_result = document_analyzer.invoke(pdf_text)

# Output:
{
    "summary": "This document discusses AI frameworks...",
    "toc": ["Introduction", "Background", "Methods", "Conclusion"],
    "stats": {"char_count": 125000, "page_count": 200}
}
```


## 👀 Parallel Generation of Attractions and Book Recommendations

- **Task**: Given a city name and a number, generate:
  - A list of recommended **attractions**.
  - A list of related **books**.
- **Requirements**: 
  - Both results should be returned in **JSON format**.
  - Both tasks should run **in parallel** to optimize speed.

{% raw %}
```python
import os
from langchain_core.runnables import RunnableParallel
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import JsonOutputParser  

os.environ["OPENAI_API_KEY"] = "*"  

# Define the LLM
model = ChatOpenAI( 
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Create a JSON parser
parser = JsonOutputParser()

# Define the prompts
prompt_attractions = ChatPromptTemplate.from_template("""
List {num} attractions in {state}. Return in JSON format:
    {{
        "num": "Index number",
        "state": "State name",
        "introduce": "Attraction introduction"
    }}
""")

prompt_books = ChatPromptTemplate.from_template("""
List {num} books related to {state}. Return in JSON format:
    {{
        "num": "Index number",
        "state": "State name",
        "introduce": "Book introduction"
    }}
""")

# Create the chains
chain1 = prompt_attractions | model | parser
chain2 = prompt_books | model | parser

# Combine them using RunnableParallel
chain = RunnableParallel(
    attractions=chain1,
    books=chain2
)


# Invoke the parallel chain
output = chain.invoke({"state": "Pennsylvania", "num": 3})
print(output)

# Output
# {'attractions': [
#     {'num': '1', 'state': 'Pennsylvania', 'introduce': 'The Liberty Bell is an iconic symbol of American independence located in Philadelphia. It is famous for its distinctive crack and is housed in the Liberty Bell Center, where visitors can learn about its history and significance.'}, 
#     {'num': '2', 'state': 'Pennsylvania', 'introduce': 'Hersheypark is a family amusement park located in Hershey, Pennsylvania. It features a variety of rides, water attractions, and entertainment options, all themed around the famous Hershey chocolate brand.'}, 
#     {'num': '3', 'state': 'Pennsylvania', 'introduce': 'The Gettysburg National Military Park preserves the site of the Battle of Gettysburg, a turning point in the Civil War. Visitors can explore the battlefield, museums, and monuments that commemorate this pivotal event in American history.'}
#     ], 
# 'books': [
#     {'num': '1', 'state': 'Pennsylvania', 'introduce': 'A comprehensive history of Pennsylvania, exploring its founding, development, and significant events that shaped the state.'}, 
#     {'num': '2', 'state': 'Pennsylvania', 'introduce': "This book delves into the rich cultural heritage of Pennsylvania Dutch communities, highlighting their traditions, crafts, and contributions to the state's identity."}, 
#     {'num': '3', 'state': 'Pennsylvania', 'introduce': "A fascinating exploration of Pennsylvania's role in the American Civil War, detailing key battles, figures, and the state's strategic importance."}
# ]}
```
{% endraw %}


#### LCEL-style Full Chain
If you plan to continue chaining (e.g., sending output into a prompt)

{% raw %}
```python
import os
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import JsonOutputParser, StrOutputParser

os.environ["OPENAI_API_KEY"] = "*"  

# Define the LLM
model = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Define the prompts
prompt_attractions = ChatPromptTemplate.from_template("""
List {num} attractions in {state}. Return in JSON format:
{{
    "num": "Index number",
    "state": "State name",
    "introduce": "Attraction introduction"
}}
""")

prompt_books = ChatPromptTemplate.from_template("""
List {num} books related to {state}. Return in JSON format:
{{
    "num": "Index number",
    "state": "State name",
    "introduce": "Book introduction"
}}
""")

final_merge_prompt = ChatPromptTemplate.from_template("""
You are a travel assistant. Based on the provided data, create a nice travel guide paragraph.

Attractions:
{attractions}

Recommended Books:
{books}

Generate a final friendly paragraph combining this information.
""")


multi_retrieval_chain = (
    {
        "attractions": prompt_attractions | model | JsonOutputParser(),
        "books": prompt_books | model | JsonOutputParser()
    }
    | final_merge_prompt
    | model
    | StrOutputParser()
)


# Invoke the parallel chain
output = multi_retrieval_chain.invoke({"state": "Pennsylvania", "num": 3})
print(output)

# Output
# Welcome to Pennsylvania, a state rich in history, culture, and family fun! Start your journey in Philadelphia, where you can marvel at the Liberty Bell, an enduring symbol of American independence that draws millions of visitors each year. For a day filled with excitement, head to Hersheypark in Hershey, where thrilling rides and chocolate-themed attractions promise fun for the whole family. Don’t miss the opportunity to explore Gettysburg National Military Park, a site that preserves the history of a pivotal Civil War battle, complete with reenactments that bring the past to life. To deepen your understanding of Pennsylvania's history, consider picking up "The Great Pennsylvania Puzzle Book" by Thomas J. McCarthy for a fun, trivia-filled challenge, or "The Civil War in Pennsylvania" by William C. Kashatus to dive into the state’s significant contributions during the war. For a thought-provoking read, John Edgar Wideman's "Philadelphia Fire" offers a compelling narrative on race and community in the city. Pennsylvania awaits with its unique blend of attractions and insightful literature!
```
{% endraw %}


**Data Flow:**

```text
Input: {"state": "Pennsylvania", "num": 3}
    ↓
RunnableParallel:
    ├── Generate attractions list
    └── Generate book list
    ↓
Merge into one dictionary {attractions, books}
    ↓
Format using final_merge_prompt
    ↓
Generate final natural language output with LLM
    ↓
Return friendly travel recommendation paragraph
```





<br>




---

🚀 Stay tuned for more insights into LangChain!



