---
title: LLM Memory Optimization for QA Workloads
date: 2025-05-20
categories: [AI, LangChain]
tags: [LangChain, LLM, Memory, RAG, Token Optimization, Milvus]
author: kai
permalink: /posts/ai/langchain/llm-memory-optimization-for-qa-workloads/
---

# 🚀 LLM Memory Optimization for QA Workloads
Modern large language models (LLMs) like GPT-4, Claude, or DeepSeek are capable of handling **complex multi-turn dialogue**. However, when deployed in real-world applications such as **customer support**, **educational assistants**, or **legal QA agents**, they face several critical challenges:

| 🔍 Problem Type | ⚠️ Description | 🧨 Consequence |
|----------------|----------------|----------------|
| Token Limits   | Long dialogues exceed the LLM's context window (e.g. 4k, 16k, or 128k tokens) | Truncated input, lost context, inaccurate replies |
| Efficiency Bottlenecks | Full-history retrieval can take >500ms | Slower response time, poor UX |
| Business Needs | Must identify past key issues quickly | Affects QA audits, escalation handling |
| Compliance Risk | Storing raw conversation may include sensitive PII | Raises data security and privacy concerns |


## 🚫 The Challenge with Long QA Sessions

1. **Lack of Context Awareness**  
   Each query is independent. Follow-up questions like “What’s my name?” result in _“I don’t know”_ because the model has no memory.

2. **Token Limit Overflow**  
   As conversations grow, the full dialogue history quickly exceeds the LLM’s token window (e.g., 4k to 128k tokens), causing:
   - Truncated input
   - Slower response
   - Lost context


## 🚀 How to Solve It
To **preserve conversation continuity** while **respecting token constraints**, we use a memory abstraction that stores **summarized history** rather than full transcripts.

### Technical Solution: Summarized Memory

LangChain provides a powerful memory system tailored to LLM interactions. Two key types:

| Memory Type                  | Description                                  |
|-----------------------------|----------------------------------------------|
| `ConversationBufferMemory`  | Stores full dialogue history (not scalable)  |
| `ConversationSummaryMemory` | Stores LLM-generated summaries (token-light) |


###  Workflow

```text
User Input ➝ Append to History ➝ LLM summarizes ➝ Summary used in next round
```

### Advantages
1. **Reduced Token Consumption**  
Summarizing prior messages dramatically decreases the number of tokens passed into the LLM, allowing you to stay within model limits and reduce API costs.

2. **Improved Long-Term Memory for Agents**  
By storing meaningful summaries instead of raw history, agents can recall key facts across multi-turn conversations — even after hundreds of turns.

3. **Scalable Architecture via External Storage**  
Summaries can be offloaded to distributed databases such as:
- **MongoDB** (for structured memory)
- **Milvus / Pinecone** (for vector-based semantic retrieval)


## 🧪 ConversationSummaryMemory
`ConversationSummaryMemory` is a memory class provided by LangChain that enables **long-term memory** for LLM agents by generating **summaries** of conversation history using the model itself.

Instead of storing the full transcript of every interaction (which consumes tokens and storage), this memory module keeps a **succinct, AI-generated summary** that evolves over time.


### Key Features

- **Summarization-Driven**: Uses an LLM prompt to compress chat history.
- **Token-Efficient**: Avoids full history replay, staying within token limits.
- **Customizable**: Supports prompt customization to preserve task-critical details.


### Prompt

```text
_DEFAULT_SUMMARIZER_TEMPLATE = 
"""Progressively summarize the lines of conversation provided, adding onto the previous summary returning a new summary.

EXAMPLE
Current summary:
The human asks what the AI thinks of artificial intelligence. The AI thinks artificial intelligence is a force for good.

New lines of conversation:
Human: Why do you think artificial intelligence is a force for good?
AI: Because artificial intelligence will help humans reach their full potential.

New summary:
The human asks what the AI thinks of artificial intelligence. The AI thinks artificial intelligence is a force for good because it will help humans reach their full potential.
END OF EXAMPLE

Current summary:
{summary}

New lines of conversation:
{new_lines}

New summary:"""
```

### Example: Storing Dialog Summaries with LLMs
```python
import os
from langchain.memory import ConversationSummaryMemory
from langchain_openai import ChatOpenAI

os.environ["OPENAI_API_KEY"] = "*"

# Initialize the LLM
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Create a summary-based memory instance
memory = ConversationSummaryMemory(llm=llm)

# Simulate a conversation
memory.save_context({"input": "What is your name?"}, {
                    "output": "Hello, I am Kai."})
memory.save_context({"input": "Can you tell me about machine learning?"}, {
                    "output": "Machine learning is a branch of artificial intelligence."})

# Retrieve the summarized memory
summary = memory.load_memory_variables({})
print("Conversation Summary:", summary)
# Conversation Summary: {'history': "The human asks for the AI's name, and the AI introduces itself as Kai. The human then asks about machine learning, and the AI explains that machine learning is a branch of artificial intelligence."}
```


## 🧠 Practical Case: A Summarized Memory Chat System with LangChain
```python
import os
from langchain.memory import ConversationSummaryMemory
from langchain_openai import ChatOpenAI
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate

os.environ["OPENAI_API_KEY"] = "*"

# Initialize the LLM
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Set Up Summarized Memory
memory = ConversationSummaryMemory(
    llm=llm,
    memory_key="chat_history",  # Must match the prompt input key
    return_messages=True
)

# Step 3: Design the Prompt Template
prompt = ChatPromptTemplate.from_messages([
    ("system",
     "You are an AI assistant that responds based on conversation history. Current summary: {chat_history}"),
    ("human", "{input}")
])

# Step 4: Build the LCEL Chain
# RunnablePassthrough: The assign method accepts keyword arguments, where the value is a function that defines how to compute the new field based on the input
chain = (
    RunnablePassthrough.assign(
        chat_history=lambda _: memory.load_memory_variables({})["chat_history"]
    )
    | prompt
    | llm
    | StrOutputParser()
)

# Step 5: Run a Multi-Turn Dialogue
user_inputs = [
    "I am Kai, I am a student of Master of computer science at KK's Classroom."
    "What is the definition of artificial intelligence?",
    "What courses are available on KK's Classroom?",
    "How is AI used in the medical field?",
    "Who am I? What is my major?"

]

for query in user_inputs:
    response = chain.invoke({"input": query})
    print(f"\nUser: {query}")
    print(f"AI: {response}")

    # Manual memory update required with LCEL pattern
    memory.save_context({"input": query}, {"output": response}) # Must manually update memory
    print("Current Memory Summary:",
          memory.load_memory_variables({})["chat_history"])


# memory = ConversationSummaryMemory(
# User: I am Kai, I am a student of Master of computer science at KK's Classroom.What is the definition of artificial intelligence?
# AI: Artificial intelligence (AI) is the branch of computer science that focuses on creating systems capable of performing tasks that typically require human intelligence. These tasks include reasoning, learning, problem-solving, understanding natural language, recognizing patterns, and making decisions. AI can be categorized into various types, such as narrow AI, which is designed for specific tasks, and general AI, which aims to perform any intellectual task that a human can do.
# Current Memory Summary: [SystemMessage(content="The human, Kai, introduces himself as a Master of Computer Science student at KK's Classroom and asks for the definition of artificial intelligence. The AI explains that artificial intelligence (AI) is a branch of computer science focused on creating systems that perform tasks requiring human intelligence, including reasoning, learning, problem-solving, understanding natural language, recognizing patterns, and making decisions. The AI also mentions the distinction between narrow AI, designed for specific tasks, and general AI, which aims to perform any intellectual task a human can do.", additional_kwargs={}, response_metadata={})]

# User: What courses are available on KK's Classroom?
# AI: I'm not aware of the specific courses available on KK's Classroom. You might want to check their official website or contact their support for the most accurate and up-to-date information on course offerings.
# Current Memory Summary: [SystemMessage(content="The human, Kai, introduces himself as a Master of Computer Science student at KK's Classroom and asks for the definition of artificial intelligence. The AI explains that artificial intelligence (AI) is a branch of computer science focused on creating systems that perform tasks requiring human intelligence, including reasoning, learning, problem-solving, understanding natural language, recognizing patterns, and making decisions. The AI also mentions the distinction between narrow AI, designed for specific tasks, and general AI, which aims to perform any intellectual task a human can do. When Kai inquires about the courses available on KK's Classroom, the AI responds that it is not aware of specific course offerings and suggests checking the official website or contacting support for accurate information.", additional_kwargs={}, response_metadata={})]

# User: How is AI used in the medical field?
# AI: AI is used in the medical field in various ways, including:

# 1. **Diagnosis and Disease Prediction**: AI algorithms analyze medical data, such as imaging scans (e.g., X-rays, MRIs), to assist in diagnosing conditions and predicting disease progression.

# 2. **Personalized Treatment Plans**: AI helps in creating tailored treatment plans based on individual patient data, including genetics, lifestyle, and preferences.

# 3. **Drug Discovery**: AI accelerates the drug discovery process by predicting how different compounds will interact with targets in the body, potentially leading to more effective medications.

# 4. **Robotic Surgery**: AI-powered robotic systems assist surgeons in performing complex procedures with precision, reducing recovery times and improving outcomes.

# 5. **Clinical Decision Support**: AI systems provide healthcare professionals with evidence-based recommendations to support clinical decision-making.

# 6. **Patient Monitoring**: AI tools monitor patients' vital signs and health metrics in real-time, allowing for early detection of complications and timely interventions.

# 7. **Natural Language Processing**: AI is used to analyze and interpret clinical notes and patient records, streamlining documentation and improving information retrieval.

# 8. **Telemedicine**: AI enhances telemedicine platforms by facilitating virtual consultations and triaging patients based on their symptoms.

# These applications help improve efficiency, accuracy, and patient outcomes in healthcare.
# Current Memory Summary: [SystemMessage(content="The human, Kai, introduces himself as a Master of Computer Science student at KK's Classroom and asks for the definition of artificial intelligence. The AI explains that artificial intelligence (AI) is a branch of computer science focused on creating systems that perform tasks requiring human intelligence, including reasoning, learning, problem-solving, understanding natural language, recognizing patterns, and making decisions. The AI also mentions the distinction between narrow AI, designed for specific tasks, and general AI, which aims to perform any intellectual task a human can do. When Kai inquires about the courses available on KK's Classroom, the AI responds that it is not aware of specific course offerings and suggests checking the official website or contacting support for accurate information. Kai then asks how AI is used in the medical field, and the AI outlines several applications, including diagnosis and disease prediction, personalized treatment plans, drug discovery, robotic surgery, clinical decision support, patient monitoring, natural language processing, and telemedicine, all of which enhance efficiency, accuracy, and patient outcomes in healthcare.", additional_kwargs={}, response_metadata={})]

# User: Who am I? What is my major?
# AI: You are Kai, and your major is Master of Computer Science at KK's Classroom.
# Current Memory Summary: [SystemMessage(content="The human, Kai, introduces himself as a Master of Computer Science student at KK's Classroom and asks for the definition of artificial intelligence. The AI explains that artificial intelligence (AI) is a branch of computer science focused on creating systems that perform tasks requiring human intelligence, including reasoning, learning, problem-solving, understanding natural language, recognizing patterns, and making decisions. The AI also mentions the distinction between narrow AI, designed for specific tasks, and general AI, which aims to perform any intellectual task a human can do. When Kai inquires about the courses available on KK's Classroom, the AI responds that it is not aware of specific course offerings and suggests checking the official website or contacting support for accurate information. Kai then asks how AI is used in the medical field, and the AI outlines several applications, including diagnosis and disease prediction, personalized treatment plans, drug discovery, robotic surgery, clinical decision support, patient monitoring, natural language processing, and telemedicine, all of which enhance efficiency, accuracy, and patient outcomes in healthcare. Later, Kai asks the AI who he is and what his major is, to which the AI confirms that he is Kai, a Master of Computer Science student at KK's Classroom.", additional_kwargs={}, response_metadata={})]
```

### Using LLMChain with Memory: No Manual Updates Required

Replaced the "Step 4: Build the LCEL Chain'

```python
from langchain.chains import LLMChain

# Define the chain using a language model, prompt, and memory
chain = LLMChain(
    llm=llm,
    prompt=prompt,
    memory=memory,
    verbose=True  # Enable detailed execution logs for debugging
)
```


## 🧩 Key Concepts Explained

### Chain Composition with `|` Operator

In LangChain's LCEL (LangChain Expression Language), components are composed using the `|` operator:

```python
chain = RunnablePassthrough.assign(...) 
    | prompt 
    | llm 
    | StrOutputParser()
```

- **RunnablePassthrough**: Passes the original input or adds dynamic variables.
- **PromptTemplate**: Formats inputs into structured prompts.
- **LLM (ChatOpenAI)**: The large language model that generates responses.
- **StrOutputParser**: Extracts the raw text from the LLM output.


### Memory Management Essentials
LangChain separates **execution logic** from **state management**. In the LCEL chain, you must explicitly update memory:
- **load_memory_variables()**: Injects current summary (or history) into the prompt.
- **save_context()**: Persists the latest user–AI turn into memory.

LCEL-style chains do **not** auto-update memory after execution — unlike traditional LLMChain — so **manual saving** is required.


### Variable Binding with assign()
Use RunnablePassthrough.assign() to **inject runtime variables** into downstream components:
- `RunnablePassthrough.assign(chat_history=lambda _: memory.load_memory_variables({})["chat_history"])`

This ensures that **memory context (chat_history) is dynamically fetched** and passed into the prompt template.


### Naming Consistency Matters
nsure the memory_key matches the variable name in the prompt. For example:

- memory_key = "**chat_history**"
- PromptTemplate: "Current summary: {**chat_history**}"

This mismatch will lead to undefined variables during prompt rendering.


## 🆚 LCEL vs. LLMChain

| Feature            | LCEL Implementation                          | LLMChain Implementation                     |
|--------------------|----------------------------------------------|---------------------------------------------|
| **Memory Management** | Requires manual `save_context()` and `load_memory_variables()` | Automatically manages memory on each call |
| **Chain Composition** | Supports arbitrary step combinations using `|` operator | Uses fixed structure (LLM + Prompt + Memory) |
| **Debug Flexibility** | Allows custom middlewares (e.g., logging, tracing) | Controlled via `verbose=True` flag only |
| **Extensibility**     | Ideal for routing, branching, and nested chains | Best for linear, straightforward workflows |







<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



