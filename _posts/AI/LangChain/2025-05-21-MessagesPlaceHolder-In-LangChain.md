---
title: MessagesPlaceholder in LangChain
date: 2025-05-21
categories: [AI, LangChain]
tags: [LangChain, MessagesPlaceholder, ChatPromptTemplate, LLM, Memory]
author: kai
permalink: /posts/ai/langchain/messagesplaceholder-in-langchain/
---

# 🚀 MessagesPlaceholder in LangChain

In LangChain, `MessagesPlaceholder` is a **special placeholder used in `ChatPromptTemplate`** to **dynamically insert a list of structured messages**—such as conversation history, tool outputs, or multi-role messages—into the prompt.

It plays a **critical role in managing context for multi-turn conversations** by allowing structured injection of prior dialogue and supporting dynamic prompt assembly.


## ✅ Key Use Cases

### Multi-role Conversations
Inject messages from different roles—`system`, `user`, `assistant`—to simulate realistic chat interactions.

### Historical Context Injection
Pass full or partial conversation history into the model to **maintain continuity and coherence** across multiple turns.

### Modular Prompt Composition
Easily combine dynamic and static prompt components (e.g., retrieved documents, user memory) within the same conversation.


## 🔄 `PromptTemplate` vs. `ChatPromptTemplate`

| Feature                  | `PromptTemplate`                        | `ChatPromptTemplate` with `MessagesPlaceholder`      |
|--------------------------|-----------------------------------------|-------------------------------------------------------|
| Input Format             | Single string template                  | Structured message objects (`HumanMessage`, etc.)     |
| Multi-turn Chat Support  | Not ideal                            | Designed for multi-turn conversations               |
| Dynamic History Insertion| Manual string formatting             | Uses `MessagesPlaceholder(variable_name="...")`     |


## 🧪 Example Usage

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])
```

**In this setup:**

- **"system"** provides initial role behavior.
- **MessagesPlaceholder("history")** dynamically injects prior messages (e.g., stored by a memory module).
- **{input}** receives the user’s current message.


## 🧠 `MessagesPlaceholder` vs Traditional List

When building conversational agents with LangChain, managing chat history is essential. Two common approaches are:

- **Using `MessagesPlaceholder`** with `ChatPromptTemplate`
- **Manually appending history as a list of strings**

Here's a detailed comparison of the two methods:

| Feature                   | `MessagesPlaceholder`                      | Traditional List Storage              |
|---------------------------|---------------------------------------------|----------------------------------------|
| **Dynamic Insertion**     | Supports real-time dynamic message control | Fixed sequence, harder to update     |
| **Role Awareness**        | Differentiates `system`, `human`, `AI` messages | Stores all as plain strings          |
| **Memory Integration**    | Seamlessly syncs with LangChain Memory modules | Manual state tracking required       |
| **Structured Operation**  | Supports metadata and typed messages (`BaseMessage`) | Pure text, no structure              |


## 📌 Step-by-Step Guide
When building conversational agents with **LangChain**, `MessagesPlaceholder` is essential for injecting dynamic message history into your chat prompts. 

### Step 1: Define the Prompt Template with a Placeholder
To support multi-turn conversations, use `ChatPromptTemplate` and declare a placeholder using `MessagesPlaceholder`.
- The **variable_name** must match the **memory_key** used in your memory object.
- The **order of messages** matters — this layout ensures historical context appears before the new input.

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])
```


### Step 2: Binding Memory
To maintain contextual awareness in multi-turn conversations, LangChain provides memory modules that can be seamlessly integrated with your chat prompt using `MessagesPlaceholder`.

- LangChain provides built-in ways to **automatically inject chat history** into your prompts using memory modules, such as `ConversationBufferMemory` or `ConversationSummaryMemory`.


```python
from langchain.memory import ConversationBufferMemory

# Create a memory instance
memory = ConversationBufferMemory(
    memory_key="chat_history",  # MUST match the variable_name in MessagesPlaceholder
    return_messages=True        # Ensures memory returns structured message objects
)
```

- memory_key="chat_history" must **exactly match** the MessagesPlaceholder(variable_name="chat_history").
- return_messages=True ensures the messages are returned as structured BaseMessage objects, not plain strings.
- This is critical for LangChain’s ChatPromptTemplate to correctly format the prompt.


### Step 3: Chain-Based Invocation with Automatic History Management
To enable **automatic chat history injection**, use either `LLMChain` or `RunnableWithMessageHistory`. 

```python
from langchain.chains import LLMChain
from langchain_community.chat_models import ChatOpenAI

llm = ChatOpenAI()

chain = LLMChain(
    llm=llm,
    prompt=prompt,
    memory=memory,
    verbose=True
)
```

## 🤷🏻 Use Case: Stateful LLM Translation Assistant

```python
import os
from langchain.memory import ConversationBufferMemory
from langchain_openai import ChatOpenAI
from langchain.chains import LLMChain
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder


os.environ["OPENAI_API_KEY"] = "*"

llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Memory Initialization
memory = ConversationBufferMemory(
    memory_key="chat_history",  # Must match the placeholder variable in the prompt
    return_messages=True
)

# Define Prompt with MessagesPlaceholder
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful translation assistant. Use prior conversation to improve translations."),
    MessagesPlaceholder(variable_name="chat_history"),
    ("human", "Please translate the following: {input}")
])

# Build the Chain with Memory Support
chain = LLMChain(llm=llm, prompt=prompt, memory=memory)

# Simulate Multi-Turn Conversation
user_inputs = [
    "Translate 'Hello' to Chinese",
    "Use the translation in a sentence",
    "Now translate 'Goodbye'"
]

for query in user_inputs:
    response = chain.invoke({"input": query})
    print(f"User: {query}")
    print(f"AI: {response['text']}\n")
    print("Current Chat History:", memory.load_memory_variables({})["chat_history"], "\n")


# memory = ConversationBufferMemory(
#   chain = LLMChain(llm=llm, prompt=prompt, memory=memory)
# User: Translate 'Hello' to Chinese
# AI: 'Hello' in Chinese is translated as '你好' (nǐ hǎo).

# Current Chat History: [HumanMessage(content="Translate 'Hello' to Chinese", additional_kwargs={}, response_metadata={}), AIMessage(content="'Hello' in Chinese is translated as '你好' (nǐ hǎo).", additional_kwargs={}, response_metadata={})] 

# User: Use the translation in a sentence
# AI: In Chinese, "Use the translation in a sentence" can be translated as "在句子中使用翻译" (zài jùzi zhōng shǐyòng fānyì).

# Current Chat History: [HumanMessage(content="Translate 'Hello' to Chinese", additional_kwargs={}, response_metadata={}), AIMessage(content="'Hello' in Chinese is translated as '你好' (nǐ hǎo).", additional_kwargs={}, response_metadata={}), HumanMessage(content='Use the translation in a sentence', additional_kwargs={}, response_metadata={}), AIMessage(content='In Chinese, "Use the translation in a sentence" can be translated as "在句子中使用翻译" (zài jùzi zhōng shǐyòng fānyì).', additional_kwargs={}, response_metadata={})] 

# User: Now translate 'Goodbye'
# AI: 'Goodbye' in Chinese is translated as '再见' (zài jiàn).

# Current Chat History: [HumanMessage(content="Translate 'Hello' to Chinese", additional_kwargs={}, response_metadata={}), AIMessage(content="'Hello' in Chinese is translated as '你好' (nǐ hǎo).", additional_kwargs={}, response_metadata={}), HumanMessage(content='Use the translation in a sentence', additional_kwargs={}, response_metadata={}), AIMessage(content='In Chinese, "Use the translation in a sentence" can be translated as "在句子中使用翻译" (zài jùzi zhōng shǐyòng fānyì).', additional_kwargs={}, response_metadata={}), HumanMessage(content="Now translate 'Goodbye'", additional_kwargs={}, response_metadata={}), AIMessage(content="'Goodbye' in Chinese is translated as '再见' (zài jiàn).", additional_kwargs={}, response_metadata={})] 
```


### Build the Chain with LECL 
```python
chain = (
    RunnablePassthrough.assign(
        chat_history = lambda _ : memory.load_memory_variables({})['chat_history']
    )
    | prompt
    | llm
    | StrOutputParser()    
)
```










<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



