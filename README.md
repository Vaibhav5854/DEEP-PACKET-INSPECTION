# Deep Packet Inspection (DPI) Engine

A high-performance **Deep Packet Inspection (DPI) Engine** developed in **C++17** for analyzing offline network traffic stored in **PCAP** files. The project classifies network applications, extracts domain information from encrypted HTTPS traffic using TLS Server Name Indication (SNI), applies user-defined filtering rules, and generates a filtered PCAP file along with detailed traffic statistics.

---

# Project Overview

Traditional packet filtering systems mainly inspect packet headers such as source IP address, destination IP address, and protocol number. Modern networks require more intelligent inspection capable of identifying applications even when traffic is encrypted.

This project implements a simplified Deep Packet Inspection engine that analyzes packet payloads to identify applications such as YouTube, Facebook, Google, GitHub, TikTok, Discord, Spotify, Zoom, HTTP websites, and DNS traffic. The engine processes an input PCAP file packet-by-packet, classifies network flows, applies blocking rules, and writes only the permitted packets into a new PCAP file.

The project demonstrates practical concepts used in enterprise firewalls, traffic monitoring systems, and network security appliances.

---

# Problem Statement

Traditional packet filtering methods rely mainly on source and destination IP addresses, protocol numbers, and port numbers to control network traffic. However, modern web applications such as YouTube, Facebook, GitHub, Netflix, and many others primarily use HTTPS encryption over the same port (443), making them difficult to distinguish using conventional filtering techniques. This limits the ability of network administrators to selectively monitor or block specific services.

To address this problem, this project implements a **Deep Packet Inspection (DPI) Engine** that analyzes network packets stored in PCAP files. The engine parses packet headers, extracts HTTP Host headers and TLS Server Name Indication (SNI) from HTTPS traffic, identifies the application associated with each network flow, and applies user-defined filtering rules based on source IP address, application, or domain name. Packets that satisfy the filtering rules are written to a new PCAP file, while blocked packets are discarded. The system also generates detailed traffic statistics, providing insights into network usage and demonstrating practical techniques used in modern network monitoring and security systems.

---

# Objectives

* Understand packet parsing and network protocols.
* Implement Deep Packet Inspection techniques.
* Classify encrypted and unencrypted network traffic.
* Track network flows using the Five Tuple.
* Apply rule-based traffic filtering.
* Generate filtered PCAP files.
* Demonstrate multi-threaded packet processing.
* Learn practical concepts used in network security systems.

---

# Features

* Reads standard PCAP files.
* Parses Ethernet, IPv4, TCP and UDP packets.
* Extracts HTTP Host headers.
* Extracts TLS Server Name Indication (SNI).
* Identifies common web applications.
* Tracks complete network flows.
* Blocks traffic using:

  * Source IP address
  * Application name
  * Domain name
* Generates filtered PCAP output.
* Displays detailed processing statistics.
* Multi-threaded implementation for improved performance.

---

# How the DPI Engine Works

```
                Input PCAP
                    │
                    ▼
           Read Network Packets
                    │
                    ▼
         Parse Ethernet/IP/TCP/UDP
                    │
                    ▼
      Extract HTTP Host or TLS SNI
                    │
                    ▼
        Identify Network Application
                    │
                    ▼
        Apply User-Defined Rules
                    │
         ┌──────────┴──────────┐
         │                     │
         ▼                     ▼
   Write to Output PCAP      Drop Packet
                    │
                    ▼
         Generate Traffic Report
```

---

# Network Protocols Used

| Layer       | Protocol               |
| ----------- | ---------------------- |
| Data Link   | Ethernet               |
| Network     | IPv4                   |
| Transport   | TCP, UDP               |
| Application | HTTP, HTTPS (TLS), DNS |

---

# Application Identification

Instead of decrypting HTTPS traffic, the engine extracts the **TLS Server Name Indication (SNI)** from the TLS Client Hello packet.

Example:

```
TLS Client Hello

Server Name:
www.youtube.com
```

The extracted domain is mapped to an application.

```
www.youtube.com
        │
        ▼
    YouTube
```

Similarly,

```
www.facebook.com
        │
        ▼
    Facebook
```

This allows encrypted traffic to be classified without decrypting packet contents.

---

# Blocking Rules

The DPI Engine supports three filtering methods.

### Block by Application

```
--block-app YouTube
```

Blocks every flow identified as YouTube.

### Block by Source IP

```
--block-ip 192.168.1.50
```

Blocks all traffic originating from the specified IP address.

### Block by Domain

```
--block-domain facebook
```

Blocks any connection whose domain name contains "facebook".

---

# Folder Structure

```
packet_analyzer/

├── include/
│   ├── pcap_reader.h
│   ├── packet_parser.h
│   ├── sni_extractor.h
│   ├── types.h
│   ├── rule_manager.h
│   ├── connection_tracker.h
│   ├── load_balancer.h
│   ├── fast_path.h
│   ├── thread_safe_queue.h
│   └── dpi_engine.h
│
├── src/
│   ├── dpi_mt.cpp
│   ├── main_working.cpp
│   ├── pcap_reader.cpp
│   ├── packet_parser.cpp
│   ├── sni_extractor.cpp
│   └── types.cpp
│
├── test_dpi.pcap
├── output.pcap
├── generate_test_pcap.py
├── README.md
└── WINDOWS_SETUP.md
```

---

# Project Workflow

1. Read packets from the input PCAP file.
2. Parse Ethernet, IPv4, TCP or UDP headers.
3. Create a Five Tuple to identify each network flow.
4. Extract HTTP Host headers or TLS SNI information.
5. Classify the application associated with the flow.
6. Apply filtering rules.
7. Forward allowed packets to the output PCAP file.
8. Drop blocked packets.
9. Generate traffic statistics.

---

### Build Commands

**Simple Version:**
```cmd
g++ -std=c++17 -O2 -I include -o dpi_simple ^
    src/main_working.cpp ^
    src/pcap_reader.cpp ^
    src/packet_parser.cpp ^
    src/sni_extractor.cpp ^
    src/types.cpp
```

**Multi-threaded Version:**
```cmd
g++ -std=c++17 -pthread -O2 -I include -o dpi_engine ^
    src/dpi_mt.cpp ^
    src/pcap_reader.cpp ^
    src/packet_parser.cpp ^
    src/sni_extractor.cpp ^
    src/types.cpp
```

### Running

**Basic usage:**
```cmd
dpi_engine.exe test_dpi.pcap output.pcap
```

**With blocking:**
```cmd
dpi_engine.exe test_dpi.pcap output.pcap ^
    --block-app YouTube ^
    --block-app TikTok ^
    --block-ip 192.168.1.50 ^
    --block-domain facebook
```

**Configure threads (multi-threaded only):**
```cmd
dpi_engine.exe input.pcap output.pcap --lbs 4 --fps 4
# Creates 4 LB threads × 4 FP threads = 16 processing threads
```

### Creating Test Data

```cmd
python generate_test_pcap.py
# Creates test_dpi.pcap with sample traffic
```

---

# Output

The engine generates two outputs.

### 1. Filtered PCAP File

The filtered packets are written into

```
output.pcap
```

This file contains only the packets that satisfy the filtering rules and can be opened using Wireshark for further analysis.

### 2. Processing Report

Example

```
==========================================
DPI ENGINE REPORT
==========================================

Total Packets      : 77
TCP Packets        : 73
UDP Packets        : 4

Forwarded Packets  : 69
Dropped Packets    : 8

Blocked Applications

- YouTube

Blocked IP

- 192.168.1.50

Detected Applications

HTTPS
HTTP
DNS
Google
YouTube
Facebook
GitHub
Discord
Spotify
Zoom

Detected Domains

www.youtube.com
www.facebook.com
www.google.com
github.com

(domains which i used to make this project)
(you can check the traffics and how it is stored by running the test_dpi.pcap file in wireshark..)

```

The report provides:

* Total packets processed
* TCP and UDP packet count
* Forwarded packets
* Dropped packets
* Active blocking rules
* Application-wise traffic distribution
* Detected domains
* Thread statistics (multi-threaded version)


# Technologies Used

* C++17
* Standard Template Library (STL)
* Multi-threading
* PCAP File Format
* TCP/IP Networking
* TLS Protocol
* Git
* MinGW / GCC

---

# Learning Outcomes

This project provided practical experience in:

* Computer Networks
* Packet Parsing
* Deep Packet Inspection
* Ethernet and IPv4 Protocols
* TCP and UDP Protocols
* TLS Handshake
* HTTP Protocol
* Server Name Indication (SNI)
* Flow Tracking
* Multi-threading
* Mutexes and Thread Synchronization
* Producer-Consumer Architecture
* Rule-based Packet Filtering
* Network Security

---

# Conclusion

This project demonstrates the implementation of a **Deep Packet Inspection (DPI) Engine** capable of parsing network packets, identifying applications from encrypted and unencrypted traffic, tracking network flows, applying user-defined filtering rules, and generating a filtered PCAP file along with detailed traffic statistics. It provides practical exposure to packet processing, network protocol analysis, concurrent programming, and modern network security techniques commonly used in enterprise firewalls and traffic management systems.


Vaibhav Raj
B.tech, BIT Mesra
