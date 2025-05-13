---
title: TCP SYN Flood Attack and ss Command
date: 2025-05-13
categories: [Architecture, Computer Networking]
tags: [Computer Networking, TCP, SYN Flood, Linux, ss command, Cybersecurity]
author: kai
---

# 🛡️ TCP SYN Flood Attack and ss Command
A **SYN Flood** is a classic form of **Denial of Service (DoS)** attack that exploits the **TCP three-way handshake** mechanism.

## 🚦 How TCP Normally Establishes a Connection

1. **Client → Server:** Sends a `SYN` packet to initiate a connection.
2. **Server → Client:** Responds with `SYN-ACK`.
3. **Client → Server:** Sends back `ACK`, and connection is established.


## 🔥 What Happens in a SYN Flood?

An attacker sends **a large volume of spoofed SYN requests**（using spoofed IPs, First Handshake）：

- The server responds with `SYN-ACK` for **each** request. (Second Handshake)
- But the **attacker never responds with ACK** (Third Handshake), so the connection **remains half-open**.
- These incomplete connections **fill up the server’s SYN queue**, consuming memory and CPU.

Eventually:
- The server **can’t accept new connections**.
- **Legitimate users are denied service** — effectively a **DoS attack**.


## 🧠 Technical Internals: TCP Queues

When a server handles TCP requests, it uses two queues:

| Queue Type       | Description                                 |
|------------------|---------------------------------------------|
| **SYN Queue**    | Half-open connections awaiting final `ACK`  |
| **Accept Queue** | Fully established connections               |

A SYN flood **targets the SYN queue**, rapidly filling it.


## 🤖 TCP Half-Open Connection Queue (SYN Queue)
In the TCP three-way handshake, when a client initiates a connection by sending a `SYN` packet, the server responds with a `SYN-ACK` and moves the connection state from `LISTEN` to `SYN_RCVD`.

At this point, **the connection is considered half-open** — meaning it’s waiting for the final `ACK` from the client to complete the handshake. These half-open connections are temporarily stored in a data structure called the **SYN queue**, also known as the **half-connection queue**.


### Server Retry Behavior and Resource Consumption

To maintain robustness and reliability:

1. **Each entry in the SYN queue** requires a small allocation of kernel memory — including a TCB (Transmission Control Block) to track the connection state.
2. Memory and CPU cycles are required to manage timers and retransmissions.
3. If the final ACK is not received in time, the server will:
  - **Resends the `SYN-ACK`**.
  - Retries this a few times (based on system configuration).
  - Finally **drops the entry** from the SYN queue after the retry limit is reached.
- The number of entries in the SYN queue is **finite**.

If too many SYN packets are received without completing the handshake, the queue fills up, and **new connection attempts are dropped**, even from legitimate clients.


### Summary

| Item                   | Description |
|------------------------|-------------|
| SYN Queue Limit        | Finite number of incomplete TCP handshakes the server can track. |
| Attack Risk            | SYN floods fill the queue with fake requests, blocking real users. |
| Server Response        | Retries SYN-ACK, waits on ACK, and eventually discards stale entries. |
| Resource Cost          | TCBs, memory, timers, and CPU cycles per entry. |


### TCP SYN Queue Size in Linux
When configuring servers for high-concurrency TCP connections, one commonly referenced parameter is `tcp_max_syn_backlog`. It's often assumed that this value directly determines the **maximum number of pending half-open (SYN_RECV) connections** a server can handle.

However, the actual SYN queue limit isn't always defined **only** by this value — the final behavior can depend on **Linux kernel version**, **listen backlog**, and **other socket-level configurations**.

#### Verifying System Settings

1. Method 1: Using sysctl: `sysctl -a | grep max_syn`
![sysctl](/assets/img/posts/Architecture/ComputerNetworking/TCPSYNFloodAttackSysctlMaxSyn.png)

2. Method 2: Reading from /proc
```bash
[ec2-user@ip-xx ~]$ cat /proc/sys/net/ipv4/tcp_max_syn_backlog
128
```

### View Current SYN Queue Size on Linux

1. Method 1: Using `ss -s`
    - Sample Output: `TCP:   50 (estab 10, closed 0, orphaned 0, synrecv 20, timewait 20)`
    - The synrecv 20 indicates 20 TCP connections are stuck in the second phase of the handshake.

2. Method 2: Using netstat and grep: `netstat -natp | grep SYN_RECV | wc -l`
```bash
[ec2-user@ip-11-0-1-178 ~]$ sudo netstat -natp | grep SYN_RECV | wc -l
20
```

### Detect SYN Queue Overflow
When a server experiences a high volume of incoming TCP connections, particularly during a **SYN Flood attack** or under **heavy load**, the **SYN queue** (also known as the *half-open connection queue*) may overflow. This queue is where the kernel stores incomplete TCP handshakes waiting for the third ACK.

If the queue overflows, **new connection requests will be silently dropped**, potentially making your service appear unavailable.

1. Detect SYN queue overflows using: `netstat -s | grep LISTEN`
    - Sample Output: 6891 SYNs to LISTEN sockets dropped
    - This indicates that the system dropped 6891 TCP connection attempts because the queue was full and could not accept more half-open connections.

Run the above command at regular intervals (e.g., every 5 seconds) and observe the number:
- If the number is stable, you’re likely fine.
- If the number is steadily increasing, your system is probably experiencing a SYN Flood attack.


## 🔍 TCP Accept Queue

In TCP networking, once the **three-way handshake** is successfully completed between a client and a server, the connection enters a state known as `ESTABLISHED`.

At this point, the connection is placed into the **Accept Queue**, also known as the **Full Connection Queue** (or `ACCEPT queue`), where it **waits to be accepted** by the server application using the `accept()` system call.

### How Is the Accept Queue Size Determined?

The **maximum number of queued `ESTABLISHED` connections** is controlled by:

1. A **kernel parameter**: `cat /proc/sys/net/core/somaxconn`
    - Sample Output: 4096
    - The default value of net.core.somaxconn was 128 in many older Linux distributions. This means at most 129 connections (0~128) could sit in the accept queue waiting for accept().
    - Modern distributions sometimes ship with a larger somaxconn default, especially in cloud or server-tuned environments. Such as 4096 for AWS EC2.

2. 4096 ≠ Guaranteed Queue Size
- Even if somaxconn = 4096, your application may still pass a smaller value in: `listen(sockfd, backlog);  // e.g., backlog = 128`
- Final effective backlog size is: `min(backlog, somaxconn)`


### Check Current TCP Accept Queue Size
To monitor the **full connection (accept) queue** size in Linux, we can use the `ss` command—a modern replacement for `netstat`.
- ss -lnt
    - -l: Show only listening sockets
	- -n: Show numeric addresses/ports (skip DNS resolution)
	- -t: Show TCP sockets only
    -  Optional: grep protNumber: Filtering by Port

![ss -lnt](/assets/img/posts/Architecture/ComputerNetworking/TCPSYNFloodSSLnt.png)

| **Field**   | **Description**                                                                 |
|-------------|----------------------------------------------------------------------------------|
| `State`     | Should be `LISTEN` for server ports that are actively waiting for connections.  |
| `Recv-Q`    | Number of **fully established connections** currently **waiting to be accepted**. |
| `Send-Q`    | The **maximum size** of the accept queue. Often `128` by default (kernel limit). |


## 🛠️ Detecting with the `ss` Command
- The `ss` (Socket Statistics) command is a powerful networking utility in Linux used to display detailed information about current network connections. It supports various protocols such as **TCP**, **UDP**, and **Unix domain sockets**.

- While similar in function to the [`netstat`]({{ site.baseurl }}/Core-Performance-Metric-In-Computer-Networks-And-Common-Commands/#-netstat) command, `ss` provides **faster output**, **richer information**, and better support for modern Linux kernels.


### 📦 Basic Usage
- Displays a summary of all sockets: `ss -s` 
- Common flags:

| Option | Description                                      |
|--------|--------------------------------------------------|
| `-a`   | Show **all sockets**, including listening and connected ones |
| `-l`   | Show **only listening** sockets                  |
| `-n`   | Show **numeric** IP addresses and port numbers (skip DNS resolution) |
| `-t`   | Display **TCP connections** only                 |
| `-u`   | Display **UDP sockets** only                     |












<br>


---

🚀 Stay tuned for more insights into networking fundamentals!