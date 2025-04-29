---
title: RunnableBranch in LangChain
date: 2025-04-29
order: 6
categories: [AI, LangChain]
tags: [LangChain, LLM, Runnable, RunnableBranch]
author: kai
---

# 🚀 RunnableBranch in LangChain
`RunnableBranch` allows you to **route inputs dynamically** based on conditions,  
similar to traditional **if-else** statements in Python, but fully **chainable** within LCEL pipelines!

## 🧠 Core Concept

```python
from langchain_core.runnables import RunnableBranch

# Build a conditional branch
branch = RunnableBranch(
    (condition1, chain1),
    (condition2, chain2),
    default_chain
)
```

### Parameter Explanation

| Parameter    | Description                                                              |
|--------------|--------------------------------------------------------------------------|
| **Condition** | A Python function (callable) that receives the input and returns `bool`. |
| **Runnable**  | A chain (or Runnable) to execute when its associated condition is `True`. |
| **Default**   | A fallback chain that runs if none of the conditions match. (Optional)    |

- Every `(Condition, Runnable)` pair defines a **branch**.  
- The **default Runnable** handles the situation when **no conditions are satisfied**.

### Technical Execution Details

| Rule                          | Behavior                                                        |
|--------------------------------|------------------------------------------------------------------|
| 1. **Order Matters**           | Conditions are evaluated **in the order you define** them.       |
| 2. **First Match Wins**        | As soon as a condition returns `True`, its associated chain executes, and the rest are skipped. |
| 3. **Default Handling**        | If **no conditions** match and you provided a **default chain**, it will be executed. |
| 4. **No Default Provided**     | If no conditions match **and** no default is provided, **an exception is raised**. |


### Data Flow

```plaintext
Input
  ↓
Condition 1 ➔ True? ➔ Yes → Execute Runnable 1
             ↓ No
Condition 2 ➔ True? ➔ Yes → Execute Runnable 2
             ↓ No
Condition 3 ➔ True? ➔ Yes → Execute Runnable 3
             ↓ No
Default chain exists? → Yes → Execute Default
                      → No → Raise Exception
             ↓ 
Final Output
```

## 🗞️ Suitable Use Cases

| Scenario                      | Description                                                                 |
|--------------------------------|-----------------------------------------------------------------------------|
| **Multi-task Classification**  | Route inputs to different chains based on task type (e.g., math vs physics questions) |
| **Error Handling Branching**   | If a main chain fails or returns an unexpected result, fall back to a backup chain   |
| **Multi-turn Dialogue Routing** | Dynamically choose response strategies based on conversation history or detected intent |


## ➕ Usage Examples
### Choosing Response Strategy Based on Conversation History

```python
from langchain_core.runnables import RunnableBranch

# Define the branching logic
from langchain_core.runnables import RunnableBranch

# Define the branching logic
branch = RunnableBranch(
    (lambda x: "complaint" in x["history"], complaint_handler),  # If user history contains "complaint", handle complaint
    (lambda x: "inquiry" in x["history"], inquiry_handler),    # If user history contains "inquiry", handle inquiry
    default_responder  # Default responder for other cases
)
```

### Intelligent Routing System Based on Input Type

```python
from langchain_core.runnables import RunnableBranch

# Step 1: Define a simple topic detection function
def detect_topic(input_text):
    if "weather" in input_text:
        return "weather"
    elif "news" in input_text:
        return "news"
    else:
        return "general"

# Step 2: Build the branching chain
branch_chain = RunnableBranch(
    (lambda x: detect_topic(x["input"]) == "weather", weather_chain),
    (lambda x: detect_topic(x["input"]) == "news", news_chain),
    general_chain  # Default: if no topic match, handle as general
)

# Step 3: Execute an example input
result = branch_chain.invoke({"input": "How is the weather in New York today?"})
```

> in RunnableBranch, the **condition part** only requires a **normal function** that **immediately returns True/False**. It is not part of the processing flow, so no need to wrap with RunnableLambda.



### Building an Intelligent Customer Service Routing System
**Automatically routes** user requests to the appropriate handler based on **detected request type**:

| Request Type    | Routed To            |
|-----------------|----------------------|
| Technical Issue | Technical Support Chain |
| Billing Issue   | Finance Chain         |
| Other Issues    | General QA Chain      |


```python
import os
from langchain_core.runnables import RunnableBranch
from langchain_core.runnables import RunnableLambda
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

os.environ["OPENAI_API_KEY"] = "*"  
# Define the LLM
model = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Step 2: Define Routing Condition Functions
# Define condition function for technical support questions
def is_tech_question(input: dict) -> bool:
    # Safely get the "input" field
    input_value = input.get("input", "")
    # Check if it contains technology-related keywords
    return "technology" in input_value or "system error" in input_value

# Define condition function for billing questions
def is_billing_question(input: dict) -> bool:
    # Safely get the "input" field
    input_value = input.get("input", "")
    # Check if it contains billing-related keywords
    return "bill" in input_value or "payment" in input_value
    
# Step 3: Define specialized chains for each request type
prompt_tech_support = ChatPromptTemplate.from_template("You are a Technical Support Agent. Please resolve the following technical issue: {input}")
technical_support_chain = prompt_tech_support | model

prompt_billing_support = ChatPromptTemplate.from_template("You are a Financial Support Agent. Please resolve the following billing issue: {input}")
billing_support_chain = prompt_billing_support | model

prompt_general_support = ChatPromptTemplate.from_template("You are a  General Customer Support. Please resolve the following issue: {input}")
general_support_chain = prompt_general_support | model


# Step 4: Build the branching logic
branch = RunnableBranch(
    (is_tech_question, technical_support_chain),       # Technical issue → handled by tech_chain
    (is_billing_question, billing_support_chain),      # Billing issue → handled by billing_chain
    general_support_chain                              # Default: handled by default_chain
)

# Step 5: Build the full chain
full_chain = RunnableLambda(lambda x: {"input": x}) | branch


# Step 6: Testing: Invoke the full chain
tech_result = full_chain.invoke("Failed to log in, show me system error")
print(tech_result)

bill_result = full_chain.invoke("cannot pay the bill?")
print(bill_result)

general_result = full_chain.invoke("whnere is the company address?")
print(general_result)
```

#### Key Principles

##### 1. Conditional Routing Logic

- `RunnableBranch` accepts a list of `(condition function, runnable)` pairs.
- **Execution order matters**: it **checks conditions sequentially**.
- **First matching condition wins**: once a condition returns `True`, the associated Runnable is executed immediately.
- If **none** of the conditions match, it will **execute the default branch** (if provided).

> **Summary:** Dynamic, prioritized routing just like classic `if-elif-else` structures.

##### 2. Input Handling

- The input to `RunnableBranch` **must be a dictionary** (e.g., `{"input": "user's message"}`).
- To ensure consistent input format,  
  we **wrap the raw user text** using `RunnableLambda` before passing it into the branch:

```python
full_chain = RunnableLambda(lambda x: {"input": x}) | branch
```

> This guarantees that all condition functions can safely access input["input"] without error.

##### 3. Chain Composition
- Each branch target (tech_chain, billing_chain, general_chain) is **an independent processing chain**.
- Once a branch is selected:
    - It **processes the input fully** (e.g., via specialized prompts and LLMs).
    - Its **output is directly returned** as the final result of the full chain.

> This makes each chain modular, reusable, and easy to maintain.


#### Data Flow

```text
User Raw Input ("cannot pay the bill?")
   ↓
RunnableLambda: Wrap as {"input": ""cannot pay the bill?"}
   ↓
RunnableBranch:
   ├── Check is_tech_question() ➔ False
   ├── else Check is_billing_question() ➔ True → billing_chain
   └── else → default_chain
   ↓
Selected chain (billing_chain) processes input
   ↓
Final Output returned
```

#### Logging Routing Decisions with `RunnableLambda`

when build the `full_chain`: 

```python
# Step 5: Define a simple logging function
def log_decision(input_data):
    print(f"[DEBUG] Branch Routing Check Input:{input_data}")
    return input_data  # Important: must return input unchanged for further processing

# Wrap the logging function into a RunnableLambda
log_chain_branch = RunnableLambda(log_decision) | branch

# Step 6: Build the full chain
full_chain = (
    RunnableLambda(lambda x: {"input": x})  # Normalize raw input into dictionary
    | log_chain_branch                      # Add logging before branch routing
)
```

##### Data Flow

```text
Raw User Input ("cannot pay the bill?")
    ↓
Wrap input {"input": "cannot pay the bill?"}
    ↓
Log input using RunnableLambda ([DEBUG] Branch Routing Check Input:{'input': 'cannot pay the bill?'})
    ↓
RunnableBranch:
   ├── Check is_tech_question() ➔ False
   ├── else Check is_billing_question() ➔ True → billing_chain
   └── else → default_chain
    ↓
Selected chain (billing_chain) processes input
    ↓
Final Output returned
```







<br>




---

🚀 Stay tuned for more insights into LangChain!



