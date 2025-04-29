---
title: LangChain Tools and How to Create Them
date: 2025-04-29
order: 8
categories: [AI, LangChain]
tags: [LangChain, LLM, Agent, Tools]
author: kai
---

# 🚀 LangChain Tools and How to Create Them

While Large Language Models (LLMs) are powerful at **text generation**,  
they have **natural limitations** in:

| Limitation                   | Description                                  |
|-------------------------------|----------------------------------------------|
| **Real-time Data Access**     | Cannot fetch live information (e.g., weather, stocks) |
| **Accurate Complex Calculation** | Struggles with precise math or financial computations |
| **Domain-Specific Expertise** | Lacks deep, updated knowledge in specialized fields (e.g., law, healthcare) |
| **External System Integration** | Cannot directly interface with APIs, databases, CRMs |

> **Tools solve these problems** —  they **"add wings"** to LLMs, enabling them to **act beyond their pre-trained knowledge**.


## 🤖 Tools
**Tool** = An **interface** between the LLM and the external world. It enables the LLM to **invoke external capabilities** like:

- APIs
- Functions
- Calculators
- Databases
- Business systems (e.g., CRM, ERP)

> Through Tools, the LLM becomes an **active agent** capable of **dynamic data retrieval, computation, and execution**.


### Core Purposes of Tools
Tools **upgrade the LLM** from a "text generator" to a **true interactive intelligent agent**.

| Purpose                          | Description                                         |
|----------------------------------|-----------------------------------------------------|
| **Break Static Knowledge Limitation** | Fetch live, updated information from the outside world |
| **Real-Time Data Retrieval**     | Example: Get live weather, stock prices, traffic info |
| **Execute Complex Logic**        | Perform mathematical, financial, scientific calculations |
| **Connect to Software Systems**  | Integrate with CRM platforms, ERP systems, and other APIs |


### Tool Lifecycle
Tools are **not hardcoded into the flow** —  Agents **decide when and how to use them** autonomously!


| Stage                 | Description                                         |
|------------------------|-----------------------------------------------------|
| **1. Tool Definition** | Create a Tool with a `name`, `description`, and a callable `function` |
| **2. Agent Registration** | Register the Tool so the Agent can recognize and reason about it |
| **3. Automatic Invocation** | LLM (via Agent) decides to use a Tool dynamically based on task needs |
| **4. Result Handling** | Capture the Tool’s output and feed it back into the reasoning cycle |


## 🛠️ 4 Ways to Create Tools in LangChain
LangChain provides multiple flexible ways to define Tools, depending on **how much control** or **simplicity** you need.

### 1. Using `@tool` Decorator
The `@tool` decorator provides a **fast and elegant way** to turn a simple function into a **fully functional Tool** that an Agent can recognize, reason about, and call autonomously.

- Just **decorate a normal function** with `@tool`.
- **Add a docstring description** to tell the LLM **what the tool does**.
- LangChain automatically generates the tool's **name**, **description**, **input arguments**, and **schema**.

**Key Points:**
- **Minimal setup, highly useful for quick prototyping!**
- Works well for most use cases.
- Easy to attach **name** and **description** metadata.
- Only supports either **sync** or **async** separately — **cannot mix** both in the same function.

#### Example 1 - Simple `@tool` Usage

```python
from langchain_core.tools import tool

@tool
def multiply(a: int, b: int) -> int:
    """Multiply two input parameters."""
    return a * b

# Inspect the tool's properties
print("Tool Name:", multiply.name)  # Tool Name: multiply
print("Tool Description:", multiply.description) # Tool Description: Multiply two input parameters.
print("Tool Arguments:", multiply.args) # Tool Arguments: {'a': {'title': 'A', 'type': 'integer'}, 'b': {'title': 'B', 'type': 'integer'}}
print("Tool Return Direct:", multiply.return_direct) # Tool Return Direct: False
print("Detailed Schema:", multiply.args_schema.model_json_schema()) 

# Detailed Schema: 
# {'description': 'Multiply two input parameters.', 
#   'properties': {'a': {'title': 'A', 'type': 'integer'}, 'b': {'title': 'B', 'type': 'integer'}}, 
#   'required': ['a', 'b'], 
#   'title': 'multiply', 
#   'type': 'object'}


# Invoke the tool manually
print(multiply.invoke({"a": 2, "b": 3})) # 6
```
#### Example 2 - Using Custom Schema and Options with pydantic

```python
from pydantic import BaseModel, Field
from langchain_core.tools import tool

# Define a structured input schema
class CalculatorInput(BaseModel):
    a: int = Field(description="First number to multiply")
    b: int = Field(description="Second number to multiply")

# Custom tool name + Structured input schema + Return immediately without post-processing
@tool("multiplication-tool", args_schema=CalculatorInput, return_direct=True)
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b

# Inspect the customized tool
print("Tool Name:", multiply.name) # Tool Name: multiplication-tool
print("Tool Description:", multiply.description) # Tool Description: Multiply two numbers.
print("Tool Arguments:", multiply.args) 
# Output: Tool Arguments: {'a': {'description': 'First number to multiply', 'title': 'A', 'type': 'integer'}, 'b': {'description': 'Second number to multiply', 'title': 'B', 'type': 'integer'}}
print("Tool Return Direct:", multiply.return_direct) # Tool Return Direct: True
print("Detailed Schema:", multiply.args_schema.model_json_schema())

# Detailed Schema: 
# {'properties': {
#   'a': {'description': 'First number to multiply', 'title': 'A', 'type': 'integer'}, 
#   'b': {'description': 'Second number to multiply', 'title': 'B', 'type': 'integer'}}, 
#   'required': ['a', 'b'], 
#   'title': 'CalculatorInput', 
#    'type': 'object'}
```

#### Key Component Overview

| Component              | Purpose                                            | Example                             |
|-------------------------|----------------------------------------------------|-------------------------------------|
| **Name (`name`)**       | Unique identifier for the tool; agents match tools by name | `"wikipedia"`, `"google_search"`    |
| **Description (`description`)** | Natural language description of the tool's purpose; agents use this to decide when to invoke it | `"Query content from Wikipedia"`    |
| **Input Parameters (`args_schema`)** | Defines the tool's expected input format using a Pydantic model; helps validate inputs and generate prompts | `query: str`                        |
| **Execution Function (`func`)** | The actual function that runs the tool’s operation (e.g., API call, shell command) | `def run(query: str): ...`           |
| **Return Mode (`return_direct`)** | If `True`, the agent will directly return the tool's output without further reasoning | Useful for simple, final answers    |


### 2. StructuredTool
`StructuredTool` is a **base class** in LangChain designed for creating **tools with structured, validated input parameters** — especially when you need **more complex, multi-parameter inputs** than a simple `@tool` decorator allows.

**Provides:**
- **Strict parameter schema definition:** Define input format using a Pydantic model
- **Multi-parameter input validation:** Ensure all inputs are correct before execution
- **Auto-generation of tool usage examples:** Makes it easier for agents to plan and invoke correctly

> `StructuredTool` shines when you are building **tools that require multiple input parameters** or **tools involving complex, structured input types**.


#### `@tool` vs `StructuredTool`

| Feature                      | `@tool` Decorator                      | `StructuredTool`                            |
|-------------------------------|----------------------------------------|--------------------------------------------|
| **Parameter Definition**      | Simple parameters (string or dict)     | Strict parameter schema using Pydantic     |
| **Input Validation**           | Weak (relies on manual code checks)    | Strong (automatic type and format validation) |
| **Multi-Parameter Support**    | Requires manual dictionary parsing     | Direct mapping to multiple named arguments |
| **Usage Complexity**           | Very easy for quick simple tools       | Designed for complex business logic tools  |


#### Example 1 - Building a StructuredTool Using `from_function`

```python
from pydantic import BaseModel, Field
from langchain_core.tools import StructuredTool

# Step 1: Define input parameter structure
class CalculatorInput(BaseModel):
    a: int = Field(description="First number")
    b: int = Field(description="Second number")


# Step 2: Define the core function
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


# Step 3: Wrap the function as a StructuredTool
calculator = StructuredTool.from_function(
    func=multiply,
    name="Calculator",
    description="A tool to calculate the product of two numbers.",
    args_schema=CalculatorInput,
    return_direct=True,
)

# Step 4: Inspect tool properties
print("Tool Name:", calculator.name) # Tool Name: Calculator
print("Tool Description:", calculator.description) # Tool Description: A tool to calculate the product of two numbers.
print("Tool Arguments:", calculator.args) # Tool Arguments: {'a': {'description': 'First number', 'title': 'A', 'type': 'integer'}, 'b': {'description': 'Second number', 'title': 'B', 'type': 'integer'}}
print("Tool Return Direct:", calculator.return_direct) # Tool Return Direct: True
print("Tool Detailed Schema:", calculator.args_schema.model_json_schema())

# Tool Detailed Schema: {
# 'properties': {
# 'a': {'description': 'First number', 'title': 'A', 'type': 'integer'}, 
# 'b': {'description': 'Second number', 'title': 'B', 'type': 'integer'}}, 
# 'required': ['a', 'b'], 'title': 'CalculatorInput', 'type': 'object'}


# Step 5: Invoke the tool
print("Tool Invocation Result:", calculator.invoke({"a": 2, "b": 3})) # Tool Invocation Result: 6
```



### 3. Runnable.as_tool()

- If you already have a **Runnable** (e.g., a pipeline that processes strings or dicts), you can convert it into a Tool using `.as_tool()`.

**Key Points:**
- Great for complex preprocessing flows.
- Can customize **parameter names, descriptions, and schemas**.
- This makes **complex pipelines usable as simple tools** for agents.

#### Example

```python
from pydantic import BaseModel, Field
from langchain_core.runnables import RunnableLambda

# Step 1: Define input schema
class CalculatorInput(BaseModel):
    a: int = Field(description="First number to multiply")
    b: int = Field(description="Second number to multiply")

# Step 2: Create a Runnable
multiply_runnable = RunnableLambda(lambda x: x["a"] * x["b"])

# Step 3: Use as_tool() directly, no RunnableTool import needed!
calculator_tool = multiply_runnable.as_tool(
    name="Calculator",
    description="Multiply two numbers together.",
    args_schema=CalculatorInput
)

# Step 4: Test the tool
print("Tool Name:", calculator_tool.name) # Tool Name: Calculator
print("Tool Description:", calculator_tool.description) # Tool Description: Multiply two numbers together.
print("Tool Invocation Result:", calculator_tool.invoke({"a": 5, "b": 6})) # Tool Invocation Result: 30
```


### 4. Subclassing BaseTool for Maximum Control
- If you need **full customization** (e.g., advanced validation, retries, complex error handling), you can **inherit from BaseTool** and manually define the tool behavior.

**Key Points:**
- Gives **complete control** over input parsing, execution, error handling.
- Requires more **boilerplate** code compared to decorators.
- Useful for **production-grade, highly specialized tools**.


#### Example - Creating a Tool by Subclassing `BaseTool`

```python
from pydantic import BaseModel, Field
from typing import Type
from langchain_core.tools import BaseTool

# Step 1: Define input schema
class CalculatorInput(BaseModel):
    a: int = Field(description="First number")
    b: int = Field(description="Second number")


# Step 2: Create a custom tool by subclassing BaseTool
class CustomCalculatorTool(BaseTool):
    name: str = "Calculator"
    description: str = "Use this tool when you need to solve math multiplication problems."
    args_schema: Type[BaseModel] = CalculatorInput
    return_direct: bool = True

    # Override the _run method
    def _run(self, a: int, b: int) -> str:
        """Execute the tool's logic."""
        return a * b


# Step 3: Instantiate and inspect the tool
calculator = CustomCalculatorTool()

print("Tool Name:", calculator.name) # Tool Name: Calculator
print("Tool Description:", calculator.description)  # Tool Description: Use this tool when you need to solve math multiplication problems.
print("Tool Arguments:", calculator.args) # Tool Arguments: {'a': {'description': 'First number', 'title': 'A', 'type': 'integer'}, 'b': {'description': 'Second number', 'title': 'B', 'type': 'integer'}}
print("Tool Return Direct:", calculator.return_direct) # Tool Return Direct: True
print("Tool Detailed Schema:", calculator.args_schema.model_json_schema())
# Tool Detailed Schema: {'properties': {'a': {'description': 'First number', 'title': 'A', 'type': 'integer'}, 'b': {'description': 'Second number', 'title': 'B', 'type': 'integer'}}, 'required': ['a', 'b'], 'title': 'CalculatorInput', 'type': 'object'}

# Step 4: Invoke the tool
print("Tool Invocation Result:", calculator.invoke({"a": 2, "b": 3})) # Tool Invocation Result: 6
```










<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



