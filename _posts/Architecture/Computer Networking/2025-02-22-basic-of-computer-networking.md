---
title: Basic Concepts of Computer Networking
date: 2025-02-22 20:19
categories: [Architecture, Computer Networking]
tags: [computer networking]
author: kai
---

# Computer Network Models

## OSI - 7 Layers and TCP/IP - 4 Layers

The **Open Systems Interconnection (OSI)** model is a conceptual framework that standardizes networking functions into **seven layers**.

The **Transmission Control Protocol/Internet Protocol (TCP/IP)** model is a more practical, real-world implementation with **four layers**, forming the foundation of the Internet.

### **Comparison of Network Models**
<ul>
  <li><strong>Application Layer:</strong> Manages communication between applications, supporting protocols like HTTP, FTP, and DNS.</li>
  <li><strong>Transport Layer:</strong> Handles end-to-end communication between hosts, ensuring reliable or fast data transfer through TCP and UDP.</li>
  <li><strong>Internet Layer:</strong> Responsible for packet encapsulation, addressing, and routing, using protocols such as IP.</li>
  <li><strong>Network Access Layer:</strong> Manages physical network communication, including MAC address translation and data transmission via network interfaces.</li>
</ul>

![Networking Models](/assets/img/posts/architecture/NetworkModel.png)

---

### Why Use Layers?

{% capture why_layer %}why-layer{% endcapture %}
<details id="{{ why_layer }}">
  <summary><strong>Without Layered Architecture</strong></summary>
  <pre>
  - Applications must handle low-level networking tasks, such as converting data to binary and controlling MAC address transmission.
  - Developers need to write additional code for networking operations, including connection management, addressing, reliability, and retransmission.
  </pre>
</details>

<details id="{{ why_layer }}">
  <summary><strong>With Layered Architecture</strong></summary>
  <pre>
  - Each layer has a clearly defined responsibility, following the Single Responsibility and Chain of Responsibility design patterns.
  - Developers focus on writing application-layer business logic without managing low-level network operations.
  - The operating system handles network connections and ensures reliable data transmission.
  - Network devices (switches, routers) manage the transmission of binary data over the physical network.
  </pre>
</details>

### Encapsulated packets like nested dolls
![Networking Models](/assets/img/posts/architecture/SimpleNetworkPacketFormat.png)





---

Stay tuned for more insights into networking fundamentals!