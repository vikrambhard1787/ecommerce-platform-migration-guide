# Order Service

## Overview

The Order Service is responsible for managing the complete lifecycle of customer orders. It acts as the system of record for orders by maintaining order details, tracking order status, coordinating with downstream systems, and publishing business events.

Once an order is successfully created, it becomes the authoritative source for order history, fulfillment, shipment, returns, refunds, and customer notifications.

---

# Business Context

The Order Service is the core transactional service within the e-commerce platform. It manages customer purchases from order creation through fulfillment, delivery, returns, and cancellation.

Business Objectives:

- Reliable order creation
- Accurate order tracking
- High availability
- Transaction consistency
- Event-driven processing
- Customer order history
- Integration with OMS and ERP

---

# Responsibilities

The Order Service is responsible for:

- Create Order
- Retrieve Order
- Update Order Status
- Cancel Order
- Return Order
- Refund Order
- Track Shipment
- Maintain Order History
- Publish Domain Events
- Integrate with OMS

---

# Functional Requirements

- Create new order
- Retrieve order details
- List customer orders
- Update order status
- Cancel order
- Return order
- Process refunds
- Track shipment
- Generate invoices
- Notify downstream services

---

# Non Functional Requirements

| Requirement | Target |
|------------|---------|
| Availability | 99.99% |
| Latency | <300ms |
| Scalability | Horizontal |
| Durability | High |
| Fault Tolerance | Yes |
| Idempotency | Mandatory |

---

# High Level Architecture

```text
                 Checkout Service
                         │
                  Create Order
                         │
                  Order Service
                         │
     ┌──────────┬───────────┬─────────────┐
     │          │           │             │
 Inventory   Payment   Notification    OMS
     │          │           │             │
     └──────────┴───────────┴─────────────┘
                     │
                  PostgreSQL
                     │
                    Kafka
```

---

# Order Lifecycle

```text
Created

↓

Confirmed

↓

Packed

↓

Shipped

↓

Out For Delivery

↓

Delivered
```

Alternative paths

```text
Created

↓

Cancelled
```

or

```text
Delivered

↓

Return Requested

↓

Returned

↓

Refund Completed
```

---

# Order Status

| Status | Description |
|----------|-------------|
| CREATED | Order successfully created |
| CONFIRMED | Payment confirmed |
| PACKED | Warehouse packed order |
| SHIPPED | Carrier picked order |
| OUT_FOR_DELIVERY | Courier delivering |
| DELIVERED | Delivered successfully |
| CANCELLED | Order cancelled |
| RETURN_REQUESTED | Customer initiated return |
| RETURNED | Product returned |
| REFUND_INITIATED | Refund processing |
| REFUND_COMPLETED | Refund completed |

---

# APIs

## Create Order

```http
POST /api/v1/orders
```

---

## Get Order

```http
GET /api/v1/orders/{orderId}
```

---

## Get Customer Orders

```http
GET /api/v1/customers/{customerId}/orders
```

---

## Cancel Order

```http
PUT /api/v1/orders/{orderId}/cancel
```

---

## Return Order

```http
POST /api/v1/orders/{orderId}/return
```

---

# Database Design

## Orders

| Field | Type |
|---------|------|
| orderId | UUID |
| customerId | UUID |
| totalAmount | Decimal |
| status | Enum |
| paymentStatus | Enum |
| createdDate | Timestamp |

---

## Order Items

| Field | Type |
|---------|------|
| orderItemId | UUID |
| orderId | UUID |
| productId | UUID |
| quantity | Integer |
| price | Decimal |

---

# Idempotency

Order creation is idempotent.

Each checkout request contains an **Idempotency-Key**.

```
Idempotency-Key:
CHK-983729
```

If Checkout retries the request, the existing order is returned rather than creating another one.

Benefits

- No duplicate orders
- Safe retries
- Better customer experience
- Simplified recovery

---

# Caching

Frequently accessed order summaries are cached.

Technology

- Redis

Cached Data

- Recent Orders
- Order Summary
- Order Status

TTL

5–10 minutes

Customer order history is cached to reduce database load.

---

# Event Publishing

The Order Service publishes:

- OrderCreated
- OrderConfirmed
- OrderCancelled
- OrderShipped
- OrderDelivered
- OrderReturned
- RefundInitiated

---

# Event Consumption

Consumes:

- PaymentAuthorized
- PaymentFailed
- InventoryReserved
- ShipmentCreated
- DeliveryCompleted

---

# Order State Machine

Every order follows predefined state transitions.

```
Created

↓

Confirmed

↓

Packed

↓

Shipped

↓

Delivered
```

Illegal transitions are rejected.

Example

```
Delivered

↓

Created

❌ Not Allowed
```

---

# Failure Handling

### Payment Failure

Cancel Order

Release Inventory

Notify Customer

---

### Inventory Failure

Cancel Order

Notify Checkout

---

### Shipment Failure

Retry

Notify Warehouse

---

### Notification Failure

Retry asynchronously

No customer impact

---

# Security

- OAuth2
- JWT
- RBAC
- HTTPS
- Order ownership validation
- Audit logging

Only the customer who owns the order or authorized support personnel can access order details.

---

# Monitoring

Business Metrics

- Orders Per Minute
- Order Success Rate
- Cancellation Rate
- Return Rate
- Refund Rate

Technical Metrics

- API Latency
- Database Latency
- Kafka Lag
- Error Rate
- Requests Per Second

---

# Scalability

The Order Service is stateless.

Supports:

- Horizontal Scaling
- Kubernetes HPA
- Read Replicas
- Redis Cache
- Database Partitioning

Historical orders can be archived to improve query performance.

---

# Design Decisions

| Decision | Reason |
|----------|--------|
| PostgreSQL | ACID compliance |
| Redis | Faster order retrieval |
| Kafka | Event-driven integration |
| Idempotency | Prevent duplicate orders |
| State Machine | Valid status transitions |
| Read Replicas | Scale read-heavy workloads |

---

# Trade-offs

| Decision | Benefit | Drawback |
|----------|---------|----------|
| SQL Database | Strong consistency | Horizontal scaling is harder |
| Event-Driven | Loose coupling | Eventual consistency |
| Redis Cache | Faster reads | Cache invalidation complexity |
| State Machine | Predictable workflow | More implementation effort |

---

# Future Enhancements

- Split Order (Multi-Warehouse Fulfillment)
- Partial Shipment
- Partial Cancellation
- Partial Returns
- Exchange Orders
- Subscription Orders
- Scheduled Deliveries
- AI-based Delivery ETA

---

# Interview Questions

### Why is Order Service the system of record?

Because it owns the complete order lifecycle and maintains the authoritative record of every customer order, independent of Checkout, Payment, or Inventory.

---

### Why does Checkout create the order instead of Payment?

Checkout orchestrates the customer journey, but Order Service owns order persistence. Payment only authorizes financial transactions and should not own order data.

---

### Why is a State Machine used?

A state machine enforces valid order transitions, prevents invalid state changes, simplifies business rules, and improves auditability.

---

### How do you prevent duplicate orders?

An Idempotency-Key is included with every order creation request. Duplicate requests return the existing order instead of creating a new one.

---

### How do you scale Order Service during Black Friday?

- Deploy multiple stateless service instances
- Use Kubernetes Horizontal Pod Autoscaler
- Cache frequently accessed order summaries
- Use read replicas for customer order history
- Partition historical data
- Process downstream activities asynchronously using Kafka