---
title: Python Model Management with Pydantic
date: 2025-04-02
categories: [AI, LangChain]
tags: [LangChain, Pydantic, LLM, Prompt Engineering, Parsing]
author: kai
---


# 🚀 Python Model Management with Pydantic
**[Pydantic](https://docs.pydantic.dev/latest/)** is a powerful **data validation and parsing library** in Python.  
It leverages **Python type hints** to enforce type checks, parse untrusted data, and serialize model instances into JSON/dict with ease.

> Think of it as a “type-safe, declarative, and user-proof” model layer — perfect for APIs, AI agents, config handling, and data pipelines.

- **Declarative Models**: Define schemas using class-based syntax and type hints.
- **Validation**: Automatically checks the data types and formats.
- **Parsing**: Converts dictionaries, JSON, and other inputs into Python objects.
- **Serialization**: Easily export models back into dicts or JSON.
- **Integration-Friendly**: Works well with FastAPI, LangChain, SQLAlchemy, etc.

## 🆚 Java vs Python vs Pydantic: Model Validation Comparison

1. **Traditional Java Validation**
In Java, developers often need to **manually write validation logic** within models:
- **Boilerplate-heavy**, not type-safe at runtime, and requires explicit checks everywhere.

```java
public class User {
    private String name;
    private int age;

    public void validate() throws IllegalArgumentException {
        if (name == null || name.isEmpty()) {
            throw new IllegalArgumentException("name is null or empty");
        }
        if (age > 150) {
            throw new IllegalArgumentException("invalid age");
        }
    }
}
```

2. **Legacy Python Validation**
Python also requires manual type and value checks:
- Repetitive, prone to errors, no automatic type coercion or validation.

```python
class User:
    def __init__(self, name: str, age: int):
        if not isinstance(name, str):
            raise TypeError("name must be string")
        if not isinstance(age, int):
            raise TypeError("age must be int")
        if age > 150:
            raise ValueError("age must be in 0-150")
        self.name = name
        self.age = age
```

3. **Pydantic: Declarative & Type-Safe**
With Pydantic, data models are **declarative**, readable, and automatically validated:
- Raises ValidationError for type or value mismatches.
- Automatically converts types when safe (e.g., “30” → 30).

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str = Field(min_length=1, max_length=50) 
    age: int = Field(ge=1, le=50) 
```

## 🧛 Installing and Using Pydantic v2
Pydantic v2 requires **Python 3.10 or above** and can be installed using pip:<br>
`pip install pydantic==2.7.4`

### 1. instance creation
Pydantic **automatically validates** input data when creating a model instance. If the data doesn't match the expected type or structure, it raises a `ValidationError`.

```python
from pydantic import BaseModel

class UserProfile(BaseModel):
    username: str
    age: int = 18
    email: str | None = None


user1 = UserProfile(username="Kai")  # Valid
print(user1) # username='Kai' age=18 email=None

user2 = UserProfile(username="Jo", age="20")  # Auto-converted to int
print(user2) # username='Jo' age=20 email=None
```

### 2. Validation

```python
from pydantic import BaseModel

class UserProfile(BaseModel):
    username: str
    age: int = 18
    email: str | None = None

# Trigger validation error by passing an integer instead of string
try:
    UserProfile(username=123)
except ValueError as e:
    print(e.errors())
    # [{
    #   'type': 'string_type',
    #   'loc': ('username',),
    #   'msg': 'Input should be a valid string',
    #   'input': 123
    # }]
```

### 3. field-level validation
Pydantic offers rich support for **field-level validation**, enabling you to enforce types like `HttpUrl`, enforce list structures, and define default values with ease.


```python
from pydantic import BaseModel,HttpUrl,ValidationError

class WebSite(BaseModel):
    url: HttpUrl # URL format validation
    visits: int = 0 # default value
    tags: list[str] = [] # list of strings

valid_data = {
    "url": "https://kaikang-sde.github.io/",
    "visits": 100,
    "tags": ['Learning', 'LangChain']
}

# Example 1: Valid data
# try:
#     website = WebSite(**valid_data)
#     print(website) # url=Url('https://kaikang-sde.github.io/') visits=100 tags=['Learning', 'LangChain']
# except ValidationError as e:
#     print(e.errors())


# Example 2: Invalid data
try:
    website = WebSite(url="kaikang-sde",visits=100)
    print(website)
except ValidationError as e:
    print(e.errors())

# Error output
[{'type': 'url_parsing', 'loc': ('url',), 
  'msg': 'Input should be a valid URL, relative URL without a base', 
  'input': 'kaikang-sde', 
  'ctx': {'error': 'relative URL without a base'}, 
  'url': 'https://errors.pydantic.dev/2.7/v/url_parsing'}]
```

### 4. Pydantic Data Parsing & Serialization
One of Pydantic’s superpowers is **seamless JSON parsing** and **structured data serialization** — making it easy to convert between raw inputs and validated Python objects.

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

# JSON input (price is a string, will be coerced)
data = '{"name": "Widget", "price": "9.99"}'

# Clean JSON input -> Strongly typed models.Model Item is defined above
item = Item.model_validate_json(data)

# Can serialize the validated model to a standard Python dict
print(item.model_dump()) # {'name': 'Widget', 'price': 9.99}
```


### 5. Debugging Tip: Print Model Structure with `model_json_schema()`

```python
from pydantic import BaseModel, HttpUrl

class Website(BaseModel):
    url: HttpUrl
    visits: int = 0
    tags: list[str] = []

# Print the full schema definition
print(Website.model_json_schema())

# Output
{
  "title": "Website",
  "type": "object",
  "properties": {
    "url": {
      "title": "Url",
      "type": "string",
      "format": "uri"
    },
    "visits": {
      "title": "Visits",
      "type": "integer",
      "default": 0
    },
    "tags": {
      "title": "Tags",
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": ["url"]
}
```


---

🚀 Stay tuned for more insights into LangChain!



