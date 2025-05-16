---
title: FastAPI with Pydantic
date: 2025-05-16
order: 2
categories: [Backend, FastAPI]
tags: [FastAPI, Pydantic, Python, API, Data Validation]
author: kai
---

# 🚀 FastAPI with Pydantic
When building APIs with **FastAPI**, defining **data models** using **Pydantic** is essential for:

- Validating client input
- Ensuring data integrity
- Structuring clean API contracts

[Python Model Management with Pydantic]({{ site.baseurl }}/Python-Data-Validation-With-Pydantic)


## ❓ What Is a Data Model

In the context of a web API:

- **Client input model** defines what kind of data the API **accepts**.
- **Server output model** defines what kind of data the API **returns**.
- **Validation rules** ensure only correct and clean data is processed.

FastAPI uses **Pydantic** to define and enforce these models.


## ✅ Basic Model Example
```python
from pydantic import BaseModel, Field

# User Registration Input Model
class UserRegister(BaseModel):
    username: str                     # Required field
    email: str | None = None          # Optional field
    age: int = Field(18, gt=0)        # Field with default value and validation
```

## 🛍️ Handling Request Body in FastAPI Using Pydantic Models
When building a modern API, validating and parsing **structured request data** is essential. With **FastAPI + Pydantic**, this process becomes seamless and Pythonic.

**API Example: Product Creation:**

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

# Define the Product model with validation and optional fields
class Product(BaseModel):
    name: str                      # Required
    description: str = None        # Optional
    price: float                   # Required
    tax: float = None              # Optional

@app.post("/products")
async def create_product(product: Product):
    return product.model_dump()    # Returns parsed and validated product as a dictionary

# Sample Response
# { 
#   "name": "Wireless Mouse",
#   "description": "Ergonomic design",
#   "price": 25.99,
#   "tax": 1.5
# }
```

## 🎯 Returning Pydantic Models in FastAPI
In FastAPI, not only can you **receive** data using `Pydantic` models, you can also **return** them directly as structured responses.

This ensures your API responses are consistent, validated, and automatically documented in Swagger UI.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# Define the response model
class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

@app.post("/items")
async def create_item(item: Item):
    return item  # FastAPI automatically serializes the model to JSON

# Sample Response
# { 
#   "name": "Wireless Mouse",
#   "description": "Ergonomic design",
#   "price": 25.99,
#   "tax": 1.5
# }
```

## 🔒 Password Strength Validation in FastAPI
[Fields Pydantic]({{ site.baseurl }}/Python-Data-Validation-With-Pydantic/#-field-in-pydantic)

With FastAPI and Pydantic, we can easily define **custom password validators** using the `@field_validator` decorator in Pydantic v2.

```python
from fastapi import FastAPI
from pydantic import BaseModel, field_validator

app = FastAPI()


class UserRegistration(BaseModel):
    username: str
    password: str

    # Custom validator for the password field
    @field_validator('password')
    def validate_password(cls, v):
        if len(v) < 8:
            raise ValueError('Must be at least 8 characters long')
        if not any(c.isupper() for c in v):
            raise ValueError('Must contain at least one uppercase letter')
        if not any(c.isdigit() for c in v):
            raise ValueError('Must contain at least one number')
        return v


@app.post("/register/")
async def register(user: UserRegistration):
    return {"username": user.username}
```











<br>

---

🚀 Stay tuned for more insights into Python!