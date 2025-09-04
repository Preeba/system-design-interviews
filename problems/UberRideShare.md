# Designing Uber (or any rideshare system)

**Designing Uber (or any rideshare system)** can cover almost all the patterns —**microservices, SAGA, event-driven architecture, message queues, circuit breakers**, etc.—but the key is **how to model the problem**. Lets break it down:

### ✅ **References:**
- https://www.youtube.com/watch?v=DGtalg5efCw
- https://www.youtube.com/watch?v=lsKU38RKQSo&t=3617s (HelloInterview.com)
- https://highscalability.com/how-uber-scales-their-real-time-market-platform/
---

### **Core Uber System Components**

1. **Ride Service** – Manages ride requests, cancellations, and status updates.
2. **User Service** – Manages riders and drivers.
3. **Payment Service** – Handles payments, refunds, and wallets.
4. **Driver Matching Service** – Matches riders with nearby drivers.
5. **Notification Service** – Sends SMS/push notifications.
6. **Pricing Service** – Calculates fares dynamically (surge pricing).
7. **Trip History / Analytics Service** – Stores completed rides and analytics data.

---

### **How it covers key system design patterns**

| Pattern                       | How Uber Example Covers It                                                                                                                                                                  |
|-------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Microservices**             | Each component above can be a separate microservice, with its own database.                                                                                                                 |
| **SAGA Pattern**              | Ride booking involves multiple steps: request → match driver → authorize payment → confirm ride. Failures trigger compensating transactions: cancel ride, release driver, refund payment.   |
| **Event-driven Architecture** | Events like `RideRequested`, `DriverMatched`, `PaymentAuthorized`, `RideCompleted` flow asynchronously between services.                                                                    |
| **Message Queue**             | Kafka, RabbitMQ, or SQS can be used to handle event streams reliably.                                                                                                                       |
| **Circuit Breakers**          | If Payment Service or Driver Matching Service is down, other services don’t fail immediately—fallback strategies or queued requests can handle the load.                                    |
| **Retries & Idempotency**     | Events may be retried if a service temporarily fails (e.g., sending a push notification). Idempotent operations prevent double charges or double matches.                                   |
| **Scalability Concerns**      | Handling surge hours, global ride requests, and geospatial queries demonstrates distributed system design considerations.                                                                   |

---

### ✅ **Conclusion**

**“Design Uber”** can be a universal system design problem for interviews, because it naturally involves multiple services, eventual consistency, async communication, failure handling, and scaling concerns.

Compared to **E-commerce Order Management**, the Uber example emphasizes **real-time, geospatial, and low-latency aspects**

---

![rideShareFig1.png](..%2Fdiagrams%2FrideShareFig1.png)

#### Data Model

Code Entities:
1. Rider
2. Driver
3. Fare
4. Ride
5. Location

![rideShareDataModelFig.png](..%2Fdiagrams%2FrideShareDataModelFig.png)

#### API Design
1. Create a new Ride object -  takes in the user's current location and desired destination and returns a Fare object with the estimated fare and eta. 
```
    POST /fares/estimate -> Fare
    Body: {
      pickupLocation, 
      destination
    }
```
   
2. Request Ride Endpoint:
    ```
   POST /rides/request -> Ride
    Body: {
    fareId
    }
   ```

3. Update Driver Location Endpoint 
```
    POST /drivers/location -> Success/Error
    Body: {
        lat, long
    }

- note the driverId is present in the session cookie or JWT and not in the body or path params
```

4. Accept Ride Request Endpoint
```
    PATCH /rides/{rideId} -> Ride
    Body: {
      accept/deny
    }
```

#### High-Level Design
##### **Driver Location Update Flow**
![DriverLocationUpdateFlow.png](..%2Fdiagrams%2FDriverLocationUpdateFlow.png)

##### **Ride Request Flow**
![RideRequestFlow.png](..%2Fdiagrams%2FRideRequestFlow.png)


### High Level Design from HelloInterview.com

![Uber.png](..%2Fdiagrams%2FUber.png)