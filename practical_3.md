# Microservices with gRPC, PostgreSQL, and Consul for Service Discovery  

## Objective  
The objective of this practical was to design and implement a **microservices architecture** where each service is independent and responsible for a specific functionality. Services communicate via **gRPC**, persist data in **PostgreSQL**, and register themselves with **Consul** for service discovery. An **API Gateway** was also implemented to handle external client requests and route them to the appropriate service.  

---

## Concepts Learned  

### 1. Microservices Architecture  
- Understood how to decompose a system into multiple independent services (`users-service`, `products-service`).  
- Each service maintains its **own database**, ensuring better decoupling and scalability.  
- Learned the benefits of **independence, fault isolation, and maintainability** compared to monolithic systems.  

### 2. gRPC for Service Communication  
- Defined service contracts using **Protocol Buffers (.proto files)**.  
- Implemented **strongly typed, fast, and efficient** RPC communication.  
- Understood how gRPC simplifies inter-service communication in a distributed system.  

### 3. Database Integration with PostgreSQL & GORM  
- Connected services to separate **PostgreSQL containers**.  
- Learned to use **GORM ORM** for defining models, migrations, and database operations.  
- Practiced **auto-migration** for schema management.  

### 4. Service Discovery with Consul  
- Learned the role of **Consul** in enabling service discovery and health checks.  
- Registered services dynamically so that the API Gateway can discover healthy service instances.  
- Understood how Consul ensures **availability and fault tolerance** in microservices.  

### 5. API Gateway  
- Implemented an **API Gateway** that receives client requests (REST/HTTP) and forwards them to gRPC services.  
- Learned how the gateway interacts with **Consul** to discover service addresses dynamically.  

---

## Problems Faced & Solutions  

### 1. **Docker Build Errors with go.mod**  
- **Problem:** During container builds, `go mod download` failed because the context did not include the `go.mod` and `go.sum` files properly.  
- **Solution:** Fixed by restructuring Dockerfiles to copy `go.mod` and `go.sum` first, run `go mod download`, and then copy the source code.  

---

### 2. **Port Conflicts (5432 & 8080 already in use)**  
- **Problem:** PostgreSQL and API Gateway ports were already occupied by local processes.  
- **Solution:** Resolved by mapping **different host ports** in `docker-compose.yml` (e.g., `55433:5432` for products-db).  

---

### 3. **Service Not Registering in Consul**  
- **Problem:** Services were not appearing in Consul UI because the `Address` field in Consul registration was incorrect.  
- **Solution:** Fixed by setting `Address` to the **Docker container name** (e.g., `users-service`, `products-service`), ensuring internal DNS resolution worked.  

---

### 4. **API Gateway: Service Unavailable Error**  
- **Problem:** API Gateway returned `Service unavailable: no healthy instances found`.  
- **Solution:** Verified that services were successfully registered in Consul, added **health checks**, and waited for services to pass health status before the gateway routed requests.  

---

## Conclusion  
From this practical, I learned how to design and implement a **microservices system** using gRPC, PostgreSQL, Consul, and Docker. I gained hands-on experience with:  
- Service decomposition and containerization.  
- Efficient gRPC-based communication.  
- Dynamic service discovery with Consul.  
- Handling practical issues like **Docker networking, port conflicts, and service registration**.  

This practical gave me strong foundational knowledge of **modern distributed system design** and how different components (DB, service registry, gateway) integrate together.  

---
