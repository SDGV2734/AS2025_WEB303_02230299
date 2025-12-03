# WEB303 Microservices & Serverless Applications

## Practical 5 Report: Refactoring Monolith to Microservices

**Student:** Sonam Dorji
**Module:** WEB303 Microservices & Serverless Applications  
**Date:** October 9, 2025  
**GitHub Repository:** https://github.com/SDGV2734/WEB303_practical_5.git

---

## Executive Summary

This report documents the comprehensive refactoring of a monolithic Student Cafe application into a microservices architecture. The project demonstrates systematic service extraction utilizing domain-driven design (DDD) principles, resulting in three independent microservices (User, Menu, and Order) coordinated through a dedicated API Gateway component. All services are containerized utilizing Docker and orchestrated via Docker Compose, exemplifying contemporary microservices architectural patterns including the database-per-service strategy, inter-service communication protocols, and centralized API gateway routing.

---

## Learning Outcomes Addressed

- **LO1:** Analyzed and compared fundamental characteristics, benefits, and trade-offs inherent to monolithic versus microservices architectural paradigms
- **LO2:** Applied domain-driven design principles to identify and delineate explicit service boundaries
- **LO3:** Systematically extracted services from the monolithic application while preserving functional integrity
- **LO4:** Implemented service discovery patterns with inter-service communication mechanisms
- **LO5:** Deployed and orchestrated multiple containerized services utilizing Docker Compose
- **LO6:** Examined migration pathways and architectural evolution toward advanced distributed systems patterns

---

## Project Deliverables

### 1. **Monolithic Application (Phase 1)**

- **Location:** `student-cafe-monolith/`
- **Network Port:** 8080
- **Database:** PostgreSQL (unified single-instance database)
- **Functionality:** Integrated application encompassing user account management, menu catalog administration, and order processing workflows
- **Implementation Status:** Fully functional baseline implementation serving as the reference monolithic architecture

### 2. **Microservices Implementation (Phases 2-4)**

#### User Service

- **Network Port:** 8081 | **Database Instance:** user_db (port 5434)
- **API Endpoints:**
  - `POST /users` - Create new user entity
  - `GET /users/{id}` - Retrieve user by unique identifier
  - `GET /users` - Enumerate all user records
- **Implementation Status:** Complete with GORM object-relational mapping models and HTTP request handlers

#### Menu Service

- **Network Port:** 8082 | **Database Instance:** menu_db (port 5433)
- **API Endpoints:**
  - `POST /menu` - Create menu item entity
  - `GET /menu/{id}` - Retrieve menu item by unique identifier
- **Data Models:** Menu and MenuItem entities with defined relational associations
- **Implementation Status:** Complete with autonomous service architecture and encapsulated data model

#### Order Service

- **Network Port:** 8083 | **Database Instance:** order_db (port 5435)
- **API Endpoints:**
  - `POST /orders` - Create order entity (with inter-service validation)
  - `GET /orders` - Enumerate all order records
- **Inter-Service Communication Protocol:**
  - Validates user existence via user-service endpoint invocation
  - Retrieves menu item specifications via menu-service endpoint invocation
  - Captures price snapshots at order creation time for historical accuracy
- **Implementation Status:** Complete with HTTP-based synchronous service-to-service communication

### 3. **API Gateway (Phase 5)**

- **Network Port:** 8080
- **Request Routing Configuration:**
  - `/api/users*` → user-service:8081
  - `/api/menu*` → menu-service:8082
  - `/api/orders*` → order-service:8083
- **Architectural Features:** Request path rewriting, reverse proxy pattern implementation, unified client entry point
- **Implementation Status:** Complete with middleware stack and comprehensive request/response logging

### 4. **Docker & Orchestration**

- **Docker Compose Configuration:** Orchestrates complete service and database deployment
- **Container Images:** Multi-stage builds utilizing Alpine Linux base image for all services to minimize image footprint
- **Database Configuration:** Isolated PostgreSQL instances provisioned for each service
- **Volume Management:** Named volumes provisioned for persistent data storage across database lifecycle
- **Implementation Status:** All services successfully compilable and executable

---

## Architecture Overview

```
┌─────────────────────┐
│   External Clients  │
└──────────┬──────────┘
           │
           ▼
   ┌───────────────────┐
   │   API Gateway     │
   │   (port 8080)     │
   └───┬───────┬───┬───┘
       │       │   │
    ┌──┴──┐ ┌─┴─┐ ┌┴──┐
    │ :81 │ │:82│ │:83│
    ▼     ▼     ▼
┌────────┬──────────┬───────┐
│ User   │  Menu    │ Order │
│ Service│ Service  │Service│
└──┬─────┴──┬───────┴───┬───┘
   │        │           │
   ▼        ▼           ▼
┌────┐  ┌────┐      ┌────┐
│user│  │menu│      │order│
│ db  │  │ db │      │ db │
└────┘  └────┘      └────┘
```

---

## Implementation Highlights

### Database-per-Service Pattern

- **User Service:** Maintains exclusive ownership of user_db with User data model
- **Menu Service:** Maintains exclusive ownership of menu_db with Menu and MenuItem data models
- **Order Service:** Maintains exclusive ownership of order_db with Order and OrderItem data models
- **Architectural Benefits:** Enables independent service scaling capabilities, facilitates heterogeneous storage strategy implementation, and reduces inter-service coupling

### Inter-Service Communication

The Order service implementation exemplifies synchronous HTTP-based inter-service communication:

```go
// Validate user entity existence via user-service endpoint
userResp, err := http.Get(fmt.Sprintf("%s/users/%d", userServiceURL, req.UserID))

// Retrieve menu item specifications with price snapshot capture
menuResp, err := http.Get(fmt.Sprintf("%s/menu/%d", menuServiceURL, item.MenuItemID))
```

### Service Discovery

- Service references are established via container hostname resolution (Docker DNS)
- Elimination of hardcoded IP address configurations
- Seamless integration with Docker Compose networking infrastructure

---

## Testing and Validation

### Build Verification

All service components successfully compile without errors:

```bash
go build ./...  # Verified for all service implementations
```

### Container Image Build Verification [Complete]

All Dockerfile specifications successfully build without compilation errors utilizing multi-stage build processes

### Functional Testing

Representative HTTP requests demonstrating API Gateway invocation:

```bash
# Create user entity
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe", "email": "john@example.com"}'

# Create menu item entity
curl -X POST http://localhost:8080/api/menu \
  -H "Content-Type: application/json" \
  -d '{"name": "Coffee", "description": "Hot coffee", "price": 2.50}'

# Create order entity (demonstrates inter-service communication)
curl -X POST http://localhost:8080/api/orders \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1, "items": [{"menu_item_id": 1, "quantity": 2}]}'

# Retrieve all order records
curl http://localhost:8080/api/orders
```

---

## Deployment

### Quick Start

```bash
cd /home/easykp8/Desktop/Year\ 3/practical5
docker-compose up --build
```

### Service Access Points

- **API Gateway (Primary Endpoint):** http://localhost:8080
- **User Service (Direct Access):** http://localhost:8081
- **Menu Service (Direct Access):** http://localhost:8082
- **Order Service (Direct Access):** http://localhost:8083

### Database Access Credentials

- PostgreSQL user-db: localhost:5434
- PostgreSQL menu-db: localhost:5433
- PostgreSQL order-db: localhost:5435
- Default credentials (all instances): `postgres` / `postgres`

---

## File Structure

```
practical5/
├── student-cafe-monolith/     [Complete] Monolithic baseline
│   ├── main.go
│   ├── models/ (user, menu, order)
│   ├── handlers/ (user, menu, order)
│   ├── database/
│   ├── Dockerfile
│   └── go.mod
├── user-service/               [Complete] Extracted user service
│   ├── main.go
│   ├── models/
│   ├── handlers/
│   ├── database/
│   ├── Dockerfile
│   └── go.mod
├── menu-service/               [Complete] Extracted menu service
│   ├── main.go
│   ├── models/
│   ├── handlers/
│   ├── database/
│   ├── Dockerfile
│   └── go.mod
├── order-service/              [Complete] Extracted order service
│   ├── main.go
│   ├── models/
│   ├── handlers/
│   ├── database/
│   ├── Dockerfile
│   └── go.mod
├── api-gateway/                [Complete] Request router
│   ├── main.go
│   ├── Dockerfile
│   └── go.mod
├── docker-compose.yml          [Complete] Complete orchestration
├── README.md                   [Complete] Comprehensive documentation
└── PRACTICAL5_REPORT.md        [Complete] This report
```

---

## Key Achievements

| Phase | Deliverable                                    | Implementation Status |
| ----- | ---------------------------------------------- | --------------------- |
| 1     | Monolithic baseline architecture               | Complete              |
| 2     | Menu service extraction and isolation          | Complete              |
| 3     | User service extraction and isolation          | Complete              |
| 4     | Order service with inter-service communication | Complete              |
| 5     | API Gateway with request routing               | Complete              |
| 6     | Docker Compose orchestration configuration     | Complete              |

---

## Production Considerations

### Current Implementation Characteristics

- Database-per-service pattern for data isolation and service autonomy
- Independent service deployment and scaling capabilities
- Synchronous HTTP-based inter-service communication
- Docker container technology for deployment consistency
- Docker Compose orchestration for local and development environments

### Recommended Future Enhancements

- Consul or Eureka integration for dynamic service discovery
- Health check endpoints and distributed monitoring infrastructure
- Circuit breaker pattern implementation for fault tolerance
- Kubernetes orchestration platform migration
- API Gateway advanced features (response caching, rate limiting, authentication)
- Distributed tracing and observability using Jaeger or similar platforms

---

## Conclusion

This practical successfully demonstrates the comprehensive refactoring journey from monolithic architecture to a distributed microservices architecture. The implementation exemplifies:

- **Service Decomposition:** Explicit service boundaries established through domain-driven design principles
- **Independent Deployment:** Each microservice maintains autonomous codebase and dedicated data store
- **Inter-Service Communication:** Functional HTTP-based synchronous communication with validation protocols
- **Container Orchestration:** Docker Compose infrastructure orchestrating the complete distributed system
- **Scalability Architecture:** Each service enables independent horizontal scaling capabilities

All designated learning outcomes have been fulfilled through practical implementation of established microservices architectural patterns. The project establishes a robust foundation for advanced distributed systems topics including service mesh architecture, Kubernetes orchestration, and event-driven communication paradigms.

---

**Submission Checklist:**

- ✓ Source code implementation for all microservices
- ✓ Dockerfile specifications for each service
- ✓ Docker Compose orchestration configuration
- ✓ Database schema initialization (via GORM automatic migration)
- ✓ Comprehensive README documentation with deployment and testing procedures
- ✓ Technical report (this document)
- ✓ Evidence of successful compilation and orchestration validation

---
