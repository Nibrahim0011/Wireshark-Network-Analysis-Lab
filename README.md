# Lab 02 — Wireshark & Network Analysis

> 🎥 **Loom Walkthrough:** (https://www.loom.com/share/60cadc7d66594bbfaccc9149fa7dc6ba)


**Hands-on network traffic analysis using Wireshark and Windows PowerShell**

**Tools:** Wireshark, Npcap, Windows PowerShell  
**Environment:** Windows PC, Wi-Fi interface and Npcap loopback capture  
**Focus:** DNS, TCP, HTTP, TLS, display filters, packet inspection, and stream analysis

## Overview

In this lab I captured and analyzed network traffic to understand how hosts resolve domains, establish connections, transmit application data, and protect it with encryption. I used Wireshark filters to isolate relevant packets and PowerShell to generate test traffic.

## Architecture

```text
Internet services ── Router ── Wi-Fi NIC ── Wireshark
                                  |
                           DNS / TCP / TLS

PowerShell test client ── 127.0.0.1:8080 ── Local HTTP listener
                              |
                      Npcap loopback capture
```

## Exercises and Results

### 1. DNS resolution

Ran `nslookup yahoo.com` in PowerShell and inspected the returned IPv4/IPv6 addresses. The configured DNS resolver was `192.168.1.1`.

<img width="490" height="374" alt="01-DNS-Lookup" src="https://github.com/user-attachments/assets/d052b89a-d835-47d7-a554-6032327e87cd" />

### 2. Capture and filter network traffic

Captured live traffic over Wi-Fi and used Wireshark display filters to narrow down traffic by protocol, TCP flags, port, and address.

<img width="2048" height="998" alt="02-Live-Packet-Capture" src="https://github.com/user-attachments/assets/0b3c9756-fc35-4186-bb77-8f49072dd180" />

Useful filters:

```wireshark
dns
tcp
tcp.flags.syn == 1
http.request.method == "POST"
tcp.len > 0
```

<img width="2048" height="805" alt="03-TCP-SYN-Filter" src="https://github.com/user-attachments/assets/8893f6c2-a130-4861-bd5f-675094541294" />


### 3. TCP three-way handshake

Isolated one TCP connection using the display filter:

```wireshark
tcp.port == 57554 && ip.addr == 104.18.39.21
```

The capture shows the full handshake:

| Packet | Direction | TCP flag | Interpretation |
|---|---|---|---|
| #337 | Local host → remote server | SYN | Client initiates connection |
| #338 | Remote server → local host | SYN, ACK | Server acknowledges |
| #339 | Local host → remote server | ACK | Client confirms |

The subsequent packets include a TLS 1.3 Client Hello and Server Hello. **This server is not identified as Yahoo; it is a separate captured connection.**
<img width="2048" height="1059" alt="04-TCP-Three-Way-Handshake" src="https://github.com/user-attachments/assets/6894597d-0648-4653-b420-727915970b23" />


### 4. HTTP plaintext data demonstration

Created a temporary **local** HTTP listener in PowerShell and submitted **dummy** test credentials over `127.0.0.1:8080`. The PowerShell output and Wireshark capture showed that form data sent over plain HTTP can be read without TLS encryption. The test server returned HTTP `200 OK`.

<img width="748" height="334" alt="05-HTTP-Test-Server" src="https://github.com/user-attachments/assets/7e6eae73-b45e-4258-99e3-0228d5e0eeb8" />

<img width="2048" height="1110" alt="06-HTTP-Cleartext-POST" src="https://github.com/user-attachments/assets/97819b26-35c3-4ed5-86fb-3846e87eeceb" />

**Security lesson:** HTTP does not provide transport encryption; HTTPS adds TLS to protect application contents in transit. Only test with invented credentials and authorized endpoints.

### 5. Follow TCP stream

Selected a TLS application-data packet and used **Follow → TCP Stream** to inspect the two directions of a connection. The stream display showed non-readable bytes, consistent with encrypted traffic. This demonstrates stream following and the visibility difference between plaintext HTTP and TLS-protected communication; **a readable HTTP request-and-response stream was not captured in this exercise**.

<img width="2048" height="1248" alt="07-TCP-Stream-Analysis" src="https://github.com/user-attachments/assets/f1328f0f-55e7-45b1-8049-f7e7d335c5a1" />


## Key Takeaways

- Display filters reveal relevant packets without deleting the underlying capture.
- TCP's `SYN → SYN, ACK → ACK` sequence establishes a connection before application exchange.
- DNS maps names to addresses, while TCP and TLS perform different roles in connectivity and security.
- Local HTTP test credentials were visible in packet data; TLS application data was not readable as plaintext.
- Following a TCP stream can group exchanged bytes by conversation, but it does not decrypt HTTPS automatically.

  
