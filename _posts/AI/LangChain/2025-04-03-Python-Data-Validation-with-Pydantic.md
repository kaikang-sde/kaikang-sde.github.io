---
title: Python Model Management with Pydantic
date: 2025-04-03
order: 1
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

1️⃣ **Traditional Java Validation**
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

2️⃣ **Legacy Python Validation**
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

3️⃣ **Pydantic: Declarative & Type-Safe**
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


## 🔍 `Field()` in Pydantic
The `Field()` function in **Pydantic** provides a powerful way to add metadata, set validation constraints, and improve documentation for each model field — especially useful for API generation or strict input validation.

| Purpose          | Example Use                              |
|------------------|-------------------------------------------|
| Add metadata     | `title`, `description`, `example`         |
| Set validation   | `min_length/max_length`, `gt/ge/lt/le`, `regex`, etc. |
| Enforce required | Use `...` (Ellipsis) to mark as required  |

**Details:**
- ...: indicates a required field (no default allowed).
- title, description: used in schema generation (OpenAPI, docs).
- min_length, max_length: for strings or lists.
- gt, lt, ge, le: for numeric boundaries.
- regex: enforce string format (e.g., phone/email).
- example: for documentation tools.

### Required Fields
In **Pydantic**, fields marked with `Field(...)` are **mandatory** — they must be provided during model instantiation. This is equivalent to saying: *"No default value is allowed; this field is required."*

The ellipsis (`...`) is a special symbol in Pydantic that acts as a **required field indicator**. It's passed as the **first argument** to `Field()` to indicate **no default value**.


```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str = Field(..., title="Username", min_length=2, description="User's name must be at least 2 characters.")

user = User(name="Alice")
print(user) # name='Alice'

user = User() # ValidationError: 1 validation error for User name -> Field required [type=missing, input_value={}, input_type=dict]
```

### Optional Fields with Default Values
In **Pydantic**, if a field is given a **default value**, it becomes **optional** — meaning you don’t have to provide it during model instantiation.

```python
from pydantic import BaseModel, Field

lass User(BaseModel):
    name: str = Field("Guest", title="username")  # Default value: "Guest"

user = User()
print(user) # name='Guest'
```

### Field vs. Direct Type Annotation in Pydantic
In **Pydantic**, both of the following declarations define a **required field**, and functionally they are equivalent:

```python
from pydantic import BaseModel

# Option 1: Implicit required field
class User(BaseModel):
    name: str  # Required

# Option 2: Explicit with Field(…)
class User(BaseModel):
    name: str = Field(...)  # Also required
```

> **Why Use Field(...)?**<br>
> While both options define a required field, Field(...) adds more control and supports rich metadata.


### Numeric Field Validation
When working with numeric data in Python applications, it's crucial to enforce **value constraints** to ensure data integrity. Pydantic’s `Field(...)` provides an elegant and declarative way to validate numeric inputs.


```python
from pydantic import BaseModel, Field, ValidationError

class Product(BaseModel):
    price: float = Field(..., title="price", gt=0)     # gt: greater than -> `price` must be greater than 0
    stock: int = Field(..., ge=0)                      # ge: greater or equal to -> `stock` must be zero or a positive integer

# Valid example
product = Product(price=99.9, stock=10)
print(product) # price=99.9 stock=10


try:
    Product(price=-5, stock=10)
except ValidationError as e:
    print(e.json()) 

# Output: 
# [{"type":"greater_than","loc":["price"],"msg":"Input should be greater than 0","input":-5,"ctx":{"gt":0.0},"url":"https://errors.pydantic.dev/2.7/v/greater_than"}]

```


### Required Nested Models
When your data structures grow complex, you often need to **nest models** inside other models. Pydantic makes this intuitive and powerful — while still ensuring validation of required fields.

```python
from pydantic import BaseModel, Field

class Address(BaseModel):
    city: str = Field(..., min_length=1)   # Required with min length
    street: str                            # Optional by default if no `Field(...)`

class User(BaseModel):
    name: str = Field(...)                 # Required
    address: Address                       # Nested model (implicitly required)

user = User(name="Alice", address={"city": "Shanghai", "street": "Main St"})
print(user) # name='Alice' address=Address(city='Shanghai', street='Main St')


user = User(name="Bob") 
# 1 validation error for User address Field required [type=missing, input_value={'name': 'Bob'}, input_type=dict]
```


### Explicit Optional Fields
In many data validation scenarios, not every field is required. Pydantic provides a clean way to **declare optional fields** using `typing.Optional` and the `Field()` helper.

- `Optional[...]`: Indicates the field **can be None**
- `Field(default_value)`: Set a default value like `None`

```python
from pydantic import BaseModel, Field
from typing import Optional

class User(BaseModel):
    name: str = Field(...)  # Required field
    email: Optional[str] = Field(None, title="email")  # Optional, default to None

user = User(name="Alice")
print(user) # Output: name='Alice' email=None

user = User(name="Bob", email="bob@example.com")
print(user) # Output: name='Bob' email='bob@example.com'
```


### Mixing Required and Optional Fields
In real-world applications, some configuration fields are **mandatory**, while others can be **optional with defaults**.  
Pydantic makes it easy to handle both with its `Field(...)` declaration.

```python
from pydantic import BaseModel, Field

class Config(BaseModel):
    api_key: str = Field(...)           # Required
    timeout: int = Field(10, ge=1)      # Optional, default=10, must be >= 1

config = Config(api_key="secret")
print(config.timeout)  # 10 (default applied)

Config(timeout=5) # Raises ValidationError: "api_key" field is required
```


## 🥷🏻 Custom Field Validation with `@field_validator`
Pydantic provides rich built-in validation via type hints and field constraints.  
But for more **advanced rules**, you can define **custom validation logic** using the `@field_validator` decorator, which allows you to **add custom validation logic** to **a single field**.

- Applied **after** base type and constraint checks (e.g., `min_length`, `gt`, `regex`).
- Can return a **transformed value** (e.g., auto-strip whitespace, lowercase emails).
- Added in **Pydantic v2.x**, replacing the older `@validator` from v1.

```python
from pydantic import BaseModel, Field, field_validator

class User(BaseModel):
    email: str

    @field_validator("email")
    def validate_email(cls, v):
        if "@" not in v:
            raise ValueError("invalid email address")
        return v.lower()  # return formatted value

# 正确
alice = User(email="ALICE@example.com")  # alice@example.com
print(alice) # email='alice@example.com'

# 错误
User(email="invalid")  
# 1 validation error for User email Value error, invalid email address [type=value_error, input_value='invalid', input_type=str]
```

### Shared Field Validator
In **Pydantic V2**, the `@field_validator` decorator allows you to attach a **single validator to multiple fields** — a concise way to enforce the same validation rule across multiple attributes.

```python
from pydantic import BaseModel, Field, field_validator

class Product(BaseModel):
    price: float
    cost: float

    @field_validator("price", "cost") # ensure that both `price` and `cost` are **positive numbers** without duplicating code.
    def check_positive(cls, v):
        if v <= 0:
            raise ValueError("Must be greater than 0")
        return v

# This will raise a ValidationError for cost
Product(price=1, cost=-2)
```


### Common Pitfall: Forgetting the Return Statement
A @field_validator **must explicitly return the field value** — either unchanged or modified.
If you forget to return, your model will **raise an error** (or **the field will be silently set to None**).


## 🔄 Pydantic v1 vs v2
With the release of Pydantic V2, many internal and external APIs have been refined or renamed for clarity and consistency.

- V2 is now the production-ready stable release
- Introduces performance optimizations (10x faster in some benchmarks)
- Significant API refinements — improves clarity but breaks backward compatibility

| **Pydantic V1**           | **Pydantic V2**             | **Purpose**                                               |
|---------------------------|------------------------------|-----------------------------------------------------------|
| `__fields__`              | `model_fields`               | Access model field definitions                            |
| `__validators__`          | `__pydantic_validator__`     | Internal validators collection                            |
| `__private_attributes__`  | `__pydantic_private__`       | Custom internal attributes                                |
| `construct()`             | `model_construct()`          | Create a model instance without validation                |
| `copy()`                  | `model_copy()`               | Deep copy a model                                         |
| `dict()`                  | `model_dump()`               | Convert model to dict (includes all validations)          |
| `json()`                  | `model_dump_json()`          | Serialize model to JSON                                   |
| `parse_obj()`             | `model_validate()`           | Validate raw dict into model                              |
| `json_schema()`           | `model_json_schema()`        | Generate JSON Schema for API/documentation                |
| `update_forward_refs()`   | `model_rebuild()`            | Rebuild forward references for self-referencing models    |





<br>



---

🚀 Stay tuned for more insights into LangChain!



