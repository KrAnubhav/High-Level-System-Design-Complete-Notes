# HLD-01: Network Protocols

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [What are Network Protocols?](#what-are-network-protocols)
3. [OSI Model Layers](#osi-model-layers)
4. [Application Layer Protocols](#application-layer-protocols)
   - [Client-Server Protocols](#client-server-protocols)
   - [Peer-to-Peer Protocols](#peer-to-peer-protocols)
5. [Client-Server Model](#client-server-model)
6. [HTTP - Hypertext Transfer Protocol](#http---hypertext-transfer-protocol)
7. [WebSocket Protocol](#websocket-protocol)
8. [FTP - File Transfer Protocol](#ftp---file-transfer-protocol)
9. [SMTP - Simple Mail Transfer Protocol](#smtp---simple-mail-transfer-protocol)
10. [Peer-to-Peer Model](#peer-to-peer-model)
11. [WebRTC Protocol](#webrtc-protocol)
12. [Transport Layer Protocols](#transport-layer-protocols)
    - [TCP/IP](#tcpip)
    - [UDP/IP](#udpip)
13. [When to Use Which Protocol](#when-to-use-which-protocol)
14. [Summary](#summary)

---

## Introduction

Network protocols are fundamental to high-level system design. Understanding which protocol to use for different applications is crucial:
- **WhatsApp/Telegram** → Which protocol?
- **Google Meet/Video Calling** → Which protocol?
- **Live Streaming** → Which protocol?

This lecture covers the essential network protocols needed for system design interviews.

---

## What are Network Protocols?

**Network Protocol** = Rules and regulations that define how two systems communicate over a network.

**Purpose:**
- Enable communication between two computers/systems
- Define the format and structure of messages
- Even if both systems "speak the same language," protocols ensure proper communication

---

## OSI Model Layers

The OSI model has **7 layers**:

```
┌─────────────────────────────┐
│   Application Layer         │ ← We focus here
├─────────────────────────────┤
│   Presentation Layer        │
├─────────────────────────────┤
│   Session Layer             │
├─────────────────────────────┤
│   Transport Layer           │ ← We focus here
├─────────────────────────────┤
│   Network Layer             │
├─────────────────────────────┤
│   Data Link Layer           │
├─────────────────────────────┤
│   Physical Layer            │
└─────────────────────────────┘
```

**In this lecture, we focus on:**
1. **Application Layer**
2. **Transport Layer**

---

## Application Layer Protocols

Application layer protocols are divided into two categories:

### Client-Server Protocols
- HTTP
- FTP
- SMTP
- WebSockets

### Peer-to-Peer Protocols
- WebRTC

**Most Important for HLD:**
- HTTP
- WebSockets
- WebRTC

---

## Client-Server Model

### Basic Structure

```
┌─────────────────┐              ┌─────────────────┐
│   Web Browser   │              │   Web Server    │
│    (Client)     │              │    (Server)     │
└─────────────────┘              └─────────────────┘
         │                                │
         │──────── Request ──────────────>│
         │                                │
         │<──────── Response ─────────────│
         │                                │
```

**Key Characteristics:**
- **One-way communication** (client initiates)
- Client makes a **request**
- Server sends a **response**
- Client always initiates the conversation

**Examples:** HTTP, FTP, SMTP

---

## HTTP - Hypertext Transfer Protocol

**Characteristics:**
- **Connection-oriented** protocol
- A connection is created between client and server
- Used for accessing web pages
- Hypertext allows jumping from one page to another

```
┌─────────┐         Connection         ┌─────────┐
│ Client  │<─────────────────────────>│ Server  │
└─────────┘                            └─────────┘
```

**Usage:** Web browsing, API calls, REST services

---

## WebSocket Protocol

### Structure

```
┌──────────┐                           ┌──────────┐
│ Client 1 │<────────────────────────>│  Server  │
└──────────┘                           └──────────┘
                                             ↕
                                       ┌──────────┐
                                       │ Client 2 │
                                       └──────────┘
```

**Key Characteristics:**
- **Bidirectional communication**
- Client can talk to Server
- Server can talk to Client
- **Still Client-Server model** (NOT peer-to-peer)

### Important Clarification

**Common Mistake:** Calling WebSocket "peer-to-peer"

**Reality:**
- Client 1 ↔ Server ✅
- Client 2 ↔ Server ✅
- Client 1 ↔ Client 2 ❌ (They do NOT talk directly)

### When to Use WebSocket?

**Use Case:** Messaging applications (WhatsApp, Telegram)

**Why?**
- When a message arrives at the server, the server needs to push it to the client
- Client shouldn't keep asking "Did message come? Did message come?"
- That would be inefficient
- Server needs the ability to initiate communication with the client

```
User 1 sends message → Server → Server pushes to User 2
                                 (Server-to-Client communication needed)
```

---

## FTP - File Transfer Protocol

**Characteristics:**
- **Two connections** are maintained:
  1. **Control Connection** (always remains)
  2. **Data Connection** (can be created and disconnected)

```
┌─────────┐                            ┌─────────┐
│ Client  │                            │ Server  │
└─────────┘                            └─────────┘
     │                                      │
     │═══════ Control Connection ══════════│ (Always on)
     │                                      │
     │------- Data Connection -------------│ (On/Off)
     │                                      │
```

### Why FTP is NOT Used Anymore?

**Security Issue:**
- Data connection is **NOT encrypted**
- File transfer happens in plain text
- Not secure for modern applications

**Alternative:** HTTPS (Hypertext Transfer Protocol Secure)

---

## SMTP - Simple Mail Transfer Protocol

**Used with:**
- **IMAP** (Internet Message Access Protocol)
- **POP3** (Post Office Protocol 3)

### Roles

| Protocol | Purpose |
|----------|---------|
| **SMTP** | Sending emails |
| **IMAP** | Receiving/Accessing/Reading emails |
| **POP3** | Downloads and deletes from server (rarely used now) |

**Modern Standard:** SMTP + IMAP

### Email Flow Diagram

```
┌──────────┐         ┌─────────────┐         ┌─────────────┐         ┌──────────┐
│   User   │────────>│ User Agent  │────────>│ MTA Client  │────────>│   MTA    │
│ (Sender) │         │  (Client)   │         │   (Client)  │         │ (Server) │
└──────────┘         └─────────────┘         └─────────────┘         └──────────┘
                                                                            │
                                                                            │
                                                                            ▼
┌──────────┐         ┌─────────────┐         ┌─────────────────────────────────┐
│   User   │<────────│ User Agent  │<────────│  Message Transfer Agent (MTA)   │
│(Receiver)│         │  (Client)   │         │           (Server)              │
└──────────┘         └─────────────┘         └─────────────────────────────────┘

Mail Send ──────────────────────────────────────────> Mail Receive
```

**Key Point:** Still follows Client-Server model
- User Agent = Client
- MTA = Server

---

## Peer-to-Peer Model

### Structure

```
┌──────────┐
│ Client 1 │
│ (Server) │
└──────────┘
     │ ╲
     │   ╲
     │     ╲
     │       ╲
     ↕         ↕
┌──────────┐   ┌──────────┐
│  Server  │   │ Client 2 │
│ (Client) │   │ (Server) │
└──────────┘   └──────────┘
```

**Characteristics:**
- All nodes can talk to each other
- Everyone can be a **client**
- Everyone can be a **server**
- Whoever has the data acts as server
- Direct communication between peers

---

## WebRTC Protocol

**WebRTC** = Web Real-Time Communication

**Type:** Peer-to-Peer Protocol

**Key Features:**
- Does NOT need to go through a server
- Direct communication between two machines
- **Fast** communication

### Why is WebRTC Fast?

```
Traditional (Client-Server):
Client 1 → Server → Client 2
(2 hops, slower)

WebRTC (Peer-to-Peer):
Client 1 ←─────────→ Client 2
(Direct, faster)
```

**Use Cases:**
- Video calling (Google Meet, Zoom)
- Live streaming
- Real-time communication

---

## Transport Layer Protocols

Two main protocols:
1. **TCP/IP** (Transmission Control Protocol)
2. **UDP/IP** (User Datagram Protocol)

---

## TCP/IP

### How TCP Works

```
Step 1: Virtual Connection
┌─────────┐ ═══════════════════════ ┌─────────┐
│ Client  │   Virtual Connection    │ Server  │
└─────────┘ ═══════════════════════ └─────────┘

Step 2: Data Packets with Sequence
┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
│ 1 │ │ 2 │ │ 3 │ │ 4 │ │ 5 │
└───┘ └───┘ └───┘ └───┘ └───┘
  │     │     │     │     │
  └─────┴─────┴─────┴─────┘
         │
         ▼
    Through Virtual
     Connection

Step 3: Server Side (Ordering Maintained)
Received: 2, 1, 4, 5 (3 missing)
After Ordering: 1, 2, _, 4, 5

Step 4: Acknowledgement
Server sends ACK for: 1 ✓, 2 ✓, 3 ✗, 4 ✓, 5 ✓
Client resends: 3
```

### TCP Characteristics

✅ **Connection-oriented**
- Virtual connection is maintained

✅ **Ordering maintained**
- Packets are sequenced
- Data arrives in correct order

✅ **Acknowledgement**
- Every packet is acknowledged
- If ACK not received → Packet is resent

✅ **Reliable**
- Guarantees data delivery
- No data loss

❌ **Slower**
- Due to connection overhead, ordering, and acknowledgements

---

## UDP/IP

### How UDP Works

```
Step 1: No Connection Maintained
┌─────────┐                        ┌─────────┐
│ Client  │                        │ Server  │
└─────────┘                        └─────────┘

Step 2: Data Divided into Datagrams
┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
│ 1 │ │ 2 │ │ 3 │ │ 4 │ │ 5 │
└───┘ └───┘ └───┘ └───┘ └───┘

Step 3: Sent on Multiple Virtual Connections (Parallel)
  │     │     │     │     │
  │     │     │     │     │
  ├─────┤     │     ├─────┤
  │     └─────┼─────┘     │
  │           │           │
  ▼           ▼           ▼
Multiple Virtual Connections

Step 4: Server Side (No Ordering)
Received: 3, 1, 5, 2, 4 (Random order)
No reordering happens
```

### UDP Characteristics

✅ **No connection maintained**
- Connectionless protocol

✅ **Parallel transmission**
- Uses multiple virtual connections
- Sends data simultaneously

❌ **No ordering**
- Packets may arrive out of order
- First sent may arrive last

❌ **No acknowledgement**
- No guarantee of delivery
- Packet loss possible

✅ **Fast**
- No overhead of connection, ordering, acknowledgement
- **Best effort mechanism**

### When to Use UDP?

**Use Cases:**
- **Live streaming**
- **Video calling**
- **Real-time applications**

**Why?**
- If some data is missed during video call → Keep moving forward
- Don't wait for missed data
- Speed is more important than reliability

**Example:**
```
Video Call Scenario:
Frame 1, Frame 2, Frame 3 (missed), Frame 4, Frame 5...
                    ↑
              We don't go back to get Frame 3
              We continue with Frame 4, 5...
```

---

## When to Use Which Protocol

### Protocol Selection Guide

| Scenario | Protocol | Reason |
|----------|----------|--------|
| **Messaging App** (WhatsApp, Telegram) | **WebSocket** | Two-way communication needed. Server must push messages to client. |
| **Video Calling** (Google Meet, Zoom) | **WebRTC + UDP** | Fast, peer-to-peer, real-time. Packet loss acceptable. |
| **Live Streaming** | **UDP** | Speed is critical. Minor data loss acceptable. |
| **Web Browsing** | **HTTP/HTTPS** | Request-response model. Security with HTTPS. |
| **File Transfer** | **HTTPS** (NOT FTP) | FTP is insecure (unencrypted data connection). |
| **Email** | **SMTP + IMAP** | SMTP for sending, IMAP for receiving. |

### WebSocket Use Case Explained

```
Messaging App Architecture:

User 1                    Server                    User 2
  │                         │                         │
  │──── Send Message ──────>│                         │
  │                         │                         │
  │                         │──── Push Message ──────>│
  │                         │    (Server to Client)   │
  │                         │                         │
```

**Why WebSocket?**
- Server needs to push messages to Client 2
- Client 2 shouldn't poll: "Message arrived? Message arrived?"
- Inefficient to keep asking
- **Bidirectional communication** solves this

### WebRTC Use Case Explained

```
Video Calling:

Traditional Approach (Slow):
User 1 → Server → User 2
       ↑        ↓
    Latency  Latency

WebRTC Approach (Fast):
User 1 ←─────────→ User 2
    Direct P2P Connection
```

**Why WebRTC?**
- Direct peer-to-peer connection
- No server in between
- **Faster** communication
- Uses **UDP** at transport layer

### TCP vs UDP Decision

**Use TCP when:**
- Data reliability is critical
- Ordering matters
- No data loss acceptable
- Example: File downloads, database transactions

**Use UDP when:**
- Speed is more important than reliability
- Real-time communication
- Minor data loss acceptable
- Example: Video calls, live streaming, online gaming

---

## Summary

### Key Takeaways

1. **Network Protocols** define rules for system communication

2. **Application Layer Protocols:**
   - **Client-Server:** HTTP, FTP, SMTP, WebSocket
   - **Peer-to-Peer:** WebRTC

3. **Client-Server Model:**
   - Client initiates, server responds
   - One-way (HTTP, FTP, SMTP)
   - Two-way (WebSocket)

4. **Peer-to-Peer Model:**
   - All nodes can be client and server
   - Direct communication
   - Fast (WebRTC)

5. **Transport Layer:**
   - **TCP:** Reliable, ordered, slower
   - **UDP:** Fast, unordered, best-effort

6. **Protocol Selection:**
   - Messaging apps → **WebSocket**
   - Video calling → **WebRTC + UDP**
   - Web browsing → **HTTP/HTTPS**
   - Live streaming → **UDP**

7. **Security:**
   - FTP is insecure (unencrypted data connection)
   - Use HTTPS instead

### Quick Reference

```
┌─────────────────────────────────────────────────────────────┐
│                    Protocol Decision Tree                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Need two-way communication?                                │
│  └─ YES → WebSocket (Messaging apps)                        │
│  └─ NO → HTTP (Web browsing, APIs)                          │
│                                                             │
│  Need peer-to-peer?                                         │
│  └─ YES → WebRTC (Video calling)                            │
│  └─ NO → Client-Server model                                │
│                                                             │
│  Need speed over reliability?                               │
│  └─ YES → UDP (Live streaming, video calls)                 │
│  └─ NO → TCP (File transfer, data integrity)                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

**End of Lecture**

This foundation is crucial for understanding high-level system design. The next lectures will build upon these protocol concepts.
