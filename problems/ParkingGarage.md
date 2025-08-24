Here's a basic implementation of a **Parking Garage API** in Java using Spring Boot. This API includes endpoints for managing vehicles, parking spots, and the parking process.

---

### **Assumptions**
- The garage has multiple parking spots, each identified by a unique ID.
- Vehicles can be parked and unparked.
- The system tracks available and occupied spots.

---

### **Directory Structure**
```
src/
├── main/
│   ├── java/com/example/parkinggarage/
│   │   ├── controller/
│   │   │   ├── ParkingController.java
│   │   ├── model/
│   │   │   ├── ParkingSpot.java
│   │   │   ├── Vehicle.java
│   │   ├── service/
│   │   │   ├── ParkingService.java
│   │   ├── repository/
│   │   │   ├── ParkingSpotRepository.java
│   │   └── ParkingGarageApplication.java
```

---

### **Implementation**

#### 1. `ParkingSpot` Model
```java
package com.example.parkinggarage.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class ParkingSpot {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private boolean isOccupied;

    private String vehicleNumber;

    // Getters and Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public boolean isOccupied() {
        return isOccupied;
    }

    public void setOccupied(boolean occupied) {
        isOccupied = occupied;
    }

    public String getVehicleNumber() {
        return vehicleNumber;
    }

    public void setVehicleNumber(String vehicleNumber) {
        this.vehicleNumber = vehicleNumber;
    }
}
```

---

#### 2. `Vehicle` Model
```java
package com.example.parkinggarage.model;

public class Vehicle {
    private String vehicleNumber;

    // Getters and Setters
    public String getVehicleNumber() {
        return vehicleNumber;
    }

    public void setVehicleNumber(String vehicleNumber) {
        this.vehicleNumber = vehicleNumber;
    }
}
```

---

#### 3. `ParkingSpotRepository`
```java
package com.example.parkinggarage.repository;

import com.example.parkinggarage.model.ParkingSpot;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface ParkingSpotRepository extends JpaRepository<ParkingSpot, Long> {
    Optional<ParkingSpot> findFirstByIsOccupiedFalse();
    Optional<ParkingSpot> findByVehicleNumber(String vehicleNumber);
}
```

---

#### 4. `ParkingService`
```java
package com.example.parkinggarage.service;

import com.example.parkinggarage.model.ParkingSpot;
import com.example.parkinggarage.model.Vehicle;
import com.example.parkinggarage.repository.ParkingSpotRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.Optional;

@Service
public class ParkingService {

    @Autowired
    private ParkingSpotRepository parkingSpotRepository;

    public ParkingSpot parkVehicle(Vehicle vehicle) {
        Optional<ParkingSpot> availableSpot = parkingSpotRepository.findFirstByIsOccupiedFalse();
        if (availableSpot.isEmpty()) {
            throw new RuntimeException("No available parking spots.");
        }

        ParkingSpot spot = availableSpot.get();
        spot.setOccupied(true);
        spot.setVehicleNumber(vehicle.getVehicleNumber());
        return parkingSpotRepository.save(spot);
    }

    public ParkingSpot unparkVehicle(String vehicleNumber) {
        Optional<ParkingSpot> occupiedSpot = parkingSpotRepository.findByVehicleNumber(vehicleNumber);
        if (occupiedSpot.isEmpty()) {
            throw new RuntimeException("Vehicle not found in the parking garage.");
        }

        ParkingSpot spot = occupiedSpot.get();
        spot.setOccupied(false);
        spot.setVehicleNumber(null);
        return parkingSpotRepository.save(spot);
    }

    public long getAvailableSpots() {
        return parkingSpotRepository.findAll().stream().filter(spot -> !spot.isOccupied()).count();
    }
}
```

---

#### 5. `ParkingController`
```java
package com.example.parkinggarage.controller;

import com.example.parkinggarage.model.ParkingSpot;
import com.example.parkinggarage.model.Vehicle;
import com.example.parkinggarage.service.ParkingService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/parking")
public class ParkingController {

    @Autowired
    private ParkingService parkingService;

    @PostMapping("/park")
    public ResponseEntity<ParkingSpot> parkVehicle(@RequestBody Vehicle vehicle) {
        ParkingSpot spot = parkingService.parkVehicle(vehicle);
        return ResponseEntity.ok(spot);
    }

    @PostMapping("/unpark")
    public ResponseEntity<ParkingSpot> unparkVehicle(@RequestParam String vehicleNumber) {
        ParkingSpot spot = parkingService.unparkVehicle(vehicleNumber);
        return ResponseEntity.ok(spot);
    }

    @GetMapping("/available-spots")
    public ResponseEntity<Long> getAvailableSpots() {
        long availableSpots = parkingService.getAvailableSpots();
        return ResponseEntity.ok(availableSpots);
    }
}
```

---

#### 6. `ParkingGarageApplication`
```java
package com.example.parkinggarage;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ParkingGarageApplication {
    public static void main(String[] args) {
        SpringApplication.run(ParkingGarageApplication.class, args);
    }
}
```

---

### **Endpoints**

1. **Park a Vehicle**  
   **POST** `/api/parking/park`  
   **Request Body**:
   ```json
   {
       "vehicleNumber": "ABC123"
   }
   ```
   **Response**:
   ```json
   {
       "id": 1,
       "occupied": true,
       "vehicleNumber": "ABC123"
   }
   ```

2. **Unpark a Vehicle**  
   **POST** `/api/parking/unpark?vehicleNumber=ABC123`  
   **Response**:
   ```json
   {
       "id": 1,
       "occupied": false,
       "vehicleNumber": null
   }
   ```

3. **Get Available Spots**  
   **GET** `/api/parking/available-spots`  
   **Response**:
   ```json
   5
   ```

---

### **Future Enhancements**
- Add authentication for users.
- Introduce different parking zones (e.g., compact, large vehicles).
- Support pricing and payment integration.
- Add detailed logs for auditing.

--- 

An **Object-Oriented Design (OOD) class model** represents the structure of a system using classes, their attributes, methods, and relationships between classes, such as associations, inheritances, and dependencies. It is typically illustrated through **class diagrams**.

A **sequence diagram** models the interaction between objects over time to accomplish a specific task or process. It shows how objects communicate through method calls and responses.

---

### **Parking Garage OOD Class Model**

#### **Classes**
1. **ParkingGarage**
    - **Attributes**: `name`, `location`, `parkingSpots`
    - **Methods**: `getAvailableSpots()`, `parkVehicle(vehicle)`, `unparkVehicle(vehicleNumber)`

2. **ParkingSpot**
    - **Attributes**: `id`, `isOccupied`, `vehicleNumber`, `type`
    - **Methods**: `assignVehicle(vehicle)`, `removeVehicle()`

3. **Vehicle**
    - **Attributes**: `vehicleNumber`, `type`
    - **Methods**: None

4. **Ticket**
    - **Attributes**: `ticketId`, `entryTime`, `exitTime`, `vehicleNumber`, `spotId`
    - **Methods**: None

5. **Payment**
    - **Attributes**: `paymentId`, `amount`, `ticketId`, `status`
    - **Methods**: `processPayment()`

6. **User**
    - **Attributes**: `userId`, `name`, `contactDetails`
    - **Methods**: `requestParking(vehicle)`, `exitParking(ticketId)`

---

### **Class Diagram**

```plaintext
+----------------+          +----------------+          +----------------+
|  ParkingGarage |<>-------|  ParkingSpot   |<>-------|    Vehicle      |
|----------------|          |----------------|          |----------------|
| - name         |          | - id           |          | - vehicleNumber|
| - location     |          | - isOccupied   |          | - type         |
| - parkingSpots |          | - vehicleNumber|          +----------------+
|----------------|          | - type         |
| + getAvailableSpots() |   |----------------|
| + parkVehicle(vehicle)|   | + assignVehicle(vehicle) |
| + unparkVehicle(id)   |   | + removeVehicle()        |
+----------------+          +----------------+
       ^
       |
       |    +-------------+
       +--> |    Ticket   |
            |-------------|
            | - ticketId  |
            | - entryTime |
            | - exitTime  |
            | - spotId    |
            | - vehicleNo |
            +-------------+
            + processPayment()
```

---

### **Sequence Diagram**

#### **Scenario: Vehicle Parking**

1. User requests to park a vehicle.
2. ParkingGarage checks for available spots.
3. If available:
    - Assigns a spot to the vehicle.
    - Creates a ticket.
4. Returns ticket details to the user.

```plaintext
User            ParkingGarage          ParkingSpot        Vehicle          Ticket
 |                     |                     |               |               |
 |  parkVehicle()      |                     |               |               |
 |-------------------->|                     |               |               |
 |                     | getAvailableSpots()|               |               |
 |                     |------------------->|               |               |
 |                     | Spot found         |               |               |
 |                     |<-------------------|               |               |
 |                     | assignVehicle()    |               |               |
 |                     |------------------->|               |               |
 |                     |   Vehicle assigned |               |               |
 |                     |<-------------------|               |               |
 |                     | createTicket()     |               |               |
 |                     |------------------->|               |               |
 |                     |   Ticket created   |               |               |
 |<--------------------|                     |               |               |
```

---

### **Key Concepts**
1. **Class Diagram**: Models the static structure of the system, focusing on the relationship between classes.
2. **Sequence Diagram**: Focuses on the dynamic interaction, showing method calls and responses in a specific scenario.

--- 

Introducing API versioning is essential to maintain backward compatibility when making changes like adding a new 
attribute, modifying existing functionality, or introducing new features. Here’s a structured approach to introducing API versioning:

---

### **1. Versioning Approaches**

#### **1.1 URL Versioning**
Include the version number in the URL.
- **Example**:
    - Old version: `/api/v1/parkingGarage`
    - New version: `/api/v2/parkingGarage`

#### **1.2 Query Parameter Versioning**
Add a version parameter to the query string.
- **Example**: `/api/parkingGarage?version=2`

#### **1.3 Header Versioning**
Use a custom header to specify the version.
- **Example**:  
  Header: `API-Version: 2`

#### **1.4 Content Negotiation**
Use the `Accept` header to specify the version.
- **Example**:  
  Header: `Accept: application/vnd.parkinggarage.v2+json`

---

### **2. Adding a New Attribute**
Let’s say you want to add an attribute, `garageType`, to the `ParkingGarage` resource.

#### **Implementation Steps**

1. **Create a New Version**  
   If using URL versioning, create a new version endpoint:
    - Old: `/api/v1/parkingGarage`
    - New: `/api/v2/parkingGarage`

2. **Update the Model**  
   Add the new attribute to the model for version 2.
   ```java
   public class ParkingGarageV2 {
       private String name;
       private String location;
       private List<ParkingSpot> parkingSpots;
       private String garageType; // New attribute
   }
   ```

3. **Implement the New Version Logic**  
   Create a new controller or handler for the v2 endpoint.
   ```java
   @RestController
   @RequestMapping("/api/v2")
   public class ParkingGarageControllerV2 {
       @GetMapping("/parkingGarage")
       public ResponseEntity<ParkingGarageV2> getParkingGarage() {
           ParkingGarageV2 garage = new ParkingGarageV2();
           garage.setName("Main Garage");
           garage.setLocation("Downtown");
           garage.setGarageType("Multi-level"); // New data
           return ResponseEntity.ok(garage);
       }
   }
   ```

4. **Document the Change**  
   Update your API documentation (e.g., OpenAPI/Swagger) to reflect the new version and attribute.

---

### **3. Supporting Multiple Versions**
You can maintain both v1 and v2 concurrently:
- Implement separate controllers for each version.
- Use routing to differentiate between versions.

#### Example of Version-Specific Controllers
```java
@RestController
@RequestMapping("/api/v1")
public class ParkingGarageControllerV1 {
    @GetMapping("/parkingGarage")
    public ResponseEntity<ParkingGarage> getParkingGarage() {
        ParkingGarage garage = new ParkingGarage();
        garage.setName("Main Garage");
        garage.setLocation("Downtown");
        return ResponseEntity.ok(garage);
    }
}

@RestController
@RequestMapping("/api/v2")
public class ParkingGarageControllerV2 {
    @GetMapping("/parkingGarage")
    public ResponseEntity<ParkingGarageV2> getParkingGarage() {
        ParkingGarageV2 garage = new ParkingGarageV2();
        garage.setName("Main Garage");
        garage.setLocation("Downtown");
        garage.setGarageType("Multi-level");
        return ResponseEntity.ok(garage);
    }
}
```

---

### **4. Testing and Migration**
1. **Test Both Versions**: Ensure both versions work independently.
2. **Deprecate Older Versions**: Notify consumers of the API to migrate to the newer version. Set a sunset date for older versions.

---

### **5. Best Practices**
1. **Use Semantic Versioning**: Follow a versioning strategy like `v1.0`, `v2.0`, or `v1.1` for minor changes.
2. **Deprecate Gracefully**: Provide adequate time and tools for API consumers to migrate.
3. **Documentation**: Maintain detailed API documentation for each version.
4. **Backward Compatibility**: Ensure that older versions are not broken by new changes.
