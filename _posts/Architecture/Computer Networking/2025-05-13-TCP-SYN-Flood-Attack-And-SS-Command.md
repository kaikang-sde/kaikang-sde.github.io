---
title: TCP SYN Flood Attack and ss Command
date: 2025-05-13
categories: [Architecture, Computer Networking]
tags: [Computer Networking, TCP, SYN Flood, Linux, ss command, Cybersecurity]
author: kai
permalink: /posts/architecture/computer-networking/tcp-syn-flood-attack-and-ss-command/
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
- `ss -lnt`
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


### Check If the TCP Accept Queue Is Overflowing
In high-concurrency environments, it's important to monitor whether the **Accept Queue** (also known as the **full connection queue**) is experiencing overflow. When too many fully established connections are waiting to be accepted by the application, new connections may be dropped.

- `netstat -s | grep TCPBacklogDrop`
- Sample Output: TCPBacklogDrop: 36
	- This means 36 fully established TCP connections were dropped because the Accept queue was full.
	- If the number keeps increasing over time, your application cannot call accept() fast enough, or your queue size is too small.

### What Happens When the TCP Accept Queue Is Full
When a server's **Accept Queue** (the full connection queue) is full, it can't accept any more fully established connections. How the server reacts to this situation depends on the system kernel parameter:
- `cat /proc/sys/net/ipv4/tcp_abort_on_overflow`
- Output: 0 or 1
    - **0 (default)**: Silently drops the final ACK from the client. The client assumes a timeout and retries, thinking the server is just slow or unresponsive.
        - Recommended in production environments. It’s more graceful, avoiding aggressive connection resets, especially under heavy load.
    - **1**: Actively sends a TCP RST (Reset) back to the client. This immediately closes the connection and informs the client of rejection.
        - Useful for diagnostics or debugging. If the client receives a `connection reset by peer error`, it can immediately detect that the server is overwhelmed — helping identify backlog saturation issues.

## 🎯 Core Insight

Both **half-open connection queues (SYN queue)** and **fully-established connection queues (accept queue)** have **maximum length limits**. If these queues overflow:

- The **Linux kernel silently drops packets** or
- Sends a **TCP RST** to reject the connection

### Why This Matters

For **short-lived connection applications**, such as:
- Reverse proxies (Nginx)
- Web services with frequent PHP requests
- API endpoints with high concurrency

the chances of queue overflow are **significantly higher**, especially under burst traffic.

### Symptoms of Overflow

When the queues are full:
- **New client connections fail** to establish
- **`connection reset by peer`** appears in logs
- CPU, thread, and memory usage may look normal
- But throughput and QPS (queries per second) **plateau unexpectedly**

This makes queue overflows a **silent bottleneck**, often overlooked during performance tuning.

### Tips for Diagnosis

- Monitor with:
  - `netstat -s | grep LISTEN` → detects dropped SYNs
  - `netstat -s | grep TCPBacklogDrop` → detect accept queue overflows
- Inspect `/proc/sys/net/core/somaxconn` and `/proc/sys/net/ipv4/tcp_max_syn_backlog`
- Use `ss -lnt` to observe current connection queue utilization

### Recommendations

- **Increase queue sizes** where appropriate:
  ```bash
  sysctl -w net.core.somaxconn=4096
  sysctl -w net.ipv4.tcp_max_syn_backlog=4096
  ```
- **For load testing or debugging**:
    - `echo 1 > /proc/sys/net/ipv4/tcp_abort_on_overflow`

## 🛡 Flood Attack - Basic Mitigation: TCP SYN Cookies
**SYN Cookies** help defend against SYN floods **without consuming server memory** for half-open connections.

### How It Works

- When `tcp_syncookies` is enabled, the **server does not allocate a TCB (Transmission Control Block)** immediately.
- Instead, it calculates a **hash (cookie)** using:
  - Source IP, source port
  - Destination IP, destination port
  - A secret server-side random number
- This hash is used as the **initial sequence number** in the SYN-ACK packet.
- If the client replies with the final ACK, the server recomputes the hash and **verifies it**.
  - If valid: A real connection is established
  - If invalid: It is ignored

### Benefits

- **No memory allocation** until final ACK — prevents queue exhaustion.
- **Stateless validation** of legitimate handshakes.

### Drawbacks

- **Extra CPU overhead** for computing hashes
- Vulnerable to **ACK flood attacks** (CPU exhaustion via fake ACKs)


### Enabling SYN Cookies on Linux
- Check with: `sysctl net.ipv4.tcp_syncookies`
- Configure in /etc/sysctl.conf: `net.ipv4.tcp_syncookies = 1`
    - 0 → Disabled
	- 1 → Enabled only when SYN backlog overflows
	- 2 → Always enabled (even if not overflowing)
- Apply changes:` sudo sysctl -p`

## 🔚 Other Mitigation Techniques
1. Increase Queue Sizes
- Half-open (SYN) queue: `net.ipv4.tcp_max_syn_backlog = 4096`
- Accept (full) queue: `net.core.somaxconn = 4096`

2. Reduce SYN-ACK Retries
- Linux retries SYN-ACK multiple times before giving up. To limit retry attempts: `sysctl -w net.ipv4.tcp_synack_retries=2`

3. Use DDoS Protection Services
- If under large-scale SYN flood or DDoS, hardware-based solutions or cloud-based traffic scrubbing (e.g., Cloudflare, Alibaba Cloud Anti-DDoS) are required.


## 🛠️ Detecting with the `ss` Command
- The `ss` (Socket Statistics) command is a powerful networking utility in Linux used to display detailed information about current network connections. It supports various protocols such as **TCP**, **UDP**, and **Unix domain sockets**.

- While similar in function to the [`netstat`]({{ site.baseurl }}/posts/architecture/computer-networking/core-performance-metric-in-computer-networks-and-common-commands/#-netstat) command, `ss` provides **faster output**, **richer information**, and better support for modern Linux kernels.


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