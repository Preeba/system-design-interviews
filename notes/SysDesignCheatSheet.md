Got it! I can expand the cheat sheet to include **a “Design Pattern / Use Case” column** showing where each technology or concept is typically applied in system designs, e.g., auctions → durability, microservices → SAGA, caching → read optimization, etc. I’ll reorganize it with 4 columns: **Term | Description | Typical Use | Design Pattern / Use Case**.

Here’s the updated cheat sheet:

---

## **1. Messaging & Event Streaming**

| Term                          | Description                                                           | Typical Use                                     | Design Pattern / Use Case                       |
| ----------------------------- | --------------------------------------------------------------------- | ----------------------------------------------- | ----------------------------------------------- |
| **Kafka**                     | Distributed event streaming platform; pub/sub messaging; durable logs | Event-driven microservices, real-time analytics | Event Sourcing, Publish/Subscribe, Durability   |
| **RabbitMQ**                  | Message broker implementing AMQP; supports queues, routing            | Reliable messaging, task queues                 | Queue-based decoupling, Asynchronous processing |
| **ActiveMQ**                  | JMS-based message broker                                              | Enterprise integration                          | Message Queue pattern                           |
| **SQS (AWS)**                 | Managed queue service                                                 | Decoupling services                             | Queue, Asynchronous task execution              |
| **Pub/Sub (GCP)**             | Messaging service                                                     | Event-driven systems                            | Publish/Subscribe                               |
| **SSE (Server-Sent Events)**  | Server push technology for unidirectional streaming                   | Live feeds, notifications                       | Real-time streaming, Event push                 |
| **WebSocket**                 | Full-duplex communication channel                                     | Real-time chat, games                           | Bidirectional streaming, Real-time updates      |
| **Event Sourcing**            | Store state as a sequence of events                                   | Audit logs, CQRS                                | Event Sourcing pattern, State durability        |
| **Change Data Capture (CDC)** | Track DB changes and stream them                                      | Data sync, ETL                                  | Event-driven integration, Replication           |

---

## **2. Microservices & Orchestration Patterns**

| Term                              | Description                                                                                               | Typical Use                                     | Design Pattern / Use Case                               |
| --------------------------------- | --------------------------------------------------------------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------- |
| **SAGA**                          | Distributed transaction pattern; sequence of local transactions with compensating transactions on failure | Distributed transactions without locking DBs    | Distributed Transactions, Eventually Consistent Systems |
| **Temporal.io**                   | Workflow orchestration platform; durable execution & retries                                              | Long-running workflows, reliable job scheduling | Orchestration, Durable Workflow, Retry Pattern          |
| **Circuit Breaker**               | Fault-tolerance pattern; stop calls to failing service                                                    | Prevent cascading failures                      | Resilience, Fault Isolation                             |
| **Bulkhead**                      | Isolation of service components to prevent total failure                                                  | Microservices resilience                        | Isolation pattern, Fault Containment                    |
| **Retry / Exponential Backoff**   | Automatic retry pattern with increasing delays                                                            | Handling transient failures                     | Fault Tolerance, Resilience                             |
| **API Gateway**                   | Single entry point for APIs                                                                               | Routing, authentication, rate-limiting          | API Composition, Security & Routing                     |
| **Service Mesh (Istio, Linkerd)** | Infrastructure layer for microservices                                                                    | Traffic control, observability, security        | Observability, Routing, Security                        |

---

## **3. Data Storage & Caching**

| Term                          | Description                              | Typical Use                        | Design Pattern / Use Case                   |
|-------------------------------|------------------------------------------|------------------------------------|---------------------------------------------|
| **RDBMS (MySQL, Postgres)**   | Structured relational DB                 | Transactions, joins                | Strong Consistency, ACID Transactions       |
| **NoSQL (MongoDB, DynamoDB)** | Schema-less, scalable                    | Large-scale, unstructured data     | Eventual Consistency, High Throughput       |
| **Redis**                     | In-memory key-value store                | Caching, pub/sub, session store    | Cache Aside, Read Optimization, Fast Lookup |
| **Memcached**                 | Simple memory cache                      | Reduce DB load                     | Cache Aside pattern                         |
| **Elasticsearch**             | Distributed search engine                | Full-text search, analytics        | Search Index pattern, Analytics             |
| **Cassandra**                 | Wide-column NoSQL DB                     | High-write, scalable workloads     | High Availability, Partition Tolerance      |
| **Sharding**                  | Splitting data horizontally across nodes | Scaling DB writes                  | Horizontal Scaling pattern                  |
| **Replication**               | Copying data across nodes                | High availability, fault tolerance | Replication for Durability & HA             |

---

## **4. Streaming / Real-Time Processing**

| Term                        | Description                           | Typical Use                    | Design Pattern / Use Case              |
|-----------------------------|---------------------------------------|--------------------------------|----------------------------------------|
| **Spark Streaming / Flink** | Stream processing frameworks          | ETL, analytics, ML pipelines   | Real-time Processing, Data Pipelines   |
| **Kafka Streams / KSQL**    | Stream processing on Kafka            | Real-time data transformations | Event Streaming, CQRS Read Model       |
| **Debezium**                | CDC connector                         | Streaming DB changes to Kafka  | Change Data Capture, Event-driven Sync |
| **Batch vs. Stream**        | Batch = periodic; Stream = continuous | Choosing processing mode       | Processing Pattern, Data Pipeline      |

---

## **5. API & Communication Patterns**

| Term                  | Description                         | Typical Use                   | Design Pattern / Use Case                     |
|-----------------------|-------------------------------------|-------------------------------|-----------------------------------------------|
| **REST**              | HTTP-based API                      | CRUD services                 | Client-Server, Stateless                      |
| **GraphQL**           | Query-based API                     | Flexible data retrieval       | API Composition, Aggregation                  |
| **gRPC**              | High-performance RPC using Protobuf | Low-latency microservices     | RPC pattern, Service-to-Service communication |
| **Thrift**            | RPC framework                       | Cross-language service calls  | RPC pattern                                   |
| **Protocol Buffers**  | Serialization format                | gRPC, efficient data exchange | Serialization, Data Exchange                  |
| **OpenAPI / Swagger** | API documentation spec              | Standardized REST APIs        | API Contract, Documentation                   |

---

## **6. Load Balancing & Scaling**

| Term                           | Description                           | Typical Use                           | Design Pattern / Use Case          |
|--------------------------------|---------------------------------------|---------------------------------------|------------------------------------|
| **Load Balancer**              | Distributes traffic across servers    | High availability, horizontal scaling | Horizontal Scaling, HA             |
| **Round-Robin / Least-Conn**   | Traffic distribution algorithms       | Balancer strategies                   | Load Distribution                  |
| **CDN**                        | Cache content geographically          | Faster static content delivery        | Edge Caching, Latency Optimization |
| **Auto-Scaling**               | Dynamically adding/removing instances | Handle variable load                  | Horizontal Scaling, Elasticity     |
| **Rate Limiting / Throttling** | Limit API usage                       | Prevent abuse, maintain stability     | Throttling, Traffic Shaping        |

---

## **7. Consistency & Reliability**

| Term                       | Description                                                      | Typical Use                             | Design Pattern / Use Case                     |
|----------------------------|------------------------------------------------------------------|-----------------------------------------|-----------------------------------------------|
| **CAP Theorem**            | Trade-off between Consistency, Availability, Partition tolerance | Distributed DB design                   | Fundamental Trade-offs, System Design Choices |
| **Eventual Consistency**   | DB may temporarily be out-of-sync                                | High-availability systems               | Eventually Consistent Systems, SAGA           |
| **Leader Election**        | Select a node to coordinate                                      | Distributed consensus (Zookeeper, Raft) | Coordination, Consensus                       |
| **Two-Phase Commit (2PC)** | Distributed transactions                                         | Ensure atomicity across DBs             | Distributed Transactions                      |
| **Distributed Lock**       | Prevent conflicts across nodes                                   | Synchronization in distributed systems  | Concurrency Control                           |

---

## **8. Observability & Monitoring**

| Term                | Description                      | Typical Use               | Design Pattern / Use Case        |
|---------------------|----------------------------------|---------------------------|----------------------------------|
| **Prometheus**      | Metrics collection               | Monitoring microservices  | Monitoring, Alerting             |
| **Grafana**         | Visualization dashboard          | Real-time metrics         | Visualization, Monitoring        |
| **Jaeger / Zipkin** | Distributed tracing              | Debugging latency issues  | Tracing, Latency Analysis        |
| **ELK Stack**       | Logs collection & search         | Monitoring, analysis      | Log Aggregation, Observability   |
| **Health Checks**   | Endpoint to check service status | Auto-recovery, monitoring | Resilience, Self-healing Systems |

---

## **9. Job Scheduling & Workflow**

| Term            | Description                    | Typical Use                       | Design Pattern / Use Case          |
|-----------------|--------------------------------|-----------------------------------|------------------------------------|
| **Cron**        | Time-based job scheduler       | Periodic tasks                    | Scheduled Jobs, Batch Jobs         |
| **Celery / RQ** | Task queue & worker system     | Asynchronous job execution        | Asynchronous Processing, Job Queue |
| **Temporal**    | Durable workflow orchestration | Retry, stateful workflows         | Orchestration, Durable Workflow    |
| **Airflow**     | DAG-based workflow scheduler   | ETL pipelines, data orchestration | Workflow Orchestration, ETL        |

---

## **10. Others / Miscellaneous**

| Term                      | Description                                | Typical Use                  | Design Pattern / Use Case            |
|---------------------------|--------------------------------------------|------------------------------|--------------------------------------|
| **CQRS**                  | Command Query Responsibility Segregation   | Separate read/write models   | CQRS pattern, Event Sourcing         |
| **Idempotency**           | Operation can be repeated safely           | API retries, payment systems | Safe Retries, Fault Tolerance        |
| **Backpressure**          | Control message flow to avoid overload     | Stream processing            | Flow Control, Resilience             |
| **Blue-Green Deployment** | Two environments for zero-downtime release | Deploy safely                | Deployment Strategy, Zero-downtime   |
| **Canary Deployment**     | Rollout to small subset                    | Gradual release              | Deployment Strategy, Risk Mitigation |
| **TTL (Time-to-Live)**    | Expiration for cache or DB record          | Cache eviction               | Cache Eviction, Expiration           |

---

✅ This version links **each tech/concept to the relevant design pattern or use case**, so when you design an online auction system, for example, you can justify:

* **Kafka + Event Sourcing → Durability**
* **SAGA → Distributed transactions (e.g., payments, bids)**
* **Redis → Fast read cache for hot items**
* **Temporal → Reliable job orchestration (like auctions closing)**

---

