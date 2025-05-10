---
title: OSI & TCP/IP Network Layer Models
date: 2025-02-22
categories: [Architecture, Computer Networking]
tags: [computer networking, OSI & TCP/IP]
permalink: /basic-of-computer-networking/
author: kai
---

# 🌐 OSI & TCP/IP Network Layer Models

In modern networking, the **layered architecture** is the foundation that makes the internet scalable, modular, and maintainable. 

## 📚 OSI 7-Layer Model (Open Systems Interconnection)

| Layer                  | Description                               | Examples                          |
|------------------------|-------------------------------------------|-----------------------------------|
| 7. Application         | End-user services                         | HTTP, FTP, DNS                    |
| 6. Presentation        | Encoding, encryption                      | SSL, JPEG, ASCII                  |
| 5. Session             | Session management                        | NetBIOS, RPC                      |
| 4. Transport           | End-to-end connections                    | TCP, UDP                          |
| 3. Network             | Logical addressing & routing              | IP, ICMP                          |
| 2. Data Link           | MAC addressing & framing                  | Ethernet, PPP                     |
| 1. Physical            | Bit-level transmission                    | Fiber, RJ45, Wireless             |


## 📦 TCP/IP 4-Layer Model

| Layer             | Description                                                  | Examples |
|------------------|--------------------------------------------------------------|-------------------|
| 4. Application Layer | High-level protocols for communication between applications | HTTP, FTP, SMTP   |
| 3. Transport Layer   | Reliable/unreliable data transmission between hosts         | TCP, UDP          |
| 2. Internet Layer    | Routing and addressing across networks                      | IP, ICMP          |
| 1. Network Interface | Physical transmission of data (hardware & drivers)          | Ethernet, Wi-Fi   |


![Networking Models](/assets/img/posts/Architecture/ComputerNetworking/NetworkModel.png)


## 🤔 Why Do We Need Layering?

Imagine you're designing a network **without layering**:

- Application developers must write **raw binary** to interface directly with the NIC (Network Interface Card)
- Every app must implement:
  - Packet formatting
  - Connection handling
  - Reliability (ACKs, retries)
  - Routing logic

This is highly **inefficient and error-prone**.

### ✅ Layered Approach = Abstraction + Responsibility

> Think of it like **software architecture**: `Controller → Service → DAO`

| Analogy Component       | Network Role                  |
|--------------------------|-------------------------------|
| Controller               | Application Layer             |
| Service                  | Transport Layer               |
| DAO (Data Access Object) | Network & Physical Layers     |

Each layer follows **Single Responsibility Principle**, which aligns with:

- **High Cohesion**: Each layer focuses on its own task
- **Low Coupling**: Layers interact through defined interfaces, not internal details


## 🧱 Encapsulation: Like Russian Dolls

Each layer **wraps the previous one**, creating a **nested packet** structure:

![Nested Packet](/assets/img/posts/Architecture/ComputerNetworking/SimpleNetworkPacketFormat.png)

This **modular design** enables:

- Protocol independence
- Simplified debugging
- Easier maintenance and upgrades


## 🥷🏻 OSI vs. TCP/IP
Both models serve as frameworks for understanding how data flows across networks. While they differ in complexity and abstraction, **each has unique advantages**.


| Model       | Advantage                         | Use Case                       |
|-------------|-----------------------------------|--------------------------------|
| **OSI (7-layer)** | More granular, detailed design       | Deep network analysis, design |
| **TCP/IP (4-layer)** | Practical, widely adopted in real systems | Real-world internet protocols  |


### OSI 7-Layer Model: Theoretical Yet Precise

OSI (Open Systems Interconnection) splits the networking stack into **7 distinct layers**, offering clarity and isolation of responsibilities.

#### Strengths:
- Better conceptual understanding for learning & design
- Clear separation of concerns (Session, Presentation Layers)
- Suitable for explaining **application-aware load balancing**


### TCP/IP 4-Layer Model: Pragmatic and Widely Used

TCP/IP condenses the OSI layers into 4 broader categories, reflecting real-world protocol stacks.

#### Strengths:
- Easier to implement
- Better maps to actual internet protocols (IP, TCP, HTTP)
- Used in **SSL/TLS encryption**, **HTTPS**, and **DNS**


###  Real-World Examples

#### HTTPS = TCP/IP Model Extension

- In the **TCP/IP model**, HTTPS operates by inserting **SSL/TLS** between **HTTP** (Application Layer) and **TCP** (Transport Layer).
- This layered encryption works seamlessly thanks to TCP/IP’s simplicity.

```text
Application:   HTTP
    ↓
(Encryption:    TLS / SSL)
    ↓
Transport:     TCP
    ↓
Internet:      IP
    ↓
Network:       MAC
```

#### Load Balancing Comparison

1️⃣. **L4 Load Balancing Example: LVS (IP-Level Forwarding), F5**

- Layer 4 load balancing works at the **Transport Layer** of the OSI/TCP-IP model, primarily using **IP address** and **port number** to route traffic.
  - A **virtual IP + port** combination is exposed to the public.
  - Incoming requests are forwarded to **real backend servers** based on IP and port rules.
- It **does not inspect or parse** the application-layer protocol (e.g., HTTP, HTTPS).
- The load balancer **does not generate or modify actual traffic**.
- It acts purely as a dispatcher.


| Feature                     | Description |
|----------------------------|-------------|
| **High Performance**       | Because there's no protocol parsing, it handles high-throughput traffic efficiently. |
| **Low Resource Usage**     | Minimal memory and CPU consumption compared to Layer 7 load balancers. |
| **Protocol-Agnostic**      | Can handle any TCP/UDP-based protocols (e.g., HTTP, FTP, MySQL). |
| **Simple Dispatching Logic** | Only considers IP and port, which simplifies routing logic. |

```text
Client
↓
Virtual IP + Port (handled by LVS or F5)
↓
Forwarding Decision (based on L4 rules)
↓
Real Server (IP + Port)
```

2️⃣. **Layer 7 Load Balancing Example: HAProxy & Nginx**
- Layer 7 load balancing operates at the **Application Layer** of the OSI model. It can inspect the content of each HTTP/HTTPS request and make decisions based on application-level data such as URL, headers, cookies, and user-agent.
- Uses a **virtual URL or hostname** to receive client requests.
- **Parses the application-layer protocol** (e.g., HTTP, HTTPS).
- Makes intelligent routing decisions based on:
  - URL paths
  - Query parameters
  - Cookies
  - User-Agent strings

| Feature                     | Description |
|----------------------------|-------------|
| **Content-Based Routing**  | Routes traffic based on specific path, domain, or header values. |
| **Session Stickiness**     | Ensures requests from the same client are sent to the same backend server. |
| **SSL Termination**        | Can handle SSL decryption and forward plain HTTP internally. |
| **Health Checks**          | Monitors application-level health of backend services. |


**Use Cases:**
- Web applications needing routing based on different subdomains or paths  
  e.g., `/api/*` goes to service A, `/admin/*` goes to service B
- Microservice architectures
- Applications requiring **SSL offloading** and **application-level logging**

```text
Client
↓
Virtual URL / Domain (handled by Nginx or HAProxy)
↓
Routing Decision (based on L7 features)
↓
Selected Backend Server
```

⚠️ Note: Layer 7 load balancers consume more system resources but offer much more **flexibility and intelligence** in routing logic.


## 🧵 Network Layers & Protocols
A **network layered model** organizes how data is transmitted from one computer to another in the form of "layers" — just like how a package is processed from the sender to the receiver through various **delivery checkpoints**.


| **Role**   | **OSI Model (7 Layers)**  | **TCP/IP Model (4 Layers)**  | **Typical Network Protocols**   |
|------------|---------------------------|-------------------------------|--------------------------------|
| **Application (User Space)**| 7. Application Layer  | 4. Application Layer   | **HTTP**, FTP, NFS       |
|                             | 6. Presentation Layer | *(Part of Application)*      | Telnet, SNMP       |
|                             | 5. Session Layer      | *(Part of Application)*      | SMTP, **DNS**      |
| **Operating System (Kernel)**| 4. Transport Layer   | 3. Transport Layer           | **TCP, UDP**       |
|                             | 3. Network Layer      | 2. Internet Layer            | **IP**, ICMP, ARP, RARP |
| **Network Devices & Drivers**| 2. Data Link Layer   | 1. Network Access Layer      | PPP, Ethernet       |
|                             | 1.Physical Layer      | *(Part of Network Access Layer)* | IEEE 802.1A, IEEE 802.2 ~ IEEE 802.11 |



The **application layer** is the topmost layer in the network protocol stack and is **directly interfacing with the end user**.
> ❗️ It **is not** the application itself, but rather a **standardized interface** that supports communication between applications over the network.

The core goal of the application layer is to:
- **Ensure interoperability**: Allow different types of applications (e.g., email clients, web browsers, file transfer tools) to **communicate using common protocols**.
- **Bridge the gap**: Abstract lower-level protocols and provide a **uniform API** for application-level data exchange.


**Sending a Packet (Data Flow: Top to Bottom)**
```text
[Application Layer]    — Message -> msg: Hi
        ↓
[Transport Layer]      — Adds Port (Process Info) -> TCP Header: msg: Hi
        ↓
[Internet Layer]       — Adds IP (Host Info) -> IP Header: TCP Header: msg: Hi
        ↓
[Network Access Layer] — Converts to bits and sends via NIC (Network Interface Card ) -> Frame Header: IP Header: TCP Header: msg: Hi : Frame Trailer
```

**Receiving a Packet (Data Flow: Bottom to Top)**

```text
[Network Access Layer]  — Frame checking (MAC). Reads bits from wire -> Frame Header: IP Header: TCP Header: msg: Hi : Frame Trailer
        ↑
[Internet Layer]        — IP check (Host address) -> IP Header: TCP Header: msg: Hi 
        ↑
[Transport Layer]       — Port check (Process address) -> TCP Header: msg: Hi
        ↑
[Application Layer]     — Final message consumed by app -> msg: Hi
```

### Real-World Analogy: Sending Ice Cream to a Friend

🎄 It's Christmas. Kai (in PA) wants to send a box of ice cream to Jojo (in VA), who lives with Bing.

- 📦 **The box of ice cream** = Application data
- 🏢 **PA → VA logistics center** = Network Layer (IP-based routing between hosts)
- 👦 **Local delivery person (looks at name & phone)** = Transport Layer (TCP/UDP — targets correct process)
- 👩‍🦰 **Jojo in her shared apartment** = Application-level process

| Real-World Item         | Network Equivalent     |
|-------------------------|------------------------|
| House in PA             | A Host (IP address)     |
| 👩 Jojo / Bing (roommates) | Processes (Port numbers) |
| 📦 Ice cream box         | Application data       |
| 🚚 Logistics center       | Network (IP routing)    |
| 🧍 Local delivery guy     | Transport layer (TCP)   |


### Network Layer/Host vs. Transport Layer/Process 

| Layer         | What it Does      | Who Talks to Who?       | Address Used | Error Checking | Real-World Analogy         |
|---------------|-------------------|-------------------------|--------------|----------------|----------------------------|
| Network Layer | Finds the right host across the internet | Host-to-Host (device)     | IP Address   | Header only    | From PA to VA (main delivery hub) |
| Transport Layer | Finds the right process on the host | Process-to-Process (apps) | Port Number  | Header + Data  | Final delivery to Jojo, not Bing |

💡 Think of it this way:
- The **network layer** makes sure the package reaches the right **house**.
- The **transport layer** ensures the package goes to the right **roommate**.




<br>


---

🚀 Stay tuned for more insights into networking fundamentals!