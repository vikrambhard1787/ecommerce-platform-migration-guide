# Domain Decomposition

## Overview

One of the key objectives of the platform modernization was to decompose the monolithic application into independently deployable microservices aligned with business capabilities. Instead of splitting the application based on technical layers (UI, Services, Database), the platform was decomposed using Domain-Driven Design (DDD) principles and bounded contexts.

Each domain owns its business logic, APIs, and data, enabling independent development, deployment, and scaling.

---

## Objectives

The primary goals of domain decomposition were:

- Identify clear business capabilities
- Reduce coupling between services
- Enable independent deployments
- Improve scalability
- Align service ownership with engineering teams
- Allow each domain to evolve independently

---

## Decomposition Approach

The decomposition strategy followed these principles:

- Business Capability Driven Design
- Domain-Driven Design (DDD)
- Bounded Contexts
- Database per Service
- API First
- Event-Driven Communication
- Loose Coupling
- High Cohesion

Instead of extracting services based on existing code modules, each microservice was designed around a single business capability.

---

## Domain Identification

The following business domains were identified during the decomposition process.

| Domain | Responsibility |
|----------|----------------|
| Homepage | Landing page, banners, CMS content |
| Search | Product search, autocomplete, filters |
| Catalog | Product details, categories, attributes |
| Pricing | Product pricing, discounts, promotions |
| Inventory | Stock availability and reservation |
| Customer | Customer profile and account management |
| Wishlist | Customer wishlists |
| Cart | Shopping cart management |
| Checkout | Checkout orchestration |
| Payment | Payment processing |
| Order | Order lifecycle management |
| Notification | Email, SMS and Push Notifications |

---

## Domain Ownership

Each service owns its business logic and persistence layer.

| Service | Own Database | Communication |
|----------|--------------|---------------|
| Homepage | No | REST |
| Search | Elasticsearch | REST |
| Catalog | PostgreSQL | REST |
| Pricing | PostgreSQL | REST + Kafka |
| Inventory | PostgreSQL | REST + Kafka |
| Customer | PostgreSQL | REST |
| Wishlist | PostgreSQL | REST |
| Cart | Redis | REST |
| Checkout | No | REST |
| Payment | PostgreSQL | REST |
| Order | PostgreSQL | Kafka |
| Notification | No | Kafka |

---

## Service Dependencies

The following diagram illustrates the interaction between business domains.

```
Homepage
    │
Search
    │
Catalog
    │
Pricing
    │
Inventory
    │
Cart
    │
Checkout
    │
Payment
    │
Order
    │
Notification
```

---

## Migration Priority

Not all domains were extracted simultaneously. The migration followed a phased approach based on business risk, dependency analysis, and implementation complexity.

| Phase | Domains |
|---------|---------|
| Phase 1 | Lift & Shift to Cloud VMs |
| Phase 2 | Homepage, Search, Catalog |
| Phase 3 | Pricing, Inventory |
| Phase 4 | Customer, Wishlist, Cart, Checkout, Payment, Order |

The migration started with low-risk, stateless services before progressively extracting transactional business capabilities.

---

## Benefits

Domain decomposition delivered several architectural and business benefits:

- Independent deployment of services
- Independent horizontal scaling
- Reduced release dependencies
- Better fault isolation
- Clear ownership for engineering teams
- Improved maintainability
- Faster feature delivery
- Easier cloud adoption
- Foundation for event-driven architecture

---

## Key Takeaways

- Services were identified based on business capabilities rather than technical layers.
- Every domain has a clearly defined responsibility and ownership.
- Each service owns its own data and exposes well-defined APIs.
- The decomposition strategy enabled incremental migration using the Strangler Pattern while minimizing business disruption.