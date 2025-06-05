---
title: Reactive Programming in Java - WebClient
date: 2025-06-05
categories: [Backend, Java]
tags: [Java, Spring, Reactive, WebClient, RestTemplate]
author: kai
permalink: /posts/backend/java/reactive-programming-in-java-webclient/
---

# 🚀 Reactive Programming in Java - WebClient
In traditional Java applications using `RestTemplate`, HTTP calls are performed **synchronously**, meaning each request blocks the current thread until a response is received.

```java
// Synchronous HTTP request with RestTemplate
RestTemplate restTemplate = new RestTemplate();
ResponseEntity<String> response = restTemplate.getForEntity(url, String.class); // Blocks thread
```
- **Thread blocking:** Threads are tied up waiting for I/O
- **High resource consumption:** Each request occupies a thread
- **Poor scalability:** System degrades quickly under high concurrency
- **Not suitable for real-time streaming or event-driven architectures**


## ✅ WebClient: A Reactive HTTP Client in the Spring Ecosystem
As the demand for **high-concurrency**, **low-latency**, and **non-blocking communication** grows in modern microservice architectures, Spring’s `WebClient` has emerged as the go-to solution for reactive HTTP requests.

Unlike traditional clients like `RestTemplate`, `WebClient` is built for **asynchronous**, **event-driven**, and **stream-oriented** data flows—making it ideal for modern cloud-native applications.


### Core Features of WebClient

#### 1. Non-Blocking I/O  
`WebClient` is powered by **Reactor Netty**, enabling fully asynchronous I/O. This allows applications to scale efficiently without blocking threads during network calls.

#### 2. Functional, Fluent API  
Designed with **functional programming principles**, WebClient offers a clean, declarative style using **chained method calls**.

```java
WebClient.create()
         .get()
         .uri("https://api.example.com/data")
         .retrieve()
         .bodyToMono(String.class)
         .subscribe(System.out::println);
```

#### 3. Streaming Support
Built-in support for:
- **Server-Sent Events (SSE)**
- **Streaming JSON**
- **Chunked file transfer**

#### 4. Spring Ecosystem Integration
WebClient seamlessly integrates with Spring Boot:
- Auto-configuration via spring-boot-starter-webflux
- Built-in support for **metrics, tracing, and error handling**
- Easily injectable as a bean


### Common Use Cases
- Inter-microservice communication
- Calling third-party APIs asynchronously
- Reactive data aggregation from multiple services
- Streaming large files with minimal resource usage


## 🔁 WebClient vs. RestTemplate: Detailed Comparison

| Feature              | **WebClient**                          | **RestTemplate**                      |
|----------------------|----------------------------------------|----------------------------------------|
| **Programming Model** | Reactive (non-blocking)               | Synchronous (blocking)                |
| **Concurrency Handling** | Event Loop (reactor-based)         | Thread-per-request (thread pool)      |
| **Resource Consumption** | Low (fixed, minimal threads)       | High (scales with thread pool size)   |
| **Best Use Case**     | High-concurrency, low-latency systems | Traditional synchronous CRUD services |

> ✅ **Recommendation**: Use **WebClient** for new microservices or event-driven applications where scalability and efficiency are critical. Stick with **RestTemplate** only for legacy applications or quick synchronous operations.


## 🧪 Basic Syntax of WebClient
### Create WebClient Instance
- **baseUrl** sets the common root URL for all requests.
- **defaultHeader** can define headers like Accept, Authorization, etc.

```java
WebClient client = WebClient.builder()
    .baseUrl("https://api.example.com")
    .defaultHeader(HttpHeaders.ACCEPT, MediaType.APPLICATION_JSON_VALUE)
    .build();
```

### Perform a GET Request
- Makes a non-blocking GET request.
- Parses the response body into a Mono<User>.

```java
Mono<User> userMono = client.get()
    .uri("/users/{id}", 1)
    .retrieve()
    .bodyToMono(User.class);
```

### Send a POST Request
- Sends JSON payload (User object) using POST.
- Returns only the status and headers (no response body).

```java
Mono<ResponseEntity<Void>> response = client.post()
    .uri("/users")
    .bodyValue(new User("Alice", 30))
    .retrieve()
    .toBodilessEntity();
```

### Handle Streaming Responses with Subscription
- When working with **reactive streams** like `Flux`, you can subscribe to incoming data chunks using a `subscribe()` method. 
- **chunk ->:** Callback for each emitted item in the stream.
- **error ->:** Handles any exceptions during stream processing.
- **() ->:** Invoked when the stream completes successfully
- This pattern allows your application to **react to each event in real-time**, making it ideal for **live dashboards, market feeds, and log pipelines**.

```java
// WebClient
WebClient webClient = WebClient.builder()
                .baseUrl("http://localhost:8000")
                .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                .defaultHeader(HttpHeaders.ACCEPT, MediaType.TEXT_EVENT_STREAM_VALUE)
                .build();

// POST request
Flux<String> response = webClient.post()
            .uri("/api/document/stream")
            .bodyValue(requestBodyJson)
            .retrieve()
            .bodyToFlux(String.class);

// Handle the streaming response
response.subscribe(
    chunk -> {
        System.out.println("Received chunk: " + chunk);
    },
    error -> {
        System.out.println("Error: " + error.getMessage());
    },
    () -> {
        System.out.println("Completed");
    }
);

// Wait for the response to complete
response.blockLast(Duration.ofSeconds(60));
```





<br>




---

🚀 Stay tuned for more insights into LangChain and Agent!



