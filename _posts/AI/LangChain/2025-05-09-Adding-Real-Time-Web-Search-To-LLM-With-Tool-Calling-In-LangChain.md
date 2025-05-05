---
title: Adding Real-Time Web Search to LLM with Tool Calling in LangChain
date: 2025-05-05
order: 5
categories: [AI, LangChain]
tags: [LangChain, Tool Integration, Web Search, LLM Agent, AI Q&A]
author: kai
---

# 🚀 Adding Real-Time Web Search to LLM with Tool Calling in LangChain
Implementing an intelligent Q&A system that **combines a large language model (LLM)**  with **external tools** (like a web search engine) using LangChain.

The system can:

- Accept **natural language input** from the user  
- Analyze the question to determine if tool usage is required  
- Automatically invoke the right tool (e.g., web search)  
- Return a final answer that may include **LLM reasoning** and **real-time results**


## 🔁 LangCgain Agent Tool Calling Flow
![LangCgain Agent Tool Calling Flow](/assets/img/posts/AI/LangChain/LangChainAgentToolCallingFlow.png)


## 🔧 Implementation Steps
1. **Configure SearchAPI Access:** Instantiate the SearchApiAPIWrapper for real-time search
2. **Define Tool Functions:** Define functions as LangChain-compatible tools
    - Web Search Tool: Used for retrieving live data and news
    - Multiplication Tool: Multiplies two integers and returns the result.
3. **Initialize the LLM and Bind Tools:**
    - Use the `ChatOpenAI` class to load your model with the desired configuration.
    - Use `ChatPromptTemplate` to guide the system’s behavior and input formatting.
4. **Construct the Execution Chain:**
    - Use `RunnablePassthrough` to feed user input into the prompt and agent.

### How It Works (Core Logic)

- **User Input Flow**
    - The **user input** is passed through **RunnablePassthrough** into the **prompt template**.
    - The **formatted prompt** is then sent to the **LLM agent**.

- **Tool Invocation**
    - Based on the LLM’s response, LangChain checks for `resp.tool_calls`.
    - If tool calls exist:
        - The corresponding **tools are executed**
        - Results are wrapped into **ToolMessage**-like structures
        - These results are **merged into the message history**
        - The updated history is reprocessed by the LLM to generate the **final answer**


## 👀 Code example

```python
from langchain_community.utilities import SearchApiAPIWrapper
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
from langchain_core.messages import ToolMessage
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import ChatPromptTemplate
import os

os.environ["SEARCHAPI_API_KEY"] = "*"
os.environ["OPENAI_API_KEY"] = "*"  


# Instantiate the search wrapper
search = SearchApiAPIWrapper()

# Step 2: Define Custom Tools
@tool("web_search",  return_direct=True)
def web_search(query: str) -> str:
    """
    Use this tool to get real-time information or current events.
    Input should be a search keyword string.
    """
    try:
        results = search.results(query)
        return "\n\n".join([
            f"Source:{res['title']}\ncontent:{res['snippet']}"
            for res in results['organic_results']
        ])
    except Exception as e:
        return f"Search failed: {str(e)}"


@tool("multiply")
def multiply(a: int, b: int) -> int:
    """Multiplies two integers."""

    return a * b


# Step 3: Initialize the LLM and Bind Tools
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Step 4: Define the Prompt Template
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are an AI assistant named Kai. Use tools when necessary to help users."),
    ("human", "{query}")
])

# Step 5: Build and Execute the Agent Chain
# Prepare the tool list
tool_dict = {
    "web_search": web_search,
    "multiply": multiply
}
tools = list(tool_dict.values())

# Bind tools to the model
llm_with_tools = llm.bind_tools(tools=tools)

# Create the full chain
chain = {
    "query": RunnablePassthrough()
} | prompt | llm_with_tools

# Run a sample query
query = "What is Amazon's stock price today?"
response = chain.invoke({"query": query})
print(response)


# Check if the model intends to call any tool
tool_calls = response.tool_calls

if not tool_calls:
    print(f"No tool required. LLM response:\n{response.content}")
else:
    # Reconstruct conversation history with the original prompt
    history_messages = prompt.invoke(query).to_messages()
    history_messages.append(response)

    print(f"Conversation history before tool execution:\n{history_messages}")

    # Loop through tool calls and execute them
    for tool_call in tool_calls:
        tool_name = tool_call.get("name")
        tool_args = tool_call.get("args")

        # Execute the selected tool
        tool_response = tool_dict[tool_name].invoke(tool_args)

        print(f"Tool called: {tool_name}")
        print(f"Arguments: {tool_args}")
        print(f"Output: {tool_response}")

        # Add the tool result to the message history
        history_messages.append(
            ToolMessage(
                tool_call_id=tool_call.get("id"),
                name=tool_name,
                content=tool_response
            )
        )

    print(f"Updated conversation history:\n{history_messages}")

    # Reinvoke the LLM using the full history + tool results
    final_response = llm_with_tools.invoke(history_messages)

    print(f"Final Response:\n{final_response}")
    print(f"Final Answer Content:\n{final_response.content}")
```

### history_messages = prompt.invoke(query).to_messages()
- If prompt is an instance of ChatPromptTemplate (as defined earlier), it contains a message structure like:
```text
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are an AI assistant named Kai. Use tools when necessary to help users."),
    ("human", "{query}")
])
```
- `invoke(query)` will:
    - Replace {query} with the actual user input (e.g., "What is Amazon's stock price today?")
    - Return a ChatPromptValue object, which holds the full message history as structured objects — e.g.:
```text
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are an AI assistant named Kai. Use tools when necessary to help users."),
    ("human", "What is Amazon's stock price today?")
])
``` 
- `.to_messages()`: This converts the ChatPromptValue into a list of message objects, ready to be used in llm.invoke() or appended to with tool responses. The result is a list of LangChain message types, like:
```text
[
    SystemMessage(content="You are an AI assistant..."),
    HumanMessage(content=What is Amazon's stock price today?)
]
``` 








<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



