---
title: Isolating Memory in LLMs with RunnableWithMessageHistory
date: 2025-05-22
categories: [AI, LangChain]
tags: [LangChain, RunnableWithMessageHistory, Multi-Session, Multi-User]
author: kai
permalink: /posts/ai/langchain/isolating-memory-in-llms-with runnablewithmessagehistory/
---

# 🚀 Isolating Memory in LLMs with RunnableWithMessageHistory

When multiple users interact with an AI system simultaneously, maintaining **isolated memory per session** is essential to avoid:

- **Data Leakage:** Conversation from User A appearing in User B’s session.
- **Context Overwriting:** Historical messages from different sessions getting mixed.
- **Privacy Concerns:** Sensitive user data not being properly compartmentalized.


## 🎯 Real-World Use Cases

| Scenario         | Memory Isolation Requirement                        |
|------------------|------------------------------------------------------|
| Customer Support | Each user query must be tracked separately          |
| EdTech Tutor     | Each student session should persist personal history |
| Medical Assistant| Patient records must remain confidential and secure  |



## 🧩 The LangChain Multi-Session Solution: `RunnableWithMessageHistory`
LangChain offers `RunnableWithMessageHistory` — a powerful abstraction to **automate session-based memory isolation** while injecting message history into every LLM execution flow.

- `from langchain_core.runnables.history import RunnableWithMessageHistory`

- **Session-Aware Memory:** Keeps history unique per `session_id`.
- **Automatic Context Injection:** Eliminates manual tracking of chat history.
- **Flexible Backend:** Supports in-memory, database, or Redis-based history stores.

```python
class RunnableWithMessageHistory(RunnableBindingBase):
    """Runnable that manages chat message history for another Runnable.

    A chat message history is a sequence of messages that represent a conversation.

    RunnableWithMessageHistory wraps another Runnable and manages the chat message
    history for it; it is responsible for reading and updating the chat message
    history.

    The formats supported for the inputs and outputs of the wrapped Runnable
    are described below.

    RunnableWithMessageHistory must always be called with a config that contains
    the appropriate parameters for the chat message history factory.

    By default, the Runnable is expected to take a single configuration parameter
    called `session_id` which is a string. This parameter is used to create a new
    or look up an existing chat message history that matches the given session_id.

    In this case, the invocation would look like this:

    `with_history.invoke(..., config={"configurable": {"session_id": "bar"}})`
    ; e.g., ``{"configurable": {"session_id": "<SESSION_ID>"}}``.

    The configuration can be customized by passing in a list of
    ``ConfigurableFieldSpec`` objects to the ``history_factory_config`` parameter (see
    example below).

    In the examples, we will use a chat message history with an in-memory
    implementation to make it easy to experiment and see the results.

    For production use cases, you will want to use a persistent implementation
    of chat message history, such as ``RedisChatMessageHistory``.

    Parameters:
        get_session_history: Function that returns a new BaseChatMessageHistory.
            This function should either take a single positional argument
            `session_id` of type string and return a corresponding
            chat message history instance.
        input_messages_key: Must be specified if the base runnable accepts a dict
            as input. The key in the input dict that contains the messages.
        output_messages_key: Must be specified if the base Runnable returns a dict
            as output. The key in the output dict that contains the messages.
        history_messages_key: Must be specified if the base runnable accepts a dict
            as input and expects a separate key for historical messages.
        history_factory_config: Configure fields that should be passed to the
            chat history factory. See ``ConfigurableFieldSpec`` for more details.

    Example: Chat message history with an in-memory implementation for testing.

    .. code-block:: python

        from operator import itemgetter
        from typing import List

        from langchain_openai.chat_models import ChatOpenAI

        from langchain_core.chat_history import BaseChatMessageHistory
        from langchain_core.documents import Document
        from langchain_core.messages import BaseMessage, AIMessage
        from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
        from pydantic import BaseModel, Field
        from langchain_core.runnables import (
            RunnableLambda,
            ConfigurableFieldSpec,
            RunnablePassthrough,
        )
        from langchain_core.runnables.history import RunnableWithMessageHistory


        class InMemoryHistory(BaseChatMessageHistory, BaseModel):
            \"\"\"In memory implementation of chat message history.\"\"\"

            messages: List[BaseMessage] = Field(default_factory=list)

            def add_messages(self, messages: List[BaseMessage]) -> None:
                \"\"\"Add a list of messages to the store\"\"\"
                self.messages.extend(messages)

            def clear(self) -> None:
                self.messages = []

        # Here we use a global variable to store the chat message history.
        # This will make it easier to inspect it to see the underlying results.
        store = {}

        def get_by_session_id(session_id: str) -> BaseChatMessageHistory:
            if session_id not in store:
                store[session_id] = InMemoryHistory()
            return store[session_id]


        history = get_by_session_id("1")
        history.add_message(AIMessage(content="hello"))
        print(store)  # noqa: T201


    Example where the wrapped Runnable takes a dictionary input:

        .. code-block:: python

            from typing import Optional

            from langchain_community.chat_models import ChatAnthropic
            from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
            from langchain_core.runnables.history import RunnableWithMessageHistory


            prompt = ChatPromptTemplate.from_messages([
                ("system", "You're an assistant who's good at {ability}"),
                MessagesPlaceholder(variable_name="history"),
                ("human", "{question}"),
            ])

            chain = prompt | ChatAnthropic(model="claude-2")

            chain_with_history = RunnableWithMessageHistory(
                chain,
                # Uses the get_by_session_id function defined in the example
                # above.
                get_by_session_id,
                input_messages_key="question",
                history_messages_key="history",
            )

            print(chain_with_history.invoke(  # noqa: T201
                {"ability": "math", "question": "What does cosine mean?"},
                config={"configurable": {"session_id": "foo"}}
            ))

            # Uses the store defined in the example above.
            print(store)  # noqa: T201

            print(chain_with_history.invoke(  # noqa: T201
                {"ability": "math", "question": "What's its inverse"},
                config={"configurable": {"session_id": "foo"}}
            ))

            print(store)  # noqa: T201


    Example where the session factory takes two keys, user_id and conversation id):

        .. code-block:: python

            store = {}

            def get_session_history(
                user_id: str, conversation_id: str
            ) -> BaseChatMessageHistory:
                if (user_id, conversation_id) not in store:
                    store[(user_id, conversation_id)] = InMemoryHistory()
                return store[(user_id, conversation_id)]

            prompt = ChatPromptTemplate.from_messages([
                ("system", "You're an assistant who's good at {ability}"),
                MessagesPlaceholder(variable_name="history"),
                ("human", "{question}"),
            ])

            chain = prompt | ChatAnthropic(model="claude-2")

            with_message_history = RunnableWithMessageHistory(
                chain,
                get_session_history=get_session_history,
                input_messages_key="question",
                history_messages_key="history",
                history_factory_config=[
                    ConfigurableFieldSpec(
                        id="user_id",
                        annotation=str,
                        name="User ID",
                        description="Unique identifier for the user.",
                        default="",
                        is_shared=True,
                    ),
                    ConfigurableFieldSpec(
                        id="conversation_id",
                        annotation=str,
                        name="Conversation ID",
                        description="Unique identifier for the conversation.",
                        default="",
                        is_shared=True,
                    ),
                ],
            )

            with_message_history.invoke(
                {"ability": "math", "question": "What does cosine mean?"},
                config={"configurable": {"user_id": "123", "conversation_id": "1"}}
            )

    """

    get_session_history: GetSessionHistoryCallable
    input_messages_key: Optional[str] = None
    output_messages_key: Optional[str] = None
    history_messages_key: Optional[str] = None
    history_factory_config: Sequence[ConfigurableFieldSpec]
```

### Key Parameters for `RunnableWithMessageHistory`

| Parameter            | Type                                         | Required | Description                                                                 |
|----------------------|----------------------------------------------|----------|-----------------------------------------------------------------------------|
| `runnable`           | `Runnable`                                   | Yes   | The base processing chain or agent that receives memory-injected input     |
| `get_session_history`| `Callable[[str], BaseChatMessageHistory]`    | Yes   | A function to fetch/create memory instances per `session_id`               |
| `input_messages_key` | `str`                                        | No    | Dict key for input text/messages (default: `"input"`)                      |
| `history_messages_key`| `str`                                       | No    | Dict key for the memory variable to inject (default: `"history"`)          |


### Example Use Cases
1. **Multi-User Chat Systems**: Enable isolated chat memory for each user, ensuring privacy and accurate context tracking.
2. **Persistent Long-Term Sessions**: 
    - Integrate with external storage (e.g., Redis, Milvus) to maintain session continuity across:
	    - Multiple user devices
	    - Session timeouts
	    - Cross-platform interactions
    - Then pass it to your agent


## 🔍 Use Case: Isolated Multi-Session Chat Memory

```python
import os
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_openai.chat_models import ChatOpenAI
from langchain_core.messages import HumanMessage
from langchain_community.chat_message_histories import ChatMessageHistory

os.environ["OPENAI_API_KEY"] = "*"

llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Session storage dictionary (can be replaced by Redis, DB, etc.)
store = {}


def get_session_history(session_id):
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]


# Define the prompt template
prompt = ChatPromptTemplate.from_messages([
    ("system",
     "You are a KK's Classroom AI assistant specialized in {ability}, answer questions within 30 words."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}"),
])

# Create the runnable chain (prompt -> model)
runnable = prompt | llm

# Wrap with session-aware memory manager
with_message_history = RunnableWithMessageHistory(
    runnable,
    get_session_history,
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
```

### Same session_id: Reusing Memory Context
- **First Input:** What is jvm?
- **Second Input:** "Please reanswer the question."
- **Behavior:** Since the session_id is the same ("user_123"), the memory from the first input is retrieved.
- **Result:** The model correctly understands that the user is referring to the previous JVM question, and Response 2 continues contextually.

```python
response2 = with_message_history.invoke(
    {"ability": "Java Development", 
     "input": HumanMessage("Please reanswer the question.")},
    config={"configurable": {"session_id": "user_123"}},
)
print(f"Response 2: {response2.content}\n")

# View conversation memory state
print(f"Stored session memory: {store}")


# Response 1: JVM (Java Virtual Machine) is an engine that executes Java bytecode, enabling platform independence by converting it into machine code for execution on any compatible system.

# Response 2: JVM (Java Virtual Machine) is a runtime environment that allows Java applications to run on any device by interpreting compiled Java bytecode into machine-specific instructions.

# Stored session memory: {
# 'user_123': InMemoryChatMessageHistory(messages=
# [HumanMessage(content='What is jvm?', additional_kwargs={}, response_metadata={}), AIMessage(content='JVM (Java Virtual Machine) is an engine that executes Java bytecode, enabling platform independence by converting it into machine code for execution on any compatible system.', additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 33, 'prompt_tokens': 44, 'total_tokens': 77, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_dbaca60df0', 'finish_reason': 'stop', 'logprobs': None}, id='run-52b84741-72c7-4167-9df8-ebfa5b9da1f5-0', usage_metadata={'input_tokens': 44, 'output_tokens': 33, 'total_tokens': 77, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}}), 
#  HumanMessage(content='Please reanswer the question.', additional_kwargs={}, response_metadata={}), AIMessage(content='JVM (Java Virtual Machine) is a runtime environment that allows Java applications to run on any device by interpreting compiled Java bytecode into machine-specific instructions.', additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 32, 'prompt_tokens': 90, 'total_tokens': 122, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_dbaca60df0', 'finish_reason': 'stop', 'logprobs': None}, id='run-0bdfb25b-af23-49d5-a329-417fcf635055-0', usage_metadata={'input_tokens': 90, 'output_tokens': 32, 'total_tokens': 122, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}})])}
```

### Different session_id: No Memory Context
- **First Input:** "What is jvm?" (under "user_123")
- **Second Input:** "Please reanswer the question." (under a new session_id, e.g., "user_666")
- **Behavior:** Because the session_id is different, no prior memory is available to the model.
- **Result:** Response 2 becomes ambiguous or confused, as the model cannot associate the second question with any previous context.

```python
response2 = with_message_history.invoke(
    {"ability": "Java Development", 
     "input": HumanMessage("Please reanswer the question.")},
    config={"configurable": {"session_id": "user_666"}},
)
print(f"Response 2: {response2.content}\n")

# View conversation memory state
print(f"Stored session memory: {store}")


# Response 1: JVM, or Java Virtual Machine, is an engine that executes Java bytecode, enabling platform independence and managing system resources for Java applications.

# Response 2: It seems there’s no specific question provided. Please ask your question about Java development, and I'll be happy to help!

# Stored session memory: {
# 'user_123': InMemoryChatMessageHistory(messages=[HumanMessage(content='What is jvm?', additional_kwargs={}, response_metadata={}), AIMessage(content='JVM, or Java Virtual Machine, is an engine that executes Java bytecode, enabling platform independence and managing system resources for Java applications.', additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 29, 'prompt_tokens': 44, 'total_tokens': 73, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_dbaca60df0', 'finish_reason': 'stop', 'logprobs': None}, id='run-0c23a75f-0a4b-404c-b8aa-aaf1ffa59cc3-0', usage_metadata={'input_tokens': 44, 'output_tokens': 29, 'total_tokens': 73, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}})]), 
# 'user_666': InMemoryChatMessageHistory(messages=[HumanMessage(content='Please reanswer the question.', additional_kwargs={}, response_metadata={}), AIMessage(content="It seems there’s no specific question provided. Please ask your question about Java development, and I'll be happy to help!", additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 25, 'prompt_tokens': 45, 'total_tokens': 70, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_dbaca60df0', 'finish_reason': 'stop', 'logprobs': None}, id='run-a9c9669b-7ee6-4819-afda-631850edef16-0', usage_metadata={'input_tokens': 45, 'output_tokens': 25, 'total_tokens': 70, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}})])}
```

## 🧠 Multi-User, Multi-Session

### Problem Statement

When building a scalable LLM application, supporting **multiple users** with **multiple parallel sessions** is essential.

**Key business requirements**:
- Each user (`user_id`) should have **completely isolated conversation history**.
- Each user can have **multiple concurrent sessions** (`session_id`) with the assistant.
- The system must support **dynamic expansion** (e.g., Redis, databases, etc.).

### Goals

| Feature             | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| Multi-Tenant Support| Each user has a unique `user_id`, ensuring complete data isolation.        |
| Multi-Session Support| Each user can maintain multiple sessions identified by `session_id`.       |
| Flexible Expansion  | Configurable parameters support custom storage or distributed systems.     |


### Key Concepts

#### `history_factory_config`

- A **configuration list** used in LangChain’s `RunnableWithMessageHistory`.
- It defines additional dynamic fields (like `user_id` and `session_id`).
- These fields affect **how chat history is created and retrieved**.

#### `ConfigurableFieldSpec`

- `from langchain_core.runnables import ConfigurableFieldSpec`

```python
history_factory_config: Sequence[ConfigurableFieldSpec]

class ConfigurableFieldSpec(NamedTuple):
    """Field that can be configured by the user. It is a specification of a field.

    Parameters:
        id: The unique identifier of the field.
        annotation: The annotation of the field.
        name: The name of the field. Defaults to None.
        description: The description of the field. Defaults to None.
        default: The default value for the field. Defaults to None.
        is_shared: Whether the field is shared. Defaults to False.
        dependencies: The dependencies of the field. Defaults to None.
    """

    id: str
    annotation: Any

    name: Optional[str] = None
    description: Optional[str] = None
    default: Any = None
    is_shared: bool = False
    dependencies: Optional[list[str]] = None
```

#### ConfigurableFieldSpec Parameters

| Parameter    | Type   | Description                                                                 | Example Value        |
|--------------|--------|-----------------------------------------------------------------------------|----------------------|
| `id`         | `str`  | A **unique identifier** for the field. Used internally to reference the field. | `"user_id"`          |
| `annotation` | `type` | Specifies the **data type** of the field. Used for type checking and validation. | `str`                |
| `name`       | `str`  | A **human-readable name** of the field. Often used in UI or logging output. | `"User ID"`          |
| `description`| `str`  | A **detailed explanation** of the field's purpose and usage.                | `"The unique identifier of a user."` |
| `default`    | `any`  | An **optional default value** used if no value is explicitly provided.      | `""` (empty string)  |
| `is_shared`  | `bool` | Indicates whether the field is **shared across the runtime**. If `True`, it remains constant throughout the session. | `True`               |

> For multi-tenant applications, fields like `user_id` and `session_id` can be declared as configurable specs, ensuring each user's context is isolated while remaining dynamically injectable into LangChain runtimes.


### Use Case: Multi-User, Multi-Session AI Assistant

```python
import os
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_openai.chat_models import ChatOpenAI
from langchain_core.messages import HumanMessage
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_core.runnables import ConfigurableFieldSpec

os.environ["OPENAI_API_KEY"] = "*"

llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Session storage dictionary (can be replaced by Redis, DB, etc.)
store = {}


def get_session_history(user_id: str, session_id):
    if (user_id, session_id) not in store:
        store[(user_id, session_id)] = ChatMessageHistory()
    return store[(user_id, session_id)]


# Define the prompt template
prompt = ChatPromptTemplate.from_messages([
    ("system",
     "You are a KK's Classroom AI assistant specialized in {ability}, answer questions within 30 words."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}"),
])

# Create the runnable chain (prompt -> model)
runnable = prompt | llm

# Wrap with session-aware memory manager
with_message_history = RunnableWithMessageHistory(
    runnable,
    get_session_history,
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
    config={"configurable": { "user_id" : "1" , "session_id" : "1" }},
)
print(f"Response 1: {response1.content}\n")
```

#### Same user_id, same session_id: Reusing Memory Context

```python
response2 = with_message_history.invoke(
    {"ability": "Java Development",
     "input": HumanMessage("Please reanswer the question.")},
    config={"configurable": { "user_id" : "1" , "session_id" : "1" }},
)
print(f"Response 2: {response2.content}\n")

# View conversation memory state
print(f"Stored session memory: {store}")

# Response 1: The JVM (Java Virtual Machine) is an engine that runs Java bytecode, enabling platform independence and managing memory, execution, and system resources for Java applications.

# Response 2: The JVM (Java Virtual Machine) is a runtime environment that executes Java bytecode, allowing Java programs to run on any platform without modification.

# Stored session memory: {('1', '1'): InMemoryChatMessageHistory(messages=
# [HumanMessage(content='What is jvm?', additional_kwargs={}, response_metadata={}), AIMessage(content='The JVM (Java Virtual Machine) is an engine that runs Java bytecode, enabling platform independence and managing memory, execution, and system resources for Java applications.', additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 33, 'prompt_tokens': 44, 'total_tokens': 77, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_dbaca60df0', 'finish_reason': 'stop', 'logprobs': None}, id='run-1637963c-acd1-4273-bc64-2b0e3c8d2a3e-0', usage_metadata={'input_tokens': 44, 'output_tokens': 33, 'total_tokens': 77, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}}), 
# HumanMessage(content='Please reanswer the question.', additional_kwargs={}, response_metadata={}), AIMessage(content='The JVM (Java Virtual Machine) is a runtime environment that executes Java bytecode, allowing Java programs to run on any platform without modification.', additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 29, 'prompt_tokens': 90, 'total_tokens': 119, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_dbaca60df0', 'finish_reason': 'stop', 'logprobs': None}, id='run-04341163-0829-48a2-ba24-6a5dffe29239-0', usage_metadata={'input_tokens': 90, 'output_tokens': 29, 'total_tokens': 119, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}})])}
```


#### Same user_id, different session_id or different user: No Memory Context
```python
response2 = with_message_history.invoke(
    {"ability": "Java Development",
     "input": HumanMessage("Please reanswer the question.")},
    config={"configurable": { "user_id" : "1" , "session_id" : "2" }},
)
print(f"Response 2: {response2.content}\n")

# View conversation memory state
print(f"Stored session memory: {store}")


# Response 1: JVM, or Java Virtual Machine, is an engine that enables Java bytecode to run on any device, providing platform independence and managing memory and execution.

# Response 2: Please provide the specific question you'd like me to reanswer, and I'll be happy to assist!

# Stored session memory: {
# ('1', '1'): InMemoryChatMessageHistory(messages=[
# HumanMessage(content='What is jvm?', additional_kwargs={}, response_metadata={}), 
# AIMessage(content='JVM, or Java Virtual Machine, is an engine that enables Java bytecode to run on any device, providing platform independence and managing memory and execution.', additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 32, 'prompt_tokens': 44, 'total_tokens': 76, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_dbaca60df0', 'finish_reason': 'stop', 'logprobs': None}, id='run-b4445413-a178-4c9d-9007-bb7f93ec2732-0', usage_metadata={'input_tokens': 44, 'output_tokens': 32, 'total_tokens': 76, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}})]), 
# ('1', '2'): InMemoryChatMessageHistory(messages=[
# HumanMessage(content='Please reanswer the question.', additional_kwargs={}, response_metadata={}), 
# AIMessage(content="Please provide the specific question you'd like me to reanswer, and I'll be happy to assist!", additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 20, 'prompt_tokens': 45, 'total_tokens': 65, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_dbaca60df0', 'finish_reason': 'stop', 'logprobs': None}, id='run-c1f13ed4-a788-43ed-b425-96b6b5eb798f-0', usage_metadata={'input_tokens': 45, 'output_tokens': 20, 'total_tokens': 65, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}})])}
```









<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



