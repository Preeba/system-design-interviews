# API Gateway

## Introduction
When building **modern distributed systems** with microservices, managing how clients interact with backend services becomes a major challenge. APIs act as the bridge between clients (web apps, mobile apps, IoT devices) and backend services.

However, exposing every microservice directly to clients can create issues:
- **Inconsistent Interfaces**: Different teams may design APIs differently, leading to poor client experience.
- **Increased Latency**: Clients may need to call multiple services separately to build a single response.
- **Security Risks**: Each exposed service increases the attack surface, making it harder to enforce consistent security policies.
- **Traffic Overload**: Backend services may get overwhelmed by direct, uncontrolled client requests.

An **API Gateway** solves these issues by acting as a **single entry point** for all client requests. It routes traffic, handles cross-cutting concerns (like authentication, logging, and rate limiting), and can even aggregate responses.

Think of it as the **traffic controller** of your microservices ecosystem—directing, protecting, and optimizing all API traffic.

---

## How an API Gateway Works
At a high level, the workflow looks like this:

1. **Client Request** – The client sends a request to the API Gateway instead of directly calling backend services.
2. **Routing & Transformation** – The gateway determines which service(s) to forward the request to and may transform the request/response format.
3. **Cross-Cutting Features** – While processing, the gateway can apply:
    - **Authentication & Authorization** – Verifies identity using OAuth, JWT, or API keys.
    - **Rate Limiting & Throttling** – Prevents abuse and protects services from overload.
    - **Caching** – Serves frequent requests from cache, reducing backend load.
    - **Logging & Monitoring** – Captures request metrics for observability.
    - **Response Aggregation** – Combines data from multiple services into a single response payload.

This setup makes **microservices easier to scale, secure, and maintain**, while keeping the client-side simple.

---

## Key Concepts
In system design interviews, you should be able to discuss the following:

- **Authentication & Authorization**: Gateways validate tokens (e.g., OAuth2, JWT) before requests reach backend services.
- **Rate Limiting & Quotas**: Controls the number of requests per client to prevent denial-of-service attacks.
- **Load Balancing**: Distributes traffic across multiple service instances for performance and reliability.
- **Service Discovery**: Integrates with service registries (like Consul, Eureka, or Kubernetes) to dynamically route requests.
- **Observability**: Provides metrics, logs, and traces for debugging and monitoring.
- **Protocol Translation**: Converts between protocols (e.g., HTTP to gRPC, WebSocket to HTTP).

---

## Common Implementations

### 1. AWS API Gateway
- Fully managed, cloud-native service.
- **Key Features**:
    - Seamless integration with AWS Lambda, DynamoDB, and other AWS services.
    - Built-in authentication, caching, and throttling.
    - Ideal for **serverless and cloud-native architectures**.

### 2. Kong Gateway
- Open-source, extensible, and highly flexible.
- **Key Features**:
    - Plugin-based architecture for adding custom policies.
    - High-performance routing and load balancing.
    - Popular choice for teams seeking **open-source control and flexibility**.

### 3. NGINX API Gateway
- Lightweight, high-performance option built on NGINX.
- **Key Features**:
    - Combines reverse proxy, load balancing, and API management.
    - Low overhead and extremely fast.
    - Best for **high-throughput, low-latency applications**.

---

## Why API Gateways Matter
API Gateways are not just a convenience—they’re an **architectural necessity** in microservices systems. They:
- Provide a **single point of entry** for client requests.
- Improve **security** by enforcing consistent authentication and authorization.
- Enhance **performance** via caching, load balancing, and request aggregation.
- Simplify **client interactions** by abstracting away the complexity of multiple backend services.
- Enable **operational visibility** with logs, metrics, and traces.

In system design interviews, API Gateways are often discussed alongside **service mesh** and **load balancers**. Be ready to explain the differences and when you’d use each.

---

# **API Gateway vs Service Mesh vs Load Balancer** 

## 🔹 API Gateway

* **Purpose**: Acts as the single entry point for external clients (mobile apps, browsers, third-party systems) to access your microservices.
* **Responsibilities**:

    * Request routing (client → service)
    * Authentication & authorization
    * Rate limiting & throttling
    * API versioning
    * Request/response transformation
    * Monitoring & logging (per client/API usage)
* **Scope**: **North-South traffic** (outside → inside the cluster).
* **Example tools**: Kong, Apigee, AWS API Gateway, NGINX (as API gateway).

---

## 🔹 Service Mesh

* **Purpose**: Manages communication between internal microservices within your system.
* **Responsibilities**:

    * Service-to-service discovery & routing
    * Secure service-to-service communication (mTLS)
    * Load balancing (at service level)
    * Circuit breaking, retries, failovers
    * Observability (tracing, metrics, logs)
* **Scope**: **East-West traffic** (service ↔ service inside the cluster).
* **Example tools**: Istio, Linkerd, Consul.

---

## 🔹 Load Balancer

* **Purpose**: Distributes incoming traffic across multiple servers or instances to ensure availability and reliability.
* **Responsibilities**:

    * Distribute client requests among servers (round robin, least connections, IP hash, etc.)
    * Health checks (remove unhealthy instances from rotation)
    * SSL termination (sometimes)
    * Basic routing (e.g., L4/L7 routing)
* **Scope**: Typically sits **in front of servers/services** (can be external or internal).
* **Example tools**: AWS ELB/ALB/NLB, HAProxy, NGINX, F5.

---

## 🔄 Key Differences at a Glance

| Feature            | API Gateway                        | Service Mesh                    | Load Balancer                     |
|--------------------|------------------------------------|---------------------------------|-----------------------------------|
| **Traffic Type**   | North-South (external → internal)  | East-West (internal ↔ internal) | Both (external and internal)      |
| **Main Role**      | Entry point for APIs               | Manage service-to-service comms | Distribute traffic across servers |
| **Security**       | AuthN/AuthZ, rate limiting         | mTLS between services           | SSL termination (optional)        |
| **Intelligence**   | High-level (API aware)             | High-level (service aware)      | Lower-level (network aware)       |
| **Examples**       | Kong, AWS API GW                   | Istio, Linkerd                  | AWS ALB, NGINX, HAProxy           |

---

👉 Think of it like this:

* **Load Balancer** = “spread the traffic”
* **API Gateway** = “control + expose APIs safely to the outside world”
* **Service Mesh** = “make microservices talk to each other reliably & securely”

---