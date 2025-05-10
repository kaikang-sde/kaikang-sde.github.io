---
title: Long-Term Memory in LLM Applications
date: 2025-05-23
categories: [AI, LangChain]
tags: [LangChain, LLM, Memory, AI Agents, Vector Database]
author: kai
---

# 🚀 Long-Term Memory in LLM Applications
Large Language Models (LLMs) are inherently **stateless** — they do not remember past interactions once a session ends. For real-world AI applications like virtual assistants, tutoring systems, or customer service bots, **long-term memory** is essential to maintain personalization, consistency, and intelligence across sessions.


## 🔑 Core Requirements of Long-Term Memory

### 1. Cross-Session Memory Persistence
Key information shared by users over multiple sessions (e.g., preferences, goals, past queries) must be stored **permanently**.

### 2. Multi-User Isolation
Memory data should be strictly **isolated by user ID** to prevent data leakage or context confusion between users.

### 3. Efficient Retrieval
Historical memory should be **retrievable within milliseconds** to ensure smooth conversational flow. Delays degrade user experience.

### 4. Dynamic Updates
Memory must support:
- **Incremental additions** (new facts, updates)
- **Aging strategies** (decay, expiration)
- **Context pruning** (removal of stale info)


## ✅ Solution Approach

### 1. Persistent Storage
Use a **database or vector store** (e.g., MongoDB, Redis, Milvus) to store messages, user profiles, summaries, or embeddings.

> Examples: 
> - Store conversation summaries using `ConversationSummaryMemory`
> - Store embeddings with vector DBs like **Milvus**, **Pinecone**, or **FAISS**


### 2. Efficient Semantic Retrieval
Use **semantic search** or **metadata filtering** to find relevant historical context:
- Match based on user intent
- Retrieve only relevant past queries or facts
- Integrate with retrieval-augmented generation (RAG)


### 3. Dynamic and Incremental Update
Continuously update memory as the conversation progresses:
- Add new facts
- Rewrite outdated context
- Implement decay mechanisms for low-use memory items


## 🧠 Persistent LLM Memory with Redis in LangChain
When building production-ready LLM applications, maintaining **fast and reliable memory** is crucial. LangChain provides an out-of-the-box solution: `RedisChatMessageHistory`, which stores conversation history in **Redis** in a structured, low-latency manner.

**LangChain Memory Docs:** [https://python.langchain.com/docs/integrations/memory/](https://python.langchain.com/docs/integrations/memory/) 

### RedisChatMessageHistory

```text
        User Input
            │
            ▼
    Is This a New Session?
            │
    ┌───────┴─────────┐
    │                 │
   Yes                No
    │                 │
    ▼                 ▼
Create Redis       Load Redis
Chat Session       Chat History
    │                 │
    ▼                 ▼
Generate LLM Response (via LangChain Agent)
            │
            ▼
Encrypt and Store in Redis
            │
            ▼
Asynchronously Backup to S3
```

`RedisChatMessageHistory` is a built-in **LangChain memory integration** that uses **Redis** to persist chat messages.

It is designed for scenarios where:

- You need **real-time memory access** for fast-response LLMs
- Memory must **persist across sessions**
- Multiple conversations or users are running in parallel

```python
class RedisChatMessageHistory(BaseChatMessageHistory):
    """Redis-based implementation of chat message history.

    This class provides a way to store and retrieve chat message history using Redis.
    It implements the BaseChatMessageHistory interface and uses Redis JSON capabilities
    for efficient storage and retrieval of messages.

    Attributes:
        redis_client (Redis): The Redis client instance.
        session_id (str): A unique identifier for the chat session.
        key_prefix (str): Prefix for Redis keys to namespace the messages.
        ttl (Optional[int]): Time-to-live for message entries in seconds.
        index_name (str): Name of the Redis search index for message retrieval.

    Args:
        session_id (str): A unique identifier for the chat session.
        redis_url (str, optional): URL of the Redis instance. Defaults to "redis://localhost:6379".
        key_prefix (str, optional): Prefix for Redis keys. Defaults to "chat:".
        ttl (Optional[int], optional): Time-to-live for entries in seconds.
                                       Defaults to None (no expiration).
        index_name (str, optional): Name of the Redis search index.
                                    Defaults to "idx:chat_history".
        redis (Optional[Redis], optional): Existing Redis client instance. If provided,
                                           redis_url is ignored.
        **kwargs: Additional keyword arguments to pass to the Redis client.

    Example:
        .. code-block:: python

            from langchain_redis import RedisChatMessageHistory
            from langchain_core.messages import HumanMessage, AIMessage

            history = RedisChatMessageHistory(
                session_id="user123",
                redis_url="redis://localhost:6379",
                ttl=3600  # Messages expire after 1 hour
            )

            # Add messages to the history
            history.add_message(HumanMessage(content="Hello, AI!"))
            history.add_message(
              AIMessage(content="Hello, human! How can I assist you today?")
            )

            # Retrieve all messages
            messages = history.messages
            for message in messages:
                print(f"{message.type}: {message.content}")

            # Clear the history
            history.clear()

    Note:
        - This class uses Redis JSON for storing messages, allowing for efficient
          querying and retrieval.
        - A Redis search index is created to enable fast lookups and potential future
          search needs over the chat history.
        - If TTL is set, message entries will automatically expire after the
          specified duration.
        - The session_id is used to group messages belonging to the same conversation
          or user session.
    """
```

### Key Features & Benefits

| Feature               | Description                                                                 |
|-----------------------|-----------------------------------------------------------------------------|
| Ultra-low latency   | Sub-millisecond access times (<1ms) ensure snappy LLM response              |
| Rich Data Structures | Stores structured messages and metadata using JSON                         |
| Persistent Storage | Leverages Redis RDB + AOF for durability and crash recovery                  |
| Cluster Support     | Supports horizontal scaling for high-concurrency production systems         |
| TTL Expiration      | Automatic message cleanup using TTL (e.g., auto-delete after 30 days)       |



### Under the Hood: JSON Storage via ReJSON

LangChain leverages the [ReJSON](https://redis.io/json/) Redis module, which allows messages to be stored and queried as JSON documents. This provides:

- Native support for complex message types
- Efficient querying of message content
- Metadata tagging and retrieval flexibility


### Redis Stack
**Redis Stack** is an official enhanced distribution of Redis, preloaded with modern modules for AI-driven applications, analytics, and real-time search.

It unifies several advanced features under a single deployment, making it ideal for large language model (LLM) memory management and chat history search in LangChain or similar frameworks.


#### Key Benefits

- **Out-of-the-box functionality**: No need to install each module manually.
- **Developer-friendly**: Tailored for full-text search, structured JSON access, graph queries, and time-series analytics.
- **Unified API**: Access advanced features through standard Redis CLI or client libraries.


#### Modules in Redis Stack

| Module         | Feature Description                              |
|----------------|--------------------------------------------------|
| `RediSearch`   | Full-text search & secondary indexing             |
| `RedisJSON`    | Native storage and querying of JSON documents     |
| `RedisGraph`   | Graph DB engine using property graph model        |
| `RedisTimeSeries` | Efficient handling of time-series data        |
| `RedisBloom`   | Probabilistic structures (Bloom filter, Count-Min) |
| `RedisInsight` | GUI-based Redis monitoring and visualization      |


#### When Should Choose Redis Stack
Redis Stack is the ideal choice for AI-enhanced applications, especially when working with **large language models (LLMs)** and intelligent agents. 

##### 1. Full-Text Search

Use **RediSearch** when your application requires:

- Searching chat logs for keywords or phrases
- Implementing semantic search in vector-augmented RAG pipelines
- Filtering large message datasets with speed and precision


##### 2. Native JSON Document Manipulation

Use **RedisJSON** if your system stores:

- Structured dialogue histories from LLM agents
- Nested user interaction trees or stateful context
- Dynamic session data across users and use cases

```json
{
  "user_id": "u_001",
  "session": "sess_abc",
  "messages": [
    { "role": "user", "text": "What is Milvus?" },
    { "role": "assistant", "text": "Milvus is a vector DB..." }
  ]
}
```

##### 3. Time Series Analytics

Use **RedisTimeSeries** when you need to:
- Track frequency of user queries
- Monitor token usage per user/session over time
- Analyze chat load and memory hits historically


##### 4. Graph-Based Reasoning

Use **RedisGraph** to:
- Model complex user-topic-intent relationships
- Visualize conversation flow across nodes
- Query social network data or agent decision paths


##### 5. Simplified Modular Development

Redis Stack prepackages all modern Redis modules, avoiding tedious manual installations like:

```text
# Instead of manually adding each module...
redis-server --loadmodule redisearch.so --loadmodule redisjson.so
```

#### LangChain Integration Note
If you’re building a LangChain + Redis integration:
- Do not use the legacy Redis server.
- Redis Stack is required to support full-text search, JSON field access, and hybrid queries.

Otherwise, you’ll encounter feature support errors (e.g., unsupported JSON or search operations).

#### Migration Tips
**If you previously used plain Redis, you can:**
- Uninstall it and install Redis Stack (recommended)
    ```text
    # Uninstall
    docker stop id
    docker rm id

    # install 
    docker run -d \
        --name redis-stack \
        --restart=always \
        -p 6379:6379 \
        -p 8001:8001 \
        -e REDIS_ARGS="--requirepass XXX" \
        redis/redis-stack:latest
    ```
- Change ports (e.g., run Redis Stack on 6380 instead of 6379)
- Enable password protection or ACL rules

**Redis Stack ≥ v7.x is recommended and supports:**
- AOF/RDB persistence
- RedisInsight for monitoring
- CLI and Python clients for AI workloads


## 👀 Use Cases:

**Install Required Packages:** `pip install langchain-redis==0.2.0 redis==5.2.1`

### Basic Testing: Redis-Powered Memory for LangChain Agents 
```python
from langchain_redis import RedisChatMessageHistory
import os

REDIS_URL = os.getenv("REDIS_URL", "redis://:****@54.163.61.180:6379")

history = RedisChatMessageHistory(session_id="user_123", redis_url=REDIS_URL)

# Add user and AI messages
history.add_user_message("Hello, AI assistant!")
history.add_ai_message("Hello! How can I assist you today?")

# Retrieve and display stored messages
print("Chat History:")
for message in history.messages:
    print(f"{type(message).__name__}: {message.content}")

# Chat History:
# HumanMessage: Hello, AI assistant!
# AIMessage: Hello! How can I assist you today?
```

![Redis Stack Testing Human Msg](/assets/img/posts/AI/LangChain/RedisStackTestingHumanMsg.png)
![Redis Stack Testing AI Msg](/assets/img/posts/AI/LangChain/RedisStackTestingAIMsg.png)


### Multi-Session
```python
import os
from langchain_core.chat_history import BaseChatMessageHistory
from langchain_core.messages import HumanMessage
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI
from langchain_redis import RedisChatMessageHistory


os.environ["OPENAI_API_KEY"] = "*"
os.environ["REDIS_URL"] = "redis://:****@54.163.61.180:6379"

llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

history = RedisChatMessageHistory(
    session_id="user_123", redis_url=os.environ["REDIS_URL"])

# Define session-based history factory
def get_redis_history(session_id: str) -> BaseChatMessageHistory:
    return RedisChatMessageHistory(session_id, redis_url=os.environ["REDIS_URL"])

# Define the prompt template
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a KK's Classroom AI assistant specialized in {ability}, answer questions within 30 words."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}"),
])

# Create the runnable chain (prompt -> model)
runnable = prompt | llm

# Wrap with session-aware memory manager
with_message_history = RunnableWithMessageHistory(
    runnable,
    get_redis_history,
    input_messages_key="input",
    history_messages_key="history",
)

# Simulate multi-turn conversation for session 'user_123'
response1 = with_message_history.invoke(
    {"ability": "Java Development",
     "input": HumanMessage("What is jvm?")},
    config={"configurable": {"session_id": "user_123"}},
)
print(f"Response 1: {response1.content}\n")

response2 = with_message_history.invoke(
    {"ability": "Java Development", 
     "input": HumanMessage("Please reanswer the question.")},
    config={"configurable": {"session_id": "user_123"}},
)
print(f"Response 2: {response2.content}\n")

# Response 1: JVM, or Java Virtual Machine, is an engine that enables Java bytecode execution, providing platform independence and memory management for Java applications.

# Response 2: JVM (Java Virtual Machine) is a runtime environment that executes Java bytecode, allowing Java programs to run on any device with a JVM implementation.
```

![Redis Stack Testing Same SessionId](/assets/img/posts/AI/LangChain/RedisStackSameSessionId.png)


### Multi-User, Multi-Session
```python
import os
from langchain_core.runnables import ConfigurableFieldSpec
from langchain_core.messages import HumanMessage
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI
from langchain_redis import RedisChatMessageHistory


os.environ["OPENAI_API_KEY"] = "*"
os.environ["REDIS_URL"] = "redis://:****@54.163.61.180:6379"

llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Define the user_id and session_id based history factory
def get_redis_history(user_id: str, session_id: str):
    uni_key = user_id+"_"+session_id

    return RedisChatMessageHistory(
        session_id=uni_key,
        redis_url=os.environ["REDIS_URL"])

# Define the prompt template
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a KK's Classroom AI assistant specialized in {ability}, answer questions within 30 words."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}"),
])

# Create the runnable chain (prompt -> model)
runnable = prompt | llm

# Wrap with session-aware memory manager
with_message_history = RunnableWithMessageHistory(
    runnable,
    get_redis_history,
    input_messages_key="input",
    history_messages_key="history",
    history_factory_config=[
        ConfigurableFieldSpec(
            id="user_id",
            annotation=str,
            name="UserId",
            description="Unique identifier for the user.",
            default="",
            is_shared=True,
        ),
        ConfigurableFieldSpec(
            id="session_id",
            annotation=str,
            name="SessionId",
            description="Unque identifier for the conversation.",
            default="",
            is_shared=True,
        ),
    ]
)
# Simulate multi-turn conversation for session 'user_123'
response1 = with_message_history.invoke(
    {"ability": "Java Development",
     "input": HumanMessage("What is jvm?")},
    config={"configurable": { "user_id" : "user_1" , "session_id" : "session_1" }},
)
print(f"Response 1: {response1.content}\n")
print(get_redis_history("user_1","session_1").messages, end="\n")


response2 = with_message_history.invoke(
    {"ability": "Java Development",
     "input": HumanMessage("Please reanswer the question.")},
    config={"configurable": { "user_id" : "user_2" , "session_id" : "session_1" }},
)
print(f"Response 2: {response2.content}\n")
print(get_redis_history("user_2","session_1").messages, end="\n")

# Response 1: JVM stands for Java Virtual Machine. It's an engine that enables Java bytecode to run on any device, providing platform independence and memory management.
# [HumanMessage(content='What is jvm?', additional_kwargs={}, response_metadata={}), AIMessage(content="JVM stands for Java Virtual Machine. It's an engine that enables Java bytecode to run on any device, providing platform independence and memory management.", additional_kwargs={'refusal': None}, response_metadata={})]

# Response 2: Please provide the question you need reanswered, and I'll assist you!
# [HumanMessage(content='Please reanswer the question.', additional_kwargs={}, response_metadata={}), AIMessage(content="Please provide the question you need reanswered, and I'll assist you!", additional_kwargs={'refusal': None}, response_metadata={})]
```
![Redis Stack Testing Same SessionId](/assets/img/posts/AI/LangChain/RedisStackSameUserIdSessionId.png)






<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



