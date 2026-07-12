# Target Architecture

## Architecture Vision

The target state is a cloud-native, event-driven microservices platform designed to support high availability, independent deployments, horizontal scalability, and rapid feature delivery. Each business capability is implemented as an independently deployable service with clear ownership and API contracts. Once migration is complete, the platform behaves as a set of resilient services connected by a consistent event bus and exposed through a unified API gateway.

## Architecture Principles

- Domain-Driven Design (DDD)
- Microservices Architecture
- API First
- Event-Driven Communication
- Database per Service
- Cloud Native Stateless Services
- Horizontal Scalability
- Security by Design
- Observability by Default

## High-Level Architecture

Internet ---> CDN/WAF ---> Load Balancer ---> API Gateway ---> Microservices ---> Kafka Event Bus ---> Independent DBs ---> Elasticsearch/Redis

The platform is built so that traffic enters through a secure edge layer, is routed through the API gateway, and is served by independently deployed microservices. Events flow asynchronously through Kafka and data is stored in service-specific stores, with search and cache layers for performance.

## Domain Decomposition

| Service        | Responsibility         |
| -------------- | ---------------------- |
| Homepage       | Landing pages          |
| Search         | Product search         |
| Catalog        | Product information    |
| Pricing        | Pricing & discounts    |
| Inventory      | Stock availability     |
| Customer       | User profile           |
| Wishlist       | Saved products         |
| Cart           | Shopping cart          |
| Checkout       | Checkout orchestration |
| Payment        | Payment processing     |
| Order          | Order lifecycle        |
| Notification   | Email/SMS              |
| Recommendation | Personalized products  |

## Microservices

Each service is a separate deployable unit with its own codebase, database, and operational boundaries. Services expose APIs for synchronous operations and publish/subscribe to events for asynchronous workflows.

## Communication

Synchronous
- REST APIs for client-facing and service APIs
- gRPC (optional) for internal service-to-service calls with low latency

Asynchronous
- Kafka for reliable event streaming and decoupled communication
- Domain Events for business state changes
- Event Streaming to enable eventual consistency across bounded contexts

Event flow example:

Order Created
↓
Kafka
↓
Inventory Updated
↓
Payment Processed
↓
Notification Sent

## Data Architecture

Each service owns its own database to reduce coupling and support independent evolution. The platform uses polyglot persistence and eventual consistency where needed.

- Database per service
- Polyglot persistence
- Eventual consistency

| Service   | Database      |
| --------- | ------------- |
| Catalog   | PostgreSQL    |
| Pricing   | PostgreSQL    |
| Inventory | PostgreSQL    |
| Customer  | MongoDB       |
| Cart      | Redis         |
| Search    | Elasticsearch |
| Order     | PostgreSQL    |

## Caching

Caching improves performance and reduces load on core services.

- Redis for session and cart cache
- CDN for static content and landing pages
- Local cache for service-level response caching

Where caching is used:
- Product Details
- Search Results
- Prices
- Homepage
- Inventory

## Security

- OAuth2
- JWT
- API Gateway
- WAF
- Secrets Manager
- TLS

Security is embedded at every layer: authentication and authorization at the gateway, encrypted transport, secure secret storage, and perimeter protection.

## Observability

- Prometheus
- Grafana
- ELK
- OpenTelemetry
- Jaeger

Explain
- Metrics for system health and service performance
- Logs for diagnostics and audit trails
- Traces for request flows and latency analysis
- Alerts for operational issues and SLA breaches

## Deployment

Developer
↓
Git
↓
CI
↓
Docker
↓
Kubernetes
↓
Production

Deployment strategies:
- Blue Green
- Canary
- Rolling Update

## Scalability

Every service can scale independently based on demand and resource utilization.

Example:
- Homepage      x20
- Search        x40
- Catalog       x30
- Pricing       x10
- Inventory     x15
- Checkout      x5
- Payment       x3

## Benefits

| Before            | After                  |
| ----------------- | ---------------------- |
| Monolith          | Microservices          |
| Shared Database   | Database per Service   |
| Manual Scaling    | Auto Scaling           |
| Shared Deployment | Independent Deployment |
| Synchronous       | Event Driven           |
| Vendor Lock       | Cloud Native           |
| Tight Coupling    | Loose Coupling         |
| Single Failure    | Fault Isolation        |
