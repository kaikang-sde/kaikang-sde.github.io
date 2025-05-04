---
title: LangChain Built-in Toolkits and Web Search Integration
date: 2025-05-04
order: 4
categories: [AI, LangChain]
tags: [LangChain, Tools, Web Search, AI Agent, LLM]
author: kai
---

# 🚀 LangChain Built-in Toolkits and Web Search Integration
LangChain offers a wide array of **built-in tools** to help developers quickly enhance LLM agents with real-world capabilities — like searching the web, retrieving documents, executing code, or querying APIs — all without needing to start from scratch.


## 🧰 LangChain Toolkits Overview
LangChain’s tool system is designed for maximum flexibility and ease of use:

- **Out-of-the-box support** for mainstream functionalities (e.g., search, code execution, shell access)
- **Standardized API** — all tools are subclasses of `BaseTool`
- **Runnable Interface** — tools are fully compatible with LangChain’s `Runnable` system (support `.invoke()`, `.stream()`, `.batch()`)
- **Self-describing metadata** to help both developers and LLMs understand tool usage (name, description, args, return_direct)

> LangChain is fully extensible — if no built-in tool meets your requirements, you can easily define your own.

🔗 [LangChain Tool Integrations Documentation](https://python.langchain.com/docs/integrations/tools/)  



## 🌐 Web Search Example - SearchAPI
🔗 [SearchApi](https://python.langchain.com/docs/integrations/tools/searchapi/)  


```python
from langchain_community.utilities import SearchApiAPIWrapper
import os

os.environ["SEARCHAPI_API_KEY"] = "*"

# Step 1: Instantiate the search tool
search = SearchApiAPIWrapper()

# Step 2: Run a real-time web query
result = search.run("What is Amazon's stock price today?")  

print(result)

# Market summary: Amazon.com Inc
# Ticker: AMZN
# Exchange: NASDAQ
# Currency: USD
# Current price: 187.39
# Date: Apr 30, 8:08 AM EDT
# Previous close: 187.7
# Price change: -0.31 (0.17%)
# Market status: Pre-market price 185.45
# Market details: Open: 183.99, High: 188.02, Low: 183.68, Mkt cap: 1.99T, P/E ratio: 33.91, Div yield: -, 52-wk high: 242.52, 52-wk low: 151.61
```

## 🔄 Advanced Usage: Customize the SearchApi Wrapper with Different Engines
The `SearchApiAPIWrapper` in LangChain is **highly customizable**.  You’re not limited to just the default search engine — you can switch to **Google News**, **Google Jobs**, **Google Scholar**, and more, simply by specifying the engine type.

`search = SearchApiAPIWrapper(engine="google_jobs")`

This flexibility makes it ideal for building intelligent agents that need to query **specific domains** of the internet.


| Engine         | Use Case Example                           |
|----------------|---------------------------------------------|
| `google`       | General search                              |
| `google_news`  | Real-time news monitoring                   |
| `google_jobs`  | Job search and listings                     |
| `google_scholar` | Academic papers and citations              |
| `bing`         | Microsoft Bing search                       |
| `yahoo`        | Yahoo-powered queries                       |
| ...            | See full list at [SearchAPI.io Docs](https://www.searchapi.io/docs) |





<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



