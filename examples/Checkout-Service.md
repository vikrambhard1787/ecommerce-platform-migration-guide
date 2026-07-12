# Checkout Service

## Overview

The Checkout Service orchestrates the customer's checkout journey by coordinating multiple business services such as Cart, Pricing, Inventory, Payment, Order, and Notification. It validates the shopping cart, calculates the final payable amount, reserves inventory, initiates payment, creates the order, and returns the final confirmation to the customer.

Unlike other domain services, the Checkout Service does not own business data. Instead, it coordinates the execution of the checkout workflow while ensuring consistency across distributed services.

---

# Business Context

Checkout is the most business-critical component of an e-commerce platform. Any failure during checkout directly impacts revenue and customer experience.

The service must provide:

- High Availability
- Low Latency
- High Reliability
- Fault Tolerance
- Idempotent Operations
- Secure Payment Processing

---

# Responsibilities

The Checkout Service is responsible for:

- Validate Shopping Cart
- Validate Customer
- Validate Product Availability
- Fetch Latest Pricing
- Calculate Taxes
- Calculate Shipping Charges
- Apply Promotions
- Reserve Inventory
- Initiate Payment
- Create Order
- Publish Order Events
- Return Order Confirmation

---

# Functional Requirements

- Retrieve Cart
- Validate Customer
- Validate Inventory
- Validate Pricing
- Calculate Discounts
- Calculate Taxes
- Calculate Shipping
- Initiate Payment
- Create Order
- Handle Payment Failure
- Handle Inventory Failure
- Return Order Summary

---

# Non-Functional Requirements

| Requirement | Target |
|------------|---------|
| Availability | 99.99% |
| Latency | <500ms |
| Scalability | Horizontal |
| Fault Tolerance | Yes |
| Idempotency | Mandatory |
| Security | PCI Compliant |

---

# High Level Architecture

```text
                Customer
                    │
             API Gateway
                    │
            Checkout Service
                    │
   ┌────────┬────────┬────────┬────────┐
   │        │        │        │        │
 Cart   Pricing  Inventory Payment Order
   │        │        │        │        │
   └────────┴────────┴────────┴────────┘
                    │
               Kafka Events
                    │
            Notification Service
```

---

# Checkout Flow

```text
Customer Clicks Checkout
        │
Retrieve Cart
        │
Validate Customer
        │
Validate Inventory
        │
Fetch Latest Pricing
        │
Apply Discounts
        │
Calculate Taxes
        │
Calculate Shipping
        │
Reserve Inventory
        │
Initiate Payment
        │
Create Order
        │
Publish OrderCreated Event
        │
Return Success Response
```

---

# APIs

## Checkout

```
POST /api/v1/checkout
```

Request

```json
{
  "customerId": "12345",
  "cartId": "cart123",
  "paymentMethod": "CreditCard",
  "shippingAddressId": "address001"
}
```

---

## Checkout Response

```json
{
  "orderId":"ORD12345",
  "status":"SUCCESS",
  "paymentStatus":"AUTHORIZED",
  "estimatedDelivery":"2026-08-10"
}
```

---

# Service Dependencies

| Service | Purpose |
|----------|---------|
| Cart Service | Retrieve customer cart |
| Product Service | Product validation |
| Pricing Service | Latest pricing |
| Inventory Service | Reserve stock |
| Payment Service | Payment authorization |
| Order Service | Create order |
| Notification Service | Confirmation email/SMS |

---

# Database

The Checkout Service **does not own a database**.

It acts as an orchestration layer and delegates persistence to domain services.

---

# Sequence Diagram

```text
Customer
    │
Checkout Service
    │
Cart Service
    │
Pricing Service
    │
Inventory Service
    │
Payment Service
    │
Order Service
    │
Notification Service
```

---

# Saga Pattern

Checkout uses the **Saga Pattern** to coordinate distributed transactions.

Example:

```
Reserve Inventory

↓

Authorize Payment

↓

Create Order

↓

Send Notification
```

If any step fails, compensating actions are executed.

Example:

```
Payment Failed

↓

Release Inventory

↓

Cancel Checkout
```

---

# Failure Handling

## Inventory Not Available

Return Out of Stock.

---

## Pricing Changed

Refresh cart.

Ask customer for confirmation.

---

## Payment Failure

Release reserved inventory.

Return payment failure.

---

## Order Creation Failure

Refund payment.

Release inventory.

---

## Notification Failure

Retry asynchronously.

---

# Idempotency

Checkout requests use an **Idempotency Key**.

Benefits:

- Prevent duplicate orders
- Prevent duplicate payment
- Safe retries
- Better customer experience

---

# Security

- OAuth2 Authentication
- JWT Validation
- HTTPS
- PCI DSS Compliance
- Input Validation
- Rate Limiting
- Fraud Detection

---

# Events Published

- CheckoutCompleted
- CheckoutFailed
- OrderCreated

---

# Events Consumed

- InventoryReserved
- PaymentAuthorized
- PaymentFailed

---

# Scalability

Checkout Service is stateless.

Supports:

- Horizontal Scaling
- Kubernetes HPA
- Load Balancer
- Auto Scaling

---

# Monitoring

Business Metrics

- Checkout Success Rate
- Checkout Abandonment
- Conversion Rate
- Average Checkout Time

Technical Metrics

- API Latency
- Error Rate
- Payment Failure Rate
- Inventory Failure Rate
- Requests Per Second

---

# Design Decisions

| Decision | Reason |
|----------|--------|
| Stateless Service | Horizontal scaling |
| Saga Pattern | Distributed transactions |
| REST APIs | Synchronous customer flow |
| Kafka | Asynchronous downstream processing |
| Idempotency | Prevent duplicate orders |
| No Database | Orchestration only |

---

# Trade-offs

| Decision | Benefit | Drawback |
|----------|---------|----------|
| Saga Pattern | No distributed transactions | More implementation complexity |
| REST APIs | Immediate customer response | Service dependencies |
| Stateless Service | Easy scaling | Requires distributed state management |

---

# Future Enhancements

- One-Click Checkout
- Buy Now Flow
- AI Fraud Detection
- Multi-Shipment Checkout
- International Tax Engine
- Dynamic Shipping Selection

---

# Interview Questions

### Why doesn't Checkout own a database?

Checkout is an orchestration service. It coordinates business operations but does not own domain data. Cart, Inventory, Payment, and Order services each own their respective data.

---

### Why use the Saga Pattern instead of a distributed transaction?

Distributed transactions don't scale well in microservices. The Saga Pattern provides eventual consistency through compensating transactions while allowing each service to remain autonomous.

---

### How do you prevent duplicate orders?

Each checkout request includes an Idempotency Key. If the same request is received multiple times, the previously processed result is returned instead of creating a new order.

---

### What happens if payment succeeds but order creation fails?

The Saga executes compensating actions:
- Refund or void the payment authorization.
- Release reserved inventory.
- Mark the checkout as failed.
- Notify the customer if required.

---

### How do you scale Checkout during Black Friday?

- Deploy multiple stateless Checkout Service instances.
- Enable Kubernetes Horizontal Pod Autoscaler (HPA).
- Scale dependent services independently.
- Use Redis for caching where appropriate.
- Use Kafka for asynchronous downstream processing.
- Continuously monitor latency, error rates, and payment success rates.