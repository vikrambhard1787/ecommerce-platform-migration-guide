# Event-Driven Architecture

## Overview

As part of the platform modernization, the communication model evolved from tightly coupled synchronous interactions to an **Event-Driven Architecture (EDA)**. Instead of services invoking one another directly for every business operation, services publish domain events whenever a significant business action occurs. Interested services subscribe to these events and react independently.

This approach reduces coupling, improves scalability, and enables each microservice to evolve independently while maintaining data consistency across the platform.

---

## Objectives

The adoption of an Event-Driven Architecture was driven by the following objectives:

* Decouple microservices
* Enable asynchronous communication
* Improve system scalability
* Increase platform resilience
* Support eventual consistency
* Reduce cascading failures
* Enable independent deployments
* Improve fault tolerance
* Simplify integration with downstream systems

---

## Architecture

```text
                  Customer
                      │
                      ▼
              Checkout Service
                      │
             Order Created Event
                      │
                 Kafka Topic
     ┌────────────┼─────────────┬──────────────┐
     ▼            ▼             ▼              ▼
Inventory     Payment      Notification    Analytics
 Service       Service        Service        Service
```

The Checkout Service publishes an **OrderCreated** event after a successful checkout. Multiple downstream services consume the event independently without requiring direct service-to-service communication.

---

## Event Broker

Apache Kafka acts as the central event broker for asynchronous communication between microservices.

Kafka provides:

* High throughput
* Fault tolerance
* Message durability
* Horizontal scalability
* Ordered event processing within partitions
* Consumer replay capability

---

## Event Flow

A typical order lifecycle follows these steps:

1. Customer places an order.
2. Checkout Service validates the request.
3. Order Service creates the order.
4. Order Service publishes an **OrderCreated** event.
5. Inventory Service reserves stock.
6. Payment Service processes the payment.
7. Notification Service sends order confirmation.
8. Analytics Service updates reporting dashboards.

Each service performs its own business logic independently.

---

## Domain Events

The platform publishes domain-specific events representing significant business activities.

| Domain       | Sample Events                                                      |
| ------------ | ------------------------------------------------------------------ |
| Catalog      | ProductCreated, ProductUpdated                                     |
| Pricing      | PriceChanged, PromotionApplied                                     |
| Inventory    | InventoryReserved, InventoryReleased, InventoryUpdated             |
| Cart         | ItemAddedToCart, ItemRemovedFromCart                               |
| Checkout     | CheckoutCompleted                                                  |
| Order        | OrderCreated, OrderCancelled, OrderShipped                         |
| Payment      | PaymentAuthorized, PaymentCaptured, PaymentFailed, RefundProcessed |
| Customer     | CustomerRegistered, CustomerUpdated                                |
| Notification | EmailSent, SmsSent, PushNotificationSent                           |

---

## Communication Pattern

The platform uses a combination of communication styles.

### Synchronous Communication

Used when an immediate response is required.

Examples:

* Product Search
* Product Details
* Customer Authentication
* Price Lookup

Protocol:

* REST APIs

---

### Asynchronous Communication

Used when services can operate independently.

Examples:

* Order Processing
* Inventory Updates
* Notifications
* Analytics
* Audit Logging

Technology:

* Apache Kafka

---

## Event Reliability

To ensure reliable event processing, the platform implements:

* At-Least-Once Delivery
* Idempotent Consumers
* Retry Mechanisms
* Dead Letter Queue (DLQ)
* Message Persistence
* Consumer Offset Management

These mechanisms prevent data loss and improve recovery from transient failures.

---

## Event Versioning

Business events evolve over time. To maintain backward compatibility:

* Events are versioned.
* Existing event contracts remain compatible.
* Consumers can process multiple event versions during transition periods.

---

## Error Handling

If event processing fails:

1. Retry processing.
2. Apply exponential backoff.
3. Move failed events to the Dead Letter Queue.
4. Notify operations teams.
5. Replay events after issue resolution.

---

## Monitoring

The event platform is monitored using metrics such as:

* Topic throughput
* Consumer lag
* Processing latency
* Failed event count
* Retry count
* Dead Letter Queue size
* Message processing time

These metrics provide visibility into the health of the event-driven ecosystem.

---

## Benefits

The Event-Driven Architecture provides several advantages:

* Loose coupling between services
* Independent deployments
* Better scalability
* Improved resilience
* Fault isolation
* Real-time data propagation
* Easier integration with external systems
* Support for eventual consistency
* Improved extensibility

---

## Key Takeaways

* Domain events are the primary mechanism for asynchronous communication between microservices.
* Apache Kafka provides a scalable and reliable event backbone for the platform.
* Event-driven communication reduces service dependencies and improves system resilience.
* Reliable event processing is achieved through retries, idempotency, dead letter queues, and comprehensive monitoring.
* The architecture enables incremental modernization while supporting future business capabilities such as analytics, personalization, and AI-driven recommendations.
