Designing a **Flight Scheduling and Information Display System** for an **airport** is a great system design interview topic. It combines real-time data updates, high availability, and a clean separation of concerns. Here's how to approach it step-by-step:

---

## 🧭 **High-Level Goals**

Build a system that:

* Manages flight schedules (arrivals, departures, delays, gates)
* Updates displays in real time across terminals
* Handles admin (airline, airport staff) and user (passenger) views
* Scales to large airports with hundreds of flights/day

---

## 🗂️ 1. **Key Features**

### 🛫 Flight Scheduling (Admin)

* Add/update/cancel flights
* Assign gates, times, statuses
* Integration with airlines and air traffic control (ATC)

### 📺 Information Display (User-Facing)

* Terminal boards showing live departures/arrivals
* Flight status (on-time, delayed, boarding, cancelled)
* Gate assignments, baggage claim

### 📱 External APIs or Apps

* Mobile app/web portal to check flight status
* Push notifications

---

## 🧱 2. **System Components**

```plaintext
+-------------------+       +------------------------+
| Airline Staff UI  |<----->|   Admin Flight Service |
+-------------------+       +------------------------+
                                   |
                                   v
                          +-------------------+
                          | Flight Schedule DB |
                          +-------------------+
                                   |
                                   v
+--------------+        +---------------------+        +------------------+
| Display UIs  |<------>|   Display Service   |<------>| Notification Svc |
+--------------+        +---------------------+        +------------------+
     |                          ↑      ↑
     |                          |      |
     v                          |      |
Passenger                     Redis / Kafka (for real-time updates)
Mobile/Web                    Message Queue / PubSub
```

---

## 🛠️ 3. **Tech Stack Suggestions**

| Layer           | Tech                                                 |
| --------------- | ---------------------------------------------------- |
| API Gateway     | Kong, NGINX, or Spring Cloud Gateway                 |
| Admin APIs      | REST or GraphQL (Java + Spring Boot / Node.js)       |
| Data Storage    | PostgreSQL for flights, Redis for real-time          |
| Real-Time       | Kafka / Redis PubSub / WebSockets                    |
| Display UI      | React / Angular front-end with auto-refresh          |
| Background Jobs | Quartz Scheduler or Celery (for delay notifications) |

---

## 🧩 4. **Data Model (Simplified)**

```sql
Flight (
  id UUID,
  flight_number VARCHAR,
  airline VARCHAR,
  from_airport VARCHAR,
  to_airport VARCHAR,
  scheduled_departure TIMESTAMP,
  scheduled_arrival TIMESTAMP,
  gate VARCHAR,
  terminal VARCHAR,
  status ENUM('Scheduled', 'Delayed', 'Boarding', 'Departed', 'Cancelled'),
  last_updated TIMESTAMP
)

DisplayBoard (
  id UUID,
  terminal VARCHAR,
  gate VARCHAR,
  last_synced TIMESTAMP
)
```

---

## ⚙️ 5. **Real-Time Updates**

* Flights update via admin panel or airline API
* Push to **Kafka or Redis Pub/Sub**
* All display boards and web/mobile apps subscribe to updates
* Display boards refresh affected flights only

---

## 🛡️ 6. **Non-Functional Requirements**

| Requirement         | Solution                                             |
| ------------------- | ---------------------------------------------------- |
| ⏱ Real-time updates | WebSockets or Redis Pub/Sub                          |
| 📈 Scalability      | Partition by terminal, Kafka consumers for each      |
| 💡 Fault-tolerance  | Use retries + fallback in Redis and Kafka            |
| 🧪 Testability      | Mock flight streams, simulate delays                 |
| 🔒 Security         | Role-based access (admin vs user), HTTPS, token auth |

---

## 🔄 7. **Flight Delay Propagation**

* Scheduled times are updated → trigger recalculation of downstream connections, gates
* Delay info propagated to:

    * Display systems
    * Notification service (e.g., SMS/Push)
    * ATC system (simulated in design)

---

## 🔁 8. **Extensions**

* Baggage tracking system
* Weather integration
* Predictive delays via ML (using historical data)
* Integration with radar / ATC feed

---

## 📋 Summary View

| Component                | Role                         |
| ------------------------ | ---------------------------- |
| **Admin Service**        | Flight CRUD + schedule logic |
| **Display Service**      | Sync flights to screens      |
| **Notification Service** | Push status changes          |
| **Flight DB (Postgres)** | Source of truth              |
| **Redis/Kafka**          | Real-time pipeline           |
| **WebSockets/REST**      | Feed to displays / apps      |

---

Let’s cover the **API design**, **database choices**, and **scaling strategies** for the **Airport Flight Scheduling and Display System**.

---

## 🧩 1. **API Design**

### ✈️ Flight Management (Admin APIs)

```http
POST   /api/flights                 # Add a new flight
PUT    /api/flights/{id}           # Update flight details (status, gate, etc.)
DELETE /api/flights/{id}           # Cancel a flight
GET    /api/flights/{id}           # Get flight by ID
GET    /api/flights?terminal=T1    # List flights by terminal or status
```

### 📺 Display Board APIs

```http
GET /api/display?terminal=T1&gate=G5   # Flights at a specific gate or terminal
GET /api/arrivals?airport=SFO          # Incoming flights
GET /api/departures?airport=SFO        # Outgoing flights
```

### 📱 Passenger-Facing APIs

```http
GET /api/flight-status?flightNumber=AA123
GET /api/notifications/{passengerId}
```

---

## 🛢️ 2. **Database Design Choices**

### Primary Database: **PostgreSQL**

Why:

* Strong relational support (good for modeling flights, schedules, terminals)
* ACID compliance
* Indexing on timestamps and flight numbers for efficient lookups

**Tables:**

* `flights`
* `terminals`
* `gates`
* `airlines`
* `passengers` (optional for notification personalization)

### Real-Time Layer: **Redis / Kafka**

Why:

* Redis: low-latency cache for display boards
* Kafka: publish/subscribe model to broadcast flight updates to terminals, apps

### Optional: **ElasticSearch**

Why:

* Full-text search on flight numbers, terminals, airlines
* Fast filtering/sorting for passenger-facing dashboards

---

## ⚙️ 3. **Scaling Strategy**

| Layer                  | Scaling Strategy                                                        |
| ---------------------- | ----------------------------------------------------------------------- |
| **API Layer**          | Deploy via Kubernetes + Horizontal Pod Autoscaling (HPA)                |
| **Database**           | Read replicas for load balancing; partition flights by date or terminal |
| **Display Updates**    | Use Kafka → Each terminal/gate has its own Kafka topic or Redis channel |
| **Caching**            | Redis stores "hot" flight data (next 3 hours) for display boards        |
| **Mobile/Web Traffic** | CDN + caching for static content; API Gateway for routing               |

---

## 📈 4. **Data Volume & Partitioning Tips**

| Table           | Partition Key                  |
| --------------- | ------------------------------ |
| `flights`       | `scheduled_date` or `terminal` |
| `gates`         | `terminal`                     |
| `notifications` | `passenger_id` or `flight_id`  |

---

## 🔐 5. **Security & Roles**

* Use **JWT tokens or OAuth2** for authentication
* **RBAC** (Role-Based Access Control)

    * Admin (airline staff)
    * Airport operator
    * Read-only (passenger, public)

---

## 🔄 6. **High Availability Considerations**

* Multi-AZ deployment of databases
* Failover replicas for Redis/Kafka
* Load balancer with health checks
* Circuit breakers for display consumers

---

## 📡 Example Real-Time Flow

1. Admin updates flight status → PUT `/api/flights/{id}`
2. Backend writes to PostgreSQL
3. Publishes event to Kafka: `FlightStatusUpdated`
4. Display services and mobile apps subscribed to Kafka update their UIs

---

## 📋 Summary

| Aspect        | Recommendation                     |
| ------------- | ---------------------------------- |
| **Core DB**   | PostgreSQL (ACID, relations)       |
| **Cache**     | Redis (real-time flight info)      |
| **Streaming** | Kafka (for display & app updates)  |
| **Search**    | ElasticSearch (optional)           |
| **Scaling**   | HPA, sharding by terminal/date     |
| **Auth**      | JWT, OAuth2, role-based access     |
| **Uptime**    | Multi-region failover, autoscaling |

---


A **Flight Information Display System (FIDS)** is used at airports to show real-time flight information (arrivals, departures, gates, status, etc.) to passengers and staff. It integrates multiple data sources and components in a real-time, fault-tolerant, and scalable architecture.

---

### ✅ **FIDS Architecture Overview**

Here’s the component **flow** (from data ingestion to display), followed by a **diagram**.

---

### 📦 Components in Flow Order:

1. **Data Sources**

  * Airline Flight Feeds (e.g., via SITA, ARINC)
  * Airport Operations DB (Gates, Check-in, Baggage)
  * Air Traffic Control (ATC) Feeds
  * Weather API
  * Internal Admin Panel

2. **Ingestion Layer**

  * APIs / Webhooks / Schedulers (polling)
  * Message Queue (Kafka / RabbitMQ) — optional for decoupling

3. **Data Processing & Aggregation Layer**

  * ETL Services / Microservices (Node.js, Java, Python, etc.)
  * Business Logic (merging delays, gate changes, priorities)

4. **Central Database / Cache**

  * Relational DB (PostgreSQL, MySQL) — for historical data
  * Redis / Memcached — for real-time access

5. **API Layer**

  * REST or GraphQL APIs for UI clients to fetch data
  * Authentication/Authorization (if required)

6. **Display Management System**

  * Display Configuration Manager (zones, screen layout)
  * Screen Controller/Client (HTML5 App, React/Angular, Raspberry Pi, etc.)
  * Real-time WebSocket/SignalR/MQTT updates

7. **Monitoring and Admin Panel**

  * Admin dashboard to manually override flight data
  * Logging and Alerting (ELK, Prometheus, Grafana)

---

### 📊 FIDS Architecture Diagram

```plaintext
                   ┌───────────────────────────────────────────────────│
                   │    Airline Systems | Weather API │ATC / Ops Feed  │
                   └──────────────────|────────────────────────────────┘
                                      │
                      ┌───────────────▼────────────────┐
                      │    Ingestion Layer (API, MQ)   │
                      └────────────────|───────────────┘
                                       │
                              ┌────────▼────────┐
                              │  Processing &   │
                              │ Business Logic  │
                              └────────┬────────┘
                                       │
                        ┌──────────────▼─────────────┐
                        │   DB (SQL) / Cache (Redis) │
                        └──────────────┬─────────────┘
                                       │
                              ┌────────▼────────┐
                              │     API Layer   │
                              └────────┬────────┘
                                       │
                ┌──────────────────────▼──────────────────────┐
                │          Display Controller / UI            │
                │ (Kiosk, Web App, Smart Screen, Mobile App)  │
                └─────────────────────────────────────────────┘
                                       │
                              ┌────────▼────────┐
                              │ Admin / Ops UI  │
                              └─────────────────┘
```

---

### 🛠️ Technologies Often Used:

* **Ingestion**: Kafka, REST APIs, cron jobs
* **Processing**: Java Spring Boot, Node.js, Python Flask
* **Database**: PostgreSQL + Redis
* **Display App**: HTML5 / React + WebSocket or MQTT
* **Deployment**: Docker, Kubernetes (for resilience)

---

A Flight Information Display System (FIDS) is a crucial part of airport operations, providing real-time flight information to passengers and staff. Its architecture is designed for high availability, data accuracy, and scalability.

**1. Data Sources:**

* **Air Traffic Control (ATC) Systems:** Provides actual flight movement data (take-offs, landings, delays, gate changes).
* **Airline Operational Systems:** Supplies scheduled flight data, including flight numbers, routes, times, and aircraft types.
* **Airport Operational Databases (AODB):** Centralized airport database that integrates data from various sources, including flight schedules, gate assignments, baggage carousel information, and check-in counter details.
* **External Data Feeds:** Weather information, connecting flight status, and sometimes even social media for passenger convenience alerts.

**2. Data Ingestion and Processing:**

* **Data Integration Layer:** This layer is responsible for collecting data from diverse sources, which often use different protocols and formats (e.g., XML, JSON, proprietary APIs). It normalizes and standardizes the data.
* **Flight Information Server(s):** This is the core of the FIDS. It processes, validates, and stores all flight-related data. Key functionalities include:
  * **Data Reconciliation:** Resolving discrepancies between different data sources (e.g., scheduled vs. actual times).
  * **Event Processing:** Triggering updates based on flight events (e.g., a flight departing a gate).
  * **Business Logic:** Applying rules for displaying information (e.g., how far in advance to show boarding calls, what messages to display for delayed flights).
  * **Data Caching:** Temporarily storing frequently accessed data for faster retrieval.

**3. Data Distribution and Management:**

* **Content Management System (CMS):** Manages the layout, templates, and multimedia content displayed on the FIDS screens. This allows airport staff to customize the appearance and information presented.
* **FIDS Database:** A robust, highly available database (often relational like PostgreSQL or SQL Server, or NoSQL for scalability) that stores all processed flight information, display configurations, and historical data.
* **Communication Protocols:** Various protocols like TCP/IP, WebSockets, or message queues (e.g., Kafka, RabbitMQ) are used to efficiently distribute data to the display clients.

**4. Display and Client Layer:**

* **FIDS Client Software:** Runs on individual display screens (LCD, LED, monitors) throughout the airport. This software retrieves data from the Flight Information Server and renders it according to the defined templates.
* **Display Controllers:** Dedicated hardware or software that manages multiple display screens in a specific area, ensuring synchronized and accurate content delivery.
* **Network Infrastructure:** A reliable and high-bandwidth network (wired and/or wireless) is essential to connect all components and ensure seamless data flow.

**5. Monitoring and Management:**

* **System Monitoring:** Tools to track the health and performance of all FIDS components, including servers, databases, network, and individual displays. Alerts are generated for any issues.
* **Administration Interface:** A web-based or desktop application for airport staff to configure FIDS settings, manage content, troubleshoot issues, and view system logs.

**Flow of Information (Simplified):**

1.  **Data Sources** (ATC, Airlines, AODB, External) send raw flight data.
2.  **Data Integration Layer** normalizes and consolidates the data.
3.  **Flight Information Server** processes the data, applies business logic, and stores it in the **FIDS Database**.
4.  **Content Management System** defines the display layouts and content.
5.  **FIDS Server** pushes relevant, processed flight data to **FIDS Client Software** running on individual **Display Screens** via the **Network Infrastructure**.
6.  **Display Controllers** manage the rendering on the screens.
7.  **Monitoring and Administration tools** oversee the entire system.

---

### Core Architecture Flow
1. **Data Sources**
  - **Airline Operational Databases (AODB)**: Primary source of flight schedules, gates, and status updates.
  - **Airport Resource Management Systems**: Assigns gates, baggage belts, and check-in counters.
  - **External Systems**: Weather feeds, security alerts, and third-party APIs

2. **Central Control Server**
  - **FIDS Core**: Processes and validates data, applies business rules, and manages templates for displays.
  - **Database (e.g., Oracle, MySQL)**: Stores configuration, user permissions, and historical data
  - **Template Designer**: Allows administrators to customize display layouts for different screen types (e.g., departures, baggage)

3. **Distribution Layer**
  - **Distribution Servers**: Relay real-time data to displays via LAN/WAN or cloud
  - **Failover Systems**: Hot-standby servers ensure continuity during outages

4. **Display Units**
  - **Check-in Counters**: Show flight-specific instructions
  - **Boarding Gates**: Dynamic gate assignments and boarding times
  - **Baggage Belts**: Carousel allocation updates
  - **Public Displays**: Large-format LCD/LED screens for arrivals/departures
  - **Web/Mobile Interfaces**: Browser-based dashboards for passengers

5. **User Interfaces**
  - **Administration Workstations**: Manage content, permissions, and emergency alerts
  - **API Integrations**: Enable data sharing with airlines, mobile apps, and signage systems

### Key Integration Points
- **Real-Time Protocols**: TCP/IP for data synchronization
- **Security**: Role-based access control, two-factor authentication, and encrypted communication
- **Cloud Components**: Optional cloud hosting for remote monitoring and updates



## FIDS Architecture: Main Components & Flow
FIDS architecture typically involves a centralized database, distribution servers and display screens. Its architecture is designed for high availability, data accuracy, and scalability. 

Centralized database stores flight data, including scheduling, gate assignments & delays. This database is connected to digital display screens placed throughout the airport.

## **Summary Flow Diagram (Textual)**

```
        Data Sources
          (Airline Feeds, Ops DB, ATC, Weather, Admin Panel)
                     ↓
            Ingestion Layer
              (APIs, Webhooks, Schedulers, MQ)
                      ↓
            Processing & Business Logic
              (ETL, Microservices, Rules)
                      ↓
            DB (SQL) / Cache (Redis)
                      ↓
            API Layer
              (REST/GraphQL, Auth)
                      ↓
            Display Management System
              (Kiosk, Web, Smart Screen, Mobile)
                      ↓
            Monitoring and Admin Panel
              (Admin Dashboard, Logging, Alerting)
```

---

## **How It All Works Together**

1. **Data is collected** from airlines, airport systems, ATC, weather, and admins.
2. **Ingestion layer** standardizes and queues this data.
3. **Processing/business logic** ensures data is accurate, merged, and actionable.
4. **Database/cache** stores the data for immediate and historical needs.
5. **API layer** delivers data securely to all display endpoints.
6. **Display systems** present the information to passengers and staff.
7. **Admin/monitoring tools** allow for manual control and system health checks.

---

**This modular, layered approach ensures high reliability, real-time updates, and flexibility for airport operations and passenger information.**

----

Here's a **detailed explanation of each component** in the Flight Information Display System (FIDS) **flow diagram** you shared:

---

### 📡 1. **Airline Systems**

* These are third-party or partner airline systems (e.g., SITA, ARINC) that provide real-time data feeds for:

  * Scheduled and actual departure/arrival times
  * Flight numbers, delays, cancellations
  * Gate and terminal assignments

🔁 **Data Flow:** Usually via APIs or flat file feeds (CSV/XML) pushed to FTP or webhooks.

---

### 🌦️ 2. **Weather API**

* External APIs (e.g., OpenWeather, FAA weather feed) provide weather data that can affect flights.
* This includes real-time and forecasted conditions such as:

  * Wind speed, visibility, precipitation
  * Weather alerts that might delay flights

🔁 **Data Flow:** Polled periodically or pushed via webhooks into the system.

---

### 🛰️ 3. **ATC / Ops Feed**

* Air Traffic Control or Airport Operational Databases (AODB) that inform:

  * Runway availability
  * Taxi routes and hold times
  * Gate occupancy
  * Baggage and check-in status

🔁 **Data Flow:** Internal feed from airport systems or government-controlled ATC data.

---

### 🔄 4. **Ingestion Layer (API, MQ)**

This is the system's entry point for external and internal data.

**Components include:**

* **API Gateways** to receive data from REST endpoints
* **Message Queues** like Kafka/RabbitMQ for asynchronous ingestion
* **Data Normalizers** to convert formats into a unified internal structure

🔁 **Flow:** Ensures all inputs, whether real-time or scheduled, are accepted and pre-processed.

---

### ⚙️ 5. **Processing & Business Logic**

Core of the FIDS logic. It:

* Merges different data sources for the same flight
* Resolves conflicts (e.g., new gate vs. old gate)
* Applies business rules (e.g., highlight "Final Call" status if departure < 15 mins)
* Calculates delays, duration, and visual flags (e.g., ON TIME, BOARDING, DELAYED)

Can be implemented via:

* **Microservices**
* **ETL pipelines**
* **Rules engines**

---

### 🛢️ 6. **Database (SQL) / Cache (Redis)**

**Storage Layer:**

* **SQL DB (e.g., PostgreSQL)** stores flight records, historical data, logs, configuration
* **Redis or Memcached** serves real-time flight data to the display layer with low latency

Caching ensures that screen updates are blazing fast and can handle concurrent requests.

---

### 🌐 7. **API Layer**

Serves data to:

* Displays (kiosks, signage)
* Mobile apps
* Admin portals

Common tech:

* **RESTful APIs**
* **GraphQL (AppSync)** for flexible data queries

Handles **filtering** (e.g., only show Terminal A), pagination, and security.

---

### 📺 8. **Display Controller / UI**

The **end-user-facing layer**, often rendered on:

* Airport monitors
* Smart screens
* Kiosks
* Mobile apps

Typically built using:

* **HTML5 + JavaScript (React, Angular, Vue)**
* Uses **WebSocket** or **MQTT** for real-time updates (gate changes, boarding calls)

Responsible for:

* Layout control
* Zone-based display (Departures, Arrivals, Baggage info)
* Multilingual support

---

### 🛠️ 9. **Admin / Ops UI**

Internal dashboard for airport staff:

* Manually override or update flight info
* Control screen configurations (which screen shows what)
* View logs, alerts, and system health

Includes authentication, audit logs, and role-based access control.

---

### 🔁 Summary of Flow:

```
External Feeds (Airlines, Weather, ATC)
         ↓
   Ingestion (API / MQ)
         ↓
  Processing & Logic Layer
         ↓
  SQL DB + Redis Cache
         ↓
      API Layer
         ↓
    ┌───────────────┬───────────────┐
Display Controller   Admin Dashboard
    (Kiosk, UI)       (Ops UI)
```
 ---

 **how a flight delay change is propagated** through your **Flight Information Display System (FIDS)**, using your architecture and layers.


## 🛫 Scenario: **Flight Delay Update**

Let’s say **Flight AA123** from an airline gets delayed.

### 🔁 End-to-End Flow:

---

### **1. Event Source: Airline / ATC / Admin Input**

* **Airline System / ATC** detects delay for flight AA123
* Sends an **event** (via REST API, MQ, or webhook)

  * Ex: `PUT /flights/AA123 {"status": "DELAYED", "estimatedDeparture": "16:45"}`

---

### **2. Ingestion Layer (API Gateway + Lambda / MQ)**

* Ingested via:

  * **API Gateway → Lambda**, or
  * **Amazon MQ (ActiveMQ) / Amazon MSK (Kafka)**

* Standardizes data to internal schema (ex: maps different formats to a unified model)

```json
{
  "flightId": "AA123",
  "status": "DELAYED",
  "delayReason": "Weather",
  "updatedTime": "2025-05-31T15:20:00Z"
}
```

---

### **3. Processing / Business Logic (Lambda / ECS / Step Functions)**

* **Business rules** apply:

  * Was the flight already marked delayed?
  * Has the estimated time changed?
  * Merge with latest ATC or weather feed (if any)
* Validate data → enrich if needed (e.g., new gate, weather context)
* Logs event for auditing

---

### **4. DB / Cache Layer (RDS + Redis)**

* Updates **RDS** record for AA123

  * `UPDATE flights SET status='DELAYED', updated_at=...`
* **Redis cache** is updated for instant lookups (used by displays)

  * Key: `flight:AA123 → {...}`

---

### **5. Notification System (SNS / EventBridge / AppSync)**

> ✅ **This is where real-time or triggered messaging happens**

* **Flight status change** is published to:

  * **Amazon SNS** topic: `flight_updates`
  * OR **Amazon EventBridge bus**

* **Consumers** of this event:

  * Passenger **mobile push (SNS → Lambda → Firebase/Apple Push)**
  * Display system refresh (via polling or **AppSync subscription**)
  * Email/SMS (via SES or Twilio)
  * Admin Dashboard metrics/logs

---

### **6. API Layer (GraphQL / REST)**

* Frontend (web, kiosk, mobile) requests latest:

  * `/flights?airport=SFO&timeWindow=next2hours`
  * Or via GraphQL: `{ flights { id, status, gate, etd } }`

* API pulls from **Redis (fast)** → fallback to **RDS (slow but complete)**

---

### **7. Display Systems (Web App, Kiosk, Smart TV)**

* On receiving update:

  * Re-renders **AA123** with `"DELAYED"` tag
  * Optionally: flashes animation / warning icon
  * May show reason or new departure time

---

### **8. Admin / Monitoring UI**

* Ops staff sees:

  * Alert on dashboard (from event system)
  * Can override or annotate (e.g., “waiting for crew”)
  * Real-time health status of ingestion/processing pipelines

---

## 🔔 Diagram: Flight Delay Propagation

```
[Airline System] → [API Gateway / MQ] → [Lambda (Biz Logic)] 
          ↓                            ↓
    [Amazon SNS/EventBridge] → [Subscribers]
          ↓                            ↓
   [RDS + Redis]                [Mobile Push / Email]
          ↓
    [Display APIs] → [Web/Kiosk/TV]
```

---

## ✅ Summary of Where Notification Happens

| Layer                    | Role in Notification                     |
| ------------------------ | ---------------------------------------- |
| Ingestion Layer          | Triggers event on change                 |
| Processing Layer         | Publishes to SNS/EventBridge             |
| DB/Cache                 | Stores latest state for fast retrieval   |
| Notification System      | Sends push/email/screen refresh          |
| API Layer                | Serves updated data to UI                |
| Display/Monitoring Layer | Reacts to change (UI update, alert, log) |

---

## 🔔 **When Are Notifications Sent in FIDS Flow?**

In your FIDS architecture, **notifications** are sent **after significant data changes are processed** — not at the point of ingestion. Here's when and what gets notified:

---

### ✅ **Events That Trigger Notifications**

These typically come from external feeds (Airline, ATC, Weather) or Admin input. The **Processing & Logic Layer** decides *whether* a change is meaningful enough to notify.

| **Event Type**            | **Notified?** | **Who/What Gets Notified**                              |
|---------------------------|---------------|---------------------------------------------------------|
| ✈️ Flight Delay           | ✅ Yes         | Display Systems, Passengers (via push), Ops Dashboard  |
| ✅ Flight On-Time Update   | ❌ No          | Only update DB, no need to notify                     |
| 🛫 Gate Change            | ✅ Yes         | Displays, Mobile App, Gate Staff                       |
| ⏱️ Estimated Time Change  | ✅ If major    | If change crosses thresholds (e.g., >10 mins)          |
| ☁️ Weather Alert          | ✅ Yes         | Ops Dashboard, Certain Flights (impacted)              |
| 🛑 Flight Cancelled       | ✅ Yes         | Passengers, Displays, Admins                           |
| ✅ Routine Status Check    | ❌ No          | Stored in DB, but not broadcast                       |
| 📝 Manual Override by Ops | ✅ Yes         | Admin Log, Displays refreshed                          |

---

### 🔁 **Where in the Flow the Notification Happens**

Here’s the timeline with clarity:

```
[External Feeds or Admin Input]
           ↓
      Ingestion Layer
           ↓
  ✅ Processing & Logic Layer
       [This is where logic decides: Is this change important?]
           ↓
     If important → 🔔 Fire notification event (SNS/EventBridge)
                   → Update Redis + RDS
           ↓
      API Layer consumes updated data
           ↓
Display UI / Admin UI pick up changes (pull or via push)
```

---

### 🔔 **Who Consumes the Notification?**

1. **Display Systems (Smart Screens, Kiosks)**

  * Trigger a refresh for changed flights (e.g., flight AA123 is delayed)

2. **Mobile App / Passenger Notification**

  * Push notification if user subscribed to a flight
    *(via SNS → Firebase / Apple Push Notification Service)*

3. **Admin Dashboard (Ops UI)**

  * Shows alerts for critical updates (cancellations, weather events)
  * May auto-highlight problem areas

4. **Logs & Audit System (Optional)**

  * For compliance or diagnostics

---

## 🧠 Example: Flight Delay Update

1. **Airline Feed** sends `AA123` is now DELAYED
2. Processing Layer:

  * Sees that status has changed from ON-TIME → DELAYED
  * Publishes event: `FlightUpdateEvent { flightId: AA123, status: DELAYED }`
3. SNS or EventBridge delivers:

  * To API/WebSocket layer → pushes to UI
  * To Notification system → push/email to passenger
  * To Admin UI → flag raised
4. Cache (Redis) updated → fast response to UI queries
5. RDS updated → for historical/logging

---

## 🧭 Summary

* **Notifications are sent only when data changes are *meaningful*.**
* The **Processing Layer** is responsible for deciding this.
* **SNS/EventBridge** or similar pub/sub system is used to broadcast.
* **Consumers** include Display UI, Ops UI, and Passenger Notification Services.

---

#### NEW CONTENT

-----

In a **Flight Information Display System (FIDS)**, **Redis** acts as a **fast in-memory cache** to serve time-sensitive flight data quickly to kiosks, web UIs, and apps.

Let’s go step-by-step:

---

## 🧠 **What is Stored in Redis (vs. SQL)?**

| **Type of Data**                   | **Stored in Redis** | **Stored in SQL (RDS)**   |
| ---------------------------------- | ------------------- | ------------------------- |
| Real-time flight status            | ✅ Yes               | ✅ Yes (for history/audit) |
| Upcoming departures                | ✅ Yes               | ✅ Yes                     |
| Display screen mappings            | ✅ Sometimes         | ✅ Yes (source of truth)   |
| Admin user data                    | ❌ No                | ✅ Yes                     |
| Flight delay logs/history          | ❌ No                | ✅ Yes                     |
| Airport config (e.g., gate layout) | ✅ (cached)          | ✅ Yes                     |

---

## 📦 **Sample Redis Key-Value Example**

### 1. **Flight Status Info**

```redis
Key:    flight:AA123
Value:  {
  "flightId": "AA123",
  "airline": "American Airlines",
  "departure": "SFO",
  "arrival": "JFK",
  "scheduledDeparture": "2025-05-31T14:30:00Z",
  "status": "DELAYED",
  "gate": "B12",
  "newDeparture": "2025-05-31T15:10:00Z"
}
TTL: 4 hours (auto-expire after flight is over)
```

---

### 2. **Upcoming Departures by Airport**

```redis
Key:    airport:SFO:departures
Value:  ["AA123", "DL456", "UA789"]
```

This is a **sorted list or set**, so you can quickly render SFO's departures.

---

### 3. **Flight List for a Display Screen**

```redis
Key:    screen:domestic:SFO:T1:panel2
Value:  ["AA123", "DL456"]
```

Display system UI can use this to fetch all related flight keys quickly.

---

### 4. **Gate-wise Flight Map (Quick Lookup)**

```redis
Key:    gate:SFO:B12
Value:  "AA123"
```

---

## 🔄 **How Redis Gets Updated (Flow)**

```mermaid
flowchart TD
  A[External Feed: Airline / ATC / Admin] --> B[Ingestion Layer (API / MQ)]
  B --> C[Processing Layer]
  C --> D[Update SQL Database]
  C --> E[Update Redis Cache]
  E --> F[API Layer / Display UI / Mobile App]

  subgraph "Fast Response Path"
    F --> E
  end
```

* **Writes**: When data changes (e.g., delay), Redis is **written by the processing layer**.
* **Reads**: Display/UI first **checks Redis** for fast access.
* SQL remains the **source of truth** for historical/audit.

---

## ⏱️ Why Redis?

* **Low latency**: UIs can load in milliseconds.
* **TTL (Time To Live)**: Expire old flights automatically.
* **Avoid overloading SQL** with frequent read traffic.

We need **Redis** for the **real-time layer in a Flight Information Display System (FIDS)** because traditional databases like PostgreSQL are optimized for durability and consistency, not ultra-fast, high-frequency reads and writes. Redis fills this performance-critical gap.

---

## ✈️ Why Redis for FIDS Real-Time Layer?

### 🔄 1. **Real-Time Updates**

* Flight info changes frequently: status (`Boarding`, `Delayed`, etc.), gates, departure times.
* Redis enables **sub-millisecond latency** for reading/writing this data.
* Displays and mobile apps can **poll or subscribe** to changes instantly.

### 🧠 Example:

* When flight `AA123` changes gate from `G5` to `G7`, Redis reflects this instantly:

  ```bash
  SET flight:AA123:gate G7
  ```

---

### ⚡ 2. **High Read/Write Throughput**

* Terminals have dozens of screens and mobile users fetching flight data.
* Redis handles **millions of operations per second** without disk I/O bottlenecks.
* PostgreSQL could get overwhelmed if used for **every screen refresh**.

---

### 🪢 3. **Pub/Sub for Push Notifications**

* Redis supports **Publish/Subscribe** pattern.
* You can publish a change like:

  ```bash
  PUBLISH flight_update AA123:Status:Boarding
  ```
* Subscribed display boards and mobile clients **instantly react**, updating the UI.

---

### 💾 4. **Caching Layer**

* Use Redis to cache:

  * Frequently queried flights (e.g., JFK departures)
  * Gate schedules
  * Statuses and terminal assignments
* Reduces repeated load on PostgreSQL.

  ```bash
  GET departures:JFK → [cached flight list]
  ```

---

### ⏱ 5. **Expiring Temporary Data**

* Redis lets you set TTLs (time-to-live) for short-lived data:

  ```bash
  SET flight:AA123:status "Boarding" EX 900
  ```
* Ensures the cache doesn't grow indefinitely and keeps stale info from lingering.

---

### 🧰 6. **Data Model Flexibility**

You can use Redis structures like:

* `HASH`: for flight details (`HGETALL flight:AA123`)
* `ZSET`: for sorting by scheduled departure
* `STREAM`: for real-time event logs
* `PUB/SUB`: for live update broadcasts

---

## 🛠 Architecture Diagram (Simplified)

```plaintext
         +-----------------+
         | PostgreSQL (DB) |  ← Authoritative Data (slow)
         +-----------------+
                  |
                  ↓ (periodic sync)
          +-------------------+
          |     Redis Cache   |  ← Real-time layer (fast, in-memory)
          +-------------------+
           ↑           ↑
   Flight APIs    Display APIs
   (mobile/web)    (terminals)
```

---

## ✅ Summary: Why Redis in FIDS

| Need                        | How Redis Helps                 |
| --------------------------- | ------------------------------- |
| Real-time flight updates    | Sub-millisecond reads/writes    |
| Scale to many users/screens | In-memory high-throughput       |
| Notify clients of changes   | Pub/Sub or Streams              |
| Reduce DB load              | Caches frequently accessed data |
| Temporary data management   | TTL for auto-expiry             |

---

```
  Data Sources (Feeds, Admin) 
         ↓
  Ingestion Layer (APIs, MQ, Webhooks)
         ↓
  Processing & Business Logic (Microservices, ETL, Rules)
         ↓
  Persistence Layer (PostgreSQL DB, Redis Cache)
         ↑
         |   (API reads/writes data)
         ↓
  API Layer (REST/GraphQL, Auth)
         ↑
         |  (Clients send requests)
         ↓
  Clients (Display UI, Mobile, Admin Panel)

```