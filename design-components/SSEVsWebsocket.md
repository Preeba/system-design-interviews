# **Server-Sent Events (SSE) vs WebSockets**

---

## SSE vs WebSockets

### **Server-Sent Events (SSE)**

* **Protocol**: Built on top of standard HTTP (one-way stream).
* **Direction**: Server → Client only (unidirectional).
* **Transport**: Text/event-stream over HTTP.
* **Use case**:

    * Real-time updates that only flow *from server to client*.
    * Examples: live sports scores, stock tickers, notification feeds.
* **Pros**:

    * Simple (works over plain HTTP).
    * Automatic reconnection built-in.
    * Lower overhead than WebSockets for one-way data.
* **Cons**:

    * No client → server push (client must still use HTTP requests).
    * Limited browser support for binary data (mostly text).

---

### **WebSockets**

* **Protocol**: Upgrades HTTP connection to a full-duplex TCP connection.
* **Direction**: Bi-directional (Client ↔ Server).
* **Transport**: Works outside HTTP after the initial handshake.
* **Use case**:

    * Real-time applications needing **two-way communication**.
    * Examples: chat apps, collaborative editing, multiplayer games.
* **Pros**:

    * True two-way, low-latency communication.
    * Supports text and binary data.
* **Cons**:

    * More complex to implement and scale (stateful connections).
    * Requires load balancer / infra that supports sticky sessions.

---

### **When to use what**

* **Use SSE** if → You only need **server → client streaming updates** (simpler, lightweight).
* **Use WebSockets** if → You need **two-way, interactive communication** or binary data.

---

👉 Example in TicketMaster context:

* **SSE**: Notify users in the virtual waiting queue about their position or when seats become available.
* **WebSocket**: Enable **real-time collaboration** (e.g., friends picking seats together, chat with support).

---
### **Comparison table (SSE vs WebSocket)** 

Here’s a crisp **comparison table** you can keep in mind for interviews:

| Feature          | **SSE (Server-Sent Events)**               | **WebSockets**                          |
|------------------|--------------------------------------------|-----------------------------------------|
| **Protocol**     | HTTP (text/event-stream)                   | HTTP upgrade → persistent TCP           |
| **Direction**    | One-way (Server → Client)                  | Two-way (Client ↔ Server)               |
| **Data type**    | Text (UTF-8)                               | Text & Binary                           |
| **Complexity**   | Simple to implement                        | More complex (connection mgmt, scaling) |
| **Scalability**  | Easier (uses plain HTTP infra)             | Harder (needs sticky sessions / state)  |
| **Reconnects**   | Built-in auto-reconnect                    | Must be handled manually                |
| **Best for**     | Notifications, feeds, streaming updates    | Chats, games, collaborative apps        |
| **Limitations**  | No client → server push, no binary support | More overhead, harder to load-balance   |

---

👉 **Rule of thumb**:

* Use **SSE** when you only need server-to-client updates (lighter, simpler).
* Use **WebSockets** when you need true two-way communication.

---

## Comparison Table of Key Protocols (System Design Relevant)

| Protocol      | Layer      | Direction         | Reliability    | Use Cases                   | Pros                                 | Cons                              |
|---------------|------------|-------------------|----------------|-----------------------------|--------------------------------------| --------------------------------- |
| **TCP**       | Transport  | Bi-directional    | Reliable       | Web traffic, apps           | Guarantees delivery/order            | Higher latency than UDP           |
| **UDP**       | Transport  | Bi-directional    | Unreliable     | DNS, streaming, gaming      | Low overhead, fast                   | No guarantee of delivery/order    |
| **QUIC**      | Transport  | Bi-directional    | Reliable       | HTTP/3, low-latency apps    | Faster, connection migration         | Newer, less mature infra support  |
| **HTTP/1.1**  | App        | Client → Server   | Reliable       | REST APIs, web pages        | Simple, widely supported             | Head-of-line blocking             |
| **HTTP/2**    | App        | Client → Server   | Reliable       | Modern APIs, web pages      | Multiplexing, header compression     | Harder debugging, infra upgrade   |
| **HTTP/3**    | App        | Client → Server   | Reliable       | High-perf APIs, browsers    | Built on QUIC, lower latency         | Still rolling out globally        |
| **WebSocket** | App        | Bi-directional    | Reliable       | Chats, multiplayer, collab  | Real-time, two-way                   | Complex scaling (sticky sessions) |
| **SSE**       | App        | Server → Client   | Reliable       | Notifications, live feeds   | Simple, auto-reconnect               | One-way only, text only           |
| **gRPC**      | App        | Bi-directional    | Reliable       | Microservices RPC, APIs     | Efficient, strongly typed (Protobuf) | Less human-readable than REST     |
| **MQTT**      | App        | Pub/Sub           | Reliable(opt)  | IoT, messaging              | Lightweight, works on poor nets      | Limited use outside IoT           |
| **AMQP**      | App        | Pub/Sub           | Reliable       | Enterprise messaging        | Rich features (routing, acks)        | Heavier protocol, higher overhead |

---

👉 **Rule of Thumb for Interviews**:

* **HTTP/REST** → simple APIs.
* **gRPC** → service-to-service comms.
* **WebSockets** → real-time, interactive.
* **SSE** → server push only.
* **MQTT/AMQP** → pub/sub messaging.
* **TCP/UDP/QUIC** → transport-level trade-offs (latency vs reliability).

---