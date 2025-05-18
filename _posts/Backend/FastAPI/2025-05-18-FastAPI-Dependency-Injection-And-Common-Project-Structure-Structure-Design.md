---
title: FastAPI Dependency Injection & Project Architecture Best Practices
date: 2025-05-18
categories: [Backend, FastAPI]
tags: [FastAPI, Dependency-injection, Architecture]
author: kai
---

# 🚀 FastAPI Dependency Injection & Project Architecture Best Practices

## 🧩 FastAPI Dependency Injection 
FastAPI’s **Dependency Injection (DI)** system is a powerful feature that promotes **clean code**, **modularity**, and **testability**. It allows developers to **extract common logic** like authentication, database sessions, and parameter validation into **reusable components** that are automatically injected into route handlers.

### Key Benefits

| Feature            | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| Code Reuse       | Avoid repeating logic like token parsing or DB session management           |
| Decoupling       | Separate business logic from infrastructure components (e.g., auth, DB)     |
| Layered Design    | Compose logic with **nested dependencies** — e.g., validate user ➝ check role |



### FastAPI vs. Java Spring DI

| Feature          | FastAPI                                 | Spring (Java)                          |
|------------------|------------------------------------------|----------------------------------------|
| Injection Style  | Function or Class-based + `Depends()`    | Annotation-based (e.g., `@Autowired`)  |
| Scope            | Per-request (default)                    | `Singleton`, `Prototype`, `Request`, etc. |
| Simplicity       | Lightweight, Pythonic                    | Full-featured but more complex         |
| Container        | Implicit (FastAPI handles it)            | Explicit (Spring IoC Container)        |


## 👀 Functions as Dependencies
FastAPI allows you to inject **dependencies**—reusable, self-contained logic—into any route handler using the `Depends()` function. These dependencies can be simple functions, classes, or even callables with parameters.

- `common_params()` is a standalone function that accepts query parameters.
- The result of common_params() is automatically injected into the read_items route handler via `Depends()`.
- You can now directly access `params["query"]` and `params["page"]` inside the handler without repeating parsing logic.


```python
from fastapi import Depends, FastAPI

app = FastAPI()

# Define a reusable dependency function
def common_params(query: str = None, page: int = 1):
    print("Executing common_params")
    page = page + 1  # Add 1 to the page
    return {"query": query, "page": page}

# Inject the dependency into the route (The return value of common_params will be passed to the params variable)
@app.get("/read_items")
async def read_items(params: dict = Depends(common_params)):
    print("Executing read_items")
    return params


# Example Request
# GET /read_items?query=fastapi&page=2


# Response 
# {
#   "query": "fastapi",
#   "page": 3   -> return 2 + 1 
# }
```

## 🥷🏻 Classes as Dependencies

FastAPI's dependency injection system is not limited to simple functions—you can also use **classes** as dependencies. This is especially useful when:

- You need to encapsulate logic with internal state
- You want to manage resources (like database sessions)
- You aim to organize complex functionality into reusable components

### Database Session Management
- The DatabaseSession class encapsulates a session-like object.
- The get_db() function acts as a dependency provider using Python’s yield.
- FastAPI will automatically handle the finally block and call close() after the request is processed.
= The db parameter in the route handler is automatically injected.

```python
from fastapi import Depends, FastAPI

app = FastAPI()


class DatabaseSession:
    def __init__(self):
        self.session = "Simulated DB session"

    def close(self):
        print("Closing DB session")


def get_db():
    db = DatabaseSession()
    try:
        yield db
    finally:
        db.close()


@app.get("/users/")
async def get_users(db: DatabaseSession = Depends(get_db)):
    return {"db_session": db.session}
```


### Pagination Parameters as a Class
- The Pagination class is instantiated automatically by FastAPI using the request query parameters.
- No need to manually parse or validate query strings.
- Clean and reusable across multiple routes.

```python
from fastapi import Depends, FastAPI

app = FastAPI()


class Pagination:
    def __init__(self, page: int = 1, size: int = 10):
        self.page = page
        self.size = size


@app.get("/articles/")
async def get_articles(pagination: Pagination = Depends()):
    return {
        "page": pagination.page,
        "size": pagination.size
    }
```

## 🌐 Global Dependencies in FastAPI
FastAPI’s powerful **dependency injection system** allows you to define **shared logic** (like authentication, logging, or rate-limiting) and apply it across your entire application—or specific groups of routes.

This keeps your code **clean, maintainable, and DRY** (Don't Repeat Yourself).

### 1. Apply a Global Dependency to All Routes
- To enforce logic (e.g., token verification) across **every** endpoint:
    - Every route in the app will automatically run verify_token() before the request is processed.

```python
from fastapi import FastAPI, Depends, HTTPException, Header

def verify_token(token: str = Header(...)):
    # Perform token validation
    if token != "valid_token":
        raise HTTPException(status_code=401, detail="Invalid token")
    return token

app = FastAPI(dependencies=[Depends(verify_token)])

# When requesting /hello, the verify_token dependency will be called, ensuring the token is valid
@app.get("/hello")
async def root():
    return {"message": "Hello World"}
```

### 2. Scoped Dependency: Route Group Level
- To apply logic only to a specific set of endpoints, use APIRouter with dependencies:
    - All endpoints registered under this router will automatically trigger log_request().

```python
from fastapi import APIRouter, Depends


def log_request():
    # Custom logging logic
    pass


router = APIRouter(
    prefix="/api/v1",
    dependencies=[Depends(log_request)]
)


@router.get("/items")
async def list_items():
    return {"message": "Logging enabled for this route group"}
```

### Secure Specific Endpoints (Admin-Only)
- For sensitive routes, add dependencies directly to the route:
    - These endpoints require successful authentication via admin_auth() before access is granted.

```python
from fastapi import Depends, HTTPException

def admin_auth():
    # Perform admin-level access check
    pass

@app.get("/admin/stats", dependencies=[Depends(admin_auth)])
def get_stats():
    return {"message": "Admin stats"}

@app.post("/admin/users", dependencies=[Depends(admin_auth)])
def create_user():
    return {"message": "User created"}
```


### Best Practices
FastAPI provides flexible control over **where and how dependencies are applied**, making it easy to enforce shared logic across different parts of your application.

| **Scope**        | **Use Case**                                             |
|------------------|----------------------------------------------------------|
| `Global App`     | Apply to all incoming requests (e.g., token authentication) |
| `APIRouter Group`| Apply to a specific set of endpoints (e.g., request logging, role filtering) |
| `Per Route`      | Fine-grained control for sensitive endpoints (e.g., admin-only access) |

🧠 **Tip:** You can **combine scopes** for layered behavior. For example, apply global authentication while also enforcing admin-specific checks on certain routes.


## 🗂️ FastAPI Project Structure Design Principles
A clean and well-organized project structure is essential for scalability, collaboration, and long-term maintenance. Below is a recommended base structure that works well for **small to medium-sized FastAPI applications**.

### Basic Layered Structure

```text
myproject/
├── main.py               # Entry point of the FastAPI app
├── routers/              # Route modules grouped by business domains
│   ├── users.py          # User-related endpoints
│   └── items.py          # Item-related endpoints
├── models/               # Pydantic models and data schemas
│   └── schemas.py        # Data validation and response models
└── dependencies.py       # Shared dependency logic (e.g., auth, DB sessions)
```

### Recommended Folder Structure (Mid-Size Projects)

```text
src/
├── app/
│   ├── core/             # Core configuration and security
│   │   ├── config.py     # App settings, env loading, constants
│   │   └── security.py   # OAuth2, JWT, password hashing
│   ├── api/              # Main routing logic
│   │   ├── v1/           # API versioning (v1, v2, ...)
│   │   │   ├── users/    
│   │   │   │   ├── endpoints.py    # Route declarations
│   │   │   │   └── schemas.py      # Request/response models
│   │   │   └── items/              # Another module (same structure)
│   ├── models/          # SQLAlchemy or ORM models
│   ├── services/        # Business logic layer (e.g., user service)
│   └── utils/           # Utility functions/helpers
├── tests/               # Pytest-based unit/integration tests
└── requirements.txt     # Project dependencies
```

### Modular FastAPI Project Structure (Version 2 for Medium-Sized Projects)

```text
myproject/
├── app/                  # Application core
│   ├── core/             # Global settings and shared utilities
│   │   ├── config.py     # Environment configs and constants
│   │   └── security.py   # Auth, token, password utilities
│   ├── api/              # API endpoints (router-level)
│   │   ├── v1/           # API version group
│   │   │   ├── users.py
│   │   │   ├── items.py
│   │   │   └── ai.py
│   │   └── deps.py       # Common dependencies (e.g., DB, auth)
│   ├── models/           # Pydantic request/response models
│   │   ├── user.py
│   │   └── item.py
│   ├── services/         # Business logic layer
│   │   ├── user_service.py
│   │   └── ai_service.py
│   ├── db/               # Database layer
│   │   ├── session.py    # DB session management
│   │   └── models.py     # SQLAlchemy model definitions
│   └── utils/            # Utility functions and loggers
│       └── logger.py
├── tests/                # Unit/integration tests
│   ├── test_users.py
│   └── conftest.py       # Fixtures and test setup
├── static/               # Static file assets (HTML, CSS, JS)
├── main.py               # Application entry point
├── requirements.txt      # Dependency lock file
└── .env                  # Environment variables
```


### Recommended Architecture Principles for FastAPI Projects
When designing a **scalable and maintainable FastAPI project**, it's essential to follow a set of clean architecture principles. While every team may have its own standards, here are some **universally recommended practices** that help improve code quality, modularity, and collaboration.

| Principle          | Implementation Method                                                                 |
|--------------------|----------------------------------------------------------------------------------------|
| **Single Responsibility** | Each file or class should **do only one thing**. Avoid mixing concerns in the same module. |
| **Dependency Inversion** | Use **dependency injection** (`Depends()`) to decouple infrastructure from business logic. |
| **Layered Separation**    | Clearly separate **route layer**, **service layer**, and **data access layer** (DAL).         |
| **API Versioning**        | Use **URL path versioning** (`/api/v1/...`) to isolate future changes.                        |
| **OpenAPI Documentation** | Provide meaningful `summary` and `description` for each route to generate helpful docs.        |









<br>

---

🚀 Stay tuned for more insights into Python!