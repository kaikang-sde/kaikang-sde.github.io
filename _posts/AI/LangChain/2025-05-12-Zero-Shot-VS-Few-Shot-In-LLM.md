---
title: Zero-Shot vs Few-Shot Learning in Large Language Models
date: 2025-05-12
categories: [AI, LangChain]
tags: [LangChain, Zero-Shot, Few-Shot, Prompt Engineering, LLM]
author: kai
---

# 🚀 Zero-Shot vs Few-Shot Learning in Large Language Models
In the world of large language models (LLMs), **Zero-Shot** and **Few-Shot** learning are powerful techniques that enable generalization and flexibility—**without the need for fine-tuning**. 

## 🔍 Zero-Shot
**Zero-Shot Learning** refers to the ability of a large language model (LLM) to complete a task **without seeing any task-specific examples**. Instead of being fine-tuned on labeled data, the model relies purely on its **pretrained general knowledge and natural language understanding** to infer the desired behavior.


**Core Principle:**<br>
Zero-Shot leverages the LLM's generalized language capabilities acquired during **pretraining** to perform tasks it has never explicitly seen before.

### Example
- **User Input:** Translate the following sentence into Chinese: Hello, how are you?
- **Model Output:** 你好

### When to Use

- You have **no labeled examples**.
- You want to **test the model’s general capabilities**.
- Tasks are **well-known** and phrased clearly (e.g., basic translation, summarization, or sentiment analysis).

### Limitations

- May produce inaccurate outputs for **ambiguous tasks**.
- **Lack of context** can reduce precision in domain-specific applications.

## 🧠 Few-Shot
**Few-Shot Learning** refers to a model's ability to perform a task after being shown just a **few examples** (typically 3–5). By observing these examples, the model learns the **task structure**, **format**, and **intent**, allowing it to **mimic the pattern** and generate accurate responses for new inputs.

**Core Principle**:<br>
Few-shot learning leverages the LLM's **In-Context Learning** ability — learning task behavior from examples provided directly in the prompt, without additional training.


### Example

- **Prompt with Few-Shot Examples**
```text
input: Apple, output: Fruit
input: Banana, output: Fruit
input: Car, output: Vehicle
input: Airplane, output: Vehicle
input Cat, output ?
```
- **Model Output**: Animal


### When to Use

- You want to **guide the model with a consistent pattern**.
- Tasks are **simple but benefit from structural clarity** (e.g., classification, mapping).
- You have a few trusted **reference examples** but no full training dataset.

### Limitations

- Limited generalization to **unseen patterns** or **noisy examples**.
- Output depends heavily on the **quality and format** of the prompt.
- May require careful **prompt crafting** to avoid confusion.

## 📊 Zero-Shot vs. Few-Shot

| Scenario                        | Zero-Shot Learning                                | Few-Shot Learning                                      |
|---------------------------------|---------------------------------------------------|--------------------------------------------------------|
| **Task Complexity**             | Simple tasks (e.g., translation, basic classification) | Complex tasks (e.g., reasoning, structured generation) |
| **Data Dependency**            | No examples needed                                | Requires a few high-quality examples                  |
| **Output Controllability**     | Lower (relies on pretrained knowledge)            | Higher (guided by explicit examples and patterns)      |
| **Typical Use Cases**          | Rapid prototyping, general-purpose Q&A            | Domain-specific tasks (e.g., legal parsing, medical term extraction) |


## 🎯 Examples
### Zero-Shot Prompt Translation with LangChain
```python
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

os.environ["OPENAI_API_KEY"] = "*"  

# Step 1: Define the zero-shot prompt
zero_shot_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a professional translator. Translate the text from {source_language} to {target_language}."),
    ("human", "{text}")
])

# Step 2: Initialize the LLM
llm = ChatOpenAI(  # Initialize the LLM model
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Step 3: Create the prompt-LLM chain
chain = zero_shot_prompt | llm

# Step 4: Invoke the chain
response = chain.invoke({
    "source_language": "Chinese",
    "target_language": "English",
    "text": "今天天气不错,我要学习AI大模型开发"
})

# Step 5: Print result
print(response.content) # The weather is nice today, and I want to study AI large model development.
```

### LangChain + FewShotPromptTemplate
```python
import os
from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate
from langchain_openai import ChatOpenAI

os.environ["OPENAI_API_KEY"] = "*"  

# Step 1: Initialize the LLM
llm = ChatOpenAI( 
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Step 2: Define Few-Shot examples
examples = [
    {
        "input": "Where is Apple Inc.'s headquarters?",
        "output": "After careful thought: Apple Inc.'s headquarters is in Cupertino, California, USA."
    },
    {
        "input": "Who is the CEO of OpenAI?",
        "output": "After careful thought: The current CEO of OpenAI is Sam Altman."
    }
]

# Step 3: Define the example format
example_template = """
Input: {input}
Output: {output}
"""
example_prompt = PromptTemplate(
    template=example_template,
    input_variables=["input", "output"]
)

# Step 4: Construct the Few-Shot Prompt Template
few_shot_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    suffix="Input: {question}\nOutput:",
    input_variables=["question"]
)

# Step 5: Build the Chain and Run Inference
few_shot_chain = few_shot_prompt | llm
response = few_shot_chain.invoke({
    "question": "Who is the founder of Amazon?"
})

# Step 6: Display the Response
print(response.content) # After careful thought: The founder of Amazon is Jeff Bezos.


print(few_shot_prompt.format(question="Who is the founder of Amazon?"))
# Input: Where is Apple Inc.'s headquarters?
# Output: After careful thought: Apple Inc.'s headquarters is in Cupertino, California, USA.

# Input: Who is the CEO of OpenAI?
# Output: After careful thought: The current CEO of OpenAI is Sam Altman.

# Input: Who is the founder of Amazon?
# Output:
```

## 🧾 Summary

| Mode         | Description                                                                 |
|--------------|-----------------------------------------------------------------------------|
| Zero-Shot    | Leverages the model’s **pretrained knowledge** to perform tasks **without examples**. Ideal for general-purpose tasks like translation or basic classification. |
| Few-Shot     | Uses **a few curated examples** to guide the model's behavior and output format. Best suited for **domain-specific** or structured tasks. |


### LangChain Implementation Tips

- **Zero-Shot**:
  - Use `AgentType.ZERO_SHOT_REACT_DESCRIPTION` when initializing an agent.
  - Focus on clean instruction formatting to guide LLM inference.
  
- **Few-Shot**:
  - Use `PromptTemplate` or `FewShotPromptTemplate` with example blocks.
  - Ensure example coverage reflects **diverse and representative** user inputs.


### Design Principles

- **Clarity of Instructions**: Clearly define the task objective, expected input, and output style.
- **Example Quality Matters**: In Few-Shot mode, a few **high-quality examples** are far more effective than many vague ones. Prioritize relevance and variety.






<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



