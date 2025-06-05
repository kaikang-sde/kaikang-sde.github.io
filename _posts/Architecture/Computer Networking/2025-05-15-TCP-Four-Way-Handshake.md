---
title: TCP Four-Way Handshake
date: 2025-05-15
categories: [Architecture, Computer Networking]
tags: [Computer Networking, TCP, Four-Way Handshake]
author: kai
permalink: /posts/architecture/computer-networking/tcp-four-way-handshake/
---

# 🔚 TCP: Four-Way Handshake
In TCP, while connections are **established** using a **three-way handshake**, they are **terminated** using a **four-way handshake**. This ensures **graceful shutdown** and **bidirectional closure** of communication.

![TCP Four-Way Handshake](/assets/img/posts/Architecture/ComputerNetworking/TCPFourWayHandshake.png)

## 📤Four-Way Handshake
### Step 1: FIN from Client (Initiating Termination)
- **Client** sends a `FIN` (Finish) segment to the server, indicating **it has finished sending data**.
- The client then transitions to the `FIN_WAIT_1` state.


### Step 2: ACK from Server

- **Server** receives the `FIN` segment and responds with an `ACK` (Acknowledgment).
- The server transitions to the `CLOSE_WAIT` state.
- The client, upon receiving the `ACK`, moves to the `FIN_WAIT_2` state.


### Step 3: FIN from Server

- The server **finishes its remaining data transmission** and sends a `FIN` to the client.
- The server enters the `LAST_ACK` state, waiting for the client’s final acknowledgment.

### Step 4: Final ACK from Client

- Client sends a final `ACK` to acknowledge the server's `FIN`.
- Client enters the `TIME_WAIT` state for **2×MSL** (Maximum Segment Lifetime) to ensure the ACK is received and old duplicates are discarded.
- Server, upon receiving the `ACK`, transitions to `CLOSED`.
- After waiting, the client also transitions to `CLOSED`.


### Why TIME_WAIT?

- Ensures the last `ACK` was delivered in case it's lost and the server retransmits `FIN`.
- Prevents delayed packets from a prior connection from being misinterpreted in a new one.


## 🧠 Summary

| Phase         | Initiator | Description                                  | Client State   | Server State  |
|---------------|-----------|----------------------------------------------|----------------|----------------|
| 1st FIN       | Client    | Close client → server communication           | FIN_WAIT_1     | —              |
| ACK for FIN   | Server    | Acknowledge client FIN                        | FIN_WAIT_2     | CLOSE_WAIT     |
| 2nd FIN       | Server    | Close server → client communication           | TIME_WAIT      | LAST_ACK       |
| Final ACK     | Client    | Acknowledge server FIN                        | CLOSED (after TIME_WAIT) | CLOSED |


## 🤷🏻 Why Does TCP Wait for 2MSL During TIME_WAIT
When a TCP connection is closed (application-level closure request — not the final TCP state CLOSED.), the **client enters the `TIME_WAIT` state**, which lasts for **2×MSL (Maximum Segment Lifetime)**.

| Purpose                    | Benefit                              |
|----------------------------|--------------------------------------|
| Clear old segments         | Prevents interference in new connections |
| Ensure ACK delivery        | Ensures graceful shutdown            |
| Enforce uniqueness         | Avoids port reuse before safe        |


### What is MSL

- **MSL (Maximum Segment Lifetime)** is the **maximum time** a TCP segment is allowed to remain in the network before being discarded.
- It is defined to avoid **delayed or duplicate packets** causing confusion after a connection is closed.

> ⏳ RFC 793 specifies MSL as **2 minutes**, but in practice, many systems use **30 seconds, 60 seconds**, or **120 seconds**.

### Why Wait 2×MSL

Waiting **2×MSL** ensures:

#### 1. Old Packets Die Out
- TCP guarantees **no stray packets** from a previous connection will be mistaken for part of a new one.
    - No leftover or delayed packets from a previous connection are allowed to interfere with a new one.
- All segments in transit during the lifetime of the connection are guaranteed to be dropped after 2×MSL.

#### 2. Handle Retransmitted FINs
- If the client’s final **ACK** to the server’s **FIN** is **lost**, the server will **retransmit the FIN**.
    - If the step 4 is lost, the server will re do step 3.
- The client (in TIME_WAIT) must be ready to receive it and **resend the ACK**.
- Round-trip for such recovery = **2×MSL**


### Example
If MSL = 60s, then:

- TIME_WAIT duration = **2 × 60s = 120 seconds**
- During this time, the system **holds socket metadata** to respond if needed


## 🆚 CLOSE-WAIT vs. TIME-WAIT

### CLOSE-WAIT: Waiting to Close
**CLOSE-WAIT** is a **server-side state**, entered **after receiving a FIN (Finish)** segment from the client, indicating the client wishes to close the connection.

At this point:
- The server has **acknowledged** the client's request.
- The connection is **half-closed**: the client can’t send more data, but the server **can still send data**.
- The server may still need to **complete data transmission or cleanup** before initiating its own FIN to fully close the connection.

**Purpose:**
Ensures the server **has time to finish processing** and cleanly shut down.


### TIME-WAIT: Waiting for Safe Close

**TIME-WAIT** is a **client-side state**, entered **after sending the final ACK** in the **fourth step** of TCP's four-way handshake for connection termination.

This state **only applies to the party that initiates the connection closure** (typically the client, but not always - Only the side that actively closes the connection will enter the TIME_WAIT state.).

The connection remains in TIME-WAIT for **2×MSL (Maximum Segment Lifetime)** to:

1. **Prevent old duplicate packets** from being mistaken as part of a new connection using the same IP and port pair.
2. **Ensure reliable connection closure** — in case the final ACK is lost, the server may retransmit its FIN, which the client must still be able to acknowledge.


### Real-World Implication: Why Servers Show Many TIME-WAITs

In HTTP/1.1, if the `Connection: close` header is used, **servers often initiate the shutdown**.

That means:
- The server becomes the **active closer**, not the client.
- Thus, the **server enters TIME-WAIT**, which may cause **many TIME-WAIT sockets to accumulate** under high load.


### What Happens Without TIME-WAIT?

If a system were to **immediately close the socket** after sending the final ACK, potential issues include:

- **Duplicate FINs from the server** (due to lost ACKs) would be answered with a **RST**, signaling an error.
- A **new connection reusing the same port** may incorrectly receive delayed packets from the previous session — violating TCP’s reliability guarantees.


### What’s the Problem with TIME-WAIT?

Too many sockets in the `TIME-WAIT` state can cause:

- **File descriptor exhaustion**  
  Every socket uses a file descriptor; a large number of sockets may exhaust system limits.

- **Port exhaustion**  
  TCP ports range from `0–65535`. If most ephemeral ports are stuck in `TIME-WAIT`, **new connections may fail** due to lack of available ports.

- **Memory & CPU overhead**  
  Each TIME-WAIT socket requires system resources for state tracking and timeout management.


#### Typical in Short-Lived TCP Services

In **high-throughput servers using short connections** (e.g., HTTP APIs, load balancers), each request may result in a full TCP connection. If the **server actively closes the connection**, it enters `TIME-WAIT`, potentially creating thousands of such sockets.

```bash
# View number of TIME-WAIT connections
netstat -an | grep TIME_WAIT | wc -l
```

#### How to Mitigate TIME-WAIT Accumulation

1. **Use Long-Lived Connections (Keep-Alive)**: Instead of creating a new connection per request, use persistent connections:
- Set HTTP header: Connection: keep-alive
- Modern HTTP/1.1+ supports this by default
- Reduces the number of TCP handshakes and teardowns

2. **Enable Port Reuse**: Allow TIME-WAIT sockets to be reused, especially when the source IP/port pair is consistent
- `sysctl -w net.ipv4.tcp_tw_reuse=1` Enable TCP port reuse in TIME-WAIT 
- Use with caution in production. Misuse may violate TCP reliability guarantees.

3. **Shorten TIME-WAIT Duration**: Reduce the TIME-WAIT duration from its default 2×MSL (often 60s or more) to a smaller interval
- `sysctl -w net.ipv4.tcp_fin_timeout=30` Set TIME-WAIT duration to 1 MSL (~30 seconds)
- This only affects how long the OS keeps closed connections in TIME-WAIT.


##### Short Connections vs. Long-Lived Connections

| Feature                        | Short Connections                             | Long-Lived Connections (Keep-Alive)           |
|-------------------------------|-----------------------------------------------|------------------------------------------------|
| **Definition**                 | A connection is created and closed per request | A connection remains open for multiple requests |
| **Connection Overhead**       | High — each request requires a new TCP handshake and teardown | Low — connection is reused, reducing handshakes |
| **Latency**                   | Higher — due to frequent connection setup      | Lower — avoids handshake latency                |
| **Server Load**               | Higher — more resources used for socket creation and cleanup | Lower — fewer sockets opened/closed            |
| **TIME-WAIT Issues**          | Common — each close may trigger TIME-WAIT      | Less frequent — fewer disconnects               |
| **Throughput**                | Lower under high concurrency                   | Higher efficiency in handling frequent requests |
| **HTTP Usage**                | HTTP/1.0 default behavior                      | HTTP/1.1+ defaults to keep-alive                |
| **Suitability**               | Simple/occasional requests (e.g. RESTful APIs with low volume) | High-performance APIs, streaming, real-time apps |
| **Example Scenario**          | REST call per image upload                     | WebSocket chat app, AI API inference server     |


## 👀 Why TCP Connection Uses Three-Way Handshake and Four-Way Termination

### Connection Establishment: Three-Way Handshake
The **three-way handshake** ensures both parties are ready to communicate and agree on initial sequence numbers.

1. **SYN**: The client sends a synchronization (SYN) packet to the server with an initial sequence number.
2. **SYN-ACK**: The server acknowledges the SYN and responds with its own SYN and sequence number.
3. **ACK**: The client acknowledges the server’s SYN.

> ✅ Both sides confirm the readiness to send and receive data.

**Why 3 steps?**  
In the second step, the server can combine both ACK and SYN in a single packet. This makes the process efficient and secure.

### Connection Termination: Four-Way Handshake

The termination of a connection involves **four separate steps** to allow both sides to complete their data transfer independently.

1. **FIN (Client → Server)**: The client finishes sending data and sends a FIN request.
2. **ACK (Server → Client)**: The server acknowledges the client’s FIN — but it might still have data to send.
3. **FIN (Server → Client)**: Once the server finishes its data transmission, it sends its own FIN.
4. **ACK (Client → Server)**: The client acknowledges the server’s FIN and enters `TIME_WAIT`.

> 🔄 Each FIN and ACK are sent independently to ensure **full-duplex** communication is properly closed.


### Why Can't We Use Three-Way Termination?

Unlike the handshake, FIN and ACK **cannot be combined safely**. After receiving a FIN, a party must:
- Acknowledge the FIN (via ACK),
- And only after completing its own data transfer, **initiate** its own FIN.

This separation allows **asymmetric closing**, where one side may still need to send data even after the other finishes.








<br>


---

🚀 Stay tuned for more insights into networking fundamentals!