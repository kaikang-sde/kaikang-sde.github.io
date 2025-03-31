---
title: LangChain ChatModel and ChatPromptTemplate
date: 2025-03-30
categories: [AI, LangChain]
tags: [LangChain, ChatModel, Prompt Engineering, Token Counting, ChatPromptTemplate]
author: kai
---

# 🚀 LangChain ChatModel: Conversational LLMs Designed for Multi-Turn Dialogues
In modern LLM development, **multi-turn dialogue** is essential — whether you’re building a chatbot, an assistant, or an agent. LangChain provides robust tools to support this through two powerful abstractions:

- `ChatModel`: A wrapper for conversational LLMs (e.g., GPT-3.5, GPT-4, Claude, Gemini)
- `ChatPromptTemplate`: A structured prompt builder for multi-role conversations

Together, they allow developers to create **context-aware, role-specific, and safe** interactions with LLMs.

## 🧑🏻‍💻 ChatModel 
A **ChatModel** is a type of large language model (LLM) **specialized for multi-turn conversations**.  
Unlike standard text generators, ChatModels are designed to understand **context, roles, and intent** — making them ideal for chatbots, assistants, and interactive AI applications.

Traditional LLMs like `text-davinci-003` are good at **single-shot generation**.  
But ChatModels like **`gpt-4`**, **`claude-2`**, or **`gemini-pro`** are optimized for **dialogue flow**, allowing them to:

- Maintain context  
- Follow instructions across turns  
- Role-play and behave according to predefined personas  
- Adapt tone and understand user emotion

### Core Features of ChatModels

| Feature             | Description                                                                 | Real-World Example                                                  |
|---------------------|-----------------------------------------------------------------------------|----------------------------------------------------------------------|
| **Context Awareness** | Maintains memory of previous turns, handles pronouns & references           | User: “What is quantum computing?” → AI explains → User: “How is *it* used?” |
| **Role Consistency** | Can be set to behave like a specific persona or professional role            | AI as a medical assistant → avoids giving diagnoses, sticks to health facts |
| **Intent Recognition** | Detects underlying intent like complaints, FAQs, or small talk               | User: “Where is my order?” → AI tags it as a delivery issue, escalates if needed |
| **Emotion Sensitivity** | Detects frustration, excitement, confusion — and adjusts tone accordingly   | User sounds upset → AI replies empathetically: “Sorry for the inconvenience…” |
| **Safety Filtering** | Avoids generating harmful, biased, or inappropriate content                  | Request for violent content → AI rejects: “I’m not able to help with that.” |


### ChatModel vs Traditional Text Model  

| 📌 Criteria         | 💬 **ChatModel**                                 | 📄 **Traditional Text Model** (`text-davinci-003`, etc.)         |
|---------------------|--------------------------------------------------|------------------------------------------------------------------|
| **Primary Use Case** | Multi-turn conversational AI                     | One-shot text generation                            |
| **Input Format**     | Structured message list (with roles like `Human`, `AI`, `System`) | Plain text prompt                                               |
| **Context Handling** | Automatically tracks dialogue history and references | Developer must manually include previous turns in prompt       |
| **Output Control**   | Built-in content filtering, role enforcement     | Must rely on prompt design to enforce constraints               |
| **Applications**     | Chatbots, virtual assistants, customer support   | Story generation, code completion, data transformation          |


### Role-Based Messaging in ChatModels
Modern ChatModels like **GPT-3.5-turbo**, **GPT-4**, and **Claude** support structured conversations using **role-based messages**.

Instead of plain prompts, messages are categorized by **roles**, allowing fine-grained control over behavior, context, and interaction flow.

| Role Type   | Identifier | Purpose / Function                                      | Example Usage                            |
|-------------|------------|----------------------------------------------------------|------------------------------------------|
| **System**  | `system`   | Define AI identity, behavior，scope, and tone                      | `("system", "You are a medical assistant...")` |
| **User**    | `user`     | Represents real user input: questions, feedback, commands| `("user", "How can I relieve a headache?")` |
| **Assistant** | `assistant` | Captures model's past replies to maintain context       | `("assistant", "You can take ibuprofen...")`   |


### OpenAI ChatModel: Role-Based Multi-Turn Conversation Example
```python
# Multi-turn conversation using role-based format
messages = [
    {"role": "system", "content": "You are a movie recommendation assistant."},
    {"role": "user", "content": "I like science fiction films. Recommend three classics."},
    {"role": "assistant", "content": "1. Blade Runner 2049. 2. Interstellar. 3. The Matrix."},
    {"role": "user", "content": "Who is the lead actor in the second one?？"}  # Follow-up question based on context
]

response = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=messages
)

print(response.choices[0].message.content)
# Output: The lead actors in Interstellar are Matthew McConaughey and Anne Hathaway…
```

## 🔢 Token Counting in Multi-Turn Chat with LLMs
When working with **ChatModels** such as `gpt-3.5-turbo` or `gpt-4`, understanding **how tokens accumulate** in multi-turn dialogue is essential for performance, cost control, and avoiding context window overflow.

### What Contributes to Token Count?

In multi-turn chat scenarios, the model processes a growing amount of **context** that typically includes:

1. **User's previous inputs**  
2. **Model's historical replies**  
3. **System prompt** (e.g., role definition or instructions)

Each of these contributes to the **total token count** in the current request.


### Token Accumulation Over Rounds (Example)
Assume:
- Each **user input** = 50 tokens  
- Each **model reply** = 100 tokens  

### Token Calculation by Turn:

| Round | New User Input | New Model Output | Previous Context | Total Tokens Sent |
|-------|----------------|------------------|------------------|--------------------|
| 1     | 50             | 100              | 0                | 50 + 100 = 150     |
| 2     | 50             | 100              | 150              | 50 + 100 + 150 = 300 |
| 3     | 50             | 100              | 300              | 50 + 100 + 300 = 450 |

With each turn, tokens **accumulate linearly**, making it crucial to track context length over time.


### Context Window Limit

Every LLM has a **maximum token limit**, known as the **context window**. If the token count exceeds this window, older messages may be **truncated** or **ignored**.

| Model             | Context Window (Max Tokens) |
|------------------|-----------------------------|
| GPT-3.5-turbo     | 4,096 tokens                |
| GPT-4 (8k)        | 8,192 tokens                |
| GPT-4 (32k)       | 32,768 tokens               |
| Claude-2          | 100k tokens (Anthropic)     |

⚠️ **If your token count exceeds the window**, OpenAI will automatically **drop earlier turns** or **raise an error**.


## 💡 `ChatPromptTemplate` in LangChain
When working with modern ChatModels like **GPT-3.5** and **GPT-4**, prompt structure matters more than ever.  
LangChain’s `ChatPromptTemplate` offers a **clean, structured, and role-aware** way to build prompts for conversational models.


### Core Differences from Traditional PromptTemplate

| Feature                        | `PromptTemplate`             | `ChatPromptTemplate`                        |
|-------------------------------|-------------------------------|---------------------------------------------|
| Input Format                  | Single string with variables  | Structured message list (role + content)    |
| Role Support                  | ❌                            | ✅ `system`, `user`, `assistant`             |
| ChatModel Compatibility       | Partial (manual wrapping)     | ✅ Native fit for GPT-3.5, GPT-4, Claude     |
| Context Maintenance           | Manual                        | ✅ Natural multi-turn support                |



### Message Type System Overview
LangChain's `ChatPromptTemplate` framework provides a modular way to build structured, **multi-role prompts** using specialized message template classes.

These message types reflect different roles in a conversation — such as **system setup**, **user input**, and **AI replies** — and make prompt design clear and maintainable.


| Message Template Class              | Role Type     | Purpose & Usage                                 |
|------------------------------------|---------------|--------------------------------------------------|
| `SystemMessagePromptTemplate`      | `system`      | Define the AI's behavior, persona, or rules      |
| `HumanMessagePromptTemplate`       | `user/human`  | Accept user input or questions                   |
| `AIMessagePromptTemplate`          | `assistant/ai`| Represent prior responses from the model         |
| `ChatPromptTemplate`               | **container** | Organize and format multiple messages into one prompt |


### Mastering two common methods from `ChatPromptTemplate`
#### `.from_template()` 
- Use Case: **Single-role message creation**
- This method is ideal for **constructing individual message templates**. It is commonly used for:
    - A single `system` message
    - A specific `user` input message
    - A placeholder to be **combined later** via `from_messages()`

```python
from langchain.prompts.chat import ChatPromptTemplate

# Create a prompt template, include system, human and ai messages. Using a list of tuples to define role and content.
# Insert name and user_input into template dynamically
chat_template = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant. Your name is {name}."),
    ("human", "Hi, how are you doing?"),
    ("ai", "I'm doing well, thank you!"),
    ("human", "{user_input}")
])

messages = chat_template.format_messages(
    name="Bob", 
    user_input="What is your favorite programming language?"
)

print(messages)
# Output:
# [SystemMessage(content='You are a helpful assistant. Your name is Bob.', additional_kwargs={}, response_metadata={}), 
#  HumanMessage(content='Hi, how are you doing?', additional_kwargs={}, response_metadata={}), 
#  AIMessage(content="I'm doing well, thank you!", additional_kwargs={}, response_metadata={}), 
#  HumanMessage(content='What is your favorite programming language?', additional_kwargs={}, response_metadata={})]
```


#### `.from_messages()`
- Use Case: **Multi-role, multi-turn dialogue**
- This method is designed for **constructing a structured conversation, supporting multiple roles** such as:
	- **system**: setup and constraints
	- **human**: user questions or inputs
	- **ai**: assistant’s historical replies
- You can provide:
	- **Tuples**: ("role", "message")
	- Or **message templates**: SystemMessagePromptTemplate, etc.

| Method | Scenario | Flexibility | Code Complexity |
|--------|----------|-------------|-----------------|
| `.from_template()` | One message, one role (e.g., only user) | Low | Simple |
| `.from_messages()` | Multi-role, multi-turn dialogue | High | Moderate(need to define a list) |

```python
from langchain_core.prompts import (
    ChatPromptTemplate, 
    SystemMessagePromptTemplate, 
    HumanMessagePromptTemplate
)

# Create single message template, using fine-grained template classes (e.g. SystemMessagePromptTemplate) to define a single message.
system_template = SystemMessagePromptTemplate.from_template(
    "your are a {role}, please use {language} to answer."
)

user_template = HumanMessagePromptTemplate.from_template("{question}")

# Combine the templates into a single prompt template
chat_template = ChatPromptTemplate.from_messages([
    system_template,
    user_template
])

# use the template
messages = chat_template.format_messages(
    role="translator", 
    language="English",
    question="Translate '我爱Python' into English."
)
print(messages)

# Output:
# [SystemMessage(content='your are a translator, please use English to answer.', additional_kwargs={}, response_metadata={}), 
#  HumanMessage(content="Translate '我爱Python' into English.", additional_kwargs={}, response_metadata={})]
```

## 🤖 LangChain ChatModel Multi-Scenario Practice (LangChain + OpenAI)

**Execution Flow: Prompt Construction Logic:**<br>
`from_template() → from_messages() → format_messages() → pass to ChatModel`

### 1. Domain Expert Prompting 

```python
from langchain_core.messages import SystemMessage, HumanMessage
from langchain_openai import ChatOpenAI
import os


os.environ["OPENAI_API_KEY"] = "*"  # replace with your actual key


# 1. Create a prompt list with fine-tuned Message Template Classes
messages = [
    SystemMessage(content="You are a Java programmer with 10 yesrs of experience, plese answer questions about Java."),
    HumanMessage(content="What is volatile keyword?")
]

# 2. Create LLM instance
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# 3. Call LLM
response = llm.invoke(messages)
print(response.content)

# Output:
# In Java, the `volatile` keyword is used as a modifier for instance variables. When a variable is declared as `volatile`, it tells the Java 
# Virtual Machine (JVM) that the value of this variable may be changed by different threads. The `volatile` keyword ensures that:
# ... more content
```

### 2. Domain Expert Prompting with Parameters
Creating **reusable prompt templates** with dynamic parameters is a **powerful pattern** in LangChain. It allows you to define expert personas on demand and adjust tone or explanation style flexibly.

**Dynamically generate expert-style system prompts like:**

> “You are a professional **Machine Learning** expert. Please answer using **metaphors and examples**.”

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

from langchain.prompts import (  
    ChatPromptTemplate,  
    SystemMessagePromptTemplate,  
    HumanMessagePromptTemplate  
) 
import os


os.environ["OPENAI_API_KEY"] = "*"  # replace with your actual key


# 1. Create system prompt
system_templet = SystemMessagePromptTemplate.from_template(
    "You are a professional {domain} expert, please answer the question with {style_guide}."
)

# 2. Create human prompt
human_templet = HumanMessagePromptTemplate.from_template(
    "Please explain: {concept}."
)

# 3. Combine system prompt and human prompt
chat_prompt = ChatPromptTemplate.from_messages(
    [system_templet, human_templet]
)

# 4. Format output
messages = chat_prompt.format_prompt(
    domain="data science",
    style_guide="using metaphors and examples",
    concept="overfitting"
)

# 5. Create LLM instance
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# 6. Call LLM
response = llm.invoke(messages)
print(response.content)

# Output:
# Imagine you are a chef preparing a special dish for a competition. ... More 
```


### 3. Compliance-Aware AI Customer Support Assistan
In enterprise AI applications — especially **customer support bots** — **compliance** is critical. You may need to prevent certain types of disclosures or automatically escalate to a human agent under specific conditions.

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
import os


os.environ["OPENAI_API_KEY"] = "*"  # replace with your actual key

# # 1. Create prompt template
compliance_template = ChatPromptTemplate.from_messages([
    (
        "system", """You are a digital assistant for {company}, please follow these rules:
        1. Do not disclose internal system names
        2. Do not provide software/financial advice
        3. if {transfer_cond} then transfer to human"""
    ), 
    ("human", "[{user_level} user]: {query}")
])

messages = compliance_template.format_messages(
    company="Microsoft", 
    transfer_cond="Software consultation, payment issues", 
    user_level="VIP", 
    query="Which software should I buy?"
)

# 2. Create LLM instance
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# 3. Call LLM
response = llm.invoke(messages)
print(response.content)

# Output:
# I'm unable to provide specific software recommendations. However, I can help you with general information about software categories or features. If you have specific needs or questions, feel free to ask! For personalized software consultation, I recommend reaching out to a human representative.
```





<br>



---

🚀 Stay tuned for more insights into LangChain!



