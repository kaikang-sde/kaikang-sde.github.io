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
from langchain_community.utilities.sql_database import SQLDatabase
from langchain_openai import ChatOpenAI
from langchain_community.agent_toolkits.sql.toolkit import SQLDatabaseToolkit
from langchain_community.agent_toolkits.sql.base import create_sql_agent
from langchain.agents import AgentType

# Step 1: Connect to Your MySQL Database
db = SQLDatabase.from_uri(
    "mysql+pymysql://root:**PWD**@54.163.**.***:3306/kcloud_aidrive",
    custom_table_info={"account": "Accounts table for querying accounts",
                       "account_file": "query files and file content",
                       "storage": "Storage table for querying storage"},  # Custom table description
)

# Step 2: Initialize Your LLM
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0,
    openai_api_key="*"
)

# Step 3: Bundle the Toolkit
toolkit = SQLDatabaseToolkit(db=db, llm=llm)
print(f"The number of tools:{len(toolkit.get_tools())}")
print("The list of tools:", [tool.name for tool in toolkit.get_tools()])


# Step 4: Create the SQL Agent
agent = create_sql_agent(
    llm=llm,
    toolkit=toolkit,
    prefix="""You are a professional cloud storage query assistant. Please follow these rules:
	1.	All responses must be in English.
	2.	Keep answers clear and concise, avoiding technical jargon.
	3.	If a query fails, try to rephrase it in a simpler way and attempt again.
	4.	When returning data results, explain them in plain and easy-to-understand language.
	5.	If the user's question is unclear, proactively ask for more details.""",
    verbose=True,  # Show reasoning steps and SQL
    agent_type=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    handle_parsing_errors=True  # Auto-handle SQL/response parsing issues
)


# Step 5: Define a Function to Handle User Queries
def ask_question(question):
    try:
        print(f"\nHandling your question:{question}")
        response = agent.invoke({"input": question})
        return response["output"]
    except Exception as e:
        print(f"System prompt: Encountered a technical issue while processing the query.")
        return "Sorry, the system is currently unable to process your request. Please try to describe your question in a simpler way, or try again later."


# Testing
questions = [
    "What is the phone number for username kai",  # Complex question since we do not provide any context, such as table names or column names
]

for q in questions:
    answer = ask_question(q)
    print(f"\nQuestion:{q}")
    print(f"Answer:{answer}")
    print("-" * 50)

```


### Execution Logs
```text
Handling your question:What is the phone number for username kai


> Entering new SQL Agent Executor chain...
Action: sql_db_list_tables  
Action Input: ""  account, account_file, file, file_chunk, file_suffix, file_type, share, share_file, storageI see that there are several tables in the database. The "account" table likely contains user information, including usernames and phone numbers. I will check the schema of the "account" table to find the relevant fields.  
Action: sql_db_schema  
Action Input: "account"  Accounts table for querying accountsI need to check the specific fields in the "account" table to find the phone number associated with the username "kai."  
Action: sql_db_schema  
Action Input: "account"  Accounts table for querying accountsIt seems that I need to clarify the schema of the "account" table to find the specific fields, including the phone number. I will check the schema again to ensure I have the correct information.  
Action: sql_db_schema  
Action Input: "account"  Accounts table for querying accountsIt seems I am not getting the specific details of the "account" table schema. I will try to list the tables again to ensure I have the correct information before proceeding.  
Action: sql_db_list_tables  
Action Input: ""  account, account_file, file, file_chunk, file_suffix, file_type, share, share_file, storageIt seems that I am unable to retrieve the schema details for the "account" table. I will try to check the query for retrieving the phone number for the username "kai" directly, but first, I need to ensure that the query is correct. 

Action: sql_db_query_checker  
Action Input: "SELECT phone_number FROM account WHERE username = 'kai';"  ```sql
SELECT phone_number FROM account WHERE username = 'kai';
```The query to retrieve the phone number for the username "kai" is correct. I will now execute this query to get the result.  
Action: sql_db_query  
Action Input: "SELECT phone_number FROM account WHERE username = 'kai';"  Error: (pymysql.err.OperationalError) (1054, "Unknown column 'phone_number' in 'field list'")
[SQL: SELECT phone_number FROM account WHERE username = 'kai';]
(Background on this error at: https://sqlalche.me/e/20/e3q8)It seems that the "phone_number" column does not exist in the "account" table. I need to check the schema of the "account" table to find the correct field for the phone number. 

Action: sql_db_schema  
Action Input: "account"  Accounts table for querying accountsIt appears that I am still unable to retrieve the schema details for the "account" table. I will try to check the fields in the "account" table again to find the correct field for the phone number. 

Action: sql_db_schema  
Action Input: "account"  Accounts table for querying accountsIt seems that I am unable to retrieve the schema details for the "account" table, which is preventing me from identifying the correct field for the phone number. I will try a different approach by checking the available fields in the "account" table directly.

Action: sql_db_query_checker  
Action Input: "SHOW COLUMNS FROM account;"  SHOW COLUMNS FROM account;The query to show the columns in the "account" table is correct. I will now execute this query to get the result.  
Action: sql_db_query  
Action Input: "SHOW COLUMNS FROM account;"  [('id', 'bigint', 'NO', 'PRI', None, ''), ('username', 'varchar(255)', 'NO', '', None, ''), ('password', 'varchar(255)', 'NO', '', None, ''), ('avatar_url', 'varchar(255)', 'YES', '', None, ''), ('phone', 'varchar(255)', 'YES', 'UNI', None, ''), ('role', 'varchar(255)', 'YES', '', 'COMMON', ''), ('del', 'bit(1)', 'NO', '', "b'0'", ''), ('est_create', 'datetime', 'NO', '', 'CURRENT_TIMESTAMP', 'DEFAULT_GENERATED'), ('est_modified', 'datetime', 'NO', '', 'CURRENT_TIMESTAMP', 'DEFAULT_GENERATED on update CURRENT_TIMESTAMP')]I have found the correct field for the phone number in the "account" table. The field is named "phone" instead of "phone_number." Now, I will query the phone number for the username "kai" using the correct field name.

Action: sql_db_query  
Action Input: "SELECT phone FROM account WHERE username = 'kai';"  [('123',)]I have successfully retrieved the phone number for the username "kai." The phone number is "123." 

Final Answer: The phone number for username "kai" is 123.
```


## ⚠️ Important Notes on Using `create_sql_agent` Effectively

### Improve SQL Generation Accuracy
- Adding `COMMENT` annotations directly in your database schema helps the LLM understand field meanings.

```sql
ALTER TABLE sales 
MODIFY COLUMN quantity INT COMMENT 'Number of items sold per transaction';
```

### Define Custom Table Descriptions
- Pass human-readable table descriptions to help LLMs interpret structure and logic:

```python
db = SQLDatabase.from_uri(
    "sqlite:///sales.db",
    include_tables=["sales", "products"],
    custom_table_info={"sales": "This table is read-only: SELECT statements are allowed; INSERT, UPDATE, and DELETE are not permitted."},
    view_support=False
)
```

### Explicitly Limit Available Fields in Prompts
```python
CUSTOM_PROMPT = """You can only use the following fields:
- products: product_id, product_name, category, price
- sales: sale_id, product_id, sale_date, quantity
Question:{question}"""
```

### Handle Complex Joins with Few-Shot Examples
- LLMs often struggle with multi-table joins. Providing a few relevant examples can significantly improve performance.
- These can be included in the system prompt or agent configuration to guide the model.

```python
examples = [
    (
        "How can I find sales records for a specific product?", 
        "SELECT * FROM sales JOIN products ON sales.product_id = products.product_id"
    )
]
```

### Others

| **Area**             | **Recommendation**                                                                 |
|----------------------|-------------------------------------------------------------------------------------|
| **Access Control**   | Use **read-only** DB users; strictly **restrict `INSERT` / `UPDATE` / `DELETE`**  |
| **Prompt Engineering** | Provide **schema details**, **field descriptions**, **few-shot examples**, and **sample rows** to improve LLM understanding |
| **Field Restrictions** | Clearly define **allowed fields and tables** in the system prompt to prevent hallucination |
| **Injection Defense** | Avoid **string concatenation** with user inputs; **sanitize all dynamic prompt content** |
| **Agent Fallbacks**  | Enable `handle_parsing_errors`; **wrap SQL execution** in error-handling logic     |
| **Result Limiting**  | Always **add `LIMIT` clauses** to generated SQL to avoid large result sets         |









<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



