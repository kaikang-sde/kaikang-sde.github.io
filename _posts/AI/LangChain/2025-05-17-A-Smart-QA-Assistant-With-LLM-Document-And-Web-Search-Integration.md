---
title: A Smart Q&A Assistant with LLM - Document + Web Search Integration
date: 2025-05-07
order: 13
categories: [AI, LangChain]
tags: [LangChain, Milvus, Web Search, Agent, VectorDB, ReAct, ChatOpenAI]
author: kai
---

# 🚀 A Smart Q&A Assistant with LLM - Document + Web Search Integration
The development of an intelligent Q&A assistant powered by an LLM agent that integrates **local document knowledge** (via Milvus vector database) with **real-time web search**. This hybrid design enables the system to solve both **technical documentation queries** and **live information requests**, supporting interactive and explainable multi-step reasoning.

## 🧠 Core Features
### Multimodal Knowledge Integration

This intelligent agent seamlessly integrates:

- **Local document knowledge** powered by the Milvus vector database
- **Real-time web search capabilities** via Search API or similar tools

Together, this enables step-by-step reasoning and answers for complex queries that combine both static knowledge and live data.

It supports:

- **Structured document Q&A** (e.g., explanations of technical frameworks)
- **Dynamic information queries** (e.g., stock prices, current dates, weather)


### Intelligent Tool Routing

The agent automatically **classifies the question type** and selects the appropriate tool:

- **Documentation-related queries**  
  → Uses the **Milvus vector search tool** to match content from local knowledge base  
  *(e.g., “What is Milvus?” → fetch answer from internal docs)*

- **Real-time data requests**  
  → Triggers the **web search tool** for the latest information  
  *(e.g., “What’s Apple's stock price?” → fetch from search engine)*

It also supports **multi-intent queries**, such as:

> “Explain Milvus, tell me today’s date, and check Apple's latest stock price.”


### Enhanced Interactive Answering

Built on the **LangChain Agent framework**, the assistant dynamically displays its decision-making process and tool execution flow:

- Follows a **"Reason → Act → Observe"** loop
- Shows intermediate steps and tool invocations for transparency
- Returns **structured outputs** with source labels, such as:
  - *"Sourced from Milvus Docs"*
  - *"Fetched via Web Search"*

This improves **traceability, explainability, and trust** in every response.


## 🧰 Technical Highlights
### Toolchain Integration

This intelligent assistant seamlessly integrates multiple components into a unified reasoning system:

- **Milvus Vector Search** — for semantic document retrieval
- **Web Search API** — to fetch real-time information
- **LLM Inference (e.g., gpt-4o)** — for reasoning, orchestration, and output generation

All components are orchestrated through the LangChain framework to deliver intelligent, multi-modal responses.


### Visualized Execution Flow with LangSmith

- Every step of the agent's decision-making process is **fully traceable** via [LangSmith](https://smith.langchain.com/)
- Developers can inspect:
  - Tool calls
  - Model thoughts
  - Intermediate observations
  - Latency and success metrics

This improves **debugging, observability, and explainability** of complex agent behavior.

### Customizable Prompt Templates

- Supports **custom prompt engineering** to control agent behavior.
- Easily switch between modes like:
  - `create_tool_calling_agent` – for structured API invocation
  - `create_react_agent` – for step-by-step reasoning (ReAct)

This makes it adaptable to a wide range of use cases.


## 📌 Application Scenarios

### Technical Documentation Q&A

- Quickly answer questions about **Milvus versions, features, and LangChain integration**
- Example:  
  *"What’s the difference between Milvus 2.x and Milvus 1.x?"*

### Real-Time Data Queries

- Fetch **up-to-date information** such as:
  - Stock prices (e.g., Tencent)
  - Current date or weather

### Multi-Step Reasoning Tasks

Handle compound queries such as:

> *"Explain how Milvus works, compare it with Faiss, and give me the latest tutorial from GitHub."*

The agent will autonomously:
1. Retrieve technical explanation from internal docs
2. Perform a comparative analysis
3. Trigger a live web search for the GitHub tutorial


## 👀 Example

### Setup and Tools

```python
from langchain_community.utilities import SearchApiAPIWrapper
from langchain_core.tools import tool
from langchain.agents import AgentExecutor, create_tool_calling_agent, create_react_agent

from langchain.tools.retriever import create_retriever_tool
from langchain_milvus import Milvus

from langchain.prompts import PromptTemplate
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
import os

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "*"
os.environ["LANGSMITH_ENDPOINT"] = "https://api.smith.langchain.com"
os.environ["LANGSMITH_PROJECT"] = "qa_agent_v1"

os.environ["SEARCHAPI_API_KEY"] = "*"
search = SearchApiAPIWrapper()

os.environ["OPENAI_API_KEY"] = "*"
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

openai_embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small",
    max_retries=3,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# Initialize the vector store
vector_store = Milvus(
    openai_embeddings,
    collection_name="doc_qa_db",  # Data already in Milvus
    connection_args={"uri": "http://54.163.61.180:19530",
                     "db_name": "my_database"}
)

retriever = vector_store.as_retriever()

# Tool 1:Milvus Vector Retrieval
# Convert the retriever to a tool
retriever_tool = create_retriever_tool(
    retriever,
    "milvus_retriever",
    "Search Milvus-related content. Must be used for any question about Milvus!"
)


# Tool 2: Real-Time Web Search Tool
@tool("web_search")
def web_search(query: str) -> str:
    """Get real-time information for unknown or current events."""
    try:
        results = search.results(query)
        return "\n\n".join([
            f"Source: {res['title']}\nSnippet: {res['snippet']}"
            for res in results['organic_results']
        ])
    except Exception as e:
        return f"Search failed: {str(e)}"


tools = [web_search, retriever_tool]
```


### Using create_tool_calling_agent

```python
# Construct the prompt template, using create_tool_calling_agent
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are an intelligent AI document assistant with access to multiple tools. Use them when necessary to answer questions."),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

agent = create_tool_calling_agent(
    llm=llm,
    tools=tools,
    prompt=prompt
)


# Initialize the agent executor with tracing and debugging enabled
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    handle_parsing_errors=True,
    return_intermediate_steps=True
)


# Wrapper function for running queries
def run_question(question: str):
    print(f"\nQuestion: {question}")
    result = agent_executor.invoke({"input": question})
    print(f"\nLLM Raw Result:\n{result}")
    print(f"\nFinal Answer:\n{result['output']}\n{'='*50}")

    for step in result["intermediate_steps"]:
        print(f"\nAction:{step[0].tool}")
        print(f"\nAction Input:{step[0].tool_input}")
        print(f"\nObservation:{step[1]}")


run_question("Explain Milvus and compare it with Faiss. Also, check today's stock price of Apple.")

# > Entering new AgentExecutor chain...

# Invoking: `milvus_retriever` with `{'query': 'Milvus overview and comparison with Faiss'}`
# What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x​About MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?


# Milvus 
#  is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.

# Default values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.
# For details, refer to Nullable & Default Value.
# Faiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.

# Milvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.
# Milvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.
# Milvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.

# What Makes Milvus so Fast？

# Milvus was designed from day one to be a highly efficient vector database system. In most cases, Milvus outperforms other vector databases by 2-5x (see the VectorDBBench results). This high performance is the result of several key design decisions:
# Hardware-aware Optimization: To accommodate Milvus in various hardware environments, we have optimized its performance specifically for many hardware architectures and platforms, including AVX512, SIMD, GPUs, and NVMe SSD.
# Advanced Search Algorithms: Milvus supports a wide range of in-memory and on-disk indexing/search algorithms, including IVF, HNSW, DiskANN, and more, all of which have been deeply optimized. Compared to popular implementations like FAISS and HNSWLib, Milvus delivers 30%-70% better performance.
# Invoking: `web_search` with `{'query': 'Apple stock price'}`


# Source: Stock Price
# Snippet: Stock Quote: NASDAQ: AAPL · Day's Open203.10 · Closing Price198.89 · Volume69.0M · Intraday High204.10 · Intraday Low198.21. Apple · Investor Relations; > ...

# Source: Apple Inc. Stock Quote (U.S.: Nasdaq) - AAPL
# Snippet: Apple Inc. · 197.76 · Recently Viewed Tickers · AAPL Overview · Key Data · Performance · Analyst Ratings · Top Stories On Apple Inc. · News From Dow Jones. Read ...

# Source: Apple Inc. (AAPL)
# Snippet: Apple Inc. (AAPL) · -6.18% · 5.19% · -11.03% · -20.87% · 9.06% · 163.67% · 154,296.64%.

# Source: Apple Inc AAPL:NASDAQ - Stock Price, Quote and News
# Snippet: Apple Inc AAPL:NASDAQ · Open198.21 · Day High199.88 · Day Low197.02 · Prev Close198.89 · 52 Week High260.10 · 52 Week High Date12/26/24 · 52 Week Low169.21 · 52 ...

# Source: Apple Inc. (AAPL) Stock Price Today
# Snippet: Apple Inc. · REAL TIME 9:33 AM EDT 05/06/25 · 198.97USD · 0.080.04% · Volume4,518,904. Volume4,518,904. 65 Day Avg Vol60,206,330. 1 ...

# Source: NASDAQ:AAPL Stock Price - Apple Inc
# Snippet: The current price of AAPL is 198.03 USD — it has decreased by −3.08% in the past 24 hours. Watch Apple Inc stock price performance more closely on the chart.

# Source: AAPL Stock Quote Price and Forecast
# Snippet: View Apple Inc. AAPL stock quote prices, financial information, real-time forecasts, and company news from CNN.

# Source: Apple Stock Price Today | NASDAQ: AAPL Live
# Snippet: What Is the Current Stock Price of Apple? The Apple Inc stock price is 198.62. Who Founded Apple? Apple was founded on April 1, 1976, by Steve Jobs, Steve ...### Overview of Milvus

# Milvus is an open-source vector database that is designed to manage and analyze large-scale vector data. It is optimized for efficiency, scalability, and performance, making it suitable for applications involving artificial intelligence, machine learning, and big data. Key features include:

# - **High Performance**: Milvus is built for speed, often outperforming other vector databases by 2-5x. It utilizes hardware-aware optimizations and advanced search algorithms to enhance efficiency.
# - **Flexible Deployment**: Milvus can be deployed in multiple configurations, including standalone for single-machine setups and distributed on Kubernetes for larger scale operations.
# - **Support for Multiple Indexing Algorithms**: It supports various indexing/search algorithms like IVF, HNSW, and DiskANN, with optimizations leading to better performance compared to competitors.

# ### Comparison with Faiss

# Faiss (Facebook AI Similarity Search) is another popular library used for efficient similarity search and clustering of dense vectors. Here's how Milvus compares with Faiss:

# - **Performance**: Milvus often delivers 30%-70% better performance than Faiss due to its specialized optimizations and advanced algorithms.
# - **Ease of Use**: Milvus provides a more user-friendly interface and is designed to handle larger datasets seamlessly, making it suitable for production environments.
# - **Deployment Options**: While Faiss is primarily a library that requires integration into applications, Milvus offers both a library and a fully managed service, making it easier to use for various applications.

# ### Current Stock Price of Apple (AAPL)

# As of now, the stock price of Apple Inc. (AAPL) is approximately **$198.62**. The stock has seen a decrease of about **3.08%** in the past 24 hours. 

# If you have any further questions or need more details, feel free to ask!

# > Finished chain.


# LLM Raw Result:
# {'input': "Explain Milvus and compare it with Faiss. Also, check today's stock price of Apple.", 'output': "### Overview of Milvus\n\nMilvus is an open-source vector database that is designed to manage and analyze large-scale vector data. It is optimized for efficiency, scalability, and performance, making it suitable for applications involving artificial intelligence, machine learning, and big data. Key features include:\n\n- **High Performance**: Milvus is built for speed, often outperforming other vector databases by 2-5x. It utilizes hardware-aware optimizations and advanced search algorithms to enhance efficiency.\n- **Flexible Deployment**: Milvus can be deployed in multiple configurations, including standalone for single-machine setups and distributed on Kubernetes for larger scale operations.\n- **Support for Multiple Indexing Algorithms**: It supports various indexing/search algorithms like IVF, HNSW, and DiskANN, with optimizations leading to better performance compared to competitors.\n\n### Comparison with Faiss\n\nFaiss (Facebook AI Similarity Search) is another popular library used for efficient similarity search and clustering of dense vectors. Here's how Milvus compares with Faiss:\n\n- **Performance**: Milvus often delivers 30%-70% better performance than Faiss due to its specialized optimizations and advanced algorithms.\n- **Ease of Use**: Milvus provides a more user-friendly interface and is designed to handle larger datasets seamlessly, making it suitable for production environments.\n- **Deployment Options**: While Faiss is primarily a library that requires integration into applications, Milvus offers both a library and a fully managed service, making it easier to use for various applications.\n\n### Current Stock Price of Apple (AAPL)\n\nAs of now, the stock price of Apple Inc. (AAPL) is approximately **$198.62**. The stock has seen a decrease of about **3.08%** in the past 24 hours. \n\nIf you have any further questions or need more details, feel free to ask!", 'intermediate_steps': [(ToolAgentAction(tool='milvus_retriever', tool_input={'query': 'Milvus overview and comparison with Faiss'}, log="\nInvoking: `milvus_retriever` with `{'query': 'Milvus overview and comparison with Faiss'}`\n\n\n", message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_eGsr6t93i1Yuuys9cSDDDhzY', 'function': {'arguments': '{"query": "Milvus overview and comparison with Faiss"}', 'name': 'milvus_retriever'}, 'type': 'function'}, {'index': 1, 'id': 'call_BIDqmuD7v96ptLtSPvSqI682', 'function': {'arguments': '{"query": "Apple stock price"}', 'name': 'web_search'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls', 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_dbaca60df0'}, id='run-baee3603-d8bf-445f-8f5d-078f95f91efa', tool_calls=[{'name': 'milvus_retriever', 'args': {'query': 'Milvus overview and comparison with Faiss'}, 'id': 'call_eGsr6t93i1Yuuys9cSDDDhzY', 'type': 'tool_call'}, {'name': 'web_search', 'args': {'query': 'Apple stock price'}, 'id': 'call_BIDqmuD7v96ptLtSPvSqI682', 'type': 'tool_call'}], tool_call_chunks=[{'name': 'milvus_retriever', 'args': '{"query": "Milvus overview and comparison with Faiss"}', 'id': 'call_eGsr6t93i1Yuuys9cSDDDhzY', 'index': 0, 'type': 'tool_call_chunk'}, {'name': 'web_search', 'args': '{"query": "Apple stock price"}', 'id': 'call_BIDqmuD7v96ptLtSPvSqI682', 'index': 1, 'type': 'tool_call_chunk'}])], tool_call_id='call_eGsr6t93i1Yuuys9cSDDDhzY'), 'What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x\u200bAbout MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?\n\n\n\nMilvus \n\n\n\n\n is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.\n\nDefault values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.\nFor details, refer to Nullable & Default Value.\nFaiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.\n\nMilvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.\nMilvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.\nMilvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.\n\nWhat Makes Milvus so Fast？\n\nMilvus was designed from day one to be a highly efficient vector database system. In most cases, Milvus outperforms other vector databases by 2-5x (see the VectorDBBench results). This high performance is the result of several key design decisions:\nHardware-aware Optimization: To accommodate Milvus in various hardware environments, we have optimized its performance specifically for many hardware architectures and platforms, including AVX512, SIMD, GPUs, and NVMe SSD.\nAdvanced Search Algorithms: Milvus supports a wide range of in-memory and on-disk indexing/search algorithms, including IVF, HNSW, DiskANN, and more, all of which have been deeply optimized. Compared to popular implementations like FAISS and HNSWLib, Milvus delivers 30%-70% better performance.'), (ToolAgentAction(tool='web_search', tool_input={'query': 'Apple stock price'}, log="\nInvoking: `web_search` with `{'query': 'Apple stock price'}`\n\n\n", message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_eGsr6t93i1Yuuys9cSDDDhzY', 'function': {'arguments': '{"query": "Milvus overview and comparison with Faiss"}', 'name': 'milvus_retriever'}, 'type': 'function'}, {'index': 1, 'id': 'call_BIDqmuD7v96ptLtSPvSqI682', 'function': {'arguments': '{"query": "Apple stock price"}', 'name': 'web_search'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls', 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_dbaca60df0'}, id='run-baee3603-d8bf-445f-8f5d-078f95f91efa', tool_calls=[{'name': 'milvus_retriever', 'args': {'query': 'Milvus overview and comparison with Faiss'}, 'id': 'call_eGsr6t93i1Yuuys9cSDDDhzY', 'type': 'tool_call'}, {'name': 'web_search', 'args': {'query': 'Apple stock price'}, 'id': 'call_BIDqmuD7v96ptLtSPvSqI682', 'type': 'tool_call'}], tool_call_chunks=[{'name': 'milvus_retriever', 'args': '{"query": "Milvus overview and comparison with Faiss"}', 'id': 'call_eGsr6t93i1Yuuys9cSDDDhzY', 'index': 0, 'type': 'tool_call_chunk'}, {'name': 'web_search', 'args': '{"query": "Apple stock price"}', 'id': 'call_BIDqmuD7v96ptLtSPvSqI682', 'index': 1, 'type': 'tool_call_chunk'}])], tool_call_id='call_BIDqmuD7v96ptLtSPvSqI682'), "Source: Stock Price\nSnippet: Stock Quote: NASDAQ: AAPL · Day's Open203.10 · Closing Price198.89 · Volume69.0M · Intraday High204.10 · Intraday Low198.21. Apple · Investor Relations; > ...\n\nSource: Apple Inc. Stock Quote (U.S.: Nasdaq) - AAPL\nSnippet: Apple Inc. · 197.76 · Recently Viewed Tickers · AAPL Overview · Key Data · Performance · Analyst Ratings · Top Stories On Apple Inc. · News From Dow Jones. Read ...\n\nSource: Apple Inc. (AAPL)\nSnippet: Apple Inc. (AAPL) · -6.18% · 5.19% · -11.03% · -20.87% · 9.06% · 163.67% · 154,296.64%.\n\nSource: Apple Inc AAPL:NASDAQ - Stock Price, Quote and News\nSnippet: Apple Inc AAPL:NASDAQ · Open198.21 · Day High199.88 · Day Low197.02 · Prev Close198.89 · 52 Week High260.10 · 52 Week High Date12/26/24 · 52 Week Low169.21 · 52 ...\n\nSource: Apple Inc. (AAPL) Stock Price Today\nSnippet: Apple Inc. · REAL TIME 9:33 AM EDT 05/06/25 · 198.97USD · 0.080.04% · Volume4,518,904. Volume4,518,904. 65 Day Avg Vol60,206,330. 1 ...\n\nSource: NASDAQ:AAPL Stock Price - Apple Inc\nSnippet: The current price of AAPL is 198.03 USD — it has decreased by −3.08% in the past 24 hours. Watch Apple Inc stock price performance more closely on the chart.\n\nSource: AAPL Stock Quote Price and Forecast\nSnippet: View Apple Inc. AAPL stock quote prices, financial information, real-time forecasts, and company news from CNN.\n\nSource: Apple Stock Price Today | NASDAQ: AAPL Live\nSnippet: What Is the Current Stock Price of Apple? The Apple Inc stock price is 198.62. Who Founded Apple? Apple was founded on April 1, 1976, by Steve Jobs, Steve ...")]}

# Final Answer:
# ### Overview of Milvus

# Milvus is an open-source vector database that is designed to manage and analyze large-scale vector data. It is optimized for efficiency, scalability, and performance, making it suitable for applications involving artificial intelligence, machine learning, and big data. Key features include:

# - **High Performance**: Milvus is built for speed, often outperforming other vector databases by 2-5x. It utilizes hardware-aware optimizations and advanced search algorithms to enhance efficiency.
# - **Flexible Deployment**: Milvus can be deployed in multiple configurations, including standalone for single-machine setups and distributed on Kubernetes for larger scale operations.
# - **Support for Multiple Indexing Algorithms**: It supports various indexing/search algorithms like IVF, HNSW, and DiskANN, with optimizations leading to better performance compared to competitors.

# ### Comparison with Faiss

# Faiss (Facebook AI Similarity Search) is another popular library used for efficient similarity search and clustering of dense vectors. Here's how Milvus compares with Faiss:

# - **Performance**: Milvus often delivers 30%-70% better performance than Faiss due to its specialized optimizations and advanced algorithms.
# - **Ease of Use**: Milvus provides a more user-friendly interface and is designed to handle larger datasets seamlessly, making it suitable for production environments.
# - **Deployment Options**: While Faiss is primarily a library that requires integration into applications, Milvus offers both a library and a fully managed service, making it easier to use for various applications.

# ### Current Stock Price of Apple (AAPL)

# As of now, the stock price of Apple Inc. (AAPL) is approximately **$198.62**. The stock has seen a decrease of about **3.08%** in the past 24 hours. 

# If you have any further questions or need more details, feel free to ask!
# ==================================================

# Action:milvus_retriever

# Action Input:{'query': 'Milvus overview and comparison with Faiss'}

# Observation:What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x​About MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?



# Milvus 
#  is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.

# Default values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.
# For details, refer to Nullable & Default Value.
# Faiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.

# Milvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.
# Milvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.
# Milvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.

# What Makes Milvus so Fast？

# Milvus was designed from day one to be a highly efficient vector database system. In most cases, Milvus outperforms other vector databases by 2-5x (see the VectorDBBench results). This high performance is the result of several key design decisions:
# Hardware-aware Optimization: To accommodate Milvus in various hardware environments, we have optimized its performance specifically for many hardware architectures and platforms, including AVX512, SIMD, GPUs, and NVMe SSD.
# Advanced Search Algorithms: Milvus supports a wide range of in-memory and on-disk indexing/search algorithms, including IVF, HNSW, DiskANN, and more, all of which have been deeply optimized. Compared to popular implementations like FAISS and HNSWLib, Milvus delivers 30%-70% better performance.

# Action:web_search

# Action Input:{'query': 'Apple stock price'}

# Observation:Source: Stock Price
# Snippet: Stock Quote: NASDAQ: AAPL · Day's Open203.10 · Closing Price198.89 · Volume69.0M · Intraday High204.10 · Intraday Low198.21. Apple · Investor Relations; > ...

# Source: Apple Inc. Stock Quote (U.S.: Nasdaq) - AAPL
# Snippet: Apple Inc. · 197.76 · Recently Viewed Tickers · AAPL Overview · Key Data · Performance · Analyst Ratings · Top Stories On Apple Inc. · News From Dow Jones. Read ...

# Source: Apple Inc. (AAPL)
# Snippet: Apple Inc. (AAPL) · -6.18% · 5.19% · -11.03% · -20.87% · 9.06% · 163.67% · 154,296.64%.

# Source: Apple Inc AAPL:NASDAQ - Stock Price, Quote and News
# Snippet: Apple Inc AAPL:NASDAQ · Open198.21 · Day High199.88 · Day Low197.02 · Prev Close198.89 · 52 Week High260.10 · 52 Week High Date12/26/24 · 52 Week Low169.21 · 52 ...

# Source: Apple Inc. (AAPL) Stock Price Today
# Snippet: Apple Inc. · REAL TIME 9:33 AM EDT 05/06/25 · 198.97USD · 0.080.04% · Volume4,518,904. Volume4,518,904. 65 Day Avg Vol60,206,330. 1 ...

# Source: NASDAQ:AAPL Stock Price - Apple Inc
# Snippet: The current price of AAPL is 198.03 USD — it has decreased by −3.08% in the past 24 hours. Watch Apple Inc stock price performance more closely on the chart.

# Source: AAPL Stock Quote Price and Forecast
# Snippet: View Apple Inc. AAPL stock quote prices, financial information, real-time forecasts, and company news from CNN.

# Source: Apple Stock Price Today | NASDAQ: AAPL Live
# Snippet: What Is the Current Stock Price of Apple? The Apple Inc stock price is 198.62. Who Founded Apple? Apple was founded on April 1, 1976, by Steve Jobs, Steve ...
```

![LangSmith Track](/assets/img/posts/AI/LangChain/create_tool_calling_agent_trace.png)


### Using create_react_agent

```python
prompt = PromptTemplate.from_template(
    '''Answer the following questions as best you can. You have access to the following tools:
{tools}

Use the following format:

Question: the input question you must answer
Thought: you should always think about what to do
Action: the action to take, should be one of [{tool_names}]
Action Input: the input to the action
Observation: the result of the action
... (this Thought/Action/Action Input/Observation can repeat N times)
Thought: I now know the final answer
Final Answer: the final answer to the original input question

Begin!

Question: {input}
Thought:{agent_scratchpad}
''')

# Create the ReAct agent
agent = create_react_agent(
    llm=llm,
    tools=tools,
    prompt=prompt
)

# Initialize the agent executor with tracing and debugging enabled
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    handle_parsing_errors=True,
    return_intermediate_steps=True
)


# Wrapper function for running queries
def run_question(question: str):
    print(f"\nQuestion: {question}")
    result = agent_executor.invoke({"input": question})
    print(f"\nLLM Raw Result:\n{result}")
    print(f"\nFinal Answer:\n{result['output']}\n{'='*50}")

    for step in result["intermediate_steps"]:
        print(f"\nAction:{step[0].tool}")
        print(f"\nAction Input:{step[0].tool_input}")
        print(f"\nObservation:{step[1]}")


run_question(
    "Explain Milvus and compare it with Faiss. Also, check today's stock price of Apple.")



# > Entering new AgentExecutor chain...
# To address the question, I need to gather information about Milvus and its comparison with Faiss, as well as the current stock price of Apple. I will start by retrieving information about Milvus and Faiss.

# Action: milvus_retriever  
# Action Input: "Explain Milvus and compare it with Faiss."  What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x​About MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?

# Milvus 
#  is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.

# Default values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.
# For details, refer to Nullable & Default Value.
# Faiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.

# Table of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub

# Milvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.
# Milvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.
# Milvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.

# What Makes Milvus so Fast？To provide a thorough response, I need to obtain more detailed information about Milvus and its comparison with Faiss, as well as check the current stock price of Apple.

# Action: milvus_retriever  
# Action Input: "Explain Milvus and compare it with Faiss."  What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x​About MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?


# Milvus 
#  is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.

# Default values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.
# For details, refer to Nullable & Default Value.
# Faiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.

# Table of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub

# Milvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.
# Milvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.
# Milvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.

# What Makes Milvus so Fast？To effectively summarize the information about Milvus and its comparison with Faiss, I will need to gather more specific details regarding their functionalities and performance metrics. Additionally, I must check the current stock price of Apple.

# Action: milvus_retriever  
# Action Input: "Explain Milvus and compare it with Faiss."  What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x​About MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?



# Milvus 
#  is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.

# Default values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.
# For details, refer to Nullable & Default Value.
# Faiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.

# Table of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub

# Milvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.
# Milvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.
# Milvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.

# What Makes Milvus so Fast？I have not yet gathered sufficient information about Milvus and its comparison with Faiss, nor have I checked the current stock price of Apple. I will continue searching for comprehensive information on the comparison.

# Action: milvus_retriever  
# Action Input: "Explain Milvus and compare it with Faiss."  What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x​About MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?



# Milvus 
#  is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.

# Default values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.
# For details, refer to Nullable & Default Value.
# Faiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.

# Table of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub

# Milvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.
# Milvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.
# Milvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.

# What Makes Milvus so Fast？To provide a thorough response, I need to gather more specific information about Milvus and its comparison with Faiss, as well as check the current stock price of Apple. 

# I will now conduct a web search for the current stock price of Apple.

# Action: web_search  
# Action Input: "current stock price of Apple"  Source: Stock Price
# Snippet: Stock Quote: NASDAQ: AAPL ; Day's Open203.10 ; Closing Price198.89 ; Volume69.0 ; Intraday High204.10 ; Intraday Low198.21.

# Source: Apple Inc AAPL:NASDAQ - Stock Price, Quote and News
# Snippet: Apple Inc AAPL:NASDAQ ; Open198.21 ; Day High199.88 ; Day Low197.02 ; Prev Close198.89 ; 52 Week High260.10 ...

# Source: Apple Inc. (AAPL) Stock Price, News, Quote & History
# Snippet: Find the latest Apple Inc. (AAPL) stock quote, history, news and other vital information to help you with your stock trading and investing.

# Source: Apple Inc. Stock Quote (U.S.: Nasdaq) - AAPL
# Snippet: 197.76 ; Volume: 69.02M · 65 Day Avg: 60.21M ; 198.21 Day Range 204.10 ; 169.21 52 Week Range 260.10 ...

# Source: AAPL Stock Quote Price and Forecast
# Snippet: 1-year stock price forecast. High $308.00 49.99% Median $237.00 15.41% Low $141.00 31.34% Current Price 205.35 past 12 months next 12 months ...

# Source: Apple Inc. (AAPL) Stock Price Today
# Snippet: Apple Inc. · REAL TIME 9:33 AM EDT 05/06/25 · 198.97USD · 0.080.04% · Volume4,518,904.

# Source: NASDAQ:AAPL Stock Price - Apple Inc
# Snippet: The current price of AAPL is 198.03 USD — it has decreased by −3.08% in the past 24 hours. Watch Apple Inc stock price performance more closely on the chart.

# Source: Apple Stock Price | AAPL Stock Quote, News, and History
# Snippet: On June 21, with Apple's stock price at $101.25, Apple issued two shares to investors at $55.62. Five years later, with Apple stock price at an ever-higher ...I have retrieved the current stock price of Apple, which is approximately $198.03 USD. Now, I need to finalize the comparison between Milvus and Faiss.

# Action: milvus_retriever  
# Action Input: "Explain Milvus and compare it with Faiss."  What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x​About MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?



# Milvus 
#  is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.

# Default values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.
# For details, refer to Nullable & Default Value.
# Faiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.

# Table of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub

# Milvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.
# Milvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.
# Milvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.

# What Makes Milvus so Fast？I have gathered enough information to provide a comprehensive answer regarding Milvus and its comparison with Faiss, as well as the current stock price of Apple.

# **Milvus:**
# Milvus is an open-source vector database designed to manage, search, and analyze large-scale embedding data. It supports various types of searches and can handle unstructured data, such as images and text. Milvus is highly scalable and optimized for performance, allowing users to perform real-time searches efficiently.

# Key features of Milvus include:
# - **Support for various data types**: It allows for flexible data modeling, including support for nullable and default values, simplifying data migration from other systems.
# - **Deployment options**: Milvus can be deployed in various configurations, including standalone and distributed modes, to cater to different needs ranging from single-machine setups to cloud-native architectures.
# - **Integration with Faiss**: Milvus has integrated the HNSW algorithm from Faiss, enhancing its stability and performance in high-dimensional data searches.

# **Faiss:**
# Faiss (Facebook AI Similarity Search) is a library for efficient similarity search and clustering of dense vectors. It is highly optimized for speed and memory usage, making it suitable for large-scale machine learning applications.

# **Comparison between Milvus and Faiss:**
# - **Purpose**: While Faiss is primarily a library focused on high-performance similarity search, Milvus is a full-fledged database that provides additional features for data management, including storage, querying, and scalability.
# - **Ease of Use**: Milvus offers a more user-friendly interface for managing vector data and integrates various functionalities for application development, whereas Faiss requires more manual setup for use.
# - **Deployment and Scalability**: Milvus supports distributed deployments and is designed to handle billions of vectors efficiently, while Faiss is typically used in more static environments.
# - **Community and Support**: Milvus has a growing community and provides extensive documentation, whereas Faiss is backed by Facebook and is widely used in research and industry.

# **Current Stock Price of Apple:**
# As of the latest information, the stock price of Apple Inc. (AAPL) is approximately **$198.03** USD.

# **Final Answer:**
# Milvus is a vector database designed for managing and searching large-scale embedding data, offering features like flexible data modeling and various deployment options, including integration with Faiss for enhanced performance. In comparison, Faiss is a specialized library for similarity search. The current stock price of Apple is approximately $198.03 USD.

# > Finished chain.


# LLM Raw Result:
# {'input': "Explain Milvus and compare it with Faiss. Also, check today's stock price of Apple.", 'output': '**\nMilvus is a vector database designed for managing and searching large-scale embedding data, offering features like flexible data modeling and various deployment options, including integration with Faiss for enhanced performance. In comparison, Faiss is a specialized library for similarity search. The current stock price of Apple is approximately $198.03 USD.', 'intermediate_steps': [(AgentAction(tool='milvus_retriever', tool_input='Explain Milvus and compare it with Faiss.', log='To address the question, I need to gather information about Milvus and its comparison with Faiss, as well as the current stock price of Apple. I will start by retrieving information about Milvus and Faiss.\n\nAction: milvus_retriever  \nAction Input: "Explain Milvus and compare it with Faiss."  '), 'What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x\u200bAbout MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?\n\n\n\nMilvus \n\n\n\n\n is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.\n\nDefault values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.\nFor details, refer to Nullable & Default Value.\nFaiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.\n\nTable of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub\n\nMilvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.\nMilvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.\nMilvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.\n\nWhat Makes Milvus so Fast？'), (AgentAction(tool='milvus_retriever', tool_input='Explain Milvus and compare it with Faiss.', log='To provide a thorough response, I need to obtain more detailed information about Milvus and its comparison with Faiss, as well as check the current stock price of Apple.\n\nAction: milvus_retriever  \nAction Input: "Explain Milvus and compare it with Faiss."  '), 'What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x\u200bAbout MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?\n\n\n\nMilvus \n\n\n\n\n is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.\n\nDefault values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.\nFor details, refer to Nullable & Default Value.\nFaiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.\n\nTable of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub\n\nMilvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.\nMilvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.\nMilvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.\n\nWhat Makes Milvus so Fast？'), (AgentAction(tool='milvus_retriever', tool_input='Explain Milvus and compare it with Faiss.', log='To effectively summarize the information about Milvus and its comparison with Faiss, I will need to gather more specific details regarding their functionalities and performance metrics. Additionally, I must check the current stock price of Apple.\n\nAction: milvus_retriever  \nAction Input: "Explain Milvus and compare it with Faiss."  '), 'What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x\u200bAbout MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?\n\n\n\nMilvus \n\n\n\n\n is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.\n\nDefault values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.\nFor details, refer to Nullable & Default Value.\nFaiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.\n\nTable of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub\n\nMilvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.\nMilvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.\nMilvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.\n\nWhat Makes Milvus so Fast？'), (AgentAction(tool='milvus_retriever', tool_input='Explain Milvus and compare it with Faiss.', log='I have not yet gathered sufficient information about Milvus and its comparison with Faiss, nor have I checked the current stock price of Apple. I will continue searching for comprehensive information on the comparison.\n\nAction: milvus_retriever  \nAction Input: "Explain Milvus and compare it with Faiss."  '), 'What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x\u200bAbout MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?\n\n\n\nMilvus \n\n\n\n\n is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.\n\nDefault values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.\nFor details, refer to Nullable & Default Value.\nFaiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.\n\nTable of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub\n\nMilvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.\nMilvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.\nMilvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.\n\nWhat Makes Milvus so Fast？'), (AgentAction(tool='web_search', tool_input='current stock price of Apple', log='To provide a thorough response, I need to gather more specific information about Milvus and its comparison with Faiss, as well as check the current stock price of Apple. \n\nI will now conduct a web search for the current stock price of Apple.\n\nAction: web_search  \nAction Input: "current stock price of Apple"  '), "Source: Stock Price\nSnippet: Stock Quote: NASDAQ: AAPL ; Day's Open203.10 ; Closing Price198.89 ; Volume69.0 ; Intraday High204.10 ; Intraday Low198.21.\n\nSource: Apple Inc AAPL:NASDAQ - Stock Price, Quote and News\nSnippet: Apple Inc AAPL:NASDAQ ; Open198.21 ; Day High199.88 ; Day Low197.02 ; Prev Close198.89 ; 52 Week High260.10 ...\n\nSource: Apple Inc. (AAPL) Stock Price, News, Quote & History\nSnippet: Find the latest Apple Inc. (AAPL) stock quote, history, news and other vital information to help you with your stock trading and investing.\n\nSource: Apple Inc. Stock Quote (U.S.: Nasdaq) - AAPL\nSnippet: 197.76 ; Volume: 69.02M · 65 Day Avg: 60.21M ; 198.21 Day Range 204.10 ; 169.21 52 Week Range 260.10 ...\n\nSource: AAPL Stock Quote Price and Forecast\nSnippet: 1-year stock price forecast. High $308.00 49.99% Median $237.00 15.41% Low $141.00 31.34% Current Price 205.35 past 12 months next 12 months ...\n\nSource: Apple Inc. (AAPL) Stock Price Today\nSnippet: Apple Inc. · REAL TIME 9:33 AM EDT 05/06/25 · 198.97USD · 0.080.04% · Volume4,518,904.\n\nSource: NASDAQ:AAPL Stock Price - Apple Inc\nSnippet: The current price of AAPL is 198.03 USD — it has decreased by −3.08% in the past 24 hours. Watch Apple Inc stock price performance more closely on the chart.\n\nSource: Apple Stock Price | AAPL Stock Quote, News, and History\nSnippet: On June 21, with Apple's stock price at $101.25, Apple issued two shares to investors at $55.62. Five years later, with Apple stock price at an ever-higher ..."), (AgentAction(tool='milvus_retriever', tool_input='Explain Milvus and compare it with Faiss.', log='I have retrieved the current stock price of Apple, which is approximately $198.03 USD. Now, I need to finalize the comparison between Milvus and Faiss.\n\nAction: milvus_retriever  \nAction Input: "Explain Milvus and compare it with Faiss."  '), 'What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x\u200bAbout MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?\n\n\n\nMilvus \n\n\n\n\n is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.\n\nDefault values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.\nFor details, refer to Nullable & Default Value.\nFaiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.\n\nTable of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub\n\nMilvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.\nMilvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.\nMilvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.\n\nWhat Makes Milvus so Fast？')]}

# Final Answer:
# **
# Milvus is a vector database designed for managing and searching large-scale embedding data, offering features like flexible data modeling and various deployment options, including integration with Faiss for enhanced performance. In comparison, Faiss is a specialized library for similarity search. The current stock price of Apple is approximately $198.03 USD.
# ==================================================

# Action:milvus_retriever

# Action Input:Explain Milvus and compare it with Faiss.

# Observation:What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x​About MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?


# Milvus 
#  is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.

# Default values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.
# For details, refer to Nullable & Default Value.
# Faiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.

# Table of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub

# Milvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.
# Milvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.
# Milvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.

# What Makes Milvus so Fast？

# Action:milvus_retriever

# Action Input:Explain Milvus and compare it with Faiss.

# Observation:What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x​About MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?

# Milvus 
#  is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.

# Default values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.
# For details, refer to Nullable & Default Value.
# Faiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.

# Table of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub

# Milvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.
# Milvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.
# Milvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.

# What Makes Milvus so Fast？

# Action:milvus_retriever

# Action Input:Explain Milvus and compare it with Faiss.

# Observation:What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x​About MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?



# Milvus 
#  is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.

# Default values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.
# For details, refer to Nullable & Default Value.
# Faiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.

# Table of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub

# Milvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.
# Milvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.
# Milvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.

# What Makes Milvus so Fast？

# Action:milvus_retriever

# Action Input:Explain Milvus and compare it with Faiss.

# Observation:What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x​About MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?



# Milvus 
#  is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.

# Default values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.
# For details, refer to Nullable & Default Value.
# Faiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.

# Table of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub

# Milvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.
# Milvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.
# Milvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.

# What Makes Milvus so Fast？

# Action:web_search

# Action Input:current stock price of Apple

# Observation:Source: Stock Price
# Snippet: Stock Quote: NASDAQ: AAPL ; Day's Open203.10 ; Closing Price198.89 ; Volume69.0 ; Intraday High204.10 ; Intraday Low198.21.

# Source: Apple Inc AAPL:NASDAQ - Stock Price, Quote and News
# Snippet: Apple Inc AAPL:NASDAQ ; Open198.21 ; Day High199.88 ; Day Low197.02 ; Prev Close198.89 ; 52 Week High260.10 ...

# Source: Apple Inc. (AAPL) Stock Price, News, Quote & History
# Snippet: Find the latest Apple Inc. (AAPL) stock quote, history, news and other vital information to help you with your stock trading and investing.

# Source: Apple Inc. Stock Quote (U.S.: Nasdaq) - AAPL
# Snippet: 197.76 ; Volume: 69.02M · 65 Day Avg: 60.21M ; 198.21 Day Range 204.10 ; 169.21 52 Week Range 260.10 ...

# Source: AAPL Stock Quote Price and Forecast
# Snippet: 1-year stock price forecast. High $308.00 49.99% Median $237.00 15.41% Low $141.00 31.34% Current Price 205.35 past 12 months next 12 months ...

# Source: Apple Inc. (AAPL) Stock Price Today
# Snippet: Apple Inc. · REAL TIME 9:33 AM EDT 05/06/25 · 198.97USD · 0.080.04% · Volume4,518,904.

# Source: NASDAQ:AAPL Stock Price - Apple Inc
# Snippet: The current price of AAPL is 198.03 USD — it has decreased by −3.08% in the past 24 hours. Watch Apple Inc stock price performance more closely on the chart.

# Source: Apple Stock Price | AAPL Stock Quote, News, and History
# Snippet: On June 21, with Apple's stock price at $101.25, Apple issued two shares to investors at $55.62. Five years later, with Apple stock price at an ever-higher ...

# Action:milvus_retriever

# Action Input:Explain Milvus and compare it with Faiss.

# Observation:What is Milvus | Milvus Documentation🚀 Try Zilliz Cloud, the fully managed Milvus, for free—experience 10x faster performance! Try Now>>Why MilvusWhat is MilvusUse CasesDocsTutorialsBootcampDemosVideoToolsAttuMilvus CLISizing ToolMilvus BackupVTSDeep SearcherBlogCommunityDiscordGitHubMore ChannelsStar34.4KContact UsTry Managed MilvusDocsTutorialsToolsBlogCommunityContact UsTry Managed MilvusSearchHomev2.5.x​About MilvusWhat is MilvusMilvus AdoptersMilvus RoadmapReleasesBenchmarksComparisonGet StartedConceptsUser GuideModelsData ImportAdministration GuideToolsIntegrationsTutorialsFAQsAPI ReferenceHomeDocsAbout MilvusWhat is MilvusWhat is Milvus?



# Milvus 
#  is a bird of prey in the genus Milvus of the hawk family Accipaitridae, celebrated for its speed in flight, keen vision, and remarkable adaptability.

# Default values and nullable properties provide greater flexibility to Milvus. Users can utilize this feature for fields with uncertain values when creating collections. It also simplifies data migration from other database systems to Milvus, allowing for handling datasets containing null values while preserving original default value settings.
# For details, refer to Nullable & Default Value.
# Faiss-based HNSW SQ/PQ/PRQThrough close collaboration with the Faiss community, the HNSW algorithm in Faiss has seen significant improvements in both functionality and performance. For considerations of stability and maintainability, Milvus 2.5 has officially migrated its support for HNSW from hnswlib to Faiss.

# Table of contentsWhat is Milvus?Unstructured Data, Embeddings, and MilvusWhat Makes Milvus so Fast？What Makes Milvus so ScalableTypes of Searches Supported by MilvusComprehensive Feature SetTry Managed Milvus for FreeZilliz Cloud is hassle-free, powered by Milvus and 10x faster.Get StartedEdit this pageCreate an issueFeedbackWas this page helpful?Made with Love  by the Devs from ZillizGet Milvus UpdatesSubscribeCopyright © Milvus. 2025 All rights reserved.ResourcesDocsBlogManaged MilvusContact UsAI Quick Reference TutorialsBootcampsDemoVideoToolsAttuMilvus CLIMilvus Sizing ToolMilvus Backup ToolVector Transport Service (VTS)CommunityGet InvolvedDiscordGithub

# Milvus Lite is a Python library that can be easily integrated into your applications. As a lightweight version of Milvus, it’s ideal for quick prototyping in Jupyter Notebooks or running on edge devices with limited resources. Learn more.
# Milvus Standalone is a single-machine server deployment, with all components bundled into a single Docker image for convenient deployment. Learn more.
# Milvus Distributed can be deployed on Kubernetes clusters, featuring a cloud-native architecture designed for billion-scale or even larger scenarios. This architecture ensures redundancy in critical components. Learn more.

# What Makes Milvus so Fast？
```

![LangSmith Track](/assets/img/posts/AI/LangChain/create_react_agent_trace.png)




## 🧭 Summary: Choosing the Right LangChain Agent for Your LLM Application
In LLM-powered agent development, different tasks require different interaction logic. Selecting the right agent type is crucial for balancing flexibility, efficiency, and reasoning capability.

### Why Different Agents?

- **Some tasks require reasoning before action** (e.g., solving math problems).
- **Others can be handled by direct tool invocation** (e.g., checking weather, using a calculator).

### Key Decision Criteria

| Task Characteristic             | Recommended Agent            | Reason                                           |
|-------------------------------|------------------------------|--------------------------------------------------|
| Requires step-by-step thinking | `create_react_agent`         | Supports explicit reasoning and decision cycles |
| Simple, direct execution       | `create_tool_calling_agent`  | Uses structured tool invocation without reasoning |



### Advanced Strategy

You can **combine both agent types** to build a hybrid workflow:

> Use `create_react_agent` to decompose complex questions → Then delegate sub-tasks to `create_tool_calling_agent` for structured execution.


### Common Ground

- Both agents are **driven by LLMs** (e.g., OpenAI, Qwen).
- Both rely on **a predefined set of tools**.
- Both support structured, pluggable prompts.


### Under the Hood

- `create_tool_calling_agent` is essentially a **simplified version** of `create_react_agent`.
- It **bypasses explicit reasoning** and executes tools based on structured prompts.









<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



