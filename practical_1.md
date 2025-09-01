Here’s your **practical report** written in **Markdown (md) format**:

```markdown
# WEB303 Microservices & Serverless Applications  
## Practical 1: From Foundational Setup to Inter-Service Communication

---

### **Objective**
The objective of this practical is to:
1. Set up a development environment for microservices using Go, Protocol Buffers (Protobuf), gRPC, Docker, and Docker Compose.  
2. Build two microservices (`Greeter Service` and `Time Service`) that communicate over gRPC.  
3. Containerize the services and run them together with Docker Compose.  
4. Test the gRPC endpoints using `grpcurl`.

---

### **Directory Structure**
After setup, the project structure looked like this:

``

.
├── docker-compose.yml
├── final\_output.png
├── go.mod
├── go.sum
├── greeter-service
│   ├── Dockerfile
│   └── main.go
├── proto
│   ├── gen
│   │   ├── greeter\_grpc.pb.go
│   │   ├── greeter.pb.go
│   │   ├── time\_grpc.pb.go
│   │   └── time.pb.go
│   ├── greeter.proto
│   └── time.proto
├── README.md
└── time-service
├── Dockerfile
└── main.go

```

---

### **Steps Performed**

1. **Created Protobuf files** (`greeter.proto` and `time.proto`) inside the `proto/` folder.  
2. **Generated Go files** from `.proto` files using:  
   ```bash
   protoc --go_out=proto/gen --go-grpc_out=proto/gen proto/greeter.proto proto/time.proto
``

3. **Implemented Services in Go**:

   * `Greeter Service` listens on port `50051`.
   * `Time Service` listens on port `50052`.
4. **Wrote Dockerfiles** for each service.
5. **Created docker-compose.yml** to run both services simultaneously.
6. **Started Containers** using:

   ```bash
   docker-compose up --build
   ```
7. **Verified running containers**:

   ```bash
   docker ps
   ```

   Example output:

   ```
   CONTAINER ID   IMAGE                           COMMAND         PORTS
   5588e029539d   practical-one-greeter-service   "/app/server"   0.0.0.0:50051->50051/tcp
   3e46c9dee075   practical-one-time-service      "/app/server"   50052/tcp
   ```
8. **Tested gRPC service** using `grpcurl`:

   ```bash
   grpcurl -plaintext \
       -import-path ./proto -proto greeter.proto \
       -d '{"name": "WEB303 Student"}' \
       0.0.0.0:50051 greeter.GreeterService/SayHello
   ```

---

### **Errors Faced & Solutions**

#### **Error 1: Permission Denied for `.proto` file**

* **Error:**

  ```
  Failed to process proto source files.: could not parse given files: open proto/greeter.proto: permission denied
  ```
* **Cause:** The `grpcurl` process didn’t have permission to read `proto/greeter.proto`.
* **Solution:** Checked file permissions using:

  ```bash
  ls -l proto/
  ```

  Files had correct read permissions. The issue was **relative path mismatch**. Fixed it by running the command from the correct directory or using the full path:

  ```bash
  grpcurl -plaintext \
      -import-path ./proto -proto ./proto/greeter.proto \
      -d '{"name": "WEB303 Student"}' \
      0.0.0.0:50051 greeter.GreeterService/SayHello
  ```

#### **Error 2: Stopping Containers**

* **Problem:** Containers kept running in the background.
* **Solution:** Stopped using:

  ```bash
  docker stop <container_id>
  ```

  Or all at once:

  ```bash
  docker-compose down
  ```

---

### **Output**

successful gRPC response:

![result](/assets/p1-1.png)

---

### **Conclusion**

In this practical, I successfully:

* Set up gRPC-based microservices in Go.
* Containerized and ran them using Docker Compose.
* Debugged issues related to file permissions and container management.
* Verified functionality by making gRPC calls with `grpcurl`.

This exercise provided hands-on experience with microservices architecture, inter-service communication, and Docker-based deployments.
