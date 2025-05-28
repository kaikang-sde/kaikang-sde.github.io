---
title: Transport Layer
date: 2025-05-09
categories: [Architecture, Computer Networking]
tags: [Computer Networking, Transport Layer, TCP, UDP]
author: kai
---

# 🧩 Transport Layer

The **transport layer** operates above the network layer and is responsible for **reliably delivering data to the correct application process** on a device.

While the network layer delivers packets to the correct device (node), the transport layer ensures that the **data reaches the correct application** via **port numbers**.

At this layer, data units are referred to as **segments** (TCP) or **datagrams** (UDP).

Two primary protocols are used:

- **TCP** (Transmission Control Protocol)
- **UDP** (User Datagram Protocol)

## 📦 UDP: User Datagram Protocol

UDP is a **connectionless** transport protocol. It sends data **without establishing a dedicated connection** between sender and receiver.

### Key Features of UDP:

- **Connectionless:** No handshake required — it starts sending data immediately.
- **Low latency, high speed:** Because it avoids connection overhead, it is faster and more efficient.
- **Unreliable delivery:** UDP does not guarantee message delivery, order, or error checking. Packets may be lost or corrupted.
- **Lightweight header:** UDP headers are simpler (8 bytes) compared to TCP.
- **No congestion control:** Packets are sent without regard for the state of the network.

### Common Use Cases:

UDP is ideal for **real-time applications** where speed is more important than reliability:

- Online gaming
- Video conferencing
- Live streaming
- VoIP (Voice over IP)

### UDP Structure

![UDP Protocol](/assets/img/posts/Architecture/ComputerNetworking/UDPStructure.png)

- The **pseudo header** is **not transmitted**, it is used internally during checksum calculation to enhance data integrity across both the IP and transport layers.
- The **actual UDP header** precedes the data in a transmitted packet and is used to identify source/destination ports and validate the payload.
- The **data** section is variable in length and holds the actual message being transmitted. It follows directly after the 8-byte header.


## 🔐 TCP: Transmission Control Protocol

TCP (Transmission Control Protocol) is a **connection-oriented**, **reliable** transport layer protocol that ensures **data integrity** and **ordered delivery** between applications across the network.

Before data transmission, TCP establishes a **reliable connection** via a **three-way handshake**. This ensures that both the sender and receiver are ready for data transfer.


### Key Features of TCP

#### 1. **Connection-Oriented**
TCP requires both parties to **establish a connection** before data can be exchanged. This process is handled through a 3-step handshake:
- SYN → SYN-ACK → ACK

#### 2. **Reliability**
TCP includes several mechanisms to ensure reliable data transfer:
- **Checksum:** Detects data corruption.
- **Acknowledgments (ACK):** Confirms successful receipt of data.
- **Retransmissions:** Automatically resends lost or damaged segments.

#### 3. **Flow Control**
TCP uses **sliding window** techniques to ensure that the **sender does not overwhelm the receiver with too much data at once**. This mechanism adjusts transmission speed based on the receiver’s buffer size.

#### 4. **Congestion Control**
TCP implements congestion control algorithms (e.g., **Slow Start**, **Congestion Avoidance**, **Fast Retransmit**, and **Fast Recovery**) to prevent **network congestion**, ensuring fair bandwidth distribution among users.


### Common Use Cases

Because of its reliability and ordered delivery, TCP is widely used in applications where **data accuracy is critical**:

- Email (SMTP, IMAP, POP3)
- File transfers (FTP)
- Web browsing (HTTP, HTTPS)
- Remote access (SSH, Telnet)

### TCP Structure

![TCP Structure](/assets/img/posts/Architecture/ComputerNetworking/TCPStructure.png)

| Field                        | Size           | Description |
|-----------------------------|----------------|-------------|
| **Source Port**             | 16 bits        | The sending application's port number. |
| **Destination Port**        | 16 bits        | The receiving application's port number. |
| **Sequence Number (SEQ)**   | 32 bits        | Identifies the position of the first data byte in the segment.<br> When a TCP segment is sent with the `SYN` flag set to `1` (during connection setup), this sequence number represents the **Initial Sequence Number (ISN)** chosen by the host. |
| **Acknowledgment Number (ACK)** | 32 bits    | If ACK flag is set, this value is the next expected byte from the sender.<br>For example, if the acknowledgment number is `1001`, it means **bytes 0 through 1000 have been successfully received**. |
| **Data Offset (Header Length)** | 4 bits     | Specifies where the data begins in the segment. |
| **Reserved**                | 3 bits         | Reserved for future use (should be set to zero). |
| **Flags (Control Bits)**    | 9 bits total   | Includes the following control bits: |
| &nbsp;&nbsp;• URG (Urgent)  | 1 bit          | Urgent pointer field is significant.<br>Rarely used; indicates that this segment should be prioritized.|
| &nbsp;&nbsp;• ACK (Acknowledgment) | 1 bit | Acknowledges receipt of previous data.<br>`ACK = 1`: This segment **confirms receipt** of data from the other party.<br>`ACK = 0`: No acknowledgment is being made in this segment.|
| &nbsp;&nbsp;• PSH (Push)    | 1 bit          | Push the data to the receiving application immediately. |
| &nbsp;&nbsp;• RST (Reset)   | 1 bit          | Abruptly resets or terminates the connection. |
| &nbsp;&nbsp;• SYN (Synchronize) | 1 bit     | Initiates a connection and synchronizes sequence numbers. |
| &nbsp;&nbsp;• FIN (Finish)  | 1 bit          | Gracefully terminate a connection. |
| **Window Size**             | 16 bits        | Size of the receive window (flow control). |
| **Checksum**                | 16 bits        | Error-checking over the header and data. |
| **Urgent Pointer**          | 16 bits        | Points to the urgent data (if URG flag is set). |
| **Options and Padding**     | Variable       | Optional settings for maximum segment size, window scaling, etc. |
| **Data**                    | Variable       | Actual application data. |


- The **sequence** and **acknowledgment numbers** are essential for reliable delivery and ordering of segments.
- **Control flags** are used during the TCP handshake (`SYN`, `ACK`) and teardown (`FIN`, `ACK`).
- **Window size** enables **flow control** to prevent buffer overflows.
- **Checksum** ensures data integrity over unreliable networks.
- **Options** (e.g., MSS, timestamps) may increase the header size beyond the minimum of 20 bytes.

#### How SYN and ACK Work Together
SYN and ACK, these two flags are **combined** during the **TCP three-way handshake** to establish a connection:

**Three-Way Handshake Steps:**

1. **Client → Server**  
   `SYN = 1`, `ACK = 0`  
   ➤ The client sends a connection request to the server. No acknowledgment yet, as this is the initial request.

2. **Server → Client**  
   `SYN = 1`, `ACK = 1`  
   ➤ The server acknowledges the client's request and also initiates its own connection request back to the client.

3. **Client → Server**  
   `SYN = 0`, `ACK = 1`  
   ➤ The client acknowledges the server’s `SYN`. The connection is now established and ready for data exchange.


#### Can SYN and FIN Be Used Together?
**NO:**
- `SYN` signals the **start** of a connection.
- `FIN` signals the **termination** of a connection.

Using them together (`SYN = 1` and `FIN = 1`) would conflict in meaning and **is not allowed**.

### TCP's Standard Data Transmission Flow

In TCP (Transmission Control Protocol), data isn't always sent immediately. Instead, TCP employs **buffering mechanisms** to ensure efficient and reliable transmission.

#### 1. Buffered Sending at the Sender

- When a host sends data, it is first placed into the **TCP send buffer**.
- The data remains in the buffer **until enough bytes accumulate** or certain conditions are met (e.g., timeout or forced flush).
- Once ready, TCP segments are created and sent to the receiving host.

#### 2. Buffered Receiving at the Receiver

- Upon arrival, TCP segments are stored in the **receive buffer**.
- If segments arrive **out of order** or **incomplete**, TCP waits for the missing pieces to ensure correct sequence before passing data to the **application layer**.
- This ensures **reliability** and **data integrity**, key features of the TCP protocol.

#### The Exception: PSH (Push) Flag

Sometimes, applications require data to be **delivered immediately** — for example:

- Real-time chat applications
- Command-line tools
- Financial trading systems

**Enter the `PSH` Flag**

- When the `PSH` (Push) flag is set, TCP tells the receiver:
  > “Don’t wait. Deliver this data to the application right away.”
  
- It overrides the default buffer-waiting behavior, ensuring **low-latency** interaction.


### How to Uniquely Identify a TCP Connection?
In TCP/IP networking, each **TCP connection** must be uniquely identified to ensure correct data routing between endpoints. This identification is critical for the transport layer to deliver data to the correct application.

#### TCP 4-Tuple (Four-Tuple)

A **TCP connection** is uniquely identified by a **4-tuple**, which includes:

- **Source IP address**
- **Source port number**
- **Destination IP address**
- **Destination port number**

These four components together ensure that a connection from `Client A:Port X` to `Server B:Port Y` is distinguished from any other connection—even if it's between the same devices but using different ports.

```text
Example:
Source IP:        192.168.1.10
Source Port:      52345
Destination IP:   172.217.160.78
Destination Port: 443
```

### Sometimes You’ll See a “5-Tuple”
Some documentation and security tools (like firewalls or intrusion detection systems) may refer to a **5-tuple**, which adds the **transport protocol** type (e.g., TCP or UDP):
- **Protocol type (TCP/UDP)**
- **Source IP address**
- **Source port number**
- **Destination IP address**
- **Destination port number**

The 5-tuple is especially useful in contexts where both TCP and UDP traffic must be monitored or filtered separately.


> ⚠️ Note: If you’re only working within TCP, the protocol type is often assumed, so the 4-tuple is sufficient. But when working across protocols (e.g., TCP vs UDP), the 5-tuple becomes necessary.



<br>


---

🚀 Stay tuned for more insights into networking fundamentals!