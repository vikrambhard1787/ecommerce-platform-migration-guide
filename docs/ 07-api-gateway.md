# API Gateway

## Overview

As part of the platform modernization, an API Gateway was introduced to provide a single entry point for all client applications. It abstracts the underlying microservices from consumers, centralizes cross-cutting concerns, and enables the gradual migration from the legacy monolithic platform to independently deployable microservices.

The API Gateway plays a critical role in implementing the **Strangler Pattern**, allowing requests to be routed dynamically to either the legacy application or the newly developed microservices during different phases of the migration.

---

## Objectives

The API Gateway was introduced to achieve the following goals:

- Provide a single entry point for all client applications
- Hide internal service topology
- Enable incremental migration using the Strangler Pattern
- Centralize authentication and authorization
- Perform request routing and load balancing
- Apply rate limiting and throttling
- Improve API security
- Simplify API versioning
- Enable request and response transformation
- Improve observability through centralized logging and metrics

---

## High-Level Architecture

```text
                    Internet
                        │
                     CDN / WAF
                        │
                  Load Balancer
                        │
                  API Gateway
        ┌───────────────┼───────────────┐
        │               │               │
   Legacy Platform   Search Service   Catalog Service
                        │
                  Pricing Service
                        │
                 Inventory Service
                        │
                    Cart Service
                        │
                 Checkout Service
                        │
                  Payment Service
                        │
                   Order Service
```

---

## Responsibilities

The API Gateway is responsible for:

- Request routing
- Authentication
- Authorization
- SSL termination
- Rate limiting
- API versioning
- Request validation
- Request aggregation
- Response transformation
- Centralized logging
- Metrics collection
- Distributed tracing

---

## Request Routing

During the migration, incoming requests are routed based on the availability of migrated services.

| Request | Target |
|----------|--------|
| /search | Search Service |
| /catalog | Catalog Service |
| /pricing | Pricing Service |
| /inventory | Inventory Service |
| /cart | Cart Service |
| /checkout | Checkout Service |
| /payment | Payment Service |
| /orders | Order Service |
| Remaining APIs | Legacy Monolith |

This routing strategy allows both the legacy platform and modern microservices to coexist during the migration.

---

## Authentication & Authorization

The API Gateway acts as the security boundary for external traffic.

Responsibilities include:

- OAuth 2.0 Authentication
- JWT Token Validation
- Role-Based Access Control (RBAC)
- API Key Validation (for partner APIs)
- HTTPS Enforcement

Authentication is performed once at the gateway before forwarding requests to downstream services.

---

## Traffic Management

To ensure platform stability, the gateway implements:

- Rate Limiting
- Request Throttling
- Connection Timeouts
- Circuit Breakers
- Retry Policies
- Request Size Limits

These mechanisms protect backend services from excessive traffic and improve overall system resilience.

---

## Observability

The API Gateway provides centralized monitoring by capturing:

- Request Count
- Response Time
- Error Rate
- Latency
- API Usage
- HTTP Status Codes
- Trace IDs
- Access Logs

These metrics are integrated with monitoring platforms such as Prometheus and Grafana.

---

## Migration Support

The API Gateway enables incremental migration by routing traffic to either the legacy platform or new microservices without impacting client applications.

Migration example:

| Phase | Routed To |
|--------|-----------|
| Phase 1 | Legacy Platform |
| Phase 2 | Homepage, Search, Catalog |
| Phase 3 | Pricing, Inventory |
| Phase 4 | Cart, Checkout, Payment, Order, Customer |

As additional services are migrated, routing rules are updated without requiring changes to frontend applications.

---

## Benefits

- Single API endpoint for clients
- Decouples clients from backend services
- Supports gradual migration
- Simplifies service discovery
- Centralized security
- Improved scalability
- Better monitoring and logging
- Independent backend evolution
- Reduced client-side complexity

---

## Key Takeaways

- The API Gateway serves as the unified entry point for all client requests.
- It enables the implementation of the Strangler Pattern by routing traffic between the legacy platform and newly developed microservices.
- Cross-cutting concerns such as security, routing, observability, and traffic management are centralized, allowing backend services to remain focused on business functionality.