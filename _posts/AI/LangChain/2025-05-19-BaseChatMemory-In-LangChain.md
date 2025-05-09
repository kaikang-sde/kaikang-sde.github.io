---
title: BaseChatMemory in LangChain - Customizable Memory for LLM Applications
date: 2025-05-09
order: 12
categories: [AI, LangChain]
tags: [LangChain, LLM Memory, AI Agent, ChatHistory]
author: kai
---

# 🚀 BaseChatMemory in LangChain: Customizable Memory for LLM Applications
In LLM applications, memory is the foundation of coherent, multi-turn conversations. Whether you're building a chatbot, a virtual assistant, or a personalized tutor, you need a way to **store and recall dialogue history**. In LangChain, this is achieved through the `BaseChatMemory` class — the abstract base class for all chat-based memory modules.

## 📦 What is `BaseChatMemory`
- `from langchain.memory.chat_memory import BaseChatMemory`
- This class defines a standard interface for storing and retrieving memory within LLM pipelines. Although it is not used directly in most use cases, all memory types in LangChain inherit from it, making it a powerful base for extending or customizing memory behavior.

> While some older APIs related to memory may be deprecated in recent LangChain versions, understanding the design principles of BaseChatMemory remains essential for backward compatibility and advanced use cases.


## 🔧 Core Capabilities
- **Standardized Interface**
    - Offers consistent APIs like `save_context()` and `load_memory_variables()` for all memory types.
- **State Management**
    - Maintains an evolving history of conversation state via the chat_memory attribute.
- **Extensibility**
    - Developers can subclass and override default behavior (e.g., encrypting memory, pruning history).


## 🧱 Key Attribute: chat_memory
- This is the core container for messages. It holds all conversation turns using the ChatMessageHistory class.
- Structure
	- Type: ChatMessageHistory
	- Stores: A messages list
	- Message types:
	    - HumanMessage – user input
	    - AIMessage – assistant output
	    - SystemMessage (optional for system role prompts)
- Example
    print(memory.chat_memory.messages)
    ```python
        # Output:
        # [
        #   HumanMessage(content="What's the weather today?"),
        #   AIMessage(content="It's sunny in Shanghai.")
        # ]
    ```

## 💡 Core Methods
LangChain provides standardized APIs for handling memory in LLM-based conversations. 

### 1. `save_context()`: Save Interaction History
This method stores both the **user input** and the **model's response** into the memory buffer.
- **Code**
    ```python
    def save_context(self, inputs: Dict[str, Any], outputs: Dict[str, str]) -> None:
        """Save context from this conversation to buffer."""
        input_str, output_str = self._get_input_output(inputs, outputs)
        self.chat_memory.add_messages(
            [
                HumanMessage(content=input_str),
                AIMessage(content=output_str),
            ]
        )
    ```
- **Example**
    ```python
    memory.save_context(
        {"input": "Hello!"}, 
        {"output": "Hollo! How can I help you?"}
    )
    ```
- **This is equivalent to**
    ```python
    memory.chat_memory.add_user_message("Hello!")
    memory.chat_memory.add_ai_message("Hollo! How can I help you?")
    ```
- Internally, chat_memory stores the interaction as structured messages.

### 2. load_memory_variables(): Retrieve History
This method fetches the current memory content in a structured format, typically used to inject chat history into LLM prompts.
- **Code**
    ```python
    @abstractmethod
    def load_memory_variables(self, inputs: dict[str, Any]) -> dict[str, Any]:
        """Return key-value pairs given the text input to the chain.

        Args:
            inputs: The inputs to the chain.

        Returns:
            A dictionary of key-value pairs.
        """
    ```
- **Example**
    ```python
    variables = memory.load_memory_variables({})
    print(variables["history"])
    ```
- **Output**
    - Human: "Hello!"
    - AI: "Hollo! How can I help you?"
- This string can be directly appended to prompts for context preservation in multi-turn dialogue.

### 3. clear(): Reset Memory
- Use this to completely wipe out the conversation history stored in memory.
- Example: `memory.chat_memory.clear()`
- Useful when starting a new conversation or resetting state between user sessions.


## 🧠 Key Subclasses of `BaseChatMemory` in LangChain

### 1. `ConversationBufferMemory`

- **Description**: Stores the complete conversation history in raw format.
- **Use Case**: Suitable for applications where the full dialogue context needs to be preserved across turns.
- **Characteristics**:
  - Retains all messages.
  - Default choice for simple chatbots.


### 2. `ConversationBufferWindowMemory`

- **Description**: Keeps only the most recent N interactions.
- **Use Case**: Ideal for resource-limited environments or short-context models.
- **Key Parameter**:  
  - `k` – Controls how many previous messages to keep.
- **Implementation Detail**:  
  - Overrides the internal message storage logic of `ConversationBufferMemory`.


### 3. `ConversationSummaryMemory`

- **Description**: Stores a **summary** of past conversations instead of full transcripts.
- **Use Case**: Useful for long conversations where summarization is more efficient than raw history.
- **How It Works**:  
  - Overrides `load_memory_variables()` to return a model-generated summary instead of a plain message history.
- **Benefit**:  
  - Helps maintain coherence with a much smaller prompt token footprint.


### 4. Custom Subclasses

- **Scenario**: You want custom logic for:
  - Filtering sensitive content.
  - Formatting history into a JSON format.
  - Compressing memory using embeddings or tags.
- **How**:  
  - Inherit from `BaseChatMemory`.
  - Override:
    - `save_context()` — for how memory is saved.
    - `load_memory_variables()` — for how memory is recalled.


| Class                          | Key Feature                     | Recommended For                         |
|-------------------------------|----------------------------------|------------------------------------------|
| `ConversationBufferMemory`     | Full raw history                | Simple chat agents                        |
| `ConversationBufferWindowMemory` | Keep last N rounds            | Token-limited models or short context     |
| `ConversationSummaryMemory`    | Summarized conversation         | Long chats where efficiency is critical   |
| Custom subclass                | Custom memory behavior          | Advanced apps needing fine-tuned control  |


## 💬 Case Study
### ConversationBufferMemory: Saving and Retrieving Conversation History
```python
from langchain.memory import ConversationBufferMemory

# Initialize the conversation memory buffer
memory = ConversationBufferMemory(
    memory_key="chat_history",  # Key used to store and retrieve memory
    return_messages=True,       # Return messages as structured objects
)

# Simulate a user message and AI reply
memory.chat_memory.add_user_message("What is your name?")
memory.chat_memory.add_ai_message("I am the Assistant.")

# Retrieve memory variables
memory_variables = memory.load_memory_variables({})
print("Memory content:", memory_variables)

# memory = ConversationBufferMemory(
# Memory content: {'chat_history': 
# [HumanMessage(content='What is your name?', additional_kwargs={}, response_metadata={}), 
# AIMessage(content='I am the Assistant.', additional_kwargs={}, response_metadata={})]}
```

### ConversationBufferWindowMemory: Keep Last 2 Exchanges
```python
from langchain.memory import ConversationBufferWindowMemory

# Initialize window memory with size = 2
memory = ConversationBufferWindowMemory(k=2, memory_key="chat_history")

# Save some messages
memory.save_context({"input": "What’s your name?"}, {
                    "output": "Hi, I'm Assistant Kai"})
# Equivalent:
# memory.chat_memory.add_user_message("What’s your name?")
# memory.chat_memory.add_ai_message("Hi, I'm Assistant Kai")

memory.save_context({"input": "What is KK's Classroom?"}, {
                    "output": "KK's Classroom is a computer learning platform for beginners"})
memory.save_context({"input": "Sounds good, I’ll check it out"}, {
                    "output": "Hope it helps!"})

# Load the current memory window
window_history = memory.load_memory_variables({})
print("Current sliding window history:", window_history)

# memory = ConversationBufferWindowMemory(k=2, memory_key="chat_history")
# Current sliding window history: {'chat_history': 
# "Human: What is KK's Classroom?\n
# AI: KK's Classroom is a computer learning platform for beginners\n
# Human: Sounds good, I’ll check it out\n
# AI: Hope it helps!"}
```









<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



