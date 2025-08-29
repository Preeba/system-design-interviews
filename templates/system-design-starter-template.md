# Design XYZ System

## System Requirements

### Functional
- List functional requirements here
- Example: User can create an account, upload files, search content, etc.

### Non-Functional
- List non-functional requirements here
- Example: High availability, low latency, scalability, security, etc.

---

## Capacity Estimation
Estimate the scale of the system you are going to design:
- Number of users (daily/monthly active users)
- Number of requests per second
- Data storage requirements
- Throughput and latency expectations

---

## API Design
List the APIs expected from the system:
- **GET /resource** – Fetch resource
- **POST /resource** – Create resource
- **PUT /resource/{id}** – Update resource
- **DELETE /resource/{id}** – Delete resource

Include request/response examples, status codes, and authentication if necessary.

---

## Database Design
- List main entities/tables
- Example: User, Product, Order, etc.
- Include fields, primary keys, indexes, relationships
- Optional: ER diagram

---

## High-Level Design
Describe the architecture of the system and its main components:
- Frontend
- Backend services
- Databases
- Caching layers
- Message queues / event buses
- Optional: Load balancers, CDN, external integrations

> You can also draw a block diagram using tools like Mermaid, Lucidchart, or PlantUML to illustrate the architecture.

---

## Request Flows
Explain how a request flows end-to-end in your high-level design:
- Example: Client → Load Balancer → API Gateway → Service → Database → Response

> Optional: Sequence diagram using Mermaid or PlantUML to visualize request flows.

---

## Detailed Component Design
Dig deeper into 2–3 critical components and explain:
- How the component works
- How it scales
- Algorithms or data structures used
- Optional: Component diagram

Example:
- **Authentication Service** – JWT-based, scales horizontally, rate limiting, etc.
- **Search Service** – ElasticSearch, inverted index, sharding, caching

---

## Trade-offs / Tech Choices
- Explain trade-offs you made in your design
- Example: SQL vs NoSQL, caching strategies, synchronous vs asynchronous processing
- Justify tech choices based on requirements

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
