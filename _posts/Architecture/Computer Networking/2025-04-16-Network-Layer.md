---
title: Network Layer
date: 2025-04-16
categories: [Architecture, Computer Networking]
tags: [Computer Networking, Network Layer, Internet Layer, IP, IPv4]
author: kai
---

# 🌐 Network Layer or Internet Layer
The **Network Layer** is a fundamental component of modern computer networks, responsible for **routing data packets from one host to another** across interconnected networks.

[Network Layer/Internet Layer]({{ site.baseurl }}/basic-of-computer-networking/#-tcpip-4-layer-model)

- When people say “**Network Layer**”, they usually refer to the same functions as the **Internet Layer** in TCP/IP model.
- So, they are **functionally equivalent**, just named differently in different models.


## 🧩 Key Protocol: IP (Internet Protocol)
The **IP protocol** is the most critical protocol within the network layer. It defines the rules for sending data **from one computer to another** across networks.

### What Does IP Do?

- Operates at the **Internet Layer** (in the TCP/IP model)
- **Delivers packets** from **source to destination** using IP addresses
- Provides a **best-effort, connectionless, and unreliable** transmission service
- Does **not guarantee** delivery, ordering, or duplicate protection

> Think of IP as a basic postal system — it gets the packet on its way, but doesn’t ensure it gets delivered or in what order.

### How the IP Protocol Works in Data Transmission
When **an application sends a request** (e.g., from a mobile app), the data packet is first delivered to the **telecom provider's switch**. The switch reads the **destination IP address** and forwards the packet through **multiple routing hops**, eventually reaching the **target server**.

- The **Internet Protocol (IP)** operates at the network layer (or Internet layer in the TCP/IP model).
- Its primary responsibility is **packet delivery** — ensuring that data is routed from the source device to the destination based on IP addresses.
- **IP is connectionless and unreliable**:
  - It does **not guarantee delivery**.
  - It does **not ensure packet order**.
  - It does **not retransmit lost packets**.

> To ensure reliable transmission (e.g., ordered delivery, error detection, and retransmission), the **Transmission Control Protocol (TCP)** — which operates at the transport layer — is used on top of IP.


### IP as the Foundation

The IP protocol serves as the **foundation for higher-level protocols** such as:

- **TCP (Transmission Control Protocol)** – reliable, connection-oriented
- **UDP (User Datagram Protocol)** – faster, connectionless, used for streaming, gaming, etc.

In essence, **IP routes the packets**, while protocols like **TCP ensure the reliability of the data transmission**.


> Think of the IP packet as a shipping box, the TCP segment as the envelope inside it, and the TCP data as the actual letter. The shipping label (IP header) gets the package to the right address; the envelope (TCP header) ensures the right person receives the right letter and can even ask for confirmation.


## 🧭 Key Functions of the IP Protocol

### 1. Addressing and Routing

Each IP datagram contains:

- **Source IP Address**: Identifies the sender (origin host). **Destination IP Address**: Identifies the receiver (target host).
- As the IP datagram travels across the internet, it may pass through multiple intermediate network nodes like **IP gateways or routers**.  
- These nodes make routing decisions **based solely on the destination IP address**, ensuring the datagram is forwarded hop-by-hop until it reaches the correct host.



### 2. Fragmentation and Reassembly

In real-world scenarios, a datagram may pass through **different types of networks**, each with its own **Maximum Transmission Unit (MTU)** — the largest packet size it can handle.

To accommodate this:

- IP supports **fragmentation**: Breaking the original datagram into smaller pieces.
- Each fragment contains:
  - An **identifier** (so fragments from the same datagram can be recognized),
  - A **fragment offset** (position of the fragment in the original datagram),
  - A **more fragments flag** (to indicate whether more parts follow).

- These fragments are routed **independently**, possibly along different paths.

- Once all fragments reach the destination host, the IP layer performs **reassembly**, reconstructing the original IP datagram.

### Summary

| Feature           | Purpose                                                  |
|------------------|-----------------------------------------------------------|
| Addressing       | Identify source and destination hosts across networks     |
| Routing          | Enable hop-by-hop delivery using destination IP address   |
| Fragmentation    | Support transmission over networks with varying MTU sizes |
| Reassembly       | Restore the original datagram at the destination          |


## 🛜 Protocols in the Network Layer
The **Network Layer** is responsible for delivering packets across different networks and plays a central role in the architecture of the Internet. Here are the **key protocols** that operate at this layer:

### 1. IP (Internet Protocol)
The foundation of the entire internet.  
- **Purpose**: Provides logical addressing and routing.
- **Key Features**:
  - Connectionless
  - Unreliable (no guarantee of delivery, ordering, or duplication prevention)
- **Types**: IPv4, IPv6


### 2. ICMP (Internet Control Message Protocol)
Used for error reporting and network diagnostics.  
- **Purpose**: Communicates control messages (e.g., unreachable hosts, TTL expired).
- **Use Case**: Powers tools like `ping` and `traceroute`.


### 3. ARP (Address Resolution Protocol)
Maps IP addresses to physical MAC addresses within a local network.  
- **Purpose**: Resolve an IP address (e.g., `192.168.1.5`) to a device's MAC address (e.g., `AA:BB:CC:DD:EE:FF`).
- **Scope**: Local Area Networks (LANs)


### 4. RARP (Reverse Address Resolution Protocol)
The opposite of ARP, now largely deprecated.  
- **Purpose**: Allows a device to discover its own IP address using its MAC address.
- **Used by**: Diskless workstations during boot in early networks.
- **Note**: Replaced by protocols like **BOOTP** and **DHCP**.


## 📡 What Are IP Address Classes?
An **IPv4 address** is a 32-bit numerical label that uniquely identifies a device on a network. These 32 bits are grouped into four **octets** (8 bits each), typically displayed in **decimal format** separated by dots for human readability.

> Example: `192.168.0.0`  
> Binary representation: `11000000.10100000.00000000.00000000`

### IP Address Structure

An IP address is logically divided into two parts:

- **Network ID**: Identifies the network segment.
- **Host ID**: Identifies the specific device (host) on that network.

This division allows IP routing to function efficiently by directing traffic to the correct network and then the correct device.


### IPv4 Address Classes

IPv4 addresses are categorized into five main **classes**, with Classes A, B, and C commonly used in commercial networks.

| Class | Binary Prefix | Network Bits | Host Bits | Address Range                | Use Case         |
|-------|----------------|--------------|-----------|------------------------------|------------------|
| A     | `0`            | 8            | 24        | `0.0.0.0` to `127.255.255.255` | Large networks   |
| B     | `10`           | 16           | 16        | `128.0.0.0` to `191.255.255.255` | Medium networks  |
| C     | `110`          | 24           | 8         | `192.0.0.0` to `223.255.255.255` | Small networks   |
| D     | `1110`         | N/A          | N/A       | `224.0.0.0` to `239.255.255.255` | Multicasting     |
| E     | `1111`         | N/A          | N/A       | `240.0.0.0` to `255.255.255.255` | Experimental use |

> Only Classes A, B, and C are commonly used for device addressing. Classes D and E are reserved for specialized use cases.

### IPv4 Capacity

- IPv4 supports a total of **2³² = 4,294,967,296** unique addresses.
- However, after subtracting **reserved and unusable blocks** (e.g., private IP ranges, loopback, multicast), the **usable pool is significantly less than 4 billion**.

### IPv4 Exhaustion: Practical Solutions

#### 1. DHCP (Dynamic Host Configuration Protocol)

**DHCP** dynamically assigns IP addresses to devices **only when they connect** to the network.

**Benefits:**
- **Efficient IP usage**: Only connected devices occupy an IP address.
- **Automation**: No manual IP assignment required.
- **Flexibility**: Devices may receive different IPs on each connection, based on availability.

> Example: A laptop connecting to Wi-Fi may get a new IP address each time it connects.


#### 2. NAT (Network Address Translation)

**NAT** allows multiple devices in a **private local network** to share a **single public IP address**.
- Devices in a LAN use **private IP addresses** (e.g., `192.168.x.x`) not routable on the internet.
- When accessing the internet, the NAT device (usually a router) **replaces** the private IP with its **public IP**, maintaining a **NAT table** to track mappings.
- Incoming responses are translated back using this table and forwarded to the correct internal device.

**Benefits:**
- **IP conservation**: Many devices share one public IP.
- **Security**: Devices in the LAN are not directly exposed to the internet.


#### 3. IPv6: The Future of the Internet

IPv6 is the **next-generation Internet Protocol**, designed to replace IPv4 and permanently solve the address exhaustion problem.
- IPv6 uses **128-bit addresses**, allowing for `2^128` unique addresses — that's **340 undecillion** IPs!
- Enough to assign a **unique IP to every grain of sand on Earth**, and still have room to spare.

**Benefits:**
- No need for NAT — **each device can have a globally unique IP**.
- Improved routing efficiency and network configuration.
- Built-in security features and support for modern internet needs.


## 🌍 Normal Networking ConceptS

### MAC Address?
A **MAC address** (Media Access Control Address) is a **unique identifier assigned to a network interface card (NIC)**. It is often referred to as a **hardware address** or **physical address**.

- Each MAC address is designed to be **globally unique**. It is burned into the hardware by the device manufacturer and cannot be changed (though some systems allow MAC spoofing at the software level for testing or anonymity).

- A MAC address consists of **48 bits** (6 bytes).
- Represented as **12 hexadecimal digits**, typically separated by hyphens or colons.

> 📌 Example: `00-16-EA-AE-3C-40` or `00:16:EA:AE:3C:40`


### ARP (Address Resolution Protocol)
**ARP** is a critical network protocol used to map an **IP address (Layer 3)** to a **MAC address (Layer 2)**. It enables devices on a local network to dynamically discover each other's hardware addresses.

- When a device wants to communicate with another device on the same local network:
1. It sends a **broadcast ARP request** asking:  
   > “Who has IP `192.168.12.4`? Tell me your MAC address.”
2. The target device replies with its **MAC address**, which the sender caches in its **ARP table**.
3. The **ARP table** is maintained automatically—**no manual configuration** is needed.

- **Example ARP Table**: 192.168.12.4(IP Address) -> 00-16-3e-14-c4-4d( MAC Address)

### RARP (Reverse Address Resolution Protocol)
**RARP** performs the **opposite function of ARP**. It maps a **MAC address (Layer 2)** back to an **IP address (Layer 3)**.

- Used primarily in **diskless workstations or embedded devices** at boot time when only the MAC address is known. The device sends a **RARP request** to a RARP server, which responds with the assigned IP address.


## 🔍 IP Address vs. MAC Address

| Attribute       | MAC Address                       | IP Address                         |
|-----------------|------------------------------------|-------------------------------------|
| Layer           | Data Link Layer (Layer 2)          | Network Layer (Layer 3)             |
| Type            | Physical / Permanent               | Logical / Changeable                |
| Uniqueness      | Globally unique                    | Unique within a network             |
| Format          | 48-bit Hex                         | IPv4 / IPv6                         |
| Example         | `00:16:EA:AE:3C:40`                | `192.168.1.1`                       |
| Use             | Device identification in LAN       | Routing across networks             |


### Why Not Use Only MAC Addresses?

Technically, MAC addresses are globally unique, so why not just use them for everything?

#### Limitations:

- To route across multiple networks, we need **hierarchical addressing**.
- **Routers would have to store and manage 2^48 (~281 trillion) MAC entries**, which would require **over 256 TB of storage** — clearly **not scalable**.
- MAC addresses are fixed to the **local network** — they are **not routable** across the internet.

Therefore, **IP addresses** provide a scalable, logical addressing system, enabling **internet-wide routing**.


## 📦 Real-World Example

| Element               | Real World Analogy              |
|------------------------|----------------------------------|
| MAC Address           | National ID card (permanent)    |
| IP Address            | Temporary work permit/location  |

When a device connects to a network for the first time, **it doesn’t yet have an IP address**. It uses its **MAC address** to identify itself (e.g., via DHCP request) and get assigned an IP.








<br>


---

🚀 Stay tuned for more insights into networking fundamentals!