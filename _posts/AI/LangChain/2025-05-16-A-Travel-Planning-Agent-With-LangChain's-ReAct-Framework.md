---
title: A Travel Planning Agent with LangChain's ReAct Framework
date: 2025-05-06
order: 12
categories: [AI, LangChain]
tags: [LangChain, LLM, AI Agent, ReAct]
author: kai
---

# 🚀 ReAct Agent for Travel Planning with LangChain

Link: [CoT and ReAct in LLM Agents]({{ site.baseurl }}/CoT-And-ReAct-In-LLM-Agents/)

**ReAct** stands for **Reasoning + Acting**. It is a hybrid intelligent agent framework that allows Large Language Models (LLMs) to:

- **Reason**: Think step-by-step like a human
- **Act**: Interact with external tools (e.g., search, calendar, map APIs)
- **Observe**: Adjust its plan dynamically based on observations

This method improves decision accuracy for multi-step or dynamic tasks such as **travel itinerary generation**, **real-time data lookup**, or **task chaining**.


## 🔁 ReAct Execution Loop

ReAct agents operate in a loop, repeatedly going through the following three stages:

```text
[Thought]      ← Reasoning: analyze the current situation and plan next step
[Action]       ← Acting: choose a tool or output a response
[Observation]  ← Observing: receive tool feedback and update context
```


## 🧠 create_react_agent
LangChain offers a dedicated method called `create_react_agent` to help developers quickly build **multi-step reasoning agents** based on the ReAct framework.

This method is ideal for tasks that require **dynamic decision-making**, such as complex question answering, multi-hop reasoning, and multi-tool coordination.


### Syntax and Parameters

```python
from langchain.agents import create_react_agent

def create_react_agent(
    llm: BaseLanguageModel,
    tools: Sequence[BaseTool],
    prompt: BasePromptTemplate,
    output_parser: Optional[AgentOutputParser] = None,
    tools_renderer: ToolsRenderer = render_text_description,
    *,
    stop_sequence: Union[bool, List[str]] = True,
) -> Runnable:
    """Create an agent that uses ReAct prompting.

    Based on paper "ReAct: Synergizing Reasoning and Acting in Language Models"
    (https://arxiv.org/abs/2210.03629)

    .. warning::
       This implementation is based on the foundational ReAct paper but is older and not well-suited for production applications.
       For a more robust and feature-rich implementation, we recommend using the `create_react_agent` function from the LangGraph library.
       See the [reference doc](https://langchain-ai.github.io/langgraph/reference/prebuilt/#langgraph.prebuilt.chat_agent_executor.create_react_agent)
       for more information.

    Args:
        llm: LLM to use as the agent.
        tools: Tools this agent has access to.
        prompt: The prompt to use. See Prompt section below for more.
        output_parser: AgentOutputParser for parse the LLM output.
        tools_renderer: This controls how the tools are converted into a string and
            then passed into the LLM. Default is `render_text_description`.
        stop_sequence: bool or list of str.
            If True, adds a stop token of "Observation:" to avoid hallucinates.
            If False, does not add a stop token.
            If a list of str, uses the provided list as the stop tokens.

            Default is True. You may to set this to False if the LLM you are using
            does not support stop sequences.

    Returns:
        A Runnable sequence representing an agent. It takes as input all the same input
        variables as the prompt passed in does. It returns as output either an
        AgentAction or AgentFinish.

    Examples:

        .. code-block:: python

            from langchain import hub
            from langchain_community.llms import OpenAI
            from langchain.agents import AgentExecutor, create_react_agent

            prompt = hub.pull("hwchase17/react")
            model = OpenAI()
            tools = ...

            agent = create_react_agent(model, tools, prompt)
            agent_executor = AgentExecutor(agent=agent, tools=tools)

            agent_executor.invoke({"input": "hi"})

            # Use with chat history
            from langchain_core.messages import AIMessage, HumanMessage
            agent_executor.invoke(
                {
                    "input": "what's my name?",
                    # Notice that chat_history is a string
                    # since this prompt is aimed at LLMs, not chat models
                    "chat_history": "Human: My name is Bob\\nAI: Hello Bob!",
                }
            )

    Prompt:

        The prompt must have input keys:
            * `tools`: contains descriptions and arguments for each tool.
            * `tool_names`: contains all tool names.
            * `agent_scratchpad`: contains previous agent actions and tool outputs as a string.

        Here's an example:

        .. code-block:: python

            from langchain_core.prompts import PromptTemplate

            template = '''Answer the following questions as best you can. You have access to the following tools:

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
            Thought:{agent_scratchpad}'''

            prompt = PromptTemplate.from_template(template)
    """
```

### ReAct Prompt Template (core format)
The ReAct agent relies on a structured prompting style to guide its behavior. 

```text
Answer the following questions using the following tools:
{tools}
Use the following format:

Question: the input question
Thought: reasoning step
Action: the action to take, must be one of [{tool_names}]
Action Input: input to the selected tool
Observation: result from the tool
... (repeat Thought/Action/Observation as needed)
Final Answer: the final answer to the question
```

### Recommended Usage (via LangChain Hub)
You can load a ready-to-use ReAct prompt directly from LangChain Hub:

```python
from langchain import hub
prompt = hub.pull("hwchase17/react")
```

This prebuilt prompt ensures:
- Clear instruction-following
- Support for tool loop cycles
- Alignment with LangChain’s built-in tool-calling mechanisms


## 🔍 Agent Method Comparison

LangChain provides multiple agent creation methods. Each has its strengths depending on your application scenario. Here's a practical comparison to help you decide:

| **Scenario Characteristic**       | **Recommended Method**           | **Why**                                                                 |
|----------------------------------|----------------------------------|-------------------------------------------------------------------------|
| Step-by-step reasoning needed    | `create_react_agent`             | Supports explicit thought chains for complex logic and decisions       |
| Requires strict parameter format | `create_tool_calling_agent`      | Structured input handling ensures safer and more accurate tool usage   |
| Fast and simple prototype        | `initialize_agent`               | Minimal setup; great for quick-start scenarios                         |
| Needs self-correction capability | `create_react_agent`             | Can re-evaluate decisions after tool feedback                          |
| Human-in-the-loop collaboration  | `create_react_agent`             | Full reasoning trace improves transparency and debugging               |

> ✅ **Tip:** Use `initialize_agent` for quick demos, but migrate to `create_react_agent` or `create_tool_calling_agent` for production-ready logic.


## Example: Smart Recommendation Assistant

```python
import os
from langchain_openai import ChatOpenAI
from langchain.tools import tool
from langchain_community.utilities import SearchApiAPIWrapper
from langchain_core.prompts import PromptTemplate
from langchain.agents import create_react_agent, AgentExecutor


os.environ["SEARCHAPI_API_KEY"] = "*"
os.environ["OPENAI_API_KEY"] = "*"


@tool
def get_weather(city: str) -> str:
    """Get the weather for a given city"""
    weather_data = {
        "New York": "Sunny, 25℃",
        "Pennsylvania": "Rainy, 20℃",
        "Virginia": "Cloudy, 28℃"
    }
    return weather_data.get(city, "City not supported.")


@tool
def recommend_activity(weather: str) -> str:
    """Recommend an activity based on the weather"""
    if "Rainy" in weather:
        return "Recommended indoor activity: Visit a museum."
    elif "Sunny" in weather:
        return "Recommended outdoor activity: Go biking in the park."
    else:
        return "Recommended general activity: Explore the city."


@tool("web_search", return_direct=True)
def web_search(query: str) -> str:
    """Search online for real-time or unknown information"""
    try:
        search = SearchApiAPIWrapper()
        results = search.results(query)
        return "\n\n".join([
            f"Source: {res['title']}\nSnippet: {res['snippet']}"
            for res in results['organic_results']
        ])
    except Exception as e:
        return f"Search failed: {str(e)}"


template = """
Answer the following questions as best you can. You have access to the following tools:
{tools}
Use the following format:
Question: the input question you must answer
Thought: you should always think about what to do
Action: the action to take, should be one of [{tool_names}]
Action Input: the input to the action
Observation: the result of the action
... (this Thought/Action/Observation can repeat N times)
Thought: I now know the final answer
Final Answer: the final answer to the original input question

Begin!

Question: {input}
Thought:{agent_scratchpad}
"""

prompt = PromptTemplate.from_template(template)

tools = [get_weather, recommend_activity, web_search]

llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

agent = create_react_agent(llm, tools, prompt)

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=3,
    handle_parsing_errors=lambda _: "Please check the input format.",
    return_intermediate_steps=True
)

response = agent_executor.invoke({
    "input": "I'm staying in New York for 3 days. Recommend me some activities based on the weather. Also, what's the stock price of APPLE?"
})

print(response)


# > Entering new AgentExecutor chain...
# I need to find out the weather in New York to recommend activities accordingly. Additionally, I need to look up the current stock price of Apple. 

# Action: get_weather  
# Action Input: "New York"  Sunny, 25℃The weather in New York is sunny and 25°C. Now, I will recommend activities based on this pleasant weather. 

# Action: recommend_activity  
# Action Input: "Sunny, 25°C"  Recommended outdoor activity: Go biking in the park.I have a recommendation for an outdoor activity in New York based on the weather. Now, I need to search for the current stock price of Apple.

# Action: web_search  
# Action Input: "current stock price of Apple"  Source: Stock Price
# Snippet: Stock Quote: NASDAQ: AAPL ; Day's Open203.10 ; Closing Price198.89 ; Volume69.0 ; Intraday High204.10 ; Intraday Low198.21.

# Source: Apple Inc AAPL:NASDAQ - Stock Price, Quote and News
# Snippet: Apple Inc AAPL:NASDAQ ; Last | 9:30 AM EDT. 197.91 quote price arrow down -0.98 (-0.49%) ; Volume. 33,153 ; 52 week range. 169.21 - 260.10.

# Source: Apple Inc. (AAPL) Stock Price, News, Quote & History
# Snippet: 154,861.52% · Previous Close 205.35 · Open 203.12 · Bid 188.98 x 100 · Ask 209.32 x 100 · Day's Range 198.21 - 204.10 · 52 Week Range 169.21 - 260.10 · Volume ...

# Source: Apple Inc. Stock Quote (U.S.: Nasdaq) - AAPL
# Snippet: 197.76 ; Volume: 69.02M · 65 Day Avg: 60.21M ; 198.21 Day Range 204.10 ; 169.21 52 Week Range 260.10 ...

# Source: AAPL Stock Quote Price and Forecast
# Snippet: Price Momentum. AAPL is trading near the bottom ; Price change. The price of AAPL shares has decreased $6.46 ; Closed at $198.89. The stock has since dropped ...

# Source: NASDAQ:AAPL Stock Price - Apple Inc
# Snippet: The current price of AAPL is 198.89 USD — it has decreased by −3.08% in the past 24 hours. Watch Apple Inc stock price performance more closely on the chart. ...

# Source: Apple Inc. (AAPL) Stock Price Today
# Snippet: Apple Inc. · REAL TIME 9:33 AM EDT 05/06/25 · 198.97USD · 0.080.04% · Volume4,518,904.

# Source: Apple Stock Price | AAPL Stock Quote, News, and History
# Snippet: On June 21, with Apple's stock price at $101.25, Apple issued two shares to investors at $55.62. Five years later, with Apple stock price at an ever-higher ...


# > Finished chain.
# {'input': "I'm staying in New York for 3 days. Recommend me some activities based on the weather. Also, what's the stock price of APPLE?", 
# 'output': "Source: Stock Price\nSnippet: Stock Quote: NASDAQ: AAPL ; Day's Open203.10 ; Closing Price198.89 ; Volume69.0 ; Intraday High204.10 ; Intraday Low198.21.\n\nSource: Apple Inc AAPL:NASDAQ - Stock Price, Quote and News\nSnippet: Apple Inc AAPL:NASDAQ ; Last | 9:30 AM EDT. 197.91 quote price arrow down -0.98 (-0.49%) ; Volume. 33,153 ; 52 week range. 169.21 - 260.10.\n\nSource: Apple Inc. (AAPL) Stock Price, News, Quote & History\nSnippet: 154,861.52% · Previous Close 205.35 · Open 203.12 · Bid 188.98 x 100 · Ask 209.32 x 100 · Day's Range 198.21 - 204.10 · 52 Week Range 169.21 - 260.10 · Volume ...\n\nSource: Apple Inc. Stock Quote (U.S.: Nasdaq) - AAPL\nSnippet: 197.76 ; Volume: 69.02M · 65 Day Avg: 60.21M ; 198.21 Day Range 204.10 ; 169.21 52 Week Range 260.10 ...\n\nSource: AAPL Stock Quote Price and Forecast\nSnippet: Price Momentum. AAPL is trading near the bottom ; Price change. The price of AAPL shares has decreased $6.46 ; Closed at $198.89. The stock has since dropped ...\n\nSource: NASDAQ:AAPL Stock Price - Apple Inc\nSnippet: The current price of AAPL is 198.89 USD — it has decreased by −3.08% in the past 24 hours. Watch Apple Inc stock price performance more closely on the chart. ...\n\nSource: Apple Inc. (AAPL) Stock Price Today\nSnippet: Apple Inc. · REAL TIME 9:33 AM EDT 05/06/25 · 198.97USD · 0.080.04% · Volume4,518,904.\n\nSource: Apple Stock Price | AAPL Stock Quote, News, and History\nSnippet: On June 21, with Apple's stock price at $101.25, Apple issued two shares to investors at $55.62. Five years later, with Apple stock price at an ever-higher ...", 
# 'intermediate_steps': [
# (AgentAction(tool='get_weather', tool_input='New York', log='I need to find out the weather in New York to recommend activities accordingly. Additionally, I need to look up the current stock price of Apple. 
# \n\nAction: get_weather  \nAction Input: "New York"  '), 'Sunny, 25℃'), 
# (AgentAction(tool='recommend_activity', tool_input='Sunny, 25°C', log='The weather in New York is sunny and 25°C. Now, I will recommend activities based on this pleasant weather. 
# \n\nAction: recommend_activity  \nAction Input: "Sunny, 25°C"  '), 'Recommended outdoor activity: Go biking in the park.'), 
# (AgentAction(tool='web_search', tool_input='current stock price of Apple', log='I have a recommendation for an outdoor activity in New York based on the weather. Now, I need to search for the current stock price of Apple.
# \n\nAction: web_search  \nAction Input: "current stock price of Apple"  '), "Source: Stock Price\nSnippet: Stock Quote: NASDAQ: AAPL ; Day's Open203.10 ; Closing Price198.89 ; Volume69.0 ; Intraday High204.10 ; Intraday Low198.21.\n\nSource: Apple Inc AAPL:NASDAQ - Stock Price, Quote and News\nSnippet: Apple Inc AAPL:NASDAQ ; Last | 9:30 AM EDT. 197.91 quote price arrow down -0.98 (-0.49%) ; Volume. 33,153 ; 52 week range. 169.21 - 260.10.\n\nSource: Apple Inc. (AAPL) Stock Price, News, Quote & History\nSnippet: 154,861.52% · Previous Close 205.35 · Open 203.12 · Bid 188.98 x 100 · Ask 209.32 x 100 · Day's Range 198.21 - 204.10 · 52 Week Range 169.21 - 260.10 · Volume ...\n\nSource: Apple Inc. Stock Quote (U.S.: Nasdaq) - AAPL\nSnippet: 197.76 ; Volume: 69.02M · 65 Day Avg: 60.21M ; 198.21 Day Range 204.10 ; 169.21 52 Week Range 260.10 ...\n\nSource: AAPL Stock Quote Price and Forecast\nSnippet: Price Momentum. AAPL is trading near the bottom ; Price change. The price of AAPL shares has decreased $6.46 ; Closed at $198.89. The stock has since dropped ...\n\nSource: NASDAQ:AAPL Stock Price - Apple Inc\nSnippet: The current price of AAPL is 198.89 USD — it has decreased by −3.08% in the past 24 hours. Watch Apple Inc stock price performance more closely on the chart. ...\n\nSource: Apple Inc. (AAPL) Stock Price Today\nSnippet: Apple Inc. · REAL TIME 9:33 AM EDT 05/06/25 · 198.97USD · 0.080.04% · Volume4,518,904.\n\nSource: Apple Stock Price | AAPL Stock Quote, News, and History\nSnippet: On June 21, with Apple's stock price at $101.25, Apple issued two shares to investors at $55.62. Five years later, with Apple stock price at an ever-higher ...")]}
```











<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



