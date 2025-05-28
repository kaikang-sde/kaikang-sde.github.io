---
title: TCP Reliability - ACK, Flow Control, Congestion Control, and Retransmission
date: 2025-05-20
categories: [Architecture, Computer Networking]
tags: [Computer Networking, TCP, Reliability, FlowControl, CongestionControl, Retransmission, ACK]
author: kai
---

# 🛜 TCP Reliability: ACK, Flow Control, Congestion Control, and Retransmission"
TCP (Transmission Control Protocol) is designed to provide **reliable, ordered, and error-checked delivery of data** between applications. Its reliability is built on four major mechanisms:

- **Acknowledgements (ACK)**  
- **Flow Control**  
- **Congestion Control**  
- **Retransmission Mechanisms**


## 🔐 ACK & Flow Control
When a server transmits data too quickly — especially over limited bandwidth or unstable network links — **packet loss** becomes inevitable. To ensure **reliable data transmission**, TCP provides several built-in mechanisms.

### ACK Mechanism (Acknowledgment)

Whenever the **receiver** successfully receives a TCP packet, it responds with an **ACK (Acknowledgment)** message.

Each ACK contains:
- The **sequence number** of the **next expected byte**.
- The **remaining buffer capacity** of the receiver (advertised window).

This tells the sender how much data the receiver is ready to accept next.


### Flow Control (Receiver-side rate management)
Flow control is a fundamental mechanism in TCP that helps **prevent the sender from overwhelming the receiver** with too much data. It is an **end-to-end** control method — imagine **shipping goods from Warehouse A to Warehouse B**: the sender must ensure the receiver has enough space before dispatching more.

Every TCP connection maintains a **buffer** on both the sender and receiver sides:

- The **sender buffer** holds unacknowledged packets.
- The **receiver buffer** stores incoming data before it is processed by the application.

If the receiver is **too busy** to consume data quickly, it must **signal the sender to slow down** — otherwise, the buffer might overflow and lead to **packet drops**.


### The Sliding Window Mechanism

TCP uses a **variable-size sliding window protocol** to implement flow control:

- The **receiver advertises a window size** (known as `win`) to indicate how much space is left in its buffer.
- This value is a **16-bit field** in the TCP header.
- The window size is updated with every ACK packet.


### Flow Control in Action

```text
Client                           Server
|                                |
|      seq=1, data (100B)        | → Sends 100 bytes
| —————————–—————————–————————–> |
|      seq=101, data (100B)      | → Sends next 100 bytes
| —————————–—————————–————————–> |
|                                |
| <—–––– ACK=201, rwnd=300 ––––  | ← Receiver acknowledges 200 bytes received, remaining buffer = 300B
|                                |
|      seq=201, data (100B)      | → Sends another 100 bytes
| —————————–—————————–————————–> |
|      seq=301, data (200B)      | → Sends next 200 bytes
| —————————–—————————–————————–> |
|                                |
| <—––––ACK=501, rwnd=0 –––––––––| ← Receiver acknowledges 300 bytes and tells sender to pause (window full)
```

#### Key Points

- **Sequence Numbers (`seq`)**: Identify the starting byte of the data being sent.
- **Acknowledgment Number (`ack`)**: Indicates the next byte the receiver expects.
- **Receive Window (`rwnd`)**: Advertised buffer space available on the receiver side.

#### What Happens When `rwnd = 0`?

- The receiver tells the sender to stop sending data.
- The sender must **pause** and wait for a new ACK with a larger window size before continuing.


## 🚦 TCP Congestion Control
TCP implements **congestion control** to ensure that the sender doesn't overwhelm the network. Unlike **flow control** (which protects the receiver), congestion control takes into account the **entire network** — including routers, links, and shared bandwidth. It's like ensuring that trucks between Warehouse A and B don’t carry more than the roads can handle.

Congestion control is **adaptive**, adjusting the sender’s rate dynamically based on real-time feedback from the network.


### 1. Slow Start (Exponential Growth)

At the beginning of a TCP connection, the sender doesn’t know the available bandwidth, so it starts conservatively.

- It begins with a small **congestion window (cwnd)** — often 1 or 2 MSS (Maximum Segment Size).
- For each ACK received, the cwnd **doubles**, increasing transmission rate exponentially.
- This continues until a threshold is reached or **packet loss** occurs.

> This prevents flooding the network with too many packets all at once.


### 2. Fast Retransmit

When the sender detects **packet loss**, it doesn’t always wait for a timeout.

- If it receives **three duplicate ACKs** (ACKs with the same acknowledgment number), it **immediately retransmits** the missing packet.
- This improves efficiency over waiting for a full timeout.

```text
Client:   Packet seq=1001 lost
Server:   ACK=1001, ACK=1001, ACK=1001
Client:   Fast retransmits seq=1001
```

> Faster recovery = better performance in lossy environments.

### Congestion Avoidance
Once slow start ends (typically when a threshold is reached), TCP enters **Congestion Avoidance mode**.
- Here, the congestion window increases **linearly** instead of exponentially.
- If **three duplicate ACKs** are received (indicating likely loss), TCP:
- Cuts the cwnd by half.
- Enters slow start again with the reduced threshold.

> Goal: Maintain high throughput while avoiding network collapse.



## 🔁 TCP Retransmission Mechanisms

TCP (Transmission Control Protocol) ensures **reliable transmission** of data by detecting and recovering from lost packets. Two key retransmission mechanisms are used to handle data loss: **Timeout Retransmission** and **Fast Retransmit**.

### 1. Retransmission Timeout (RTO)

#### How It Works:

- After sending a packet, the sender starts a **timer**.
- If the **ACK** (Acknowledgement) is not received before the timer expires, the packet is retransmitted.
- This cycle continues until the packet is acknowledged.

#### Key Term: `RTO`

- **Retransmission Timeout** should be slightly **greater than the RTT** (Round Trip Time).
- RTO is dynamically adjusted based on network latency measurements.

#### Drawback:

- **Delayed recovery**: If a packet is lost, the sender has to **wait for the timeout**, which **increases end-to-end delay**.


### 2. Fast Retransmit (Triggered by Duplicate ACKs)

#### Motivation:

Waiting for a timeout may be too slow. TCP can detect losses earlier using **duplicate ACKs**.

#### How It Works:

- Each TCP segment is assigned a **sequence number (seq)**.
- If a packet is missing (e.g., packet 14), the receiver **continues to acknowledge the last in-order packet** it received (e.g., ACK=13).
- If the sender receives **3 duplicate ACKs** (e.g., ACK=13 repeated 3 times), it assumes packet 14 was lost and **retransmits it immediately**, without waiting for the timeout.

#### Example:

```text
Received Packets: [6, 7, 8, 9, 10, 11, 12, 13, (14 is missing), 15, 16...]
Receiver sends: ACK=14 (expecting 14)
Sender sees: ACK=14 repeated 3 times → Triggers fast retransmit of packet 14
```

#### Benefit:
- Much faster than waiting for RTO.
- Improves overall TCP performance in lossy networks.

#### Limitations of Fast Retransmit

While faster than timeout-based recovery, Fast Retransmit still has challenges:
- It may **retransmit older packets unnecessarily** (even if only one packet is lost).
- It can **misinterpret packet reordering as loss**.
- There is **ambiguity**: should we **retransmit just one packet** or **all unacknowledged packets**?

To improve upon this, **Selective Acknowledgment (SACK)** and **Duplicate SACK (D-SACK)** are introduced in advanced TCP implementations.






<br>


---

🚀 Stay tuned for more insights into networking fundamentals!