# Design Chat System
## References used
Alex Xu's System design (https://bytebytego.com/courses/system-design-interview/design-a-chat-system)

## System Requirements

### Functional
- User should be able to send/receive realtime messages(1-on-1, and in group)
- User's presence - online/offline 
- Multi-device support
- Push Notification for new messages (push/email/web)
- Optional: File attachments (images, documents), emojis, and message reactions.

### Non-Functional
- High availability/reliabilty - No message should be dropped
- Security - end-2-end encryption, authentication and authorization
- Low latency for realtime messaging (< 200ms for message delivery)
- Scalability to support millions of concurrent users

---

## Capacity Estimation
Estimate the scale of the system you are going to design:
- Number of users (daily/monthly active users) - 10 millions Monthly Active Users(MAU)
- Number of requests per second: ~50K - 100K
- Data storage requirements - 10TB/month
- Throughput and latency expectations

---

## API Design
- POST /users/register - create a new user
- POST /users/login - authenticate user
- POST /messages - Send a message  (Request: { "senderId": "123", "receiverId": "456", "content": "Hello!" })
- GET messages/{chatId}?limit=50&offset=0 -- Fetch chat history
- GET /users/{userId}/contacts - Get Contact list

---

## Database Design
Main Entities:
- message (message_id, sender_id, receiver_id, content, created_at)
- group_message(channel_id, message_id, user_id, content, created_at)
- user (used_id(PK), username, email, password_hash, created_at, last_login)

---

## High-Level Design
![chatFig1.png](..%2Fdiagrams%2FchatFig1.png)

When a client intends to start a chat, it connects the chat service using one or more protocols. For a chat service, the choice of network protocols is important.
## **Network Protocol**:
### **HTTP (REST/HTTP APIs)**
- In most client/server applications, requests are initiated by the client. HTTP protocol works request -> response: client asks, server responds.
- Each request is independent (Stateless)
- Typical flow for sending messages via HTTP:
  - Client POSTs message to /messages.
  - Server stores it and responds with confirmation
  - Client polls the server to check for new messages
#### Pros
- Simple to implement
- Easy to scale horizontally
- Works through standard HTTP/HTTPS proxies

#### Cons
- Not real-time: polling introduces latency
- Polling frequently wastes resources and increases bandwidth
- Hard to maintain live presence updates (eg: User is typing....")

### **WebSocket**
- A persistent bi-directioal connection between client and server.
- Starts life  as a HTTP Connection, and after handshake (HTTP upgrade) upgrades to WebSocket connection.
- Server and client can push data anytime without new requests.

#### Pros
- Low-latency: instant message delivery (<200ms if network is good).
- Efficient: no repeated HTTP headers; single TCP connection.
- Bidirectional: server can send data without client asking.
- Great for real-time features: typing indicators, presence, live notifications.

#### Cons:
- More complex to implement and scale (connection state management required).
- Must handle network interruptions and reconnections.
- Not all proxies or firewalls handle WebSockets cleanly (but this is rare today).

So, by using WebSocket for both sending and receiving, it simplifies the design and implementation ==> **Stateful services**
![chatFig2.png](..%2Fdiagrams%2FchatFig2.png)


Most of the features (sign-up, login, user profile etc) of a chat application can use the traditional request/response method over HTTP ==> **Stateless services**
![chatFig3.png](..%2Fdiagrams%2FchatFig3.png)

---

## Detailed Component Design
The request flow in the below figure, shows a single server design as a starting point(small scale, but single point of failure to fit all in one diagram). 
![chatFig4.png](..%2Fdiagrams%2FchatFig4.png)

``` 
  Step-by-step flow in the diagram  
  1. Login request
    * User A logs in through the Load Balancer → request routed to one of the API servers.
    * API server validates credentials (Auth DB / token service).
    * On success, the API server tells User A:
      ** Which Chat server to connect to for WebSocket (step 4).
      ** Returns a session token.
  2. API server → Service discovery
    * API server queries Service Discovery (e.g., ZooKeeper, Consul, or etcd).
    * Service discovery knows:
      ** Which chat servers are alive.
      ** Which users are connected to which chat server.
      ** Load balancing info (to avoid hotspots).
  3. Chat server assignment
    * API server chooses Chat Server 2 (based on load/user affinity) for User A.
    * It sends this info back to the client.
  4. WebSocket connection
    * User A establishes a persistent WebSocket connection with Chat Server 2.
    * Now User A is “online” and can send/receive real-time messages.
```


## Message Flows
1 on 1 flow
Below figure explains the 1-0n-1 flow
```
  1. User A sends a chat message to Chat server 1.
  2. Chat server 1 obtains a message ID from the ID generator.
  3. Chat server 1 sends the message to the message sync queue.
  4. The message is stored in a key-value store.
  5.a. If User B is online, the message is forwarded to Chat server 2 where User B is connected.
  5.b. If User B is offline, a push notification is sent from push notification (PN) servers.
  6. Chat server 2 forwards the message to User B. There is a persistent WebSocket connection between User B and Chat server 2.
```

Perfect 👍 Let me redraw the architecture you uploaded, but this time I’ll include both **login flow** and **messaging flow** with the **Message Queue** and persistence.

Here’s a **Mermaid diagram**:

```mermaid
flowchart TD
    %% Login Flow
    UserA[User A] -->|1. login| LB[Load Balancer]
    LB -->|2| API[API Servers]
    API -->|3| SD[Service Discovery (ZooKeeper)]
    API -->|assigned server| CS2[Chat Server 2]
    UserA -->|4. WebSocket| CS2
    
    %% Messaging Flow
    CS2 -->|send message| MQ[Message Queue]
    MQ --> KV[KV Store (Message persistence)]
    MQ --> CS1[Chat Server 1]
    MQ --> CSN[Chat Server N]
    MQ --> PN[Push Notification Servers]
    
    CS1 -->|deliver to online user| UserB[User B]
    CSN -->|deliver to online group members| GroupUsers[Group Members]
    PN -->|notify offline users| OfflineUsers[Offline Users]
```
![chatFig6.png](..%2Fdiagrams%2FchatFig6.png)

---

### 🔹 Explanation

* **Login Flow**

  1. User A logs in → LB → API Servers.
  2. API queries **Service Discovery** to pick a chat server.
  3. User A gets assigned to **Chat Server 2** and opens a WebSocket.

* **Messaging Flow**

  1. User A sends a message via WebSocket to **Chat Server 2**.
  2. Chat Server 2 pushes it into the **Message Queue**.
  3. Message is:
    * Persisted in **KV Store** (history).
    * Delivered to other **Chat Servers** (where User B or group members are online).
    * Sent to **Push Notification Servers** for offline users.


![chatFig5.png](..%2Fdiagrams%2FchatFig5.png)

## Trade-offs / Tech Choices
- SQL + NoSQL: SQL for relational data, NoSQL for high-throughput messages.
- WebSocket vs Polling: WebSocket chosen for low-latency real-time delivery.
- Kafka: decouples message delivery and notifications for reliability.
- Caching: Redis reduces DB load and speeds up presence checks.

---

## Failure Scenarios / Bottlenecks
- Discuss potential failure points
- Database failure, network partitions, service crashes, high traffic spikes
- How the system handles them (retry, failover, replication)

---

## Future Improvements
- Suggest future enhancements
- Example: Horizontal scaling, advanced caching, monitoring, AI/ML recommendations
- How to mitigate previously discussed failure scenarios

![chatSystem.png](..%2Fdiagrams%2FchatSystem.png)