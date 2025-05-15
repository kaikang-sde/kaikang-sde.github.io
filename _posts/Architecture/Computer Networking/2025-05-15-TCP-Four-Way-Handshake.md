---
title: TCP Four-Way Handshake
date: 2025-05-15
categories: [Architecture, Computer Networking]
tags: [Computer Networking, TCP, Four-Way Handshake]
author: kai
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
When a TCP connection is closed, the **client enters the `TIME_WAIT` state**, which lasts for **2×MSL (Maximum Segment Lifetime)**.

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




<br>


---

🚀 Stay tuned for more insights into networking fundamentals!