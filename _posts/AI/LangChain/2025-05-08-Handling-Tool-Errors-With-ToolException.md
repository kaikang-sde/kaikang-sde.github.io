---
title: Handling Tool Errors with ToolException
date: 2025-05-07
order: 4
categories: [AI, LangChain]
tags: [LangChain, ToolException, Error Handling, AI Agents, Tools]
author: kai
---

# 🚀 Handling Tool Errors with ToolException
When an intelligent agent (Agent) in LangChain calls external tools such as APIs, databases, or search engines, it must be prepared to deal with **unexpected runtime errors**.

Without proper handling, these issues can:
- Crash the program with unhandled exceptions
- Confuse users with raw stack traces
- Block recovery — no way to retry or fallback


## 📌 Common Tool Failure Scenarios

| Scenario                  | Example                                                         |
|---------------------------|-----------------------------------------------------------------|
| Network error           | API timeout or connection reset                                |
| Authentication failure | Expired or invalid API key                                      |
| Bad input               | Malformed or unsupported query parameters                       |
| Resource limit          | API quota exceeded or rate-limited                              |



## 🛡️ Solution: `ToolException` for Robust Error Handling

LangChain provides a built-in mechanism called **`ToolException`** to **catch and handle tool-related failures** in a structured and recoverable way.

### Core Benefits

- **Standardized error format**  
- **Preserves context** like tool name and failure reason  
- **Enables intelligent fallback and retries**  
- **Improves user-facing responses** with friendly error messages


### Method 1. Handling Tool Errors with `handle_tool_error=True`
LangChain makes it simple to **gracefully catch and return errors** from tools using a built-in switch:  
`handle_tool_error=True`.

By enabling this flag when creating a tool, LangChain will **automatically catch internal exceptions** and convert them into structured, readable outputs — without crashing the pipeline.

```python
from langchain_core.tools import ToolException, StructuredTool

# Define a tool function that raises a ToolException
def search(query: str) -> str:
    """
    Simulate a failed search query.
    """
    raise ToolException(f"No results found for query: {query}")


search_tool = StructuredTool.from_function(
    func=search,
    name="search",
    description="Search tool that may return empty results.",
    handle_tool_error=True  #  Enable error handling here!
)

# Simulate a user query that will trigger the exception
response = search_tool.invoke({"query": "What is Amazon's stock price today?"})

# ToolException will be caught and returned as a message
print(response) # No results found for query: What is Amazon's stock price today?
```


### Method 2: Use a Custom Error Message
LangChain also allows you to **define a custom fallback error message** when a tool fails, by setting `handle_tool_error` to a **string**, rather than `True`.

This is useful when:
- You want to suppress internal error details
- You want to return a **predefined user-friendly message**
- You want the agent to stay polite and informative during failure



```python
from langchain_core.tools import ToolException, StructuredTool

# Simulate a failing tool
def search(query: str) -> str:
    """
    Simulate a failed search query.
    """
    raise ToolException(f"No search results found for: {query}")


search_tool = StructuredTool.from_function(
    func=search,
    name="search",
    description="A tool that searches online resources.",
    handle_tool_error="Search failed, please try again."  # Custom error message will override the error message from the ToolException
)

response = search_tool.invoke({"query": "What is Amazon's stock price today?"})
print(response)  # Search failed, please try again.
```


### Method 3: Use a Custom Exception Handler Function
In addition to simple `True` or static string fallback, LangChain also supports **fully custom error handling logic** by passing a **callable (function)** to `handle_tool_error`.

This gives you maximum flexibility to:

- Log errors
- Dynamically generate fallback messages
- Route errors differently based on exception content
- Perform analytics or retries

```python
from langchain_core.tools import ToolException, StructuredTool

def search(query: str) -> str:
    """
    Simulate a failed search query.
    """
    raise ToolException(f"No search results found for: {query}")


def _handle_tool_error(error: ToolException) -> str:
    """
    Custom logic to handle tool errors. Can do anything, retry, loging, MQ message, etc
    """
    return f"Search failed (custom handler): {error}"


search_tool = StructuredTool.from_function(
    func=search,
    name="search",
    description="Search tool with custom error fallback.",
    handle_tool_error=_handle_tool_error  # Pass Custom Handler function here
)

response = search_tool.invoke({"query": "What is Amazon's stock price today?"})
print(response)
```





<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



