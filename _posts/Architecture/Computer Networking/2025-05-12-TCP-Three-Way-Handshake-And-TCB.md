---
title: TCP Three-Way Handshake and TCB
date: 2025-05-12
categories: [Architecture, Computer Networking]
tags: [Computer Networking, TCP, Transport Layer, Three-Way Handshake]
author: kai
---

# 🔍 TCP Three-Way Handshake
TCP (Transmission Control Protocol) is a **connection-oriented** protocol designed to ensure **reliable** and **ordered** data transmission. One of its most critical mechanisms is the **Three-Way Handshake**, used to establish a connection between two endpoints before data transfer begins.

![TCP Structure](/assets/img/posts/Architecture/ComputerNetworking/TCPStructure.png)


## ⚙️ Three-Way Handshake
The **TCP Three-Way Handshake** is a process involving three steps (SYN → SYN-ACK → ACK) that initializes a TCP connection.

![TCP Three-Way Handshake](/assets/img/posts/Architecture/ComputerNetworking/TCPThreeWayHandshake.png)

### First Handshake: Client Sends SYN
- The client sends a **SYN (Synchronize)** packet to the server to initiate a connection.
- This packet includes:
  - `SYN = 1` 
  - A **random Initial Sequence Number (ISN)**: `seq = x`
- Client transitions to the **SYN_SENT** state.
- `Client → Server : SYN = 1, seq = x`

### Second Handshake: Server Replies with SYN-ACK
- The server receives the SYN packet, allocates resources, and responds with a **SYN-ACK** packet.
- This packet includes:
    - SYN = 1, ACK = 1
    - seq = y (server’s ISN - a radom initial sequence number)
	- ack = x + 1 (to acknowledge client’s SYN)
	- Server transitions to the **SYN_RECEIVED** state.
- `Server → Client : SYN = 1, ACK = 1, seq = y, ack = x + 1`

### Third Handshake: Client Sends ACK
- The client receives the SYN-ACK, and replies with an **ACK** packet:
	- ACK = 1
	- seq = x + 1
	- ack = y + 1
	- Client transitions to **ESTABLISHED** state.
	- Server also enters **ESTABLISHED** state upon receiving this ACK.
- `Client → Server : ACK = 1, seq = x + 1, ack = y + 1`

### State Transitions
- **Client**: CLOSED -> SYN_SENT -> ESTABLISHED
- **Server**: CLOSED -> LISTEN-> SYN_RECEIVED -> ESTABLISHED


### Data Transfer Note
- **First SYN packet cannot carry data** (for security reasons).
- **Third ACK can optionally carry data**, though this depends on the implementation.

### Key TCP Flags
1. **ACK (Acknowledgment)**
    - Value: `1` if the acknowledgment number is valid.
    - Meaning: "I have received the previous segment(s), and here's the next expected byte."
    - Often seen in every TCP packet **after the initial handshake**.
2. **SYN (Synchronize)**
    - Value: `1` if the packet is used to initiate a connection.
    - Purpose: Used during **connection setup**, specifically in the **three-way handshake**.
    - Carries the **Initial Sequence Number (ISN)**.
3. **Sequence Number (seq)**
    - Indicates the **first byte of data** in this segment.
    - During the handshake, it holds the ISN.
    - Used by the receiver to reassemble packets in the correct order.
4. **Acknowledgment Number (ack)**
    - Indicates the **next byte the sender expects to receive**.
    - If `ACK=1`, then this field is valid.
    - For example, if `ack=1001`, it means the receiver has successfully received bytes 1–1000.


## 🧠 TCP Control Block (TCB): Connection State Management
TCP (Transmission Control Protocol) is a **stateful protocol**, meaning it **maintains connection information** throughout its lifetime. Unlike UDP, which is connectionless, **each TCP connection maintains context and state** using a data structure known as the **Transmission Control Block (TCB)**.

- Each connection is **uniquely identified** by a **4-tuple**:
  - `Source IP address`
  - `Source port`
  - `Destination IP address`
  - `Destination port`

- For every incoming TCP packet, the system uses this 4-tuple to **lookup the corresponding TCB** from a container (e.g., a hash map or list) to determine the connection's context.
- Each TCB typically consumes at least 280 bytes.


### TCB Lifecycle Step-by-Step
How a TCP server manages the **Transmission Control Block (TCB)** during the connection setup process?

1. **Server is listening for new connections**  
   → Connection state: `LISTEN`

2. **Receives a SYN packet from the client**  
   → A new **TCB** is created and initialized

3. **Server responds with a SYN+ACK packet**  
   → Connection state changes to `SYN_RECEIVED`

4. **The new TCB is placed in the "half-open connection queue"**  
   → This is also known as the **SYN backlog queue**

5. **Client receives SYN+ACK and sends ACK back to complete the handshake**

6. **Server receives the ACK from the client**  
   → The connection is now fully established

7. **The TCB is removed from the half-open queue**  
   → And added to the **fully-established connection queue**

8. **Connection state changes to `ESTABLISHED`**  
   → Data transfer can now begin

> This is the server-side state transition flow that handles a new incoming TCP connection using the TCB data structure.


```text
[LISTEN]
   ↓ recv SYN
[Create TCB]
   ↓ send SYN+ACK
[SYN_RECEIVED]
   ↓ queue TCB (half-open)
   ↓ recv ACK
[ESTABLISHED]
   ↓ move TCB to full-connection queue
```


## 🤔 Why TCP Requires Three-Way Handshake (Not Two or Four)

### What Happens in a Two-Way Handshake?

Let’s explore a simplified **two-way handshake scenario** where both sides exchange initial sequence numbers:

1. **Step 1 – SYN from Client:**  
   The client sends a SYN packet to the server, containing its initial sequence number, e.g., `seq = x`.

2. **Step 2 – SYN+ACK from Server:**  
   The server responds with a SYN+ACK packet. This acknowledges the client’s sequence number with `ack = x+1` and also includes its own starting sequence number, `seq = y`.


### Case 1: Sequence Number Synchronization Incomplete

- **Step 1:** The **client** sends a **SYN packet** to the server with a sequence number: `SYN, seq = x`
- **Step 2:** The **server** receives this and responds with: `SYN-ACK, ack = x + 1, seq = y`
- **Problem:**
   - The server confirms the client's sequence number, but the **client never confirms the server's**.
   - If the server's SYN-ACK packet gets **lost** during transmission, the client:
      - Doesn’t know the server's initial sequence number.
      - Can’t establish a reliable connection.
      - May continue sending data incorrectly.

- **Solution with Third Handshake**
   - The **third message (ACK)** from client to server:
      - Confirms receipt of the server’s sequence number `y`.
      - Ensures **mutual agreement** on sequence tracking.


### Case 2: Stale Connection Requests
![Stale Connection Requests](/assets/img/posts/Architecture/ComputerNetworking/TwoWayHandshakeProblem2.png)

Imagine the following:

1. The **client** sends a **SYN** but due to **network congestion**, it gets **delayed**.
2. The client **times out** and sends a second **SYN**.
3. The second SYN succeeds; the connection is established and eventually closed.

#### Problem
The **first, delayed SYN** finally arrives at the server **after the connection is closed**.

If using **two-way handshake**, the server would:
- Accept the SYN.
- Move to **ESTABLISHED** state.
- Wait for client interaction that **will never come**.

This causes:
- **Resource leaks**.
- **Zombie connections** on the server.
- **Service degradation** under high concurrency.


#### Solution with Third Handshake
In a proper **three-step process**:

1. Server responds with `SYN-ACK`.
2. **Client must respond with an ACK**.

If the client has already closed the session:
- It won’t reply with ACK.
- The server will **not complete the handshake**, preventing stale connections.

In some TCP implementations, the client may even send an **RST** (reset) packet to explicitly refuse the connection.


### Final Conclusion

The **primary purpose** of the TCP three-way handshake is to ensure a **reliable, full-duplex communication channel** between client and server. It is **not arbitrary** — it's a carefully designed protocol mechanism based on the following goals:

1. **Ensure Bidirectional Communication Availability**
   - Verifies that **both client and server** are ready to **send and receive data**.
   - Avoids half-open connections where only one side is responsive.

2. **Prevent Reuse of Stale or Duplicate Connections**
   - Protects against outdated or duplicated `SYN` packets initializing ghost connections.
   - Saves **system resources** and avoids **zombie sockets**.

3. **Synchronize Initial Sequence Numbers**
   - Ensures both sides are aware of each other’s **starting sequence numbers** for reliable communication.


#### Sequence and Acknowledgment Numbers

- **`seq` (Sequence Number)**:
  - Used to **identify the order of bytes** in the TCP stream.
  - Helps in **reordering packets** in case they arrive out of order.

- **`ack` (Acknowledgment Number)**:
  - Confirms **successful receipt** of data.
  - Helps in detecting **packet loss**, which triggers **retransmission**.


### Reliability Mechanism

TCP’s **reliability** does **not solely depend** on the three-way handshake — it is further reinforced by the **retransmission mechanism**:

- Lost packets are **re-sent**.
- Unacknowledged segments trigger **timeouts and retries**.
- Ensures **end-to-end delivery integrity**.





<br>


---

🚀 Stay tuned for more insights into networking fundamentals!