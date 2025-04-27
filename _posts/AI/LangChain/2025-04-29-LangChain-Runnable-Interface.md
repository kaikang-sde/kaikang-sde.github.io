---
title: LangChain - Runnable Interface
date: 2025-04-27
order: 3
categories: [AI, LangChain]
tags: [LangChain, LLM, Runnable, ICEL, Async-processing]
author: kai
---

# 🚀 LangChain - Runnable Interface
LangChain’s power lies not just in its modular components but in the unifying abstraction that ties them together: the **`Runnable` interface**. This post explores what `Runnable` is, why it’s essential, and how it enables flexible, scalable chain construction through the **LangChain Expression Language (LCEL)**.


## ⚙️ What is the Runnable Interface?

In LangChain, a **Runnable** is the **core abstraction** for **any executable logic unit**—whether it's:

- A model invocation
- A prompt formatting step
- A document retrieval
- An external API call

The Runnable interface **standardizes** the way components are **executed, batched, streamed, and composed**.



## 🤷🏻  Why Use Runnable?
**1. Unified Interface**
- Every major component — **PromptTemplates, Models, Parsers, Retrievers** — implements the Runnable interface.
    - Guarantees **type compatibility**
	- Enables **seamless chaining** without custom adapters
- `from langchain_core.runnables`


**2. Flexible Composition**
- Using the pipe operator (`|`), you can **chain multiple Runnables** together declaratively, just like a dataflow: `runnable_chain = prompt | llm | parser`
    - Each component passes its output to the next
    - Complex workflows become elegant and readable

**3. Dynamic Configuration**
- Highly dynamic, fault-tolerant, and easy to maintain
    - **Runtime parameter binding** (e.g., passing new variables into the pipeline)
    - **Component swapping** (replace a model or retriever without rewriting chains)
    - **Error handling** with features like `.with_retry()`

**4. Async and Parallel Execution**
- Perfect for high-concurrency applications like chatbots, document summarization pipelines, or streaming APIs.
- Built-in support for:
    - `ainvoke()`: Async single execution
	- `abatch()`: Async batch execution
	- `RunnableParallel`: Run multiple Runnables concurrently



## 🧩  What is `RunnableSequence` in LangChain?

`RunnableSequence` is a **core component** in LangChain that allows you to build **sequential execution chains**. It is a **subclass of `Runnable`**, specifically designed for **linear workflows** where the output of one step becomes the input to the next.

- `from langchain_core.runnables import RunnableSequence`


```python
# a simplified view of how the invoke() method operates internally
def invoke(
    self, input: Input, config: Optional[RunnableConfig] = None, **kwargs: Any
) -> Output:
    # Invoke all steps in sequence
    try:
        for i, step in enumerate(self.steps):
            # Mark each step as a child run for better traceability
            config = patch_config(
                config, callbacks=run_manager.get_child(f"seq:step:{i + 1}")
            )
            with set_config_context(config) as context:
                if i == 0:
                    input = context.run(step.invoke, input, config, **kwargs)
                else:
                    input = context.run(step.invoke, input, config)
    except Exception as e:
        raise e
    return input
```


## 🧠 Building Chains with LCEL

`chain = prompt | model | output_parser`

- Each component is connected sequentially using the pipe operator (`|`), defining the logic of data flow.
    - **prompt:** A PromptTemplate Runnable that formats the input.
    - **model:** A ChatModel or LLM Runnable that generates text.
    - **output_parser:** A Runnable that processes the model’s output into structured data.


### Data Flow: One-Way Execution
In an LCEL chain:
- **Each Runnable’s output becomes the input of the next Runnable**.
- This forms a single-directional data stream.
- **Example:** `A | B | C`:  
> Invoke A (user input) -> invoke B (output of A) -> invoke C (output of B) -> Final Output / output of C


###  Unified Interface: Seamless Component Chaining
- Every component (prompt templates, models, retrievers, parsers) implements the **Runnable** interface.
- This guarantees **type compatibility** and **seamless chaining**.
- You can freely connect different types of components without worrying about integration mismatches.


### Lazy Execution: Define Now, Run Later
When you write: `chain = prompt | model | output_parser`
- **No computation actually happens yet.**
- You are merely **defining the relationship** between components.
- **Execution is triggered** only when you explicitly call:
    - `chain.invoke(input)` or `chain.stream(input)`

This lazy design enables:
- **Dynamic parameter binding** at runtime
- **Flexible configuration** (e.g., swapping models or setting retries)
- **Efficient resource management**

### How the `|` Operator Works Internally
The `|` (pipe) operator in Python is overridden for Runnables using the __or__ method.

```python
def __or__(self, other: Runnable) -> RunnableSequence:
    return RunnableSequence(steps=[self, other])
```
- When you chain with `|`, it **creates a RunnableSequence**.
- The sequence **stores all connected Runnables** in an internal steps list.
- Upon execution, it **iterates through steps** and calls `invoke()` on each component sequentially.

```text
User Input
   ↓
[PromptTemplate.invoke()]
   ↓
[Model.invoke()]
   ↓
[OutputParser.invoke()]
   ↓
Final Answer
```


## 🔎 Core Methods Defined by the `Runnable` Interface
In LangChain, the `Runnable` interface standardizes **multiple execution modes** to handle different application needs—whether it's **single calls**, **batch processing**, or **streaming** outputs for real-time responses.

Here’s the essential method set defined by the `Runnable` base class:

```python
class Runnable(Generic[Input, Output]):
    # Process a single input synchronously and return a single output
    def invoke(self, input: Input) -> Output: ...

    # Process a single input asynchronously
    async def ainvoke(self, input: Input) -> Output: ...

    # Stream outputs chunk-by-chunk for real-time response generation
    def stream(self, input: Input) -> Iterator[Output]: ...

    # Process a batch of inputs to improve throughput
    def batch(self, inputs: List[Input]) -> List[Output]: ...
```


| Method       | Description            | Typical Use Case                         |
|--------------|-------------------------|-------------------------------------------|
| `invoke()`   | Synchronous execution    | Single call workflows, quick tasks        |
| `batch()`    | Batch synchronous execution | Processing multiple inputs efficiently (e.g., datasets) |
| `stream()`   | Streamed output generation | Real-time text generation, chatbot streaming |
| `ainvoke()`  | Asynchronous execution   | High-concurrency web service integration, async APIs |


## 🏗️ Runnable Subclass Implementations in LangChain
To support different control flows and execution patterns, the LangChain `Runnable` interface has **multiple specialized subclasses**.  
Each subclass is designed to address a specific type of data processing need—whether it's **sequential workflows**, **branching logic**, or **parallel execution**.

| Component             | Key Feature                   | Typical Use Case                          |
|------------------------|--------------------------------|-------------------------------------------|
| `RunnableSequence`     | Sequential execution          | Linear data processing pipelines          |
| `RunnableBranch`       | Conditional branching          | Dynamic routing based on runtime decisions |
| `RunnableParallel`     | Parallel execution             | Independent multi-task processing         |
| `RunnablePassthrough`  | Data passthrough               | Forwarding original input without change  |


### 1. `RunnableSequence`

- Executes multiple Runnables **in a strict order**.
- Each step’s output becomes the next step’s input.
- Equivalent to chaining components with the `|` operator.

✅ Best for **linear, deterministic pipelines** like prompt → model → output parser.


### 2. `RunnableBranch`

- Implements **conditional routing**.
- At runtime, **selects a different branch** based on input data or context.
- Each branch is itself a Runnable.

✅ Ideal for **dynamic workflows** where different models, prompts, or retrieval strategies are chosen depending on user intent.


### 3. `RunnableParallel`

- Executes **multiple Runnables concurrently** on the same input.
- Returns a **dictionary of outputs**, keyed by each Runnable’s name.

✅ Great for **multi-tasking scenarios**, such as simultaneously summarizing, translating, and classifying the same document.


### 4. `RunnablePassthrough`

- Simply **forwards the input unchanged** to the next step.
- Useful for:
  - Preserving user input alongside retrieved context
  - Feeding raw inputs into prompt templates
  - Debugging intermediate stages

✅ Essential in complex pipelines where **you need to pass both original and processed data** downstream.









<br>




---

🚀 Stay tuned for more insights into LangChain and RAG!



