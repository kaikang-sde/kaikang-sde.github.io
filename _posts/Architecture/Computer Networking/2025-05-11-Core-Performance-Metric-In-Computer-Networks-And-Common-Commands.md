---
title: Core Performance Metrics in Computer Networks and Common Commands
date: 2025-05-11
categories: [Architecture, Computer Networking]
tags: [computer networking, Linux, Performance, CLI, ifconfig, netstat]
permalink: /Core-Performance-Metric-In-Computer-Networks-And-Common-Commands/
author: kai
---

# 📌 Core Performance Metrics in Computer Networks and Common Commands
When evaluating computer networks, it's essential to understand **key performance metrics** that quantify speed, efficiency, and reliability. Some common commands used to inspect network interface status on Unix-like systems.

## 🧠 Preliminaries

### Bits vs Bytes

| Term       | Description                            | Symbol |
|------------|----------------------------------------|--------|
| **Bit**    | The smallest unit of data. A `bit` is either a `1` or a `0`. | `b`      |
| **Byte**   | A collection of 8 bits. It's the standard unit for measuring file/data size. | `B`      |

#### Case Sensitivity Matters

- `1 Byte (B)` = `8 bits (b)`
- `Mbps` = Mega**bits** per second (network bandwidth)
- `MBps` = Mega**bytes** per second (file size or transfer rate)

#### Examples

| Metric           | Explanation                                      |
|------------------|--------------------------------------------------|
| **kbps**         | kilobits per second (1000 bits/sec)              |
| **MB**           | Megabytes (1 MB = 1024 KB = 8,388,608 bits)      |
| **1B = 8b**      | Conversion rule: always multiply bytes by 8 to get bits |
| **100 Mbps**     | Network speed in megabits → ~12.5 MBps in real throughput |


### Network Speed (Bitrate)
**Bitrate** or **transmission speed** refers to how fast bits (the smallest unit of data) can be transmitted over a digital communication channel. It's a fundamental performance metric in computer networks.

#### Unit of Measurement

| Unit    | Meaning                         |
|---------|----------------------------------|
| `bps`   | bits per second                 |
| `kbps`  | kilobits per second = 1,000 bps |
| `Mbps`  | megabits per second = 1,000,000 bps |
| `Gbps`  | gigabits per second = 1,000,000,000 bps |


#### Real-World Example: File Transfer Calculation

**Problem:**<br>
You want to send a **100MB** file using a **100Mbps** Ethernet connection.

> Transmission is measured in **bits**, but file sizes are usually in **Bytes**.


#### Step-by-step Calculation:

1. Convert file size from **MB to bits**: `100 MB = 100 × 8 = 800 Megabits`
2. Calculate the transmission time: `Time = Total Data / Transmission Rate = = 800 Mb / 100 Mbps = = 8 seconds`


### Bandwidth
**Bandwidth** refers to the maximum amount of data that can be transmitted over a network connection in a given amount of time. It represents the **maximum data-carrying capacity** of the connection — **not** the actual speed at which you are downloading or uploading data.


- **Unit**: **bits per second (bps)**
- Common forms:
  - `Mbps` — Megabits per second
  - `Gbps` — Gigabits per second

#### Example
- **Scenario**: Kai subscribes to a **home internet plan of 800 Mbps**.
- **Question**: What is his maximum theoretical download speed in MB/s (megabytes per second)?
- **Answer**: 800 Mbps ÷ 8 = 100 MB/s (ideal)
- **Bandwidth ≠ Download Speed (Directly)**: While bandwidth is measured in **bits per second (bps)**, the speed shown in download managers or browsers is often in **bytes per second (B/s)**.


### Throughput
**Throughput** refers to the **actual amount of data** successfully transmitted over a network **per unit of time**. While **bandwidth** is the theoretical maximum capacity, **throughput is what you actually get**.

- Unit: bits per second (bps) or bytes per second (B/s)
- Throughput is often **less than or equal to bandwidth**, depending on network congestion, latency, and overhead.
- Network Utilization tells you how efficiently the available bandwidth is being used. 
   - `Network Utilization (%) = Throughput / Bandwidth`


#### Common Throughput Units

| **Metric** | **Full Name**           | **Description**                                           |
|------------|--------------------------|-----------------------------------------------------------|
| **PPS**    | Packets Per Second       | Number of individual network packets sent/received per second. |
| **Bps**    | Bytes Per Second         | Actual data transferred per second, measured in **bytes** (1B = 8 bits). |
| **bps**    | Bits Per Second          | Data transfer rate in **bits** per second, the most fundamental unit. |


### Network Delay (Latency)
**Delay** refers to the time it takes for data (a bit, packet, or message) to travel from the source to the destination across the network.

A full end-to-end delay typically consists of:

- **Transmission Delay**: Time to push data onto the link.
- **Propagation Delay**: Time for the signal to physically travel through the medium.
- **Processing Delay**: Time routers and switches take to examine and forward the packet.
- **Queuing Delay**: Time spent waiting in buffer queues during congestion.

> The sum of these delays determines overall network **latency**.


### Round-Trip Time (RTT)

**RTT** is the time it takes for a message to go from the source to the destination **and back again**.

- Measured in milliseconds (ms).
- Commonly used in ping or ICMP tests to assess network responsiveness.
- Critical in real-time applications like video conferencing or online gaming.

### Packet Loss Rate

**Packet loss rate** refers to the percentage of packets that are **lost** during transmission over a network.

**Formula**: `Packet Loss Rate = (Lost Packets / Total Sent Packets) × 100%`


## 🧰 ifconfig 
The `ifconfig` (interface configuration) command in Unix/Linux is used to **display or configure network interfaces**. It’s particularly useful for **viewing IP configurations**, **MAC addresses**, and **packet transmission statistics**.

![ifconfig](/assets/img/posts/Architecture/ComputerNetworking/CommonCmdIfconfig.png)

### Key Output Fields Explained

| Field                             | Description                                                                                     |
|----------------------------------|-------------------------------------------------------------------------------------------------|
| `flags=4163<UP,BROADCAST,...>`   | **RUNNING** means the interface is connected (physically up). Missing `RUNNING` implies cable unplugged. |
| `mtu`                             | **Maximum Transmission Unit** – maximum packet size in bytes (default: 1500).                  |
| `inet`                            | IPv4 address assigned to the interface. Public IPs (like Elastic IPs in ECS) aren’t shown here.<br>172.31.30.145 is the Private IPv4 addresses in this EC2 Instance|
| `netmask`                         | Subnet mask that determines the network’s IP range.                                            |
| `broadcast`                       | Broadcast address for the network.                                                             |
| `ether`                           | MAC address of the interface.                                                                  |

**Special Note on ECS Public IPs**

If you're using an ECS instance (e.g., AWS Cloud) with a public EIP, note:

- You **won't** see the public IP under `inet` in `ifconfig`.  
- That's because the **public IP is mapped at the gateway level**, not directly on the instance’s network interface.


### Packet Statistics (RX/TX)

| Field         | Meaning                                                                                   |
|---------------|--------------------------------------------------------------------------------------------|
| `RX packets`  | Number of **received** packets.                                                            |
| `TX packets`  | Number of **transmitted** packets.                                                         |
| `bytes`       | Data volume transferred (in bytes).                                                       |
| `errors`      | Number of packets with **errors** (e.g., checksum failure).                                |
| `dropped`     | Packets **discarded** (e.g., buffer overflow or invalid headers).                          |
| `overruns`    | Packets lost due to interface buffer **overrun** (e.g., too fast for CPU to handle).       |
| `carrier`     | Carrier-level **transmission errors** (e.g., bad cable or electrical issue).               |
| `collisions`  | Detected **collisions** on the network (mostly in shared media like old Ethernet hubs).   |


## 🧭 netstat
`netstat` (short for **network statistics**) is a powerful command-line tool for **monitoring network connections, routing tables, and interface statistics** in Linux/Unix systems.

It’s frequently used for:
- Checking which ports are open or listening.
- Viewing TCP/UDP connection statuses.
- Diagnosing network issues.
- Analyzing port and process mappings.


### Common Parameters

| Parameter     | Description                                                                           |
|---------------|----------------------------------------------------------------------------------------|
| `-r`          | Show the kernel **routing table**.                                                    |
| `-n`          | Display **numerical addresses** instead of resolving hostnames.<br>Use -n to speed up output by disabling reverse DNS lookup.                        |
| `-s`          | Show **per-protocol statistics** (TCP, UDP, ICMP, etc.).                              |
| `-p`          | Display the **process name and PID** associated with each connection.<br> Requires root privileges to show program names.                  |
| `-l`          | Show **listening ports** only.                                                        |
| `-a`          | Show **all connections and listening ports**.                                         |
| `-u`          | Show **UDP** connections.                                                             |
| `-t`          | Show **TCP** connections.                                                             |
| `-i`          | Show network **interface statistics**.                                                |


### Common Use Cases

| Command | Use Case |
|--------|----------|
| `netstat -anp` | Show all connections with associated programs and ports (TCP + UDP). |
| `netstat -anp | grep 8080` | Check if port 8080 is being used and by which process. |
| `netstat -nupl` | Display all **UDP** listening ports with their PIDs. |
| `netstat -ntpl` | Display all **TCP** listening ports with their PIDs. |
| `netstat -na | grep ESTABLISHED | wc -l` | Count all active **established** TCP connections. |
| `netstat -l` | List all ports that are currently **listening**. |
| `netstat -lt` | List only **TCP listening** ports. |


### `netstat -atnlp`
The `netstat -atnlp` command is widely used in Linux systems to inspect all **active TCP sockets**, both listening and established, in a detailed, numeric, and process-aware format.

![netstat -atnlp](/assets/img/posts/Architecture/ComputerNetworking/NetstatAtnlp.png)

#### Field Descriptions

| **Field**         | **Explanation** |
|-------------------|-----------------|
| `Proto`           | **Protocol type**, usually `tcp` or `udp`. |
| `Recv-Q`          | **Receive Queue**: Number of bytes received and stored in the kernel buffer but not yet read by the application. Non-zero values can indicate application lag. |
| `Send-Q`          | **Send Queue**: Number of bytes sent by the application but not yet acknowledged by the receiver. High values may indicate network latency or slow receivers. |
| `Local Address`   | **Local endpoint**: Format `IP:Port`. Describes the socket’s local binding. |
| `Foreign Address` | **Remote endpoint**: Where the socket is connected to, or `0.0.0.0:*` if not yet connected. |
| `State`           | **TCP Connection State** (only applicable for TCP).<br>UDP (User Datagram Protocol) is connectionless — it does not maintain any state. |
| `PID/Program`     | Shows the **process ID and executable name** that owns the socket (requires root access). |


#### Special Address Meanings

| **Address Format**         | **Meaning** |
|----------------------------|-------------|
| `127.0.0.1:port`           | **Loopback interface**, only accessible from localhost. |
| `0.0.0.0:port`             | **Open to all IPv4 interfaces**, accessible from the external network. |
| `:::port`                  | **IPv6 equivalent** of `0.0.0.0:port`, open to all. |
| `0.0.0.0:*` / `:::*`       | Socket not bound to a specific port, not yet listening or connected. |


#### Common TCP Connection States
When using tools like `netstat`, you may encounter various TCP connection states. These represent the different stages of a TCP connection lifecycle—covering both the **three-way handshake** used to establish a connection and the **four-way termination** process to close it.

> UDP (User Datagram Protocol) is connectionless — it does not maintain any state.

Below is a breakdown of the **11 common TCP states + 1 UNKNOWN**:

| **State**       | **Description** |
|-----------------|-----------------|
| **LISTEN**      | The socket is waiting for incoming connections. Common for servers. |
| **SYN_SENT**    | The client has sent a SYN request to initiate a connection, waiting for a response. |
| **SYN_RECV**    | The server has received the SYN and replied with SYN-ACK, waiting for final ACK. |
| **ESTABLISHED** | Connection is fully open. Both ends can send/receive data. |


**Connection Termination States**

| **State**       | **Role**          | **Description** |
|-----------------|------------------|-----------------|
| **FIN_WAIT1**   | Active Closer     | The side initiating closure sends a FIN packet and enters this state. |
| **CLOSE_WAIT**  | Passive Closer    | The other side received the FIN and acknowledged it but hasn't sent its own FIN yet. |
| **FIN_WAIT2**   | Active Closer     | After receiving ACK for its FIN, waits for the other side’s FIN. |
| **LAST_ACK**    | Passive Closer    | Sent its own FIN, waiting for the final ACK from the other side. |
| **TIME_WAIT**   | Active Closer     | After receiving the other FIN and sending ACK, waits to ensure no lost ACKs. |
| **CLOSING**     | Both              | Rare state when both sides send FINs simultaneously and wait for the final ACK. |


**Connection Ends**

| **State**   | **Description** |
|-------------|-----------------|
| **CLOSED**  | The connection is completely closed, and resources have been released. |
| **UNKNOWN** | An undefined or invalid socket state (rare). |

#### Visual: TCP Lifecycle (Simplified)

```text
Client                           Server
  |                                |
  |---- SYN ---------------------> |  (SYN_SENT)
  |                                |
  |<--- SYN + ACK ---------------- |  (SYN_RECV)
  |                                |
  |---- ACK ---------------------> |  (ESTABLISHED)
  |                                |
  |========= Data Exchange ======>|
  |                                |
  |---- FIN ---------------------> |  (FIN_WAIT1)
  |                                |
  |<--- ACK ---------------------- |  (FIN_WAIT2)
  |<--- FIN ---------------------- |  (CLOSE_WAIT → LAST_ACK)
  |---- ACK ---------------------> |  (TIME_WAIT → CLOSED)
```













<br>


---

🚀 Stay tuned for more insights into networking fundamentals!