---
title: Modular Route Management in FastAPI Using APIRouter
date: 2025-05-17
categories: [Backend, FastAPI]
tags: [FastAPI, APIRouter, Code Structure, API Versioning, Security]
author: kai
---

# 🚀 Modular Route Management in FastAPI Using APIRouter
As your FastAPI project grows, **maintaining all routes in `main.py` becomes a nightmare**. Without proper routing management, your code will face:

1. **Code Bloat**
All routes in a single `main.py` leads to:
- Unreadable, bloated files
- Difficult navigation and debugging

2. **Repetitive Config**
Every endpoint might repeat:
- Updating prefix/tag? You’ll need to edit every line manually.

    ```python
    @app.get("/api/v1/users", tags=["users"])
    @app.post("/api/v1/users", tags=["users"])
    ```

3. **Scattered Permissions**
If each admin route sets up its own authentication:
- it’s tedious, error-prone, and inconsistent.

    ```python
    @app.get("/admin", dependencies=[Depends(auth_admin)])
    ```


## 👀 Use APIRouter

FastAPI provides APIRouter to organize routes into modular, reusable, and configurable routers.

| Practice             | Description                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| **Modular Structure** | Organize routes by business domain (e.g., `users.py`, `items.py`)           |
| **Prefix Management** | Use the `prefix` argument to avoid duplicating path prefixes in every route |
| **Doc Friendly**      | Utilize `tags`, `summary`, and status codes to generate clean OpenAPI docs  |
| **Layered Dependencies** | Apply shared dependencies (like auth) at the module level; keep route-level logic focused |


### Why Modularize?

- **Separation of concerns** — one file per business domain
- **Reusability** — routers can be imported in different projects
- **Maintainability** — easier navigation, debugging, and testing

## 📦 Suggested File Structure

```text
project/
├── main.py                # Application entry point
└── routers/               # All modular API route files
    ├── __init__.py        # Ensures routers is a Python module
    ├── users.py           # User-related endpoints
    ├── items.py           # Item/product-related endpoints
    ├── admin/             # Admin-only routes (RBAC ready)
    │   ├── dashboard.py   # Admin dashboard operations
    │   └── audit.py       # Logs, auditing, and monitoring
    └── v2/                # Versioned API for /v2 endpoints
        └── users.py       # Updated user routes for API v2
```

### Code Example
```python
# main.py
from fastapi import FastAPI
from routers import users, items, orders 

app = FastAPI()

app.include_router(users.router)
app.include_router(admin.router)
app.include_router(orders.router)


# routers/admin.py
from fastapi import APIRouter, Depends

# A dummy admin authentication dependency
def admin_auth():
    # Add your JWT / OAuth2 / session validation here
    pass

# Define admin router with prefix and shared dependencies
admin_router = APIRouter(
    prefix="/admin",
    dependencies=[Depends(admin_auth)],
    tags=["Admin"]
)

# Admin-only endpoints - /admin/stats
@admin_router.get("/stats")
def get_stats():
    return {"message": "This is an admin-only stats endpoint."}

@admin_router.post("/users")
def create_user():
    return {"message": "New user created by admin."}

```

### Best Practices
- Use `prefix="/admin"` to namespace all admin routes.
- Use `dependencies=[Depends(...)]` to enforce centralized authentication.
- Use `tags=["Admin"]` for Swagger UI grouping.
- Don’t duplicate `Depends(admin_auth)` in every route


## 🧩 Core Configuration Parameters

| **Parameter**   | **Purpose**                                                | **Example Value**                         |
|------------------|-------------------------------------------------------------|--------------------------------------------|
| `prefix`         | Shared path prefix for all endpoints in this router         | `"/api/v1"`                                |
| `tags`           | Group endpoints in the OpenAPI UI                           | `["Authentication"]`                       |
| `dependencies`   | Apply shared dependencies (e.g., authentication) to routes  | `[Depends(verify_token)]`                  |
| `responses`      | Define default OpenAPI response schemas for all endpoints   | `{400: {"model": ErrorModel}}`             |


## 🔁 Example

- **app/users.py**

```python
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel

router = APIRouter(
    prefix="/users/v1",       # Prefix for all user-related routes
    tags=["User Management"],  # Grouping in OpenAPI docs
    dependencies=[],           # Shared dependencies (if any)
    responses={404: {"description": "Not found"}},
)


@router.get("/", summary="Get user list")
async def list_users():
    return [{"id": 1, "name": "Alice"}]


@router.post("/", summary="Create a new user")
async def create_user():
    return {"id": 2, "name": "Bob"}


class UserCreate(BaseModel):
    username: str
    password: str


@router.post("/register", status_code=201)
async def register(user: UserCreate):
    """Register a new user."""
    # In production, you should persist this to a database
    return {
        "message": "User successfully created",
        "username": user.username
    }


@router.get("/{user_id}")
async def get_user(user_id: int):

    if user_id > 100:
        raise HTTPException(status_code=404, detail="User not found")
    return {"user_id": user_id, "name": "Virtual User"}
```

- **app/products.py**

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/products/v1",       # All routes prefixed with /products
    tags=["Product Management"],  # OpenAPI tag group
    dependencies=[]           # Shared dependencies (if needed)
)


@router.get("/search", summary="Search products")
async def search_products(
    q: str,
    min_price: float = None,
    max_price: float = None
):
    # TODO: Implement search logic using query parameters
    return {"message": "Search successful", "query": q, "min": min_price, "max": max_price}


@router.get("/{product_id}", summary="Get product details")
async def get_product_details(product_id: int):
    # TODO: Fetch product by ID from the database
    return {"id": product_id, "name": f"Mock product #{product_id}"}
```

- **app/main.py**

```python
from fastapi import FastAPI

from app import users, products

app = FastAPI()
app.include_router(users.router)
app.include_router(products.router)

for route in app.routes:
    print(route.path, route.methods)

# /openapi.json {'GET', 'HEAD'}
# /docs {'GET', 'HEAD'}
# /docs/oauth2-redirect {'GET', 'HEAD'}
# /redoc {'GET', 'HEAD'}
# /users/v1/ {'GET'}
# /users/v1/ {'POST'}
# /users/v1/register {'POST'}
# /users/v1/{user_id} {'GET'}
# /products/v1/search {'GET'}
# /products/v1/{product_id} {'GET'}
```

- **Another way to start the server**

```python
if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```


![FastAPI Router](/assets/img/posts/Backend/FastAPI/FastAPIRouter.png)





<br>

---

🚀 Stay tuned for more insights into Python!