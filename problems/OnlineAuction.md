# Design Online Auction

# Goals / Requirements

**Functional**

* Sellers can **create auctions** (item, start/end time, starting price, images, description).
* Buyers can **place bids** on auctions.
* Users can **view auction details** and **see live updates** (current highest bid, new bids).
* Support queries: list auctions, get bids for an auction, get auction by id.
* Basic payment/closing workflow (winner pays once auction ends).

**Non-functional**

* Low-latency updates to watchers (near real-time).
* High write throughput for bids (hot auctions).
* Strong ordering for bids per auction (so we know which bid came first).
* Durability — no lost bids.
* Scalability: many concurrent auctions and bidders.
* Reasonable consistency: latest accepted bid must be authoritative (no two winners).
* Fraud detection and rate limits.

---

# Core entities & example schemas

A relational-style representation (diagram shows **Postgres**) — good for transactions and constraints.

**Auction**

```sql
Auction {
  id UUID,
  itemId UUID,
  sellerId UUID,
  startTime TIMESTAMP,
  endTime TIMESTAMP,
  startingPrice DECIMAL,
  maxBidId UUID,   -- optional cached pointer
  status ENUM('upcoming','live','closed','cancelled'),
  createdAt TIMESTAMP,
  updatedAt TIMESTAMP
}
```

**Item**

```sql
Item {
  id UUID,
  name TEXT,
  description TEXT,
  imageLinks JSONB,
  category TEXT,
  metadata JSONB
}
```

**Bid**

```sql
Bid {
  id UUID,
  auctionId UUID,   -- partition key for ordering
  userId UUID,
  amount DECIMAL,
  createdAt TIMESTAMP,
  status ENUM('accepted','rejected')  -- rejection reasons: outbid, late, invalid
}
```

**User**

```sql
User {
  id UUID,
  username,
  paymentInfoRef,
  reputationScore,
  ...
}
```
---

# High-level components (from diagram)

* **Client** (web/mobile) — views auctions, places bids, receives live updates.
* **API Gateway** — routing, authentication, rate limiting, TLS termination.
* **Auction Service** — manages auctions lifecycle (create, schedule start/end, compute final winner).
* **Bid Service** — validates and accepts/rejects incoming bids; high-throughput write path.
* **Kafka (or message broker)** — durable ordered log for bids/events; used for decoupling and replay.
* **Pub/Sub / Push subsystem** — broadcasts events to watchers (WebSocket / SSE / Push).
* **Database (Postgres )** — canonical store for auctions, items, bids (or at least auctions & metadata).
* **Cache (Redis)** — hot item/maxBid caching for low latency reads.
* **Search/index (Elasticsearch)** — item search and filtering.
* **Payment/Settlement Service** — handle payments after auction close.
* **Fraud and Monitoring subsystems**.

---

# Data flow — placing a bid (step-by-step)

1. **Client** sends `POST /auctions/{id}/bids` to API Gateway (via WebSocket or HTTPS).
2. **API Gateway** authenticates, rate-limits, forwards to **Bid Service**.
3. **Bid Service** performs **quick validation**:

    * Auction is `live` (startTime <= now <= endTime).
    * Bid amount > current highest + minIncrement.
    * User is allowed (not blocked).
4. If quick checks pass, **Bid Service** writes an event to **Kafka** (topic: `bids`, partitioned by `auctionId`) — this gives durable, ordered append.

    * Alternatively, for very small setups you could write directly to DB in a single transaction, but you lose the decoupling & replay benefits.
5. A **consumer** of Kafka (could be the same Bid Service or a dedicated worker) pulls the message:

    * Performs authoritative validation **in a transaction** (read current highest bid from DB or cache, ensure this bid still wins).
    * If accepted:

        * Insert `Bid` row (or append to bids table).
        * Update `Auction.maxBidId` (or update materialized view).
        * Publish `BidAccepted` event to a **pub/sub** topic for real-time notifications.
        * Update cache (Redis) with new highest bid.
    * If rejected:

        * Insert `Bid` with `rejected` status or emit `BidRejected` event (with reason).
6. **Push**: Pub/sub pushes events to connected clients (WebSocket/SSE). Clients subscribed to auction updates receive the new highest bid instantly.
7. **End of Auction**: Auction Service (or cron job) consumes `bids` topic/DB and when `endTime` is reached:

    * Marks `Auction.status = closed`.
    * Determines winner (highest accepted bid).
    * Triggers Payment Service (invoicing).
    * Sends notifications.

Key guarantees:

* **Ordering per auction**: use Kafka partitioning keyed by `auctionId` so all bids for a given auction are ordered.
* **Durability**: Kafka + DB ensure events are persisted; job replay can reconstruct state if needed.
* **Low-latency reads**: Redis holds currentMaxBid per auction for constant-time checks and immediate responses to viewers.

---

![OnlineAuction.png](..%2Fdiagrams%2FOnlineAuction.png)

# API surface (examples)

**Create auction**

```
POST /auctions
Body: { itemId, startTime, endTime, startingPrice, sellerId, ... }
Response: { auctionId }
```

**Get auction**

```
GET /auctions/{id}
Response: { auction, currentMaxBid, topNRecentBids }
```

**List auctions**

```
GET /auctions?status=live&category=...
```

**Place bid**

```
POST /auctions/{id}/bids
Body: { userId, amount, idempotencyKey? }
Response: { status: 'accepted'|'rejected', bidId, reason? }
```

**Subscribe (real-time)**

* WebSocket: subscribe to `auction:{id}` events or use SSE endpoint `GET /auctions/{id}/stream`.

---

# Database choices & rationale

**Primary choice: Postgres (relational)** — diagram shows Postgres, and it's a sensible default for:

* ACID transactions (ensuring single-winner consistency at auction close).
* Complex queries/joins (items, sellers, bids).
* Constraints and easy schema evolution (JSONB for flexible metadata).
* Analytical queries (with read replicas).

**Complementary stores**

* **Kafka** as durable, ordered event log for bids (good for high write throughput, ordering per auction, replay).
* **Redis** as an in-memory cache for current highest bid & rate limiting / locks.
* **Elasticsearch** for search/filtering over items/auctions.
* For extreme scale: store bids in an append-only store like **Cassandra** or S3 if needed (cheap write-heavy storage) and keep a materialized top-k in Postgres/Redis.

---

# Concurrency & correctness strategies

* **Single-writer per auction semantic**: by routing all bid writes through Kafka partitioned by `auctionId`, the consumer processes them serially — simplifies concurrent acceptance logic.
* **Transactional update**: consumer uses DB transaction to compare stored highest bid and insert/update atomically. Alternatively use `SELECT ... FOR UPDATE` or optimistic concurrency with version numbers.
* **Idempotency**: clients provide idempotency keys so repeated submissions aren’t double-counted.
* **Anti-sniping**: if a valid bid arrives within N seconds of `endTime`, automatically extend `endTime` by M seconds (configurable). Auction Service enforces this inside the closing transaction.
* **Rate limiting & anti-fraud**: per-user and per-IP limits; flag unusual rapid bidding.

---

# Scalability & availability

* **Stateless services** (API, Bid/Auction services) behind load balancers; autoscale based on load.
* **Kafka** handles throughput; partition by auctionId to spread load across brokers.
* **Database scaling**:

    * Read replicas for query load.
    * Partition/shard auctions by range or hash for very large scale.
    * Use materialized views for common read patterns.
* **Cache**: Redis cluster for hot auctions.
* **Backpressure**: if consumer lag grows, throttle new bids or return transient error to clients.
* **Disaster recovery**: backups of Postgres and retention policy for Kafka topics; event replay can rebuild read models.

---

# Observability & operational pieces

* Audit logs for bids (immutable log).
* Metrics: bids/sec, latency, consumer lag, cache hit rate.
* Alerts on DB replication lag, Kafka partition under-replicated, high error rates.
* Reconciliation job to compare Kafka event log and DB state periodically.

---

# Edge cases & failure modes

* **Network partition / consumer down**: Kafka retains events; bids are durable but may not be accepted until consumer processes them—respond to client with “queued” vs “accepted”.
* **Simultaneous identical bids**: accept earliest based on ordering from Kafka + DB transaction; if exact equal amounts, tie-break by earlier timestamp or bidder priority.
* **Clock skew**: use server time for start/end checks; UTC, monotonic clocks recommended.
* **DB outage**: continue to accept into Kafka (if the bid service can append), then replay to DB when restored — but must surface to clients the difference between “persisted” and “processed”.

---

# Security & compliance

* Authentication (OAuth/JWT), authorization checks (seller vs buyer), rate limiting.
* Secure storing of PII (payment details via PCI-compliant provider).
* Prevent injection and validate inputs (amounts, IDs).
* Fraud detection subsystem (sudden bid spikes, shill bidding detection).

---

# Variations & tradeoffs

* **Direct DB writes vs event log**:

    * Direct DB writes are simpler for small systems but harder to scale and harder to handle replay and stream processing.
    * Kafka + consumers add complexity but give ordering, decoupling, durability, and replay.
* **Relational vs NoSQL for bids**:

    * Relational gives ACID guarantees for final winner selection.
    * For extremely high throughput you can append to a NoSQL append store and maintain small relational transactional state for currentWinner.
* **WebSocket vs SSE**:

    * Use WebSocket when you need bidirectional comms (client-side bid confirmations, heartbeats).
    * SSE is simpler for server→client updates (one-way).

---

# Short sequence diagram (textual)

Client → API Gateway → Bid Service → Kafka (append)
Kafka → Bid Consumer → DB (insert bid & update auction) → Pub/Sub → Push to Client (WebSocket/SSE)

---

# Final summary (one-line)

Use a **hybrid architecture**: Postgres as canonical store for auctions/metadata and authoritative writes, **Kafka** as an ordered durable event log for incoming bids (partitioned by auctionId), **Bid Service** to validate/append bids, **consumer** to make authoritative DB updates and publish events, and **WebSocket/SSE + Redis** for near-real-time, low-latency updates to many watchers — plus search (Elasticsearch), caching, and proper transaction/ordering safeguards for correctness and scale.

----
# **Temporal.io** can be *very* useful in an online auction platform like this.

---

## 💡 First — What Temporal solves

Temporal is ideal for:

* **Long-running workflows** (hours, days, even weeks).
* **Reliable orchestration** of steps that must survive restarts and retries.
* **Guaranteed execution** even if a service crashes or is redeployed.
* **Complex state transitions** across multiple services (with retries, compensation logic, timers).

Essentially, Temporal ensures **“exactly-once workflow execution”** even if the underlying infrastructure is unreliable.

---

## ⚙️ Where Temporal fits in this eBay-like design

Let’s overlay Temporal into your existing architecture.

---

### 🧩 1. **Auction Lifecycle Orchestration**

**Best fit.**

Each **auction** naturally has a lifecycle:

1. Wait until `startTime` to transition to “live”.
2. Stay open for bids until `endTime`.
3. When `endTime` arrives:

    * Determine winner.
    * Trigger payment.
    * Send notifications.
    * Archive the auction.

This entire lifecycle is a **long-running, event-driven process** — perfect for a Temporal **Workflow**.

**Workflow name:** `AuctionWorkflow`

**Activities inside the workflow:**

* `WaitUntil(startTime)`
* `MarkAuctionLive()`
* `WaitUntil(endTime)`
* `DetermineWinner()`
* `TriggerPaymentWorkflow(winnerId)`
* `SendNotifications()`
* `ArchiveAuctionData()`

**Why Temporal helps here:**

* You avoid relying on **cron jobs** or **polling schedulers** that check DB rows every few seconds/minutes.
* If your Auction Service crashes midway (e.g., between closing and payment), Temporal automatically resumes the workflow.
* Temporal timers scale well — millions of timers can wait efficiently.

**Placement:**

* Temporal Worker runs inside the **Auction Service**.
* Each auction creation triggers a new Temporal Workflow instance.

---

### 🧩 2. **Bid Settlement / Payment Workflow**

After an auction closes, you typically:

1. Charge the winner’s payment method.
2. Notify both parties.
3. Handle potential payment failures (retry, cancel, escalate).
4. Update statuses atomically.

Each of these steps may:

* Call external systems (e.g., Stripe, PayPal).
* Require retries and compensations (refund if failure).

So you can have another **Workflow** called `PaymentWorkflow`.

**Activities:**

* `ChargeUser(winnerId, amount)`
* `NotifySellerAndBuyer()`
* `RetryOnFailure()`
* `MarkAsPaid()`

---

### 🧩 3. **Anti-sniping extensions**

If you implement the rule:

> “If a valid bid comes within the last 10 seconds, extend the auction by 30 seconds,”

Temporal helps here too.

* You can set a **timer** in the `AuctionWorkflow` for the `endTime`.
* If a **signal** (new high bid event) arrives within that window, the workflow can **reset its timer** dynamically.

This avoids messy DB race conditions or scheduling updates in multiple systems.

---

### 🧩 4. **Bid Processing Pipeline (optional)**

If you rely on Kafka for ordered bid ingestion, Temporal might not be ideal for high-throughput bid-level handling (Kafka is faster and more specialized).
However, Temporal can help for:

* **Audit correction**: A workflow that replays bids from Kafka and reconciles DB state nightly.
* **Retry logic**: If bid processing fails due to DB or cache issues, Temporal guarantees retry with exponential backoff.

Still, this is less common — Kafka + consumer retries usually suffice for real-time bids.

---

### 🧩 5. **User Notification / Email Pipeline**

After key events (auction started, outbid, auction ended, etc.), Temporal can orchestrate notification workflows.

For example:

* **`OutbidNotificationWorkflow`**

    * Wait for Kafka event “bid_rejected”.
    * Send notification.
    * Retry on failure.
    * Log result.

These can run as lightweight asynchronous workflows that fan out from event streams.

---

## 🏗️ Architectural integration

Here’s how Temporal fits into the flow:

```
Client → API Gateway → Auction Service → Temporal Workflow
                                            ↓
                                      [AuctionWorkflow]
                                          ↓ timers
                                          ↓ signals (bids)
                                          ↓
                                 triggers Bid Service / Payment
                                           ↓
                                 Temporal Activities:
                                   - DB updates
                                   - Pub/Sub notifications
                                   - Payment integration
```

**In short:**

* **Auction creation** → triggers a Temporal `AuctionWorkflow`.
* **Bid Service** → sends **signals** (like “new high bid”) to the relevant `AuctionWorkflow`.
* **Workflow** → manages timers and transitions (live → closed).
* **Workflow completion** → triggers `PaymentWorkflow`.

---

## 🧠 Summary — Temporal’s roles in this design

| Use Case                          | Why Temporal Fits                | Benefits                              |
| --------------------------------- | -------------------------------- | ------------------------------------- |
| **Auction lifecycle**             | Long-lived, timed, multi-step    | No cron, reliable timers, auto-resume |
| **Payment orchestration**         | External API calls with retries  | Exactly-once, easy rollback           |
| **Notifications**                 | Background async tasks           | Reliable fan-out                      |
| **Anti-sniping**                  | Dynamic timer resets via signals | Clean event-driven extension          |
| **Bid reconciliation (optional)** | Reliable batch reprocessing      | Resilient to transient errors         |

---

## 🚀 TL;DR

✅ **Use Temporal for**:

* Orchestrating auction start/end timers and closing logic.
* Payment and notification workflows.
* Any multi-step or retry-prone async operations.

❌ **Don’t use Temporal for**:

* Real-time, high-throughput bid ingestion or updates (Kafka remains better here).
* Short synchronous request-response APIs.

---
