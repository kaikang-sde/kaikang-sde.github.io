---
title: Binding Tools to LLMs in LangChain
date: 2025-05-02
order: 5
categories: [AI, LangChain]
tags: [LangChain, LLM, Agent, Tools, tool_calls]
author: kai
---

# 🚀 Binding Tools to LLMs in LangChain
Large Language Models (LLMs) are powerful at reasoning and language generation — but they **cannot access real-time data** like weather, stock prices, or backend orders on their own.

> **Solution**: Bind external **Tools** to the LLM, so it can **call real-world functions** when needed.


## 🎯 Use Case Scenarios

| Domain        | Tool Use                                      |
|---------------|-----------------------------------------------|
| Weather       | Query current temperature and forecast        |
| Finance       | Fetch real-time stock prices or exchange rates|
| E-commerce    | Check order status, update inventory          |


## 🧠 Mental Model: J.A.R.V.I.S. Analogy

> Binding tools to an LLM is like **installing arms and legs** for Iron Man’s AI assistant J.A.R.V.I.S.

- The **LLM is the brain** — it understands your instructions.
- The **Tools are the actuators** — they execute things in the real world (e.g., querying weather, running shell commands).


## ⚠️ Important Clarification

- Despite the term **"tool calling"**, the LLM **does not execute the tool itself**.  
- It only **generates structured parameters** for the tool.  
- Whether the tool is **actually invoked** depends on your application logic.


## 🛠️ Tool Binding Workflow

| Component        | Role                                                            | Example                                                   |
|------------------|------------------------------------------------------------------|------------------------------------------------------------|
| **Tool Description** | Teaches the LLM what the tool does and how to use it           | `"Query the current weather for a given city and date"`    |
| **Parameter Parsing** | Maps natural language to structured tool inputs              | From `"What's the weather in New York today?"` extract `{"city": "New York", "date": "today"}` |
| **Execution Feedback** | Converts the tool's raw response into human-like answer      | From JSON `{ "temp": 25 }` to `"It's 25°C and sunny."`     |

## 🔁 Full Process Flow

```text
user input 
    ↓
LLM Intention Analysis 
    ↓
Tool Selection 
    ↓
Parameter Extraction 
    ↓
Tool Execution (The LangChain AgentExecutor receives the parameters and invokes the actual tool function)
    ↓
Result Formatting 
    ↓
Final Response
```


## 🔧 Binding Tools to an LLM via `.bind_tools()`
`.bind_tools()` is a method available on **chat model objects** (like `ChatOpenAI`) that allows you to bind one or more tools directly to the LLM. This enables the model to **intelligently decide** when and how to invoke a tool in response to a user query.

### What It Enables:
- Tool-aware reasoning (model decides if/when to use tools)
- Automatic tool schema registration
- Seamless tool parameter generation from user input
- Multi-tool orchestration support

> ⚠️ While LangChain provides the `.bind_tools()` API to bind external tools to language models, **not every LLM backend or model implementation supports it.**

```python
def bind_tools(
    self,
    tools: Sequence[Union[Dict[str, Any], Type, Callable, BaseTool]],
    *,
    tool_choice: Optional[
        Union[dict, str, Literal["auto", "none", "required", "any"], bool]
    ] = None,
    strict: Optional[bool] = None,
    parallel_tool_calls: Optional[bool] = None,
    **kwargs: Any,
) -> Runnable[LanguageModelInput, BaseMessage]:
```

### Example

```python
# Prepare tool list
tools = [add, multiply]

# Create a tool-aware LLM
llm = ChatOpenAI(model="gpt-4o-mini")  # Must be tool-compatible

# Bind tools to the LLM
llm_with_tools = llm.bind_tools(tools)

# Run a query
query = "What is 3 * 12?"
response = llm_with_tools.invoke(query)

# Output the final response
print(response.content)
```


## 🔍 Tool Calling Step-by-Step: How LLM Triggers Tools in LangChain

### Step 1: Model Decides to Call a Tool
In LangChain, when a tool-enabled large language model (LLM) decides that a tool is required to answer a user query, **it doesn't respond with natural language immediately**. Instead, the LLM returns an **empty `.content`**, and attaches **tool invocation metadata** to the `.tool_calls` attribute. 
This mechanism allows the model to **offload execution** to real tools (e.g., API calls, calculators, databases).


#### Where Tool Instructions Live
When you use a bound tool model via `.bind_tools()`, LangChain makes tool calling structured and observable. If the model chooses to use a tool, it populates the `.tool_calls` property with a **list of tool call dictionaries**.

Each dictionary includes:

| Field         | Meaning                                  |
|---------------|-------------------------------------------|
| `id`          | Unique identifier for the tool call       |
| `function.name` | The registered tool name (e.g., `"multiply"`) |
| `function.arguments` | A JSON string of structured parameters |
| `type`        | Always `"function"` (indicating a function/tool call) |
| `index`       | Position of the tool in a multi-call sequence (e.g., `0`, `1`) |

If the model doesn't call any tool, `.tool_calls` will simply be **an empty list**.

##### Output Example

```python
# Example setup: using a tool-bound LLM to handle a user query
ai_msg = llm_with_tools.invoke(messages)

# Inspect tool call information in the AIMessage
print(ai_msg.tool_calls)

{
  "tool_calls": [
    {
      "id": "call_ea723d86cf804a088b946a",
      "function": {
        "name": "multiply",
        "arguments": "{\"a\": 3, \"b\": 12}"
      },
      "type": "function",
      "index": 0
    }
  ]
}
```

### Step 2: Execute the Selected Tool
Once an LLM response contains a `.tool_calls` list, it's your job as the developer to:

1. **Identify** which tool the model chose  
2. **Execute** the corresponding function  
3. **Inject the result** back into the conversation for further reasoning or final response

This step connects **LLM reasoning** to **real-world execution**.

#### Example
```python
# Assume you already received this from step 1
ai_msg = llm_with_tools.invoke(messages)

# Tool name to function mapping
tool_registry = {
    "add": add,
    "multiply": multiply
}

# Iterate over tool calls
for tool_call in ai_msg.tool_calls:
    # Extract tool name from model response
    tool_name = tool_call["function"]["name"].lower()

    # Find the matching Python function
    selected_tool = tool_registry[tool_name]
    print(f"Selected tool: {selected_tool}")

    # Invoke the tool with arguments from the LLM
    tool_msg = selected_tool.invoke(tool_call)
    print(f"Tool result message: {tool_msg}")

    # Append the tool result back to the conversation
    messages.append(tool_msg)
```

After invoking a tool using `.invoke(tool_call)`, LangChain returns a structured object called **`ToolMessage`**, which contains the **result of the tool execution** — ready to be passed back to the LLM.

This message links **tool execution** to the **original LLM tool call** by including a unique `tool_call_id`.

```python
ToolMessage(
    content='36',
    name='multiply',
    tool_call_id='call_1319a58494c54998842092'
)
```


### Step 3: Passing Results Back to the LLM for Final Response
Once you've successfully **executed the tool** and received a `ToolMessage`, the final step is to **feed that result back into the LLM** so it can generate a **natural language response**.

This step closes the loop: `LLM → Tool → Result → LLM response`

1. Recognize that it already called a tool
2. Use the tool’s output as new context
3. Generate a final user-facing response


#### Example

```python
# Step 1: After executing the tool, append its result
messages.append(tool_msg)  # ToolMessage with content + tool_call_id

# Step 2: Feed the updated message history back to the model
final_result = llm_with_tools.invoke(messages)

# Step 3: Print the final content
print(f"Final response: {final_result.content}")

AIMessage(
    content='The result of 3 multiplied by 12 is 36.',
    additional_kwargs={'refusal': None},
    response_metadata={
        'token_usage': {'completion_tokens': 16, 'prompt_tokens': 272, 'total_tokens': 288},
        'model_name': 'gpt-4o-mini',
        'finish_reason': 'stop',
        ...
    },
    id='run-8046aa25-091a-4df3-a49a-aa36811c8d44-0',
    usage_metadata={
        'input_tokens': 272,
        'output_tokens': 16,
        'total_tokens': 288
    }
)
```

![Tool Invocation](/assets/img/posts/AI/LangChain/ToolInvocation.png)
![Tool Result](/assets/img/posts/AI/LangChain/ToolResult.png)



## 🔩 Complete LangChain tool-calling workflow

```python
import os
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage

os.environ["OPENAI_API_KEY"] = "*"

# Step 1: Define Tools
@tool
def add(a: int, b: int) -> int:
    """Adds a and b."""
    return a + b


@tool
def multiply(a: int, b: int) -> int:
    """Multiplies a and b."""
    return a * b


# Step 2: Initialize the LLM and Bind Tools
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

tools = [add, multiply]
llm_with_tools = llm.bind_tools(tools)


# Step 3: Process User Query and Capture Tool Calls
query = "What is the result of 3 * 12?"
messages = [HumanMessage(query)]

# Run initial inference
ai_msg = llm_with_tools.invoke(messages)
print("AI Message:", ai_msg)
# AI Message: content='' 
# additional_kwargs={'tool_calls': [{'id': 'call_SYRbTQAG2Jf7YEdYkuIe4D5p', 
# 'function': {'arguments': '{"a":3,"b":12}', 'name': 'multiply'}, 'type': 'function'}], 'refusal': None} 
# response_metadata={'token_usage': {'completion_tokens': 18, 'prompt_tokens': 80, 'total_tokens': 98, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_dbaca60df0', 'finish_reason': 'tool_calls', 'logprobs': None} id='run-3b05c214-5ad3-438b-b75a-5e3aeee57867-0' 
# tool_calls=[{'name': 'multiply', 'args': {'a': 3, 'b': 12}, 'id': 'call_SYRbTQAG2Jf7YEdYkuIe4D5p', 'type': 'tool_call'}] usage_metadata={'input_tokens': 80, 'output_tokens': 18, 'total_tokens': 98, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}}

# The result of tool calls
print("Tool Calls:", ai_msg.tool_calls)
# Tool Calls: [{'name': 'multiply', 'args': {'a': 3, 'b': 12}, 'id': 'call_SYRbTQAG2Jf7YEdYkuIe4D5p', 'type': 'tool_call'}]

# Add AI message to conversation history
messages.append(ai_msg)


# Step 4: Execute Tool(s) and Append Results
# Tool dispatch logic
for tool_call in ai_msg.tool_calls:
    tool_name = tool_call["name"].lower() # Selected tool, not all tools
    selected_tool = {"add": add, "multiply": multiply}[tool_name]

    print(f"Selected Tool: {selected_tool.name}") # Selected Tool: multiply

    # Execute the tool
    tool_msg = selected_tool.invoke(tool_call)
    print(f"Tool Result Message: {tool_msg}") 
    # Tool Result Message: content='36' name='multiply' tool_call_id='call_qKmwSsjP6DHJWuwbg0sUR1lu'

    # Add result to conversation
    messages.append(tool_msg)


print(messages)
# [HumanMessage(content='What is the result of 3 * 12?', additional_kwargs={}, response_metadata={}), 
# AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_ecteeaymOoy80jduziRkQzSI', 'function': {'arguments': '{"a":3,"b":12}', 'name': 'multiply'}, 'type': 'function'}], 'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 18, 'prompt_tokens': 80, 'total_tokens': 98, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_0392822090', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-e5ca74f2-b99c-45c3-8815-7172837d8d34-0', 
# tool_calls=[{'name': 'multiply', 'args': {'a': 3, 'b': 12}, 'id': 'call_ecteeaymOoy80jduziRkQzSI', 'type': 'tool_call'}], usage_metadata={'input_tokens': 80, 'output_tokens': 18, 'total_tokens': 98, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}}), 
# ToolMessage(content='36', name='multiply', tool_call_id='call_ecteeaymOoy80jduziRkQzSI')]

# Step 5: Feed Tool Results Back to LLM for Final Response
# Final call with tool result injected
final_response = llm_with_tools.invoke(messages)

# Print final result
print("Final LLM Response:", final_response.content) 
# Final LLM Response: The result of \( 3 \times 12 \) is 36.
```

### Data flow
```text
User Input (query string)
   ↓
[HumanMessage(query)]
   ↓
Initial LLM Call → llm_with_tools.invoke(messages)
   ↓
LLM Output (AIMessage with .tool_calls)
   └─ tool_calls = [{ name: "multiply", arguments: {"a": 3, "b": 12} }]
   ↓
Tool Execution
   └─ selected_tool = multiply
   └─ result = multiply(3, 12) → 36
   └─ ToolMessage(content="36", name='multiply', tool_call_id=...)
   ↓
messages.append(ToolMessage)
   ↓
Second LLM Call → llm_with_tools.invoke(messages)
   ↓
Final Response (AIMessage with content)
   └─ "The result of \( 3 \times 12 \) is 36."
```









<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



