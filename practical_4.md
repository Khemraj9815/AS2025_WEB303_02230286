# Practical_04 Report — Kubernetes Microservices with Kong Gateway & Resilience Patterns

## Overview
The Student Cafe is a microservices-based web application deployed on Kubernetes to demonstrate modern DevOps practices such as service discovery, API gateway management with Kong, and containerized deployments.

## Architecture Components

### 1. Frontend Service (React UI)
- **Technology:** React.js with hooks  
- **Port:** 80 (served via Nginx)  
- **Docker Image:** `cafe-ui:v2`  
- **Features:** Interactive menu display, shopping cart, order placement with real-time feedback, responsive UI with emojis.

### 2. Food Catalog Service (Go)
- **Technology:** Go with Chi router  
- **Port:** 8080  
- **Docker Image:** `food-catalog-service:latest`  
- **Endpoints:**
  - `GET /health` — health check
  - `GET /items` — retrieve menu items  
- **Sample data:** Coffee ($2.50), Sandwich ($5.00), Muffin ($3.25)

### 3. Order Service (Go)
- **Technology:** Go with Chi router  
- **Port:** 8081  
- **Docker Image:** `order-service:v2`  
- **Endpoints:**
  - `GET /health` — health check
  - `POST /orders` — create a new order  
- **Behavior:** Communicates with the catalog service to validate items and create orders.

### 4. Infrastructure Services
- **Kong API Gateway:** Routes external requests to services.  
- **Consul:** Service discovery and health monitoring.  
- **Kubernetes:** Container orchestration and service management.

## Deployment Topology
Internet → Kong Proxy → Kubernetes Services → Pods

Port-forward used during testing:
```bash
kubectl port-forward service/kong-kong-proxy 8080:80 -n student-cafe
```

API routes configuration:
- `/api/catalog/*` → Food Catalog Service  
- `/api/orders/*` → Order Service  
- `/*` → Cafe UI Service

## Issues Encountered and Solutions

### Issue 1 — Minikube Service Access
- **Problem:** `minikube service` failed due to QEMU builtin network issues.  
- **Screenshot:** `/assets/practical4Screenshots/minikube-fail.png`  
- **Solution:** Used `kubectl port-forward` as a workaround:
```bash
kubectl port-forward service/kong-kong-proxy 8080:80 -n student-cafe
```
- **Access screenshot:** `/assets/practical4Screenshots/access-application.png`

### Issue 2 — React Build and Deployment
- **Problem:** UI displayed a blank page because the React build was incorrect.  
- **Screenshot:** `/assets/practical4Screenshots/blank.png`  
- **Solution:**
  1. Rebuilt the React app with production optimizations.  
  2. Created a new Docker image that serves static files correctly.  
  3. Updated the Kubernetes deployment to use the new image.

### Issue 3 — Service Discovery Failure
- **Problem:** Order service could not locate `food-catalog-service` via Consul, causing order placement to fail.  
- **Screenshots:**  
  - `/assets/practical4Screenshots/order-fail.png`  
  - `/assets/practical4Screenshots/postman-fail.png`  
- **Root cause:** Consul health checks were failing, preventing service registration/discovery.  
- **Solution:** Implemented a fallback in the order service to use Kubernetes DNS when Consul lookup fails. Example change:
```go
catalogAddr, err := findService("food-catalog-service")
if err != nil {
    log.Printf("Warning: Could not find catalog service (%v), but continuing with order", err)
    catalogAddr = "http://food-catalog-service:8080" // Kubernetes DNS fallback
}
```

## Build and Deployment Commands

### Build Docker images (using Minikube's Docker daemon)
```bash
eval $(minikube docker-env)
docker build -t cafe-ui:v2 ./cafe-ui
docker build -t order-service:v2 ./order-service
docker build -t food-catalog-service:latest ./food-catalog-service
```

### Deploy to Kubernetes
```bash
kubectl apply -f app-deployment.yaml
kubectl apply -f kong-ingress.yaml
```

### Update deployments with new images
```bash
kubectl set image deployment/cafe-ui-deployment cafe-ui=cafe-ui:v2 -n student-cafe
kubectl set image deployment/order-deployment order-service=order-service:v2 -n student-cafe
```

### Access the application (during testing)
```bash
kubectl port-forward service/kong-kong-proxy 8080:80 -n student-cafe
# Open browser to: http://localhost:8080
```

## Screenshots (referenced)
- Food Menu Interface: `/assets/practical4Screenshots/food-menu-display.png`  
- Order Placement Success: `/assets/practical4Screenshots/order-placement-success.png`  
- Kubernetes Pods Status: `/assets/practical4Screenshots/kubectl-get-pods.png`  
- Kubernetes Services Overview: `/assets/practical4Screenshots/kubectl-get-services.png`  
- API Testing (successful): `/assets/practical4Screenshots/postman-success.png`

## Conclusion
- Task 1 — Deploy and Test: All microservices were deployed on Kubernetes and the React UI is reachable via Kong (http://localhost:8080) after using port-forwarding. Order submission initially failed as observed.  
- Task 2 — Debug and Fix: Diagnosis revealed a service-discovery failure between the order and catalog services due to failing Consul health checks. A Kubernetes DNS fallback was added in the order service to maintain functionality when Consul is unavailable.  
- **Final status:** Services are operational and orders can be placed successfully.
