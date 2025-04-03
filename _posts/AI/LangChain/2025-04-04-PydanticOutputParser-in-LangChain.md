---
title: PydanticOutputParser in LangChain
date: 2025-04-03
order: 2
categories: [AI, LangChain]
tags: [LangChain, Pydantic, OutputParser, LLM]
author: kai
---


# 🚀 PydanticOutputParser in LangChain
In modern LLM-based applications, the **output from a large language model (LLM)** is typically **unstructured text**. If you're building structured workflows or API-driven agents, you need to **validate, parse, and safely transform** that output into well-defined Python objects.

That's where `PydanticOutputParser` comes into play.

| Reason              | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| **Structured Output** | Converts raw LLM responses into structured Python objects (`BaseModel`)     |
| **Data Validation**   | Automatically enforces types, ranges, and constraints using Pydantic rules  |
| **Developer Speed**  | Removes the need for custom parsing logic for each response                 |
| **Error handling**   | Offers better error handling and response correction if the format is wrong |

`JsonOutputParser` just converts raw JSON text → Python object, **no validation**!        

## 1️⃣ Case Study: extract structured information from unstructured LLM output

**Goal:** From a natural language description of a user, extract:
- `name`: string  
- `age`: integer (must be > 0)  
- `hobbies`: list of strings

**Key Technologies Used**
- `LangChain`: Prompt + Model + Output parser chaining
- `Pydantic`: Data validation and structured typing
- `PydanticOutputParser`: Converts LLM output into `BaseModel` objects

{% raw %}
```python
from langchain_openai import ChatOpenAI
from pydantic import BaseModel, Field  
from langchain_core.output_parsers import PydanticOutputParser  
from langchain_core.prompts import ChatPromptTemplate  
import os


os.environ["OPENAI_API_KEY"] = "*"  # replace with your actual key

# 1. Create LLM instance
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# 2: Define Pydantic model for User Infromation
class UserInfo(BaseModel):  
    name: str = Field(...,description="username")  
    age: int = Field(..., description="age", gt=0)  
    hobbies: list[str] = Field(description="hobbies")

# 3. Create output parser
parser = PydanticOutputParser(pydantic_object=UserInfo)  

# 4. Build Prompt Template
prompt = ChatPromptTemplate.from_template(
    """  
    You are a text analyzer. Extract user information strictly in the following format:
    {format_instructions}  

    input: 
    {input}  
    """
)  

# 5. inject format instructions
prompt = prompt.partial(  
    format_instructions=parser.get_format_instructions()  
)  

# 5. Chain
chain = prompt | llm | parser  

# 6. Invoke
result = chain.invoke({  
    "input": "My name is Kai, I am 20 years old, and I like playing basketball and watching movies. I am very happy today."
})  

print(type(result))  # <class '__main__.UserInfo'>
print(result)  # name='Kai' age=20 hobbies=['playing basketball', 'watching movies']
print(result.model_dump()) # {'name': 'Kai', 'age': 20, 'hobbies': ['playing basketball', 'watching movies']}
```
{% endraw %}


## 2️⃣ Case Study: E-commerce Sentiment Analysis

**Goal:** Analyze customer reviews (in natural language), and extract:
- `sentiment`: overall sentiment of the review  
- `confidence`: confidence score (0.0 ~ 1.0)  
- `keywords`: extracted list of key phrases 

### JsonOutputParser vs. PydanticOutputParser
`JsonOutputParser` and `PydanticOutputParser` are both used to **parse structured output** from LLM responses, but there are some subtle differences and strengths worth understanding:

| Feature                        | `JsonOutputParser`                                      | `PydanticOutputParser`                                  |
|-------------------------------|----------------------------------------------------------|----------------------------------------------------------|
| **Model Binding Support**      | Supports `pydantic_object` (optional enhancement)     | Requires `pydantic_object` to define the output shape |
| **Use Without Schema**          | Can be used without Pydantic schema                   | Requires a valid Pydantic model                       |
| **Stream Compatibility**        | Supports parsing **partial JSON** in streaming        | Not ideal for stream parsing                          |
| **Built-in Validation**         | If schema provided; otherwise raw parsing             | Full type-safe validation                             |
| **Output Format Enforcement**   | Guides LLM using format instructions                  | More strict structure (fewer hallucinations)          |
| **Use Case**                    | Best for **lightweight JSON use** and **stream parsing**| Best for **strong validation and typed models**         |


### Key Takeaway
1. `JsonOutputParser` offers **flexibility** and **streaming support**, while `PydanticOutputParser` offers **strict structure enforcement** and **data validation**.

    You can choose between them based on:
    - Do you need strict typing? → Use `PydanticOutputParser`
    - Do you stream responses or want relaxed structure? → Use `JsonOutputParser`

2. The `pydantic_object` parameter in `JsonOutputParser` is **optional**, but if provided, it **enhances validation** and guides **format generation** via `.get_format_instructions()`.
   `parser = JsonOutputParser(pydantic_object=MyModel)`  # Optional



{% raw %}
```python
from langchain_openai import ChatOpenAI
from pydantic import BaseModel, Field  
from langchain_core.output_parsers import JsonOutputParser   
from langchain_core.prompts import ChatPromptTemplate  
import os


os.environ["OPENAI_API_KEY"] = "*"  # replace with your actual key

# 1. Create LLM instance
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# 2: Define Pydantic model for SentimentResult
class SentimentResult(BaseModel):  
    sentiment: str  
    confidence: float  
    keywords: list[str]

# 3. Create output parser
parser = JsonOutputParser(pydantic_object=SentimentResult) 

# 4. Build Prompt Template
prompt = ChatPromptTemplate.from_template(
    """  
    Analyze the sentiment of the following review:  {input}  
    Respond in JSON format: {format_instructions}  
    """
).partial(format_instructions=parser.get_format_instructions())  

# 4. Chain
chain = prompt | llm | parser  

# 6. Invoke
result = chain.invoke({  
    "input": "The food was great, but the service was slow."
})  
# print(result) # {'sentiment': 'mixed', 'confidence': 0.75, 'keywords': ['food', 'great', 'service', 'slow']}


# 6. Streaming
for chunk in chain.stream({"input": "Delivery is slow, the packaging is broken."}):
   print(chunk)  
# {'sentiment': 'negative', 'confidence': 0.85, 'keywords': ['slow delivery', 'broken packaging']}
```
{% endraw %}








<br>



---

🚀 Stay tuned for more insights into LangChain!



