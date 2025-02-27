---
title: Basic Concepts of Computer Networking
date: 2025-02-22 20:19
categories: [Architecture, Computer Networking]
tags: [computer networking]
author: kai
---

# Computer Network Models

## OSI - 7 Layers and TCP/IP - 4 Layers

The **Open Systems Interconnection (OSI)** model is a conceptual framework that standardizes networking functions into **seven layers**. (Load Balancing Under High Concurrency)

The **Transmission Control Protocol/Internet Protocol (TCP/IP)** model is a more practical, real-world implementation with **four layers**, forming the foundation of the Internet. (Encrypted Network Transmission - HTTPS incorporates the SSL/TLS protocol between the HTTP and TCP layers.)

### **Comparison of Network Models**
- **Application Layer:** Manages communication between applications, supporting protocols like HTTP, FTP, and DNS.
- **Transport Layer:** Handles end-to-end communication between hosts, ensuring reliable or fast data transfer through TCP and UDP.
- **Internet Layer:** Responsible for packet encapsulation, addressing, and routing, using protocols such as IP.
- **Network Access Layer:** Manages physical network communication, including MAC address translation and data transmission via network interfaces.

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
## Network Layered Models and Key Protocols
![Networking Models](/assets/img/posts/architecture/NetworkLayeredModelsandKeyProtocols.png)
### Data Transmission Between Layers in the Network Layered Model
Similar to express delivery, where parcels are distributed step by step through transfer stations: Province → City → County → District → Village → House Number → Specific Recipient.

- Sending Data Packets
  - The data is processed layer by layer from top to bottom within the network protocol stack until it reaches the network card for transmission.
- Receiving Data Packets
	- The data goes through layer-by-layer processing from bottom to top in the network protocol stack before being delivered to the application for use.
- Notes
	- The Application Layer is the layer that directly interacts with users, providing a unified protocol interface for applications, but it is not the application itself.
	- The goal is to ensure that different types of applications follow consistent lower-layer communication protocols, enforcing certain constraints.
  ![Networking Models](/assets/img/posts/architecture/DataTransmissionBetweenLayers.png)

### Transport Layer VS Internet Layer
- **Internet Layer** protocols provide **host-to-host** logical communication.
	- There are numerous devices on the Internet. How does a device determine where to send data? Why doesn’t it mistakenly send data to other machines? → **IP addresses identify communication hosts, allowing the network layer to pinpoint the exact device.**
	•	The network layer only verifies the checksum field in the IP datagram header for errors but does not check the data payload. The checksum field in the IP datagram header is a mechanism used to detect errors in the IP header. It ensures that the data in the header has not been corrupted during transmission.



---

Stay tuned for more insights into networking fundamentals!