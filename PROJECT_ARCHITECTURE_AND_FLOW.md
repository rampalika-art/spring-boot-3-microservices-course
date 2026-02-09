# Spring Boot 3 Microservices - Project Architecture and Flow

## Table of Contents
1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Services Breakdown](#services-breakdown)
4. [Technology Stack](#technology-stack)
5. [Communication Flow](#communication-flow)
6. [Request Flow Examples](#request-flow-examples)
7. [Infrastructure Components](#infrastructure-components)
8. [Security](#security)
9. [Observability](#observability)

---

## Overview

This is a **Spring Boot 3 Microservices** project that demonstrates a complete e-commerce shop backend system using modern microservices architecture patterns. The system allows users to browse products, place orders, check inventory availability, and receive notifications about their orders.

### Key Features
- **Microservices Architecture**: Independent, scalable services
- **Event-Driven Communication**: Asynchronous messaging with Kafka
- **API Gateway Pattern**: Single entry point for all client requests
- **Service Discovery & Load Balancing**: Built-in resilience
- **Security**: OAuth 2.0 / OpenID Connect with Keycloak
- **Observability**: Distributed tracing, logging, and metrics
- **Containerization**: Docker and Kubernetes ready

---

## Architecture

The project follows a **microservices architecture** with the following components:

```
┌─────────────┐
│   Angular   │  Frontend Application
│  Frontend   │
└──────┬──────┘
       │
       ↓
┌─────────────────┐
│   API Gateway   │  Single Entry Point (Port 9000)
│  (Spring Cloud  │  - Routing
│   Gateway MVC)  │  - Circuit Breaker
└────────┬────────┘  - Load Balancing
         │
    ┌────┴────┬────────────┬───────────┐
    ↓         ↓            ↓           ↓
┌─────────┐ ┌────────┐ ┌──────────┐ ┌───────────┐
│ Product │ │ Order  │ │Inventory │ │Notification│
│ Service │ │Service │ │ Service  │ │  Service   │
└────┬────┘ └───┬────┘ └────┬─────┘ └─────┬──────┘
     │          │            │             │
     ↓          ↓            ↓             ↓
┌─────────┐ ┌────────┐ ┌─────────┐   ┌────────┐
│ MongoDB │ │ MySQL  │ │  MySQL  │   │ Kafka  │
└─────────┘ └────────┘ └─────────┘   └────────┘
```

---

## Services Breakdown

### 1. **Product Service**
**Purpose**: Manages product catalog (CRUD operations for products)

**Key Features:**
- Create new products
- Retrieve all products
- Store product information (name, description, price)

**Technology:**
- Database: **MongoDB** (NoSQL for flexible product schema)
- Port: Configured dynamically
- API Endpoints:
  - `POST /api/product` - Create a product
  - `GET /api/product` - Get all products

**Data Model:**
```java
Product {
  id: String
  name: String
  description: String
  price: BigDecimal
}
```

---

### 2. **Order Service**
**Purpose**: Handles order placement and orchestrates the order workflow

**Key Features:**
- Place new orders
- Validate inventory before order placement
- Publish order events to Kafka
- Implements resilience patterns (Circuit Breaker, Retry)

**Technology:**
- Database: **MySQL** (Relational data for transactional integrity)
- Messaging: **Kafka** (Event publishing)
- HTTP Client: Spring WebClient for inter-service communication
- Port: Configured dynamically

**API Endpoints:**
- `POST /api/order` - Place an order

**Data Model:**
```java
Order {
  id: Long
  orderNumber: String (UUID)
  skuCode: String
  price: BigDecimal
  quantity: Integer
}
```

**Key Dependencies:**
- Calls **Inventory Service** to check stock availability
- Publishes **OrderPlacedEvent** to Kafka topic `order-placed`

**Resilience Patterns:**
- `@CircuitBreaker` - Prevents cascading failures
- `@Retry` - Automatic retry on failures
- Fallback method for graceful degradation

---

### 3. **Inventory Service**
**Purpose**: Manages product inventory and stock availability

**Key Features:**
- Check if a product is in stock
- Validate sufficient quantity is available

**Technology:**
- Database: **MySQL**
- Port: Configured dynamically

**API Endpoints:**
- `GET /api/inventory?skuCode={code}&quantity={qty}` - Check stock availability

**Data Model:**
```java
Inventory {
  id: Long
  skuCode: String
  quantity: Integer
}
```

**Business Logic:**
- Returns `true` if product exists and quantity is sufficient
- Returns `false` otherwise

---

### 4. **Notification Service**
**Purpose**: Sends email notifications to customers

**Key Features:**
- Listens to Kafka events
- Sends order confirmation emails
- Asynchronous event-driven communication

**Technology:**
- Messaging: **Kafka Consumer**
- Email: **JavaMailSender** (Spring Mail)
- Port: Configured dynamically

**Event Consumption:**
- Listens to Kafka topic: `order-placed`
- Processes **OrderPlacedEvent**

**Data Flow:**
```java
OrderPlacedEvent {
  orderNumber: String
  email: String
  firstName: String
  lastName: String
}
```

**Email Template:**
```
Hi {firstName} {lastName},

Your order with order number {orderNumber} is now placed successfully.

Best Regards
Spring Shop
```

---

### 5. **API Gateway**
**Purpose**: Single entry point for all client requests

**Key Features:**
- Request routing to backend services
- Circuit breaker for each service
- Load balancing
- Centralized error handling
- Swagger documentation aggregation

**Technology:**
- Framework: **Spring Cloud Gateway MVC**
- Port: **9000**

**Route Configuration:**
```
/api/product    → Product Service
/api/order      → Order Service
/api/inventory  → Inventory Service
```

**Resilience:**
- Each route has its own circuit breaker
- Fallback route returns "Service Unavailable" message
- Graceful degradation on service failures

---

## Technology Stack

### Backend Services
- **Java 21** - Programming language
- **Spring Boot 3.2.4** - Application framework
- **Spring Cloud 2023.0.1** - Microservices patterns
- **Spring Cloud Gateway MVC** - API Gateway
- **Spring Data JPA** - Database access
- **Spring Data MongoDB** - MongoDB integration
- **Spring Kafka** - Event streaming
- **Resilience4j** - Fault tolerance library

### Databases
- **MongoDB 7.0.5** - NoSQL for Product Service
- **MySQL 8.3.0** - Relational DB for Order & Inventory Services

### Messaging & Events
- **Apache Kafka 7.5.0** - Event streaming platform
- **Zookeeper 7.5.0** - Kafka coordination
- **Kafka UI** - Web-based Kafka management

### Security
- **Keycloak 24.0.1** - Identity and Access Management
- **OAuth 2.0 / OpenID Connect** - Authentication & Authorization

### Observability Stack
- **Grafana 10.1.0** - Visualization dashboards
- **Prometheus 2.46.0** - Metrics collection
- **Loki** - Log aggregation
- **Tempo 2.2.2** - Distributed tracing

### Frontend
- **Angular 18** - Modern web framework
- **TypeScript** - Type-safe JavaScript
- **Node.js & NPM** - Runtime and package management

### DevOps & Deployment
- **Docker** - Containerization
- **Kubernetes** - Container orchestration
- **Kind** - Local Kubernetes cluster
- **Maven** - Build tool
- **TestContainers** - Integration testing

---

## Communication Flow

### Synchronous Communication (HTTP/REST)
Used for **request-response** patterns requiring immediate feedback:

1. **Frontend → API Gateway** (HTTP)
2. **API Gateway → Services** (HTTP)
3. **Order Service → Inventory Service** (HTTP via Spring WebClient)

**Example:**
```
Client → API Gateway → Order Service → Inventory Service
                           ↓
                        MySQL DB
```

### Asynchronous Communication (Events/Kafka)
Used for **fire-and-forget** operations and decoupling:

1. **Order Service publishes** `OrderPlacedEvent` to Kafka
2. **Notification Service consumes** event from Kafka
3. Email sent asynchronously

**Benefits:**
- Services remain decoupled
- No direct dependency between Order and Notification services
- Scalable event processing
- Fault tolerance (events are persisted)

---

## Request Flow Examples

### Example 1: Creating a Product

```
┌─────────┐    POST /api/product      ┌─────────────┐
│ Client  │ ───────────────────────→ │ API Gateway │
└─────────┘                           └──────┬──────┘
                                            │
                                            ↓
                                     ┌──────────────┐
                                     │   Product    │
                                     │   Service    │
                                     └──────┬───────┘
                                            │
                                            ↓
                                     ┌──────────────┐
                                     │   MongoDB    │
                                     └──────────────┘

Response: ProductResponse (201 Created)
```

**Request:**
```json
POST http://localhost:9000/api/product
{
  "name": "iPhone 15",
  "description": "Latest Apple smartphone",
  "price": 999.99
}
```

**Response:**
```json
{
  "id": "65a1b2c3d4e5f6789",
  "name": "iPhone 15",
  "description": "Latest Apple smartphone",
  "price": 999.99
}
```

---

### Example 2: Placing an Order (Complete Flow)

This is the **most complex flow** involving multiple services:

```
┌─────────┐
│ Client  │ (1) POST /api/order
└────┬────┘
     │
     ↓
┌────────────┐
│   API      │ (2) Routes to Order Service
│  Gateway   │
└────┬───────┘
     │
     ↓
┌─────────────┐
│   Order     │ (3) Receives order request
│  Service    │
└──┬──────┬───┘
   │      │
   │      └────────────────────────┐
   │                               ↓
   │                        ┌──────────────┐
   │                        │  Inventory   │ (4) Check stock
   │                        │   Service    │
   │                        └──────┬───────┘
   │                               │
   │                               ↓
   │                        ┌──────────────┐
   │                        │ MySQL (Inv)  │ (5) Query inventory
   │                        └──────┬───────┘
   │                               │
   │         (6) Returns true/false│
   │ ◄─────────────────────────────┘
   │
   ↓
┌──────────────┐
│ MySQL (Ord)  │ (7) Save order
└──────┬───────┘
       │
       ↓
┌──────────────┐
│    Kafka     │ (8) Publish OrderPlacedEvent
└──────┬───────┘
       │
       │ (9) Event consumed
       ↓
┌────────────────┐
│ Notification   │ (10) Send email
│   Service      │
└────────────────┘
```

**Detailed Steps:**

1. **Client sends order request** to API Gateway
   ```json
   POST http://localhost:9000/api/order
   {
     "skuCode": "iphone_15",
     "price": 999.99,
     "quantity": 2,
     "userDetails": {
       "email": "customer@example.com",
       "firstName": "John",
       "lastName": "Doe"
     }
   }
   ```

2. **API Gateway** routes request to Order Service
   - Circuit breaker protects against Order Service failures
   - Falls back to error message if service is down

3. **Order Service** processes the request:
   - Receives `OrderRequest`
   - Generates UUID for order number

4. **Order Service calls Inventory Service** via HTTP
   ```
   GET http://inventory-service/api/inventory?skuCode=iphone_15&quantity=2
   ```
   - Uses Spring WebClient (declarative HTTP client)
   - Circuit breaker with retry pattern
   - Fallback returns `false` on failure

5. **Inventory Service checks database**
   - Queries MySQL for `skuCode="iphone_15"`
   - Validates `quantity >= 2`

6. **Inventory Service responds**
   - `true` if in stock
   - `false` if out of stock

7. **If in stock, Order Service saves order**
   ```java
   Order {
     orderNumber: "550e8400-e29b-41d4-a716-446655440000"
     skuCode: "iphone_15"
     price: 1999.98  // (999.99 * 2)
     quantity: 2
   }
   ```

8. **Order Service publishes event to Kafka**
   ```java
   Topic: "order-placed"
   Event: OrderPlacedEvent {
     orderNumber: "550e8400-..."
     email: "customer@example.com"
     firstName: "John"
     lastName: "Doe"
   }
   ```

9. **Notification Service consumes event**
   - Kafka listener picks up the event
   - Runs asynchronously (non-blocking)

10. **Notification Service sends email**
    ```
    To: customer@example.com
    Subject: Your Order with OrderNumber 550e8400-... is placed successfully
    
    Hi John Doe,
    Your order with order number 550e8400-... is now placed successfully.
    
    Best Regards
    Spring Shop
    ```

11. **Response returned to client**
    ```
    "Order Placed Successfully"
    ```

**Error Scenarios:**

- **Inventory Service down**:
  - Circuit breaker opens
  - Fallback returns `false`
  - Order rejected: "Product with SkuCode iphone_15 is not in stock"

- **Product out of stock**:
  - Order rejected with exception
  - No event published
  - Client receives error

- **Email service down**:
  - Order still saved successfully
  - Email failure logged
  - Does not affect order placement (asynchronous)

---

### Example 3: Checking Inventory

```
┌─────────┐    GET /api/inventory?skuCode=xyz&quantity=5    ┌─────────────┐
│ Client  │ ────────────────────────────────────────────→  │ API Gateway │
└─────────┘                                                  └──────┬──────┘
                                                                    │
                                                                    ↓
                                                             ┌──────────────┐
                                                             │  Inventory   │
                                                             │   Service    │
                                                             └──────┬───────┘
                                                                    │
                                                                    ↓
                                                             ┌──────────────┐
                                                             │    MySQL     │
                                                             └──────────────┘

Response: true or false
```

---

## Infrastructure Components

### 1. **MongoDB**
- **Purpose**: Document store for Product Service
- **Port**: 27017
- **Credentials**: root/password
- **Database**: product-service
- **Why MongoDB?**: Flexible schema for product attributes

### 2. **MySQL (2 instances)**
- **Purpose**: Relational database for Order and Inventory services
- **Port**: 3306
- **Credentials**: root/mysql
- **Databases**: Initialized via init.sql script
- **Why MySQL?**: ACID compliance for transactions

### 3. **Apache Kafka**
- **Purpose**: Event streaming platform
- **Broker Port**: 9092 (external), 29092 (internal)
- **Components**:
  - **Zookeeper**: Cluster coordination (port 2181)
  - **Schema Registry**: Schema management (port 8085)
  - **Kafka UI**: Web console (port 8086)
- **Topics**: `order-placed`

### 4. **Keycloak**
- **Purpose**: Identity and Access Management (IAM)
- **Port**: 8181 (mapped from 8080)
- **Admin Console**: http://localhost:8181
- **Credentials**: admin/admin
- **Features**:
  - OAuth 2.0 / OpenID Connect
  - User management
  - Role-based access control (RBAC)
  - Single Sign-On (SSO)

### 5. **Observability Stack**

#### Prometheus
- **Purpose**: Metrics collection and storage
- **Port**: 9090
- **Scrapes**: Application metrics endpoints
- **Query Language**: PromQL

#### Grafana
- **Purpose**: Visualization and dashboards
- **Port**: 3000
- **URL**: http://localhost:3000
- **Features**:
  - Pre-configured dashboards
  - Anonymous access enabled
  - Multiple data sources (Prometheus, Loki, Tempo)

#### Loki
- **Purpose**: Log aggregation
- **Port**: 3100
- **Integration**: Collects logs from all services

#### Tempo
- **Purpose**: Distributed tracing
- **Ports**: 3110, 9411 (Zipkin compatible)
- **Features**: Trace requests across services

---

## Security

### Authentication & Authorization (Keycloak)

The system uses **OAuth 2.0** and **OpenID Connect** for security:

1. **User authentication** via Keycloak
2. **JWT tokens** issued by Keycloak
3. **API Gateway validates** tokens
4. **Services trust** the gateway (no token validation needed)

**Flow:**
```
User → Login → Keycloak → JWT Token → API Gateway (validates) → Services
```

**Configuration:**
- Realm configuration in `/docker/keycloak/realms/`
- Client IDs and secrets configured per service
- Role-based access control for fine-grained permissions

---

## Observability

### Three Pillars of Observability

#### 1. **Metrics (Prometheus + Grafana)**
- **What**: Quantitative measurements
- **Examples**:
  - Request rate (requests/second)
  - Error rate (4xx, 5xx responses)
  - Latency (response times)
  - JVM metrics (heap, threads)
  - Database connection pool stats

#### 2. **Logs (Loki)**
- **What**: Event records
- **Examples**:
  - Application logs
  - Error stack traces
  - Audit logs
  - Business events

#### 3. **Traces (Tempo)**
- **What**: Request flow across services
- **Examples**:
  - Order request spanning multiple services
  - Performance bottlenecks
  - Service dependencies
  - Error propagation

**Correlation:**
All three are correlated using **trace IDs**, allowing you to:
1. See a spike in error rate (metrics)
2. View related error logs (logs)
3. Trace the failing request (traces)

---

## Deployment Options

### 1. **Local Development (Docker Compose)**

**Start infrastructure:**
```bash
docker-compose up -d
```

**Start services:**
```bash
cd product-service && mvn spring-boot:run
cd order-service && mvn spring-boot:run
cd inventory-service && mvn spring-boot:run
cd notification-service && mvn spring-boot:run
cd api-gateway && mvn spring-boot:run
```

---

### 2. **Kubernetes (Production-like)**

**Prerequisites:**
- Kind cluster
- Docker images built

**Steps:**

1. **Create Kind cluster:**
   ```bash
   ./k8s/kind/create-kind-cluster.sh
   ```

2. **Build and push Docker images:**
   ```bash
   mvn spring-boot:build-image -DdockerPassword=<password>
   ```

3. **Deploy infrastructure:**
   ```bash
   kubectl apply -f k8s/manifests/infrastructure.yaml
   ```
   This deploys:
   - MySQL
   - MongoDB
   - Kafka
   - Keycloak
   - Grafana stack

4. **Deploy application services:**
   ```bash
   kubectl apply -f k8s/manifests/applications.yaml
   ```

5. **Access services via port-forwarding:**
   ```bash
   # API Gateway
   kubectl port-forward svc/gateway-service 9000:9000
   
   # Keycloak
   kubectl port-forward svc/keycloak 8080:8080
   
   # Grafana
   kubectl port-forward svc/grafana 3000:3000
   ```

---

## Design Patterns Used

### 1. **API Gateway Pattern**
- Single entry point for all client requests
- Routing, load balancing, circuit breaking
- Simplifies client-side code

### 2. **Database per Service**
- Each service owns its database
- Data isolation and independence
- Enables technology diversity (MongoDB + MySQL)

### 3. **Event-Driven Architecture**
- Asynchronous communication via Kafka
- Loose coupling between services
- Scalability and resilience

### 4. **Circuit Breaker Pattern**
- Prevents cascading failures
- Graceful degradation
- Automatic recovery

### 5. **Retry Pattern**
- Automatic retry on transient failures
- Configurable backoff strategies

### 6. **CQRS (Command Query Responsibility Segregation)**
- Separate read and write models
- Optimized for different access patterns

### 7. **Saga Pattern (Implicit)**
- Distributed transaction management
- Compensation on failures (not yet fully implemented)

---

## Testing Strategy

### 1. **Unit Tests**
- Test individual components in isolation
- Mock dependencies

### 2. **Integration Tests**
- Test service interactions
- Use **TestContainers** for real databases
- Spin up temporary Docker containers

### 3. **Contract Tests**
- Ensure API contracts are maintained
- Prevent breaking changes

### 4. **End-to-End Tests**
- Test complete user flows
- Frontend → Gateway → Services → Databases

---

## Scalability Considerations

### Horizontal Scaling
Each service can be scaled independently:
```bash
kubectl scale deployment order-service --replicas=3
```

### Load Balancing
- API Gateway distributes requests
- Kubernetes service load balancing

### Database Scaling
- **Read replicas** for heavy read workloads
- **Sharding** for horizontal partitioning

### Kafka Scaling
- **Partitions** for parallel processing
- **Consumer groups** for load distribution

---

## Monitoring & Alerts

### Key Metrics to Monitor

1. **Service Health**
   - Uptime/downtime
   - Health check status

2. **Performance**
   - Request latency (p50, p95, p99)
   - Throughput (requests/second)

3. **Errors**
   - Error rate (4xx, 5xx)
   - Circuit breaker state

4. **Infrastructure**
   - CPU, Memory, Disk usage
   - Database connections
   - Kafka lag

### Alerting Rules
- Circuit breaker open for > 1 minute
- Error rate > 5%
- Latency p95 > 1 second
- Service down

---

## Future Enhancements

1. **Service Mesh** (Istio/Linkerd)
   - Advanced traffic management
   - mTLS between services
   - Fine-grained observability

2. **Config Server**
   - Centralized configuration management
   - Dynamic configuration updates

3. **Service Discovery**
   - Netflix Eureka or Consul
   - Dynamic service registration

4. **API Versioning**
   - Support multiple API versions
   - Backward compatibility

5. **Rate Limiting**
   - Protect services from overload
   - Per-user rate limits

6. **Caching**
   - Redis for distributed caching
   - Reduce database load

7. **GraphQL Gateway**
   - Alternative to REST
   - Client-specific data fetching

---

## Conclusion

This Spring Boot 3 Microservices project demonstrates a **production-ready** e-commerce backend with:

✅ **Scalable architecture** - Independent services  
✅ **Resilient communication** - Circuit breakers, retries  
✅ **Event-driven design** - Asynchronous processing  
✅ **Security** - OAuth 2.0 with Keycloak  
✅ **Observability** - Metrics, logs, traces  
✅ **Cloud-native** - Docker and Kubernetes ready  

The architecture is designed to handle real-world challenges like:
- Service failures
- High traffic
- Data consistency
- Security threats
- Monitoring and debugging

This makes it an excellent foundation for building and scaling modern microservices applications.
