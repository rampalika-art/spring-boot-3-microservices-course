# Quick Start Guide - Spring Boot 3 Microservices

## 🚀 What is This Project?

This is an **e-commerce microservices application** built with Spring Boot 3 that demonstrates modern cloud-native architecture patterns.

**In Simple Terms:** It's a backend system for an online shop where customers can:
- Browse products 📦
- Place orders 🛒
- Check inventory 📊
- Receive email notifications 📧

---

## 🏗️ Architecture at a Glance

```
Frontend (Angular) 
       ↓
API Gateway (Port 9000) - Your single entry point
       ↓
┌──────┴───────┬────────────┬───────────┐
│              │            │           │
Product      Order      Inventory   Notification
Service      Service     Service      Service
   ↓            ↓           ↓            ↓
MongoDB      MySQL       MySQL        Kafka
```

---

## 🎯 Core Concepts

### 1️⃣ Microservices = Small, Independent Apps
Each service does ONE thing well:
- **Product Service**: Manages products (like a product catalog)
- **Order Service**: Handles orders (like a shopping cart checkout)
- **Inventory Service**: Tracks stock (like warehouse management)
- **Notification Service**: Sends emails (like order confirmations)

### 2️⃣ API Gateway = Front Door
- All requests go through one door (port 9000)
- Routes traffic to the right service
- Protects services if one goes down (Circuit Breaker)

### 3️⃣ Two Ways Services Talk

**Synchronous (Request-Response):**
```
Order Service: "Hey Inventory, do we have 2 iPhones?"
Inventory Service: "Yes!" or "No!"
```

**Asynchronous (Fire-and-Forget):**
```
Order Service: "New order placed!" → Kafka
Notification Service: "Got it! Sending email..."
```

---

## 📊 Complete Order Flow (Step-by-Step)

### Scenario: Customer orders 2 iPhones

```
1. Customer clicks "Place Order" in Angular app
   ↓
2. Request goes to API Gateway (http://localhost:9000/api/order)
   ↓
3. Gateway routes to Order Service
   ↓
4. Order Service asks Inventory: "Do we have 2 iPhones?"
   ↓
5. Inventory Service checks MySQL database
   ↓
6. Inventory responds: "Yes, in stock!"
   ↓
7. Order Service saves order to MySQL
   ↓
8. Order Service publishes event to Kafka: "Order placed!"
   ↓
9. Notification Service picks up the event
   ↓
10. Notification Service sends confirmation email
    ↓
11. Customer sees: "Order Placed Successfully"
```

**If inventory is out of stock:**
- Step 6: Inventory says "No!"
- Order Service returns error
- No order saved, no email sent

---

## 🔧 Technology Stack Explained

### Backend
- **Java 21** - Programming language (modern, fast)
- **Spring Boot 3** - Framework that makes Java easy
- **Maven** - Builds and manages the project

### Databases
- **MongoDB** - For products (flexible, JSON-like storage)
- **MySQL** - For orders & inventory (structured, transactional)

### Messaging
- **Kafka** - Event bus for async communication
  - Think of it as a high-speed message queue

### Security
- **Keycloak** - Manages logins and permissions
  - Like "Sign in with Google" but for your app

### Monitoring
- **Grafana** - Dashboards showing app health
- **Prometheus** - Collects metrics (requests, errors, speed)
- **Loki** - Stores logs
- **Tempo** - Traces requests across services

---

## 🚦 Running the Application

### Option 1: Local Development (Quick)

**Step 1: Start Infrastructure**
```bash
docker-compose up -d
```
This starts: MongoDB, MySQL, Kafka, Keycloak, Grafana

**Step 2: Start Services (in separate terminals)**
```bash
cd product-service && mvn spring-boot:run
cd order-service && mvn spring-boot:run
cd inventory-service && mvn spring-boot:run
cd notification-service && mvn spring-boot:run
cd api-gateway && mvn spring-boot:run
```

**Step 3: Start Frontend**
```bash
cd frontend
npm install
npm run start
```

**Step 4: Access the app**
- Frontend: http://localhost:4200
- API Gateway: http://localhost:9000
- Keycloak: http://localhost:8181
- Grafana: http://localhost:3000
- Kafka UI: http://localhost:8086

---

### Option 2: Kubernetes (Production-like)

**Step 1: Create cluster**
```bash
./k8s/kind/create-kind-cluster.sh
```

**Step 2: Build Docker images**
```bash
mvn spring-boot:build-image
```

**Step 3: Deploy**
```bash
kubectl apply -f k8s/manifests/infrastructure.yaml
kubectl apply -f k8s/manifests/applications.yaml
```

**Step 4: Port-forward to access**
```bash
kubectl port-forward svc/gateway-service 9000:9000
```

---

## 🧪 Testing It Out

### 1. Create a Product
```bash
curl -X POST http://localhost:9000/api/product \
  -H "Content-Type: application/json" \
  -d '{
    "name": "iPhone 15",
    "description": "Latest Apple smartphone",
    "price": 999.99
  }'
```

**Response:** Product created with ID

---

### 2. Get All Products
```bash
curl http://localhost:9000/api/product
```

**Response:** List of all products

---

### 3. Check Inventory
```bash
curl "http://localhost:9000/api/inventory?skuCode=iphone_15&quantity=2"
```

**Response:** `true` or `false`

---

### 4. Place an Order
```bash
curl -X POST http://localhost:9000/api/order \
  -H "Content-Type: application/json" \
  -d '{
    "skuCode": "iphone_15",
    "price": 999.99,
    "quantity": 2,
    "userDetails": {
      "email": "customer@example.com",
      "firstName": "John",
      "lastName": "Doe"
    }
  }'
```

**Response:** "Order Placed Successfully"

**What happens behind the scenes:**
1. ✅ Inventory checked
2. ✅ Order saved to database
3. ✅ Event sent to Kafka
4. ✅ Email sent to customer

---

## 🎓 Key Patterns Explained

### Circuit Breaker
**Problem:** If Inventory Service is down, should Order Service keep trying forever?

**Solution:** Circuit Breaker
- Tries 3 times
- If all fail, "opens" the circuit
- Returns fallback: "Service unavailable"
- Periodically checks if service is back

**Like:** An electrical circuit breaker in your home

---

### Event-Driven Architecture
**Problem:** Order Service shouldn't wait for email to be sent

**Solution:** Kafka events
- Order Service: "I'm done, here's the order info" → Kafka
- Notification Service: Picks up event when ready
- Order Service doesn't wait, doesn't care if email fails

**Like:** Leaving a voicemail vs waiting on hold

---

### API Gateway
**Problem:** Frontend needs to know URLs of all services

**Solution:** API Gateway
- One URL for everything (port 9000)
- Gateway routes internally
- Frontend code stays simple

**Like:** A hotel reception desk routing you to the right room

---

## 📈 Monitoring Your Services

### View Metrics (Grafana)
1. Open http://localhost:3000
2. Navigate to dashboards
3. See:
   - Request rate per service
   - Error rates
   - Response times
   - Database performance

### View Logs (Loki)
1. In Grafana, go to "Explore"
2. Select Loki datasource
3. Query logs: `{service="order-service"}`

### Trace Requests (Tempo)
1. In Grafana, go to "Explore"
2. Select Tempo datasource
3. See request flowing through services
4. Find bottlenecks and errors

---

## 🛠️ Common Issues & Solutions

### Port already in use
```bash
# Find what's using the port
lsof -i :9000

# Kill the process
kill -9 <PID>
```

### Services can't connect to databases
```bash
# Check if Docker containers are running
docker ps

# Restart Docker Compose
docker-compose restart
```

### Kafka not working
```bash
# Check Kafka UI
http://localhost:8086

# Check topics exist
docker exec -it broker kafka-topics --list --bootstrap-server localhost:9092
```

---

## 🎯 Next Steps

### For Learning
1. ✅ Understand the architecture (you've done this!)
2. Try the example requests above
3. Check Grafana dashboards
4. Add a new product and place an order
5. Watch the Kafka topic in Kafka UI

### For Development
1. Add a new endpoint to Product Service
2. Create a new microservice (e.g., Payment Service)
3. Add more product fields (color, size, category)
4. Implement order cancellation
5. Add unit and integration tests

### Advanced Topics
1. Implement Saga pattern for distributed transactions
2. Add Redis caching
3. Implement rate limiting
4. Add GraphQL API
5. Set up CI/CD pipeline

---

## 📚 Additional Resources

- **Full Documentation**: See `PROJECT_ARCHITECTURE_AND_FLOW.md`
- **Video Tutorial**: [YouTube Link](https://youtu.be/yn_stY3HCr8)
- **Spring Boot Docs**: https://spring.io/projects/spring-boot
- **Kubernetes Docs**: https://kubernetes.io/docs/
- **Kafka Docs**: https://kafka.apache.org/documentation/

---

## 🤝 Contributing

This is a learning project. Feel free to:
- Report issues
- Suggest improvements
- Add new features
- Improve documentation

---

## ❓ FAQs

**Q: Why so many services for a simple shop?**  
A: To demonstrate microservices patterns. In production, you'd have 10-100+ services.

**Q: Why not use a monolith?**  
A: Microservices allow independent scaling, deployment, and technology choices.

**Q: Is this production-ready?**  
A: It's a great starting point. Add more error handling, tests, and security for production.

**Q: Why both MySQL and MongoDB?**  
A: To show polyglot persistence. Each service uses the best database for its needs.

**Q: What's the performance impact of microservices?**  
A: Network calls add latency, but benefits (scalability, resilience) often outweigh costs.

---

## 🎉 Conclusion

You now understand:
- ✅ What each service does
- ✅ How services communicate
- ✅ The complete order flow
- ✅ Key design patterns
- ✅ How to run and test the app

**Happy learning! 🚀**
