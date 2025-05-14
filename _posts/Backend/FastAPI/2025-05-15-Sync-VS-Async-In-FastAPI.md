---
title: Sync vs Async in FastAPI
date: 2025-05-14
order: 2
categories: [Backend, FastAPI]
tags: [FastAPI, AsyncIO, Python, Backend Development]
author: kai
---

# 🚀 Sync vs Async in FastAPI
Understanding the difference between **synchronous** and **asynchronous** code can be tricky. So let’s simplify it using a real-life analogy: **ordering bubble tea**!

## 🤝 Synchronous (Sync) – Traditional Bubble Tea Shop

### Scenario

You walk into a traditional milk tea shop:

1. **You:** "I'd like a pearl milk tea."
2. **Clerk:** Starts making it (takes 5 minutes).
3. **You:** Wait and do nothing.
4. **Clerk:** "Here you go!"
5. **Next customer:** Still waiting in line.

### Key Characteristics

- Tasks are processed **one at a time**.
- Each operation **blocks** the next.
- If something takes 5 seconds, nothing else happens during that time.


## 👏🏻 Asynchronous (Async) – Smart Bubble Tea Shop

### Scenario

You visit a smart, self-service milk tea shop:

1. **You:** Place an order via app.
2. **System:** Accepts the order and assigns a number.
3. **You:** Sit and scroll your phone.
4. **Clerk:** Makes multiple drinks in parallel.
5. **System:** Notifies you when your drink is ready.

### Key Characteristics

- Tasks can run **concurrently**.
- While waiting, other things **keep happening**.
- Ideal for I/O-bound operations (like waiting for a database, API, etc.)


## 🔁 Sync vs Async in FastAPI: Key Differences with Code
For I/O-heavy APIs (like external HTTP requests), asynchronous FastAPI endpoints drastically reduce response time and improve throughput — a must-have for production-grade performance.

### Synchronous Version (Slower, Blocking)
In a synchronous function, **each request waits for the previous one to finish** — even if they are independent.

```python
import requests
from fastapi import FastAPI

app = FastAPI()

@app.get("/sync-news")
def get_news_sync():
    # Sequential calls: total time = 2s + 2s = 4s
    news1 = requests.get("https://api1.com").json()  # ⏳ 2s
    news2 = requests.get("https://api2.com").json()  # ⏳ 2s
    return {"total_time": 4}
```

### Asynchronous Version (Faster, Concurrent)
With httpx.AsyncClient and asyncio.gather, the two requests `run in parallel`.

```python
import httpx
import asyncio
import time
from fastapi import FastAPI

app = FastAPI()

@app.get("/async-news")
async def get_news_async():
    async with httpx.AsyncClient() as client:
        start = time.time()
        task1 = client.get("https://api1.com")  # 👈 Starts both
        task2 = client.get("https://api2.com")
        res1, res2 = await asyncio.gather(task1, task2)  # 👈 Awaits both
        elapsed = time.time() - start
        return {
            "total_time": round(elapsed, 2),  # ≈ 2 seconds (not 4!)
            "news1": res1.json(),
            "news2": res2.json()
        }
```

## ⚙️ Async Programming in Python with `asyncio`
Python’s built-in `asyncio` library is the foundation for **asynchronous programming**, enabling high-concurrency applications with minimal thread usage.

The **core concepts** of `asyncio`, including **coroutines**, **event loops**, **tasks**, and **futures**.


### Why Async?

Traditional (synchronous) code blocks the program while waiting for I/O — e.g., network, disk, or database. This **limits throughput**.

With `asyncio`, you can **suspend** operations using `await`, freeing the event loop to do other work.

> _Ideal for I/O-bound scenarios like web services, crawlers, bots, and API clients._


### 1. Coroutine
A coroutine is a special function defined using `async def`. It returns a coroutine object and doesn't execute until `await`ed or scheduled.

When a coroutine encounters `await some_async_operation()`, two things happen:

1. The **coroutine is suspended** — it *pauses execution* at that line.
2. The **event loop regains control** and is free to run **other tasks**.

Once the awaited operation is finished (e.g., HTTP response arrives, file read completes), the coroutine is **resumed right where it left off**.


```python
import asyncio

async def my_coroutine():
    await asyncio.sleep(1)
    print("Done!")
```

### 2. Event Loop
In Python's asynchronous programming model, the **event loop** is the **orchestrator** of everything. It runs your coroutines, handles I/O events, manages timers, and makes sure tasks get scheduled and executed.

If you're writing `async` code in Python, you're working **with** (and **inside**) the event loop — even if you don’t see it directly.

The **event loop** is:

- A continuously running process
- That **watches for completed I/O operations**
- And **dispatches control** back to the appropriate coroutine

It’s the core engine behind libraries like `asyncio`, `FastAPI`, `httpx`, and more.
```python
import asyncio

async def say_hello():
    print("Hello...")
    await asyncio.sleep(1)  # wait 1 second -> During the pause, the event loop can run other coroutines.
    print("...World!")

# Start the event loop and run coroutine
asyncio.run(say_hello())
```

#### Under the Hood
- When you call: `asyncio.run(say_hello())`
- **It does the following:**
    1.	Creates an event loop
	2.	Wraps your coroutine as a Task
	3.	Runs the event loop until the Task finishes
	4.	Cleans up the loop
- You could also manually manage the loop: (**use asyncio.run() in most cases.**)
    ```python
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    loop.run_until_complete(my_coroutine())
    loop.close()
    ```

#### Event Loop FLow

```text
Coroutine A starts
↓
await async_task() → yields control
↓
Event loop picks Coroutine B
↓
B does work → await other_task()
↓
A's task completes → resumed by event loop
```

### 3. Task
In asynchronous Python programming, a **Task** is a wrapper around a coroutine that lets it run **concurrently** within the event loop.

Unlike a plain coroutine, a Task is **scheduled for execution** immediately and can run alongside other tasks — perfect for high-performance I/O operations.

- A **Task** is created from a coroutine using `asyncio.create_task()`
- It is then **scheduled** by the event loop
- The task will run **asynchronously**, allowing other tasks to run concurrently

```python
import asyncio

async def my_coroutine():
    print("Start working...")
    await asyncio.sleep(2)
    print("Finished!")

async def main():
    # Create a task from the coroutine
    task = asyncio.create_task(my_coroutine())

    print("Task started... doing other work")
    await task
    print("Task finished")

asyncio.run(main())

# Task started... doing other work
# Start working...
# (wait 2 seconds)
# Finished!
# Task finished
```

- The event loop schedules my_coroutine() as a Task
- Meanwhile, main() continues execution until await task is called


### 4. Future
In Python’s `asyncio` framework, a **Future** represents a placeholder for a result that **doesn’t exist yet**, but will be available in the future. It's the low-level building block for managing asynchronous operations.

A `Future` object acts as a container for the eventual result of an asynchronous operation.

- It's **not a coroutine**.
- It is **used internally by the event loop** to track tasks.
- You don’t `await` a `Future` directly in most application code.
- Instead, it’s usually **wrapped by a `Task`**, which is a higher-level construct.

```python
import asyncio

async def set_future(fut):
    await asyncio.sleep(1)
    fut.set_result("Future is done!")

async def main():
    loop = asyncio.get_running_loop()
    fut = loop.create_future() # creates an empty Future.

    asyncio.create_task(set_future(fut))
    result = await fut
    print(result)

asyncio.run(main())
```

- In **most real-world scenarios**, developers use Task instead of manually dealing with Future.
- A Task is a subclass of Future that wraps coroutines and integrates with the event loop automatically.


## 🌐 Modern HTTP Requests in Python with `httpx`
In modern Python web development, efficient HTTP communication is key. [`httpx`](https://www.python-httpx.org/) is a **powerful, modern alternative to `requests`**, supporting both **sync** and **async** usage out of the box — with built-in **HTTP/2**, timeout management, streaming, and connection pooling.

- **Installation**: `pip install httpx`

### Why Use `httpx`?

| Feature               | Description                                       |
|-----------------------|---------------------------------------------------|
| Sync & Async        | Unified API for both synchronous and asynchronous use |
| HTTP/1.1 & HTTP/2   | Supports modern HTTP protocols                    |
| Secure & Fast       | Built-in timeout, retry, and connection pool support |
| Testing             | Native test client with mock server support      |


### Synchronous Request Example
-  Simple syntax, just like requests, but with better performance and modern support.

```python
import httpx

response = httpx.get("https://httpbin.org/get")
print(response.status_code)
print(response.json())
```

### Asynchronous Request Example
- Use httpx.AsyncClient() to perform fully non-blocking HTTP operations in async workflows.

```python
import httpx
import asyncio

async def fetch_data():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://httpbin.org/get")
        print(response.json())

asyncio.run(fetch_data())
```

### Recommended: Use a Client Instance
Using **a client instance** provides:
- Connection reuse (better performance)
- Timeout, retry, and base URL management

- **Synchronous Client:**
    ```python
    with httpx.Client() as client:
        response = client.get("https://httpbin.org/get")
        print(response.json())
    ```

- **Asynchronous Client:**
    ```python
    async with httpx.AsyncClient() as client:
        response = await client.get("https://httpbin.org/get")
        print(response.json())
    ```










<br>

---

🚀 Stay tuned for more insights into Python!