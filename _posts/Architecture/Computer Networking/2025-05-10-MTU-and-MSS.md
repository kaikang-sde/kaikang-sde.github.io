---
title: MTU and MSS
date: 2025-05-10
categories: [Architecture, Computer Networking]
tags: [Computer Networking, MTU, MSS]
author: kai
---

# 🧠 MTU and MSS
**Question:  What’s the Maximum Data TCP or UDP Can Transmit?**<br>
To answer this, we need to understand two critical networking concepts: MTU and MSS.


## 🔍 MTU (Maximum Transmission Unit)
**MTU** refers to **the largest size (in bytes) that a single data packet can be transmitted** over a network without fragmentation. It plays a key role at the **Data Link Layer** of the OSI model (**Network Access Layer/ Link Layer** of the TCP/IP Layer) and directly affects the performance and reliability of network communication.

- **MTU (Maximum Transmission Unit)** is the **maximum number of bytes** a device (like a network card, router, or switch) can transmit in a single Ethernet frame.
- It defines the **payload limit** for each data frame at the **link layer**.
- Exceeding the MTU results in **fragmentation** (splitting data into smaller packets), which can hurt performance.
- **Path MTU (PMTU)**: The smallest MTU value along the path determines the maximum transmission size (like the weakest plank in a barrel).
- Standard Ethernet **MTU = 1500 bytes** (excluding Ethernet header and trailer)

> MTU can vary depending on physical media and encapsulation overhead.

### How MTU Works Across the Stack

| Layer | Model | MTU Relevance | 
|-------|-------|---------------|
| Data Link | OSI | MTU defines max payload in Ethernet frame |
| Linked Layer | TCP/IP | Same role, just broader: MTU still defines frame size limits |
| Network Layer | Both | IP checks MTU to decide whether to fragment packets |
| Transport Layer | Both | TCP uses MSS (Maximum Segment Size), which is MTU minus IP and TCP headers | 


## 🔧 MSS (Maximum Segment Size)

**MSS** defines the **maximum number of bytes** of data that a TCP segment can carry, **excluding TCP and IP headers**. It's a **crucial parameter** that ensures efficient data transfer while preventing IP fragmentation.

- MSS defines the largest chunk of application data that TCP can send in a single segment.
- TCP uses MSS when **segmenting large payloads** and during **retransmissions**.
- It helps avoid IP fragmentation by ensuring packets stay within the network's MTU (Maximum Transmission Unit).


### How MSS Works in TCP

1. **Negotiation** during the **TCP 3-way handshake**:  
   - Both hosts declare their supported MSS value in the SYN packet.
   - The actual MSS used in the session is the **minimum** of the two.

2. **Segmentation**:  
   - TCP divides large payloads into **chunks no larger than MSS**.
   - This avoids triggering IP-level fragmentation.

3. **Retransmission Unit**:  
   - When a segment is lost, it is **retransmitted in MSS-sized units**.


### How Is MSS Calculated?

> `MSS = MTU - IP Header Size - TCP Header Size`

- **Standard Ethernet MTU** = 1500 bytes  
- **IP Header (IPv4)** = 20 bytes  
- **TCP Header** = 20 bytes  
- 👉 **Default MSS** = `1500 - 20 - 20 = 1460 bytes`


## 🆚 MTU vs MSS: Key Differences

| Feature         | MTU                              | MSS                                |
|-----------------|----------------------------------|------------------------------------|
| Layer           | Data Link (e.g., Ethernet)       | Transport (TCP)                    |
| Includes Header | Yes                              | No                                 |
| Defines         | Max size of **entire** frame     | Max size of **TCP payload**        |
| Default Value   | 1500 bytes (Ethernet)            | 1460 bytes (MTU - 40 bytes)        |
| Usage           | Applied at the network interface | Applied during TCP data transfer   |


## 🤝 Relationship: How MTU and MSS Work Together

When a TCP connection is established, both sides negotiate an MSS value. This ensures that no TCP segment exceeds the agreed size and stays within the path's MTU, avoiding fragmentation.

> **Why care?**  
> Fragmentation adds overhead and can degrade performance or cause packet loss.

### Quick Summary
- **MTU (Maximum Transmission Unit)**
   - Refers to the **maximum number of bytes** that can be transmitted in a **single frame** at the **data link layer**. 
   - **Set by hardware limits** of network devices (e.g., Ethernet MTU is typically **1500 bytes**).
   - Includes the entire packet: headers + data.

- **MSS (Maximum Segment Size)**
   - Refers to the **maximum size of the TCP payload** (data only) that can be transmitted in one segment.
   - MSS = MTU - IP Header - TCP Header

- **Analogy: Think of MTU and MSS like a shipping box:**
   - **MTU** is the **maximum allowed weight of the box** (e.g., 1000g total).
   - **MSS** is the **maximum weight of the actual product** inside (e.g., 900g), after accounting for packaging and labels (headers).


## 📦 How Much Data Can TCP and UDP Transmit?
To understand the maximum amount of data that can be sent in one transmission by **TCP** and **UDP**, we need to analyze how data is encapsulated at each protocol layer, starting from the **Ethernet frame**.

### Ethernet Packet Breakdown

- **Total size of Ethernet frame**: `1522 bytes`
- **Header size (Ethernet)**: `22 bytes`
- **Payload (MTU)**: `1500 bytes`

The **Maximum Transmission Unit (MTU)** is the max payload size allowed in an Ethernet frame, and it's **1500 bytes**.


### IP Packet Structure

An **IP packet** fits inside the Ethernet payload (1500 bytes):

- **IP header**: `20 bytes` (minimum)
- **Available for TCP/UDP data**: `1480 bytes`


### TCP Packet Size Calculation

A **TCP packet** sits inside the IP packet:

- **TCP header**: `20 bytes` (minimum)
- **Maximum Segment Size (MSS)** for TCP:
   - `MSS = MTU - IP header - TCP header = 1500 - 20 - 20 = 1460 bytes`

In practice, due to optional headers or security features (like TCP options, IP options), the effective payload may be around **1400 bytes**.

### UDP Packet Size Calculation

UDP has a smaller header than TCP:

- **UDP header**: `8 bytes`
- **Maximum payload size**: 
   - `MSS = MTU - IP header - UDP header = 1500 - 20 - 8 = 1472 bytes`


### Summary Table

| Protocol | Header Sizes | Max Payload (Bytes) |
|----------|--------------|---------------------|
| TCP      | IP (20) + TCP (20) = 40B | **1460** |
| UDP      | IP (20) + UDP (8)  = 28B | **1472** |

Note: The actual values may vary depending on network settings and whether header options (e.g., IP options, TCP options) are used.

### Key Takeaways

- **TCP MSS is typically 1460 bytes**.
- **UDP can carry slightly more per packet (1472 bytes)** due to its smaller header.
- Real-world payloads may be **smaller** due to optional headers or encryption layers (e.g., VPN, IPSec).


### Scenario: What Happens When You Send a 10MB File?
When a **10MB file** is sent over the network using **TCP**, it cannot be delivered as a single chunk. Due to **MTU** (Maximum Transmission Unit) limits and **TCP segment size**, the file must be broken into **thousands of smaller packets**.


#### Calculation
Assume:
- **Each TCP segment** carries approximately **1400 bytes** of payload (excluding headers).
- **10MB = 10 × 1024 × 1024 = 10,485,760 bytes**
- **Total TCP packets = 10,485,760 / 1400 ≈ ~7489 segments**

So, to transmit a 10MB file, the system sends roughly **7,100–7,500 packets**.
> Different application protocols handle payload structure and compression differently:<br>
> HTTP/1.x: Sends headers and body as separate entities. Less efficient for large files.<br>
> HTTP/2: Supports header compression, multiplexing, and more efficient binary framing. Optimized for performance and large data transfers.

####  Server-Side Logic
When a server sends this 10MB file:
1.	**TCP divides** the file into segments.
2.	Each segment is **wrapped by IP and Ethernet headers**.
3.	The **network transmits** these packets independently.
4.	The **receiving OS reassembles** all packets based on TCP sequence numbers.
5.	The **complete byte stream** is handed over to the application.

> The application layer (e.g., an HTTP server) doesn’t handle or even see the segmentation. It only reads the final reassembled data.








<br>


---

🚀 Stay tuned for more insights into networking fundamentals!