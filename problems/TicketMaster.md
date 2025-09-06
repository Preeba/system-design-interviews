# **TicketMaster**

🎟️ What is Ticketmaster?
- Ticketmaster is an online platform that allows users to purchase tickets for concerts, sports events, theater, and other live entertainment.

## Functional Requirement (**User should be able to..**)
- View the events
- Search for the events
- Book tickets for the events

**Out-Of-Scope**
- User should be able to view their booked events
- Admin or event coordinators should be able to add events
- Popular events should have dynamic pricing

## Non-Functional Requirement (**-lities of the system, features **)
**Note! CAP Theorem --> Consistency (all nodes see the same data), 
Availability (every request gets a response), and Partition Tolerance (the system continues to operate despite network failures)**
0. Low latency search
1. Strong consistency for booking tickets (no double booking)
2. High availability for search and viewing the events
3. read >> write
4. Scalability to handle surges from popular events (Taylor swift show, as an example)

**Out-Of-Scope**
- GDPR compliance
- Fault tolerance
- The system should provide secure transactions for purchases
- The system should have regular backups

## **Core Entities**
1. Event 
2. User
3. Venue
4. Performer
5. Booking
6. Ticket

## **API**
1. View the events
   GET /events/{event_id} --> Event & Venue & Performer & Ticket
2. Search for events
   GET /search?performer=xxx&&venue=xxx&&date={date} --> Event[]
3. Booking ticket -- First reserve the ticket and then confirm with payment. Its 2 step process
   ```
   POST /booking/reserve
   body: {ticked_id}
   ```
    ```
    POST /booking/confirm
      body: {ticked_id,
          paymentDetails(stripe)    
      }
   ```     

## **High-Level Design**


![TicketMasterHighLevel.png](..%2Fdiagrams%2FTicketMasterHighLevel.png)

## **Deep Dive**
### How do we improve booking experience while reserving tickets?

Distributed locking is a method we use in distributed system so that only a single process can access or modify a shared resource at a time, this achieves data consistency and integrity for any transactions.

We can use in-memory data store like Redis which supports distributed locking for our use case. 

#### How does it work?

- We acquire a lock in Redis using a identifier e.g. Booking_ID. We keep a TTL which acts as expiration time.
- If the transaction is completed/committed, the status in the DB is updated a RESERVED/BOOKED and the lock in released after TTL.
- If the TTL expires (indicating the user did not complete the purchase in time), Redis will automatically release the lock. This makes the ticket available to all other page visitors.

##### **Cons:** 
If Redis goes down then we might have users not able to book tickets even after filling in all form details. However, we would not make a double booking as our DB concurrency control will still be there. As system design is a trade of, this appears to be the best one for a better scalable ticket master system.****

---

### what to do in case of high demand/page visit?
#### Handling High Demand (Virtual Waiting Queue)

During peak events (e.g., popular concerts), letting all users hit the booking flow at once risks **timeouts, failures, and overselling**. A simple and effective solution is a **virtual waiting queue**:

* **How it works**:

   * Users are placed in a queue *before* accessing the seat map or booking page.
   * A **WebSocket connection** (or SSE/long polling fallback) keeps users informed of their position in real time.
   * As capacity frees up, users are dequeued and granted access to the booking interface.

* **Pros**:

   * Prevents backend overload during traffic spikes.
   * Provides predictable, fair access and improves user trust.
   * Smooth UX: users see progress instead of failing requests.

* **Cons / trade-offs**:

   * Users may face long waits.
   * Requires additional infra (queue manager, WebSocket servers).
   * Still need **fairness logic** (e.g., randomization vs FIFO) to prevent bots/gaming.

---

### How can you improve search to ensure we meet our low latency requirements?
Our current search implementation is not going to cut it. Queries to search for events based on keywords in the name, description, or other fields will require a full table scan because of the wildcard in the LIKE clause. This can be very slow, especially as the number of events grows
```sql
-- slow query
   SELECT * 
   FROM Events
   WHERE name LIKE '%Taylor%' 
      OR description LIKE '%Taylor%'
```

#### Search with Elasticsearch

Elasticsearch (or OpenSearch) enables **fast, scalable full-text search** across events and venues. It uses **inverted indexes**, mapping terms to documents for sub-second queries—far faster than SQL scans.

##### Integration with PostgreSQL

* Keep data in sync via **Change Data Capture (CDC)** tools (e.g., Debezium, Kafka Connect).
* CDC streams inserts, updates, and deletes from PostgreSQL to Elasticsearch in near real-time.
* SQL remains the **source of truth**; Elasticsearch powers search and discovery.

##### Features

* **Fuzzy search / typo tolerance** (e.g., “Taylor Swift” vs “Tayler Swift”).
* **Autocomplete, filters, synonyms, relevancy boosts**.
* Handles **high query volume** efficiently.

##### Challenges

* **Eventual consistency**: small sync delays possible.
* **Infra complexity & cost**: maintaining an ES cluster.
* **Denormalization**: must flatten relational data into ES docs.

---

#### How can you speed up frequently repeated search queries and reduce load on our search infrastructure?

Here’s a refined, clearer version of your answer — trimmed for conciseness but still showing depth and trade-offs:

---

## Speeding Up Repeated Search Queries

### Approach

* **Elasticsearch caching**:

    * **Node query cache** (LRU): caches results of frequently used filters.
    * **Shard-level request cache**: caches aggregation-heavy search responses.
    * Supports adaptive caching (frequently executed queries get cached automatically).

* **CDN caching**:

    * Cache popular search results closer to users (e.g., via CloudFront).
    * Works best for **non-personalized queries**, where results are the same for all users (e.g., “Taylor Swift concerts”).

### Challenges / Trade-offs

* **Cache invalidation**: must expire or refresh cached queries when new events are added or existing ones change.
* **Consistency vs freshness**: cached results may lag behind real-time DB updates.
* **Infra overhead**: requires managing cache layers (ES + CDN) and adaptive strategies.

---

👉 Basically: **built-in ES cache → CDN → trade-offs**.

