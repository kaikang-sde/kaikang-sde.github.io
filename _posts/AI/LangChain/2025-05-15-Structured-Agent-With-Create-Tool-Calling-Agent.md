---
title: Structured Agent with create_tool_calling_agent
date: 2025-05-07
order: 11
categories: [AI, LangChain]
tags: [LangChain, LLM, AI Agent, create_tool_calling_agent]
author: kai
---

# 🚀 Structured Agent with create_tool_calling_agent
`create_tool_calling_agent` is a **new agent constructor** introduced in LangChain 0.3. It enables the LLM to **return structured tool invocation parameters directly**—typically in JSON format. This method significantly reduces parsing errors and enhances control for complex, multi-step tasks.

> 🧠 Think of it as a more **predictable, structured, and precise** version of ReAct agents.



## 🎯 Key Benefits

1. Structured Tool Invocation
- The LLM returns clearly defined parameters for tools, not just free-form text.
- Supports **complex argument types** (e.g., nested objects, arrays).
2. Multi-Step Task Handling
- Ideal for workflows where **several tools** must be called in a specific order.
3. High Precision Control
- Custom prompts can guide the agent’s **decision-making and tool usage rules** more accurately than generic prompt templates.


## 🧱 Method Signature

```python
from langchain.agents import create_tool_calling_agent

def create_tool_calling_agent(
    llm: BaseLanguageModel,
    tools: Sequence[BaseTool],
    prompt: ChatPromptTemplate,
    *,
    message_formatter: MessageFormatter = format_to_tool_messages,
) -> Runnable:
    """Create an agent that uses tools.

    Args:
        llm: LLM to use as the agent.
        tools: Tools this agent has access to.
        prompt: The prompt to use. See Prompt section below for more on the expected
            input variables.
        message_formatter: Formatter function to convert (AgentAction, tool output)
            tuples into FunctionMessages.

    Returns:
        A Runnable sequence representing an agent. It takes as input all the same input
        variables as the prompt passed in does. It returns as output either an
        AgentAction or AgentFinish.

    Example:

        .. code-block:: python

            from langchain.agents import AgentExecutor, create_tool_calling_agent, tool
            from langchain_anthropic import ChatAnthropic
            from langchain_core.prompts import ChatPromptTemplate

            prompt = ChatPromptTemplate.from_messages(
                [
                    ("system", "You are a helpful assistant"),
                    ("placeholder", "{chat_history}"),
                    ("human", "{input}"),
                    ("placeholder", "{agent_scratchpad}"),
                ]
            )
            model = ChatAnthropic(model="claude-3-opus-20240229")

            @tool
            def magic_function(input: int) -> int:
                \"\"\"Applies a magic function to an input.\"\"\"
                return input + 2

            tools = [magic_function]

            agent = create_tool_calling_agent(model, tools, prompt)
            agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

            agent_executor.invoke({"input": "what is the value of magic_function(3)?"})

            # Using with chat history
            from langchain_core.messages import AIMessage, HumanMessage
            agent_executor.invoke(
                {
                    "input": "what's my name?",
                    "chat_history": [
                        HumanMessage(content="hi! my name is bob"),
                        AIMessage(content="Hello Bob! How can I assist you today?"),
                    ],
                }
            )

    Prompt:

        The agent prompt must have an `agent_scratchpad` key that is a
            ``MessagesPlaceholder``. Intermediate agent actions and tool output
            messages will be passed in here.
    """
```

| Parameter | Type                    | Required | Description                                                                 |
|-----------|-------------------------|----------|-----------------------------------------------------------------------------|
| `llm`     | `BaseLanguageModel`     | Yes      | The language model that supports tool calling, such as `ChatOpenAI`.       |
| `tools`   | `Sequence[BaseTool]`    | Yes      | A list of tools; each tool must be defined using the `@tool` decorator.     |
| `prompt`  | `ChatPromptTemplate`    | Yes      | A prompt template that controls agent behavior. It must include `tools` and `tool_names` as placeholders. |


## ✅ Applicable Scenarios 

- **API Integration**  
  Ideal for use cases like weather queries, payment interfaces, or other external APIs where strict parameter formatting is required.

- **Multi-Tool Orchestration**  
  Suitable when the model needs to dynamically select and invoke multiple tools based on user input.

- **High-Precision Tasks**  
  Recommended for low-error-tolerance domains such as financial calculations or medical diagnostics, where structured and accurate execution is critical.


## 🧠 Example: LangChain Agent for Stock & Flight Tasks 

```python
import os
from langchain_openai import ChatOpenAI
from langchain.agents import create_tool_calling_agent, Tool, AgentExecutor
from langchain.tools import tool
from datetime import datetime
from langchain_core.prompts import ChatPromptTemplate

os.environ["OPENAI_API_KEY"] = "*"


@tool
def get_current_date() -> str:
    """Return today's date in YYYY-MM-DD format."""
    return f"The current date is {datetime.now().strftime('%Y-%m-%d')}"


@tool
def search_flights(from_city: str, to_city: str, date: str) -> str:
    """Search for flights given departure, arrival cities and date."""
    return f"Found flight: {from_city} -> {to_city} on {date}, price: ¥1200"


@tool
def book_flight(flight_id: str, user: str) -> str:
    """Book a specific flight for a user."""
    return f"User {user} successfully booked flight {flight_id}"


def get_stock_price(symbol) -> str:
    return f"The price of {symbol} is $100."


# Tool Registration
tools = [
    Tool(name="get_stock_price", func=get_stock_price,
         description="Get the price of a stock."),
    search_flights,
    book_flight,
    get_current_date
]

# Model and Prompt Setup
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are an personalized AI assistant. Call tools when necessary."),
    ("human", "My name is Kai. I travel a lot. My ID is 5454767654639876."),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}"), # must have
])

# Agent Construction & Execution
agent = create_tool_calling_agent(llm, tools, prompt)

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    return_intermediate_steps=True
)

result = agent_executor.invoke({
    "input": "What is Apple's stock price? Also check tomorrow's flight from PA to CA and book it."
})
print(f"Final Result: {result}")


# > Entering new AgentExecutor chain...
# Invoking: `get_stock_price` with `AAPL`
# The price of AAPL is $100.

# Invoking: `get_current_date` with `{}`
# The current date is 2025-05-06

# Invoking: `search_flights` with `{'from_city': 'PA', 'to_city': 'CA', 'date': '2025-05-07'}`
# Found flight: PA -> CA on 2025-05-07, price: ¥1200

# Invoking: `book_flight` with `{'flight_id': 'PA-CA-2025-05-07', 'user': '5454767654639876'}`
# User 5454767654639876 successfully booked flight PA-CA-2025-05-07

# - The current price of Apple's stock (AAPL) is $100.
# - Your flight from PA to CA on May 7, 2025, has been successfully booked!

# > Finished chain.

# Final Result: 
# {'input': "What is Apple's stock price? Also check tomorrow's flight from PA to CA and book it.", 
# 'output': "- The current price of Apple's stock (AAPL) is $100.\n- Your flight from PA to CA on May 7, 2025, has been successfully booked!", 
# 'intermediate_steps': [(ToolAgentAction(tool='get_stock_price', tool_input='AAPL', log='\nInvoking: `get_stock_price` with `AAPL`\n\n\n', 
# message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [
# {'index': 0, 'id': 'call_Uybe6j5mcpMsY44spDePc3gg', 'function': {'arguments': '{"__arg1": "AAPL"}', 'name': 'get_stock_price'}, 'type': 'function'}, 
# {'index': 1, 'id': 'call_iCa62iLZZejU6XIBOWp21Ino', 'function': {'arguments': '{}', 'name': 'get_current_date'}, 'type': 'function'}]}, 
# response_metadata={'finish_reason': 'tool_calls', 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_0392822090'}, id='run-0b7268c0-338d-42d5-9eaa-9823a9004d22', 
# tool_calls=[{'name': 'get_stock_price', 'args': {'__arg1': 'AAPL'}, 'id': 'call_Uybe6j5mcpMsY44spDePc3gg', 'type': 'tool_call'}, {'name': 'get_current_date', 'args': {}, 'id': 'call_iCa62iLZZejU6XIBOWp21Ino', 'type': 'tool_call'}], 
# tool_call_chunks=[{'name': 'get_stock_price', 'args': '{"__arg1": "AAPL"}', 'id': 'call_Uybe6j5mcpMsY44spDePc3gg', 'index': 0, 'type': 'tool_call_chunk'}, {'name': 'get_current_date', 'args': '{}', 'id': 'call_iCa62iLZZejU6XIBOWp21Ino', 'index': 1, 'type': 'tool_call_chunk'}])], tool_call_id='call_Uybe6j5mcpMsY44spDePc3gg'), 'The price of AAPL is $100.'), (ToolAgentAction(tool='get_current_date', tool_input={}, log='\nInvoking: `get_current_date` with `{}`\n\n\n', 
# message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_Uybe6j5mcpMsY44spDePc3gg', 'function': {'arguments': '{"__arg1": "AAPL"}', 'name': 'get_stock_price'}, 'type': 'function'}, {'index': 1, 'id': 'call_iCa62iLZZejU6XIBOWp21Ino', 'function': {'arguments': '{}', 'name': 'get_current_date'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls', 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_0392822090'}, id='run-0b7268c0-338d-42d5-9eaa-9823a9004d22', 
# tool_calls=[{'name': 'get_stock_price', 'args': {'__arg1': 'AAPL'}, 'id': 'call_Uybe6j5mcpMsY44spDePc3gg', 'type': 'tool_call'}, {'name': 'get_current_date', 'args': {}, 'id': 'call_iCa62iLZZejU6XIBOWp21Ino', 'type': 'tool_call'}], tool_call_chunks=[{'name': 'get_stock_price', 'args': '{"__arg1": "AAPL"}', 'id': 'call_Uybe6j5mcpMsY44spDePc3gg', 'index': 0, 'type': 'tool_call_chunk'}, {'name': 'get_current_date', 'args': '{}', 'id': 'call_iCa62iLZZejU6XIBOWp21Ino', 'index': 1, 'type': 'tool_call_chunk'}])], 
# tool_call_id='call_iCa62iLZZejU6XIBOWp21Ino'), 'The current date is 2025-05-06'), (ToolAgentAction(tool='search_flights', tool_input={'from_city': 'PA', 'to_city': 'CA', 'date': '2025-05-07'}, log="\nInvoking: `search_flights` with `{'from_city': 'PA', 'to_city': 'CA', 'date': '2025-05-07'}`\n\n\n", message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_DqDPnSSth9AVKX7Kh8KhIQ9S', 'function': {'arguments': '{"from_city":"PA","to_city":"CA","date":"2025-05-07"}', 'name': 'search_flights'}, 'type': 'function'}]}, 
# response_metadata={'finish_reason': 'tool_calls', 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_129a36352a'}, id='run-3021a055-2c5e-4ea2-8a72-db5811056eec', tool_calls=[{'name': 'search_flights', 'args': {'from_city': 'PA', 'to_city': 'CA', 'date': '2025-05-07'}, 'id': 'call_DqDPnSSth9AVKX7Kh8KhIQ9S', 'type': 'tool_call'}], 
# tool_call_chunks=[{'name': 'search_flights', 'args': '{"from_city":"PA","to_city":"CA","date":"2025-05-07"}', 'id': 'call_DqDPnSSth9AVKX7Kh8KhIQ9S', 'index': 0, 'type': 'tool_call_chunk'}])], tool_call_id='call_DqDPnSSth9AVKX7Kh8KhIQ9S'), 'Found flight: PA -> CA on 2025-05-07, price: ¥1200'), (ToolAgentAction(tool='book_flight', tool_input={'flight_id': 'PA-CA-2025-05-07', 'user': '5454767654639876'}, log="\nInvoking: `book_flight` with `{'flight_id': 'PA-CA-2025-05-07', 'user': '5454767654639876'}`\n\n\n", message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_k4TGNkUH6RAJO8hTGwLV1gPd', 'function': {'arguments': '{"flight_id":"PA-CA-2025-05-07","user":"5454767654639876"}', 'name': 'book_flight'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls', 'model_name': 'gpt-4o-mini-2024-07-18', 'system_fingerprint': 'fp_dbaca60df0'}, id='run-f66dc817-c4df-49cb-818e-6d8c1ae24c94', tool_calls=[{'name': 'book_flight', 'args': {'flight_id': 'PA-CA-2025-05-07', 'user': '5454767654639876'}, 'id': 'call_k4TGNkUH6RAJO8hTGwLV1gPd', 'type': 'tool_call'}], 
# tool_call_chunks=[{'name': 'book_flight', 'args': '{"flight_id":"PA-CA-2025-05-07","user":"5454767654639876"}', 'id': 'call_k4TGNkUH6RAJO8hTGwLV1gPd', 'index': 0, 'type': 'tool_call_chunk'}])], tool_call_id='call_k4TGNkUH6RAJO8hTGwLV1gPd'), 'User 5454767654639876 successfully booked flight PA-CA-2025-05-07')]}
```



<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



