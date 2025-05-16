---
title: FastAPI - A Modern Web Framework
date: 2025-05-14
categories: [Backend, FastAPI]
tags: [FastAPI, Python, Web Development, Async, API Design, HTTP]
author: kai
---

# 🚀 FastAPI: A Modern Web Framework
In modern backend development, traditional Python web frameworks such as **Flask** and **Django** pose several challenges:

- Limited support for asynchronous programming (slower I/O-bound performance).
- Manual API documentation is **time-consuming** and **prone to becoming outdated**.
- Weak built-in data validation can lead to **runtime bugs** and **security vulnerabilities**.

## 🔍 What is FastAPI?

[**FastAPI**](https://fastapi.tiangolo.com/) is a **modern, high-performance web framework** for building APIs with Python **3.8+**. It leverages:

- **Starlette** for the web server layer (async support).
- **Pydantic** for robust data validation and parsing.

> Designed for speed, type safety, and developer productivity.


## 🧩 Key Features

| Feature           | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| **High Performance**  | Built on Starlette (async) — handles I/O-bound workloads efficiently.         |
| **Auto Docs**       | Swagger UI & Redoc support built-in — no extra configuration needed.       |
| **Strong Typing**    | Python type hints + Pydantic models = fewer bugs & better maintainability. |
| **Validation First** | Automatic request/response validation — catch bugs early.                   |
| **Developer Friendly** | Simple syntax, fewer lines of code, better editor support (e.g., autocomplete). |


## ✅ When to Use

- Building **RESTful APIs** or **LLM-based services**.
- Developing **microservices** where strong validation and speed matter.

> **FastAPI = Performance + Type Safety + Auto Documentation**

### Pros

- **High development velocity** — reduce code size by **40%+**.
- **Fewer runtime errors** thanks to type-safe interfaces.
- **Self-updating API documentation** for better collaboration.

### Cons

- **Ecosystem is newer** — fewer third-party plugins vs Django.
- Requires understanding **async/await** patterns for full performance benefits.


## 🔗 What is Uvicorn?

**Uvicorn** is a **lightweight, high-performance ASGI server** used to run asynchronous Python web applications such as:

- [FastAPI](https://fastapi.tiangolo.com/)
- [Starlette](https://www.starlette.io/)
- [Django Channels](https://channels.readthedocs.io/)

Think of Uvicorn as the **Tomcat of the Python ASGI ecosystem** — it bridges HTTP protocols with your Python async app.

### Core Features

| Feature             | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| **Async Handling**    | Built on `asyncio`, supports native `async/await` — ideal for high concurrency. |
| **High Performance**  | Powered by **`uvloop`** and **`httptools`** (written in C), delivering ultra-fast performance. |
| **Protocol Support**  | Supports **HTTP/1.1**, **WebSockets**, and optional **HTTP/2** (with extra config). |
| **Auto Reloading**   | Use `--reload` in dev mode to restart server on code changes. |
| **ASGI Compliant**    | Fully supports the **ASGI standard**, enabling real-time features like WebSockets. |



### Relationship with FastAPI

FastAPI itself is **just a web framework**, not a web server.

To make your FastAPI app accessible over HTTP, you need a **server like Uvicorn** to run it.

#### Typical Workflow

```text
Client Request → Uvicorn (ASGI Server) → FastAPI App → Response → Uvicorn → Client
```

- Uvicorn listens for incoming requests.
- It parses and forwards those requests to your FastAPI app via ASGI.
- FastAPI processes the request and returns a response.
-	Uvicorn then sends the response back to the client.



## 📊 Comparative Overview of Modern Web Frameworks

Choosing the right web framework is essential for your project's **performance**, **scalability**, and **development speed**. Below is a side-by-side comparison of popular web frameworks across languages.

| Framework    | Language   | Key Features                                                   | Best Use Cases                            |
|--------------|------------|------------------------------------------------------------------|-------------------------------------------|
| **Flask**     | Python     | Lightweight, flexible, minimal by design; synchronous only      | Small projects, quick prototypes          |
| **Django**    | Python     | Batteries-included (ORM, Admin, Auth); powerful but heavy       | Full-stack web apps, CMS, enterprise apps |
| **Express.js**| Node.js    | Minimalist, high concurrency; weak typing can cause runtime bugs| APIs in JavaScript ecosystem              |
| **Spring Boot** | Java    | Enterprise-grade, modular, secure; verbose and slower startup    | Large-scale enterprise systems            |
| **FastAPI**   | Python     | Async-first, auto docs, type-safe, ultra-fast via Starlette      | High-performance APIs, microservices      |


## ⚙️ FastAPI Environment Setup

### Prerequisites

- Python 3.8 or higher
- `pip` for package management
- [FastAPI Installation](https://fastapi.tiangolo.com/#installation) 

### Installation Steps
1. Create a Virtual Environment (Recommended)
2. Install FastAPI: `pip install "fastapi[standard]"`
3. Install Uvicorn (ASGI Server): `pip install "uvicorn[standard]"`

### Quick Test
- **Step 1**: main.py

    ```python
    from typing import Union
    from fastapi import FastAPI

    app = FastAPI()

    @app.get("/")
    def read_root():
        return {"message": "FastAPI is working!"}

    #  Accessing Path and Query Parameters
    # http://127.0.0.1:8000/items/5?q=kai → Returns {"item_id": 5, "q": "kai"}
    @app.get("/items/{item_id}")
    def read_item(item_id: int, q: Union[str, None] = None):
        return {"item_id": item_id, "q": q}
    ```
- **Step 2: Run the server**: 
    - Basic Syntax: `uvicorn main:app --reload`
    - Start with Custom Port: `uvicorn main:app --port 8002 --reload`
    - Alternative: FastAPI Dev Command: `fastapi dev main.py`

- **Step 3:** After running uvicorn, visit:
    - Swagger UI: http://127.0.0.1:8000/docs
	- ReDoc: http://127.0.0.1:8000/redoc


## 🤖 Basic Routing with HTTP Methods
In FastAPI, defining routes and associating them with HTTP methods like `GET`, `POST`, `PUT`, and `DELETE` is simple and intuitive. Each route handler corresponds to a specific operation in a RESTful API.


```python
from fastapi import FastAPI

app = FastAPI()

# READ: Get an item by ID
@app.get("/items/{item_id}")
def read_item(item_id: int):  # FastAPI auto-validates item_id as int
    return {"item_id": item_id}


# CREATE: Add a new item
@app.post("/items/")
def create_item(item: dict):
    return {"item": item}


# UPDATE: Modify an existing item
@app.put("/items/{item_id}")
def update_item(item_id: int, item: dict):
    return {"item_id": item_id, "updated_item": item}


# DELETE: Remove an item by ID
@app.delete("/items/{item_id}")
def delete_item(item_id: int):
    return {"status": "deleted", "item_id": item_id}
```

## 📬 Accessing HTTP Headers
FastAPI makes it easy to extract values from incoming request headers using the `Header` utility.

This is particularly useful when handling **authentication tokens**, **custom client metadata**, or any data passed via HTTP headers.

```python
from fastapi import FastAPI, Header

app = FastAPI()

@app.get("/header")
def read_item(item_id: int, token: str = Header(default="token")):
    return {
        "token": token,
        "item_id": item_id
    }
```

- Header names are case-insensitive, but FastAPI converts underscores (_) to dashes (-) by default.
- To prevent this behavior, use convert_underscores=False:
    -  `token: str = Header(..., convert_underscores=False)`

### Retrieve All Request Headers
- uses the **Request object** to access all HTTP headers.

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.get("/all-headers/")
def get_all_headers(request: Request):
    return dict(request.headers)

# {
#   "host": "127.0.0.1:8000",
#   "connection": "keep-alive",
#   "sec-ch-ua-platform": "\"macOS\"",
#   "user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36",
#   "accept": "application/json",
#   "sec-ch-ua": "\"Chromium\";v=\"136\", \"Google Chrome\";v=\"136\", \"Not.A/Brand\";v=\"99\"",
#   "sec-ch-ua-mobile": "?0",
#   "sec-fetch-site": "same-origin",
#   "sec-fetch-mode": "cors",
#   "sec-fetch-dest": "empty",
#   "referer": "http://127.0.0.1:8000/docs",
#   "accept-encoding": "gzip, deflate, br, zstd",
#   "accept-language": "en-US,en;q=0.9"
#  }
```

### Set Custom Response Headers
In many scenarios, you'll want to return **custom headers** with your FastAPI response—for example, to manage caching, security metadata, or custom identifiers.

FastAPI provides built-in support via `JSONResponse`.

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()


@app.get("/custom-header/")
def set_custom_header():
    content = {"message": "Hello World"}
    headers = {
        "X-Custom-Header": "my_value kai.com",
        "Cache-Control": "max-age=3600"
    }
    return JSONResponse(content=content, headers=headers)

# Response headers
#  cache-control: max-age=3600 
#  content-length: 25 
#  content-type: application/json 
#  date: Wed,14 May 2025 11:23:02 GMT 
#  server: uvicorn 
#  x-custom-header: my_value kai.com 
```

- For more complex response customization (status code, content type, etc.), JSONResponse and Response give you full control.
- Use consistent naming (e.g. X-Custom-*) for custom headers to avoid collision with standard headers.


## 🚦 Custom Response Status Codes
In many RESTful applications, we want to return different **HTTP status codes** based on the result of the operation. FastAPI gives you flexible ways to define both **default** and **dynamic** status codes for your API endpoints.

```python
from fastapi import FastAPI, status, Response

app = FastAPI()

@app.get("/status_code", status_code=200)
def create_item(name: str):
    if name == "Foo":
        # Explicitly return a 404 Not Found response
        return Response(status_code=status.HTTP_404_NOT_FOUND)
    return {"name": name}
```

- The @app.get(..., status_code=200) decorator sets the default status code.
- Inside the function, we override that code conditionally using `Response(...)`.


## 📘 Common HTTP Status Codes in FastAPI


| Code | Constant                                 | Meaning                     |
|------|------------------------------------------|-----------------------------|
| 200  | `status.HTTP_200_OK`                     | ✅ Successful request        |
| 201  | `status.HTTP_201_CREATED`                | 📦 Resource created         |
| 204  | `status.HTTP_204_NO_CONTENT`             | 🔇 No response body         |
| 400  | `status.HTTP_400_BAD_REQUEST`            | ⚠️ Client input error       |
| 401  | `status.HTTP_401_UNAUTHORIZED`           | 🔒 Missing auth credentials |
| 403  | `status.HTTP_403_FORBIDDEN`              | 🚫 Permission denied        |
| 404  | `status.HTTP_404_NOT_FOUND`              | 🔍 Resource not found       |
| 500  | `status.HTTP_500_INTERNAL_SERVER_ERROR`  | 💥 Server error             |






<br>

---

🚀 Stay tuned for more insights into Python!