---
title: Basic of Computer Networking
date: 2025-02-22 20:19
categories: [Architecture, Computer Networking]
tags: [computer networking]
author: kai
---

# Computer Network Models

## OSI - 7 Layers and TCP/IP - 4 Layers
The **Open Systems Interconnection** (OSI) model is a 7-layer conceptual framework.

The **Transmission Control Protocol/Internet Protocol** (TCP/IP) model is a 4-layer practical implementation used in real-work networking, including the Internet.

<ul>
  <li><strong>Application Layer:</strong> Responsible for the communication between applications, such as HTTP, FTP, DNS, etc</li>
  <li><strong>Transport Layer:</strong> Responsible for data transmission between two hosts and end-to-end communication, such as TCP, UDP, etc</li>
  <li><strong>Internet Layer:</strong> Responsible for packet encapsulation, addressing, and routing, such as IP, etc</li>
  <li><strong>Network Access Layer:</strong> Responsible for transmitting network packets in the physical network, such as MAC address translation and transmitting network data frames through network interface, etc.</li>
</ul>

![Networking Models](/assets/img/posts/architecture/network%20model.png)

### Why layers?
{% capture why_layer %}why-layer{% endcapture %}
<details id="{{ why_layer }}">
  <summary><strong>No Layers</strong></summary>
  <p>
    - The application transfers data into binary and requires writing code to manipulate MAC to send data.<br>
    - More code is needed to establish networking connections, handle addressing, ensure reliable transmission, and manage failure retransmission, etc.
  </p>
</details>
<details id="{{ why_layer }}">
  <summary><strong>Layers</strong></summary>
   <p>
    - Each layer has a clear division of responsibility. (Single Responsibility and Chain of Responsibility Pattern)<br>
    - Developers are responsible for writing <strong>application-layer</strong> business logic.<br>
    - The OS is responsible for establishing newtwork connection and reliable transmission.<br>
    - Switches and routers are responsible for transmitting binary data over the physical medium.
    </p>
</details>


More on editing...








