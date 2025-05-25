---
title: FastAPI + LLM - Building a Streaming AI-Powered Q&A Assistant
date: 2025-05-19
categories: [AI, LangChain]
tags: [LangChain, LLM, StreamingResponse, AI Assistant]
author: kai
---

# 🚀 FastAPI + LLM: Building a Streaming AI-Powered Q&A Assistant

How to build an **AI-powered Q&A assistant** that uses **FastAPI** and a **Large Language Model (LLM)** to generate concise, real-time answers for a given knowledge topic.


## 📌 Project Goals

- Develop a **FastAPI-based web service** that accepts a query and returns an AI-generated explanation.
- Utilize an LLM to **stream generated text**, allowing partial output to be sent to the client in real-time.
- Ensure **efficient memory usage** and a responsive user experience with FastAPI’s `StreamingResponse`.


## ⚙️ StreamingResponse?

FastAPI’s `StreamingResponse` is designed for **progressive data transmission** and is ideal for:

- **Long-running tasks**: e.g., real-time LLM generation, audio/video processing.
- **Large content delivery**: stream files or logs without memory overflow.
- **Real-time feedback**: enhance UX by pushing data as soon as it's ready.
- `from fastapi.responses import StreamingResponse`

```python
class StreamingResponse(Response):
    body_iterator: AsyncContentStream

    def __init__(
        self,
        content: ContentStream,
        status_code: int = 200,
        headers: typing.Mapping[str, str] | None = None,
        media_type: str | None = None,
        background: BackgroundTask | None = None,
    ) -> None:
        if isinstance(content, typing.AsyncIterable):
            self.body_iterator = content
        else:
            self.body_iterator = iterate_in_threadpool(content)
        self.status_code = status_code
        self.media_type = self.media_type if media_type is None else media_type
        self.background = background
        self.init_headers(headers)
```

###  Core Benefits

| Feature           | Description                                                                 |
|-------------------|-----------------------------------------------------------------------------|
| Streamed Output | Sends data chunk-by-chunk to the client as it’s generated.                  |
| Memory Efficient | Avoids loading the full content into memory, saving server resources.       |
| Real-time UX     | Enables immediate frontend rendering with partial LLM responses.            |
| Async Friendly   | Supports both synchronous and asynchronous generators for I/O-bound tasks. |


###  Key Parameters
To stream data, your route handler should return a `StreamingResponse` object and pass a generator or async generator as its content source.

| **Parameter**   | **Type**              | **Description**                                                                 |
|-----------------|-----------------------|---------------------------------------------------------------------------------|
| `content`       | generator / async     | A generator or async generator that yields bytes or string chunks to stream.   |
| `media_type`    | `str`                 | The MIME type of the response (e.g., `"text/event-stream"`, `"application/json"`). |
| `headers`       | `dict`                | Optional HTTP headers (e.g., `{"Content-Disposition": "attachment; filename=data.csv"}`). |
| `status_code`   | `int`                 | Custom HTTP status code to return (default is `200 OK`).                        |

### Code Example: Streaming AI Text Generation
- A client sends a GET request to `/ai_write?query=your_text`.
- `ai_writer.run_stream(query)` asynchronously yields text chunks.
- Each chunk is wrapped as a Server-Sent Event: data: {...}\n\n.
- The client receives content in real time.

#### `writer_app.py`
```python
import json
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import uvicorn

from ai_writer import AIWriter

app = FastAPI()
ai_writer = AIWriter()


async def ai_qa_stream_generator(query: str):
    """Yield AI-generated content in streaming chunks."""
    try:
        async for chunk in ai_writer.run_stream(query):
            json_data = json.dumps({"text": chunk})
            yield f"data: {json_data}\n\n"
    except Exception as e:
        error_msg = json.dumps({"error": str(e)})
        yield f"data: {error_msg}\n\n"


@app.get("/ai_write")
async def ai_writer_endpoint(query: str):
    """
    AI Writer Endpoint: streams LLM-generated text in real-time.
    """
    return StreamingResponse(
        ai_qa_stream_generator(query),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive"
        }
    )

# start the server
if __name__ == "__main__":
    uvicorn.run(app, port=8000)
```

#### `ai_write.py`

```python
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from typing import AsyncGenerator

os.environ["OPENAI_API_KEY"] = "*"  


# AIWriter Class Definition
class AIWriter:
    def __init__(self):
        # Initialize the language model
        self.llm = self.llm_model()

    # Define a method to return the custom language model
    def llm_model(self):
        # Define the LLM
        model = ChatOpenAI(
            model="gpt-4o-mini",
            temperature=0.7,
            stream=True, 
            openai_api_key=os.environ["OPENAI_API_KEY"]
        )
        return model


    # Streaming Prompt Execution
    async def run_stream(self, query: str) -> AsyncGenerator[str, None]:
        """Run AI question and return streaming response"""
        try:
            # Define a prompt template, request a summary of the document
            prompt_template = "Use 100 words to explain the following knowledge point or introduce:{concept}"
            prompt = ChatPromptTemplate.from_template(prompt_template)

            # Define the LLM chain, combining the custom language model and prompt
            chain = prompt | self.llm | StrOutputParser()

            # Use a streaming output
            async for chunk in chain.astream({"concept": query}):
                if isinstance(chunk, str):
                    yield chunk
                elif isinstance(chunk, dict) and "content" in chunk:
                    yield chunk["content"]
                else:
                    yield str(chunk)
        except Exception as e:
            yield f"Error: {str(e)}"

    async def chat(self, query: str):
        """Handle user message and return a streaming response"""
        try:
            async for chunk in self.run_stream(query):
                yield chunk
        except Exception as e:
            yield f"Error: {str(e)}"
```


## 🧪 How to test SSE (Server-Sent Events)
Tools like **Postman** or **Apifox** support built-in debugging for **Server-Sent Events (SSE)** — which is especially helpful when working with **streaming responses from LLMs**.

- Headers: `Accept: text/event-stream`





<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



