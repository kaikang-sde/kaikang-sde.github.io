---
title: LLM Accessing MySQL - SQL Agent
date: 2025-06-04
categories: [AI, LangChain]
tags: [LangChain, LLM, MySQL, SQL Agent, Natural-Language-Query]
author: kai
---

# 🚀 LLM Accessing MySQL - SQL Agent with LangChain

In future-facing business environments, users—including **product managers**, **analysts**, and **sales teams**—increasingly prefer **natural language interactions** over writing raw SQL.

For example:

> Instead of writing  
> `SELECT product_name FROM sales ORDER BY revenue DESC LIMIT 1;`  
> Users simply ask:  
> **"What is the top-selling product?"**

This trend leads to a clear need: **bridge the gap between LLMs and business databases**.



## 🔍 Use Case: LLM Accessing MySQL

LangChain enables the creation of AI agents that can:

- Interpret user questions in **natural language**
- **Generate SQL queries** automatically
- **Execute** them against MySQL databases
- **Translate results** into readable, business-friendly language

This functionality makes **non-technical users empowered** to retrieve business insights without knowing SQL.


## ⚙️ What Is `create_sql_agent`?

LangChain’s `create_sql_agent` is a utility that builds an AI **agent** capable of interacting with an SQL database.

### Core Capabilities

| Capability             | Description |
|------------------------|-------------|
| **Natural Language → SQL** | Converts prompts like “Get sales by region” into proper SQL queries |
| **Secure Execution**       | Executes in **read-only mode** to avoid unintended data changes |
| **Result Parsing**         | Converts raw database rows into human-friendly answers |
| **Error Handling & Retry** | Automatically detects and fixes syntax/logic errors (e.g., column name typos) |


- **LLM** handles query generation and explanation.
- **LangChain** wraps DB connection, schema awareness, and tool usage.
- **MySQL** serves as the backend business data source.


### Architecture Overview

```text
User (NL question)
↓
SQL Agent (via LangChain + LLM)
↓
SQL Generation → MySQL Execution
↓
Result Parsing & Natural Language Output
```


### Key Features for Production Safety

- **Read-only connections** by default: Prevents data corruption
- **Controlled query templates**: Optional whitelisting of query types
- **Schema introspection**: Helps LLM avoid hallucinations or invalid field references
- **Prompt engineering**: Guides the model toward safe, accurate, and meaningful SQL



## 📌 Example Prompt Flow

> **User:** “What was the total sales last month in the North region?”  
> → LLM generates:  
> `SELECT SUM(sales) FROM orders WHERE region = 'North' AND date >= '2024-05-01';`  
> → Result: `[$35000]`  
> → Response:  
> **“Total sales in the North region last month were $35,000.”**


## 🤖 Business Impact

- **Empower non-technical users** to self-serve insights
- **Accelerate decision-making** with on-demand querying
- **Reduce engineering workload** on BI/SQL support tasks
- **Enable conversational BI assistants** in dashboards or internal tools


## 💡 Real-World Applications of SQL Agent

### 1. Conversational Data Access for Non-Technical Users

Enable **product managers**, **sales analysts**, and **executives** to query business data by simply asking questions:

> "What’s the revenue trend over the last 6 months?"  
> "How many new users signed up this week?"

- No SQL knowledge needed  
- Instant, natural-language insights

### 2. Automated Data Analysis & Report Generation

Use LLMs to automate:

- Monthly/quarterly sales reports
- Regional breakdowns and comparisons
- Inventory forecasting

LLMs can **generate, execute, and summarize** complex queries across multiple tables, all through natural instructions.


### 3. Business Q&A Bots with Backend Database Access

Integrate SQL Agents into internal **chatbots** to answer real-time business questions:

> **User:** “Which product had the highest return rate last quarter?”  
> **Bot:** “Product A had a return rate of 12.3%.”

This brings the power of conversational AI directly into CRM, ERP, or BI dashboards.


## ⚙️ Core Syntax: How to Build an SQL Agent with LangChain

LangChain provides a simple interface to create SQL-capable agents using your LLM and MySQL database.

### Required Import

```python
from langchain_community.agent_toolkits.sql.base import create_sql_agent

def create_sql_agent(
    llm: BaseLanguageModel,
    toolkit: Optional[SQLDatabaseToolkit] = None,
    agent_type: Optional[
        Union[AgentType, Literal["openai-tools", "tool-calling"]]
    ] = None,
    callback_manager: Optional[BaseCallbackManager] = None,
    prefix: Optional[str] = None,
    suffix: Optional[str] = None,
    format_instructions: Optional[str] = None,
    input_variables: Optional[List[str]] = None,
    top_k: int = 10,
    max_iterations: Optional[int] = 15,
    max_execution_time: Optional[float] = None,
    early_stopping_method: str = "force",
    verbose: bool = False,
    agent_executor_kwargs: Optional[Dict[str, Any]] = None,
    extra_tools: Sequence[BaseTool] = (),
    *,
    db: Optional[SQLDatabase] = None,
    prompt: Optional[BasePromptTemplate] = None,
    **kwargs: Any,
) -> AgentExecutor:
    """Construct a SQL agent from an LLM and toolkit or database.

    Args:
        llm: Language model to use for the agent. If agent_type is "tool-calling" then
            llm is expected to support tool calling.
        toolkit: SQLDatabaseToolkit for the agent to use. Must provide exactly one of
            'toolkit' or 'db'. Specify 'toolkit' if you want to use a different model
            for the agent and the toolkit.
        agent_type: One of "tool-calling", "openai-tools", "openai-functions", or
            "zero-shot-react-description". Defaults to "zero-shot-react-description".
            "tool-calling" is recommended over the legacy "openai-tools" and
            "openai-functions" types.
        callback_manager: DEPRECATED. Pass "callbacks" key into 'agent_executor_kwargs'
            instead to pass constructor callbacks to AgentExecutor.
        prefix: Prompt prefix string. Must contain variables "top_k" and "dialect".
        suffix: Prompt suffix string. Default depends on agent type.
        format_instructions: Formatting instructions to pass to
            ZeroShotAgent.create_prompt() when 'agent_type' is
            "zero-shot-react-description". Otherwise ignored.
        input_variables: DEPRECATED.
        top_k: Number of rows to query for by default.
        max_iterations: Passed to AgentExecutor init.
        max_execution_time: Passed to AgentExecutor init.
        early_stopping_method: Passed to AgentExecutor init.
        verbose: AgentExecutor verbosity.
        agent_executor_kwargs: Arbitrary additional AgentExecutor args.
        extra_tools: Additional tools to give to agent on top of the ones that come with
            SQLDatabaseToolkit.
        db: SQLDatabase from which to create a SQLDatabaseToolkit. Toolkit is created
            using 'db' and 'llm'. Must provide exactly one of 'db' or 'toolkit'.
        prompt: Complete agent prompt. prompt and {prefix, suffix, format_instructions,
            input_variables} are mutually exclusive.
        **kwargs: Arbitrary additional Agent args.

    Returns:
        An AgentExecutor with the specified agent_type agent.

    Example:

        .. code-block:: python

            from langchain_openai import ChatOpenAI
            from langchain_community.agent_toolkits import create_sql_agent
            from langchain_community.utilities import SQLDatabase

            db = SQLDatabase.from_uri("sqlite:///Chinook.db")
            llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
            agent_executor = create_sql_agent(llm, db=db, agent_type="tool-calling", verbose=True)

    """
```

### Example

```python
agent = create_sql_agent(
    llm=ChatOpenAI(temperature=0, model="gpt-5"),                  # Required: LLM of choice
    toolkit=SQLDatabaseToolkit(db=db, llm=llm),                    # Required: SQL database toolkit
    agent_type=AgentType.ZERO_SHOT_REACT_DESCRIPTION,              # Recommended agent type
    verbose=True,                                                  # Optional: Show execution trace
    prefix="""You are a MySQL expert...""",                        # Optional: Custom prompt prefix
    suffix="""Please always check your query results..."""         # Optional: Custom prompt suffix
)
```

## 👀 What Is `SQLDatabaseToolkit`?
In LangChain, `SQLDatabaseToolkit` is a **dedicated utility for integrating SQL databases** with large language models. It acts as a **tool container**—bundling your database connection along with a suite of built-in SQL tools to support querying, schema introspection, and result handling.

> Think of it as the *bridge* that enables an LLM agent to reason about and interact with a live SQL database.

![SQLDatabaseToolkit](/assets/img/posts/AI/LangChain/SQLDatabaseToolkit.png)


### Under the Hood: `SQLDatabase`

Before using the toolkit, you need to establish a database connection using `SQLDatabase`:

#### Class Overview

```python
class SQLDatabase:
    def __init__(
        engine: Engine,
        schema: Optional[str] = None,
        metadata: Optional[MetaData] = None,
        ignore_tables: Optional[List[str]] = None,
        include_tables: Optional[List[str]] = None,
        sample_rows_in_table_info: int = 3,
        indexes_in_table_info: bool = False,
        custom_table_info: Optional[dict] = None,
        view_support: bool = False,
        max_string_length: int = 300,
        lazy_table_reflection: bool = False,
    )
```

#### from_uri
```python
@classmethod
def from_uri(
    cls,
    database_uri: Union[str, URL],
    engine_args: Optional[dict] = None,
    **kwargs: Any,
) -> SQLDatabase:
    """Construct a SQLAlchemy engine from URI."""
    _engine_args = engine_args or {}
    return cls(create_engine(database_uri, **_engine_args), **kwargs)
```


#### Example
```python
from langchain_community.utilities.sql_database import SQLDatabase

db = SQLDatabase.from_uri(
    database_uri="mysql+pymysql://user:password@host:port/dbname",  
    include_tables=["account_file", "storage"],                     # Optional: limit scope
    custom_table_info={"account": "Query account related tables"}   # Optional: manual table descriptions
)
```


### Creating the Toolkit

```python
from langchain_community.agent_toolkits.sql.toolkit import SQLDatabaseToolkit
from langchain.chat_models import ChatOpenAI

toolkit = SQLDatabaseToolkit(
    db=db,
    llm=ChatOpenAI(temperature=0, model="gpt-4")
)
```


## 🧪 Case Study: Building a SQL Agent with LangChain (Pseudocode Demo)

```python
from langchain_community.utilities.sql_database import SQLDatabase
from langchain.chat_models import ChatOpenAI
from langchain_community.agent_toolkits.sql.toolkit import SQLDatabaseToolkit
from langchain_community.agent_toolkits.sql.base import create_sql_agent
from langchain.agents import AgentType

# Step 1: Connect to Your MySQL Database
db = SQLDatabase.from_uri(
    "mysql+pymysql://root:xdclass.net168@39.108.115.128:3306/dcloud_aipan"
)

# Step 2: Initialize Your LLM
llm = ChatOpenAI(
    temperature=0,
    model="gpt-4",
    openai_api_key="sk-your-key"
)

# Step 3: Bundle the Toolkit
toolkit = SQLDatabaseToolkit(db=db, llm=llm)

# Step 4: Create the SQL Agent
agent = create_sql_agent(
    llm=llm,
    toolkit=toolkit,
    verbose=True,  # Show reasoning steps and SQL
    agent_type=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    handle_parsing_errors=True  # Auto-handle SQL/response parsing issues
)

# Step 5: Query with Natural Language
questions = [
    "How many customers we have?",
    "The top 5 products and their total sales amount",
]

for question in questions:
    print(f"\nQuestion:{question}")
    response = agent.invoke({"input": question})
    print(f"Answer:{response['output']}")
```

















<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



