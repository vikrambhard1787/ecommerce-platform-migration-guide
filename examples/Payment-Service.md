# Payment Service

## Overview

The Payment Service is responsible for securely processing customer payments and managing the complete payment lifecycle. It integrates with external payment gateways to authorize, capture, refund, and reconcile payments while ensuring transactional integrity and preventing duplicate transactions.

The service acts as the single source of truth for payment transactions and is designed to support multiple payment methods, high availability, and secure financial processing.

---

# Business Context

Payment processing is one of the most critical components of an e-commerce platform. Customers expect secure, reliable, and seamless payment experiences.

Business Objectives

- Secure payment processing
- High payment success rate
- Prevent duplicate charges
- Support multiple payment methods
- Fast payment authorization
- Reliable refund processing
- Complete payment audit trail

---

# Responsibilities

The Payment Service is responsible for:

- Payment Authorization
- Payment Capture
- Payment Status
- Refund Processing
- Payment Cancellation
- Payment Reconciliation
- Payment Audit Logs
- Payment Gateway Integration
- Publish Payment Events

---

# Functional Requirements

- Authorize Payment
- Capture Payment
- Cancel Payment
- Refund Payment
- Retrieve Payment Status
- Retry Failed Payments
- Support Multiple Payment Methods
- Generate Payment Reference
- Payment Reconciliation

---

# Non Functional Requirements

| Requirement | Target |
|------------|---------|
| Availability | 99.99% |
| Latency | <300ms |
| Security | PCI DSS |
| Scalability | Horizontal |
| Idempotency | Mandatory |
| Durability | High |

---

# Supported Payment Methods

- Credit Card
- Debit Card
- UPI
- Net Banking
- Wallets
- Gift Cards
- Buy Now Pay Later (BNPL)

---

# High Level Architecture

                    Checkout Service
                            │
                     Payment Service
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
     Payment Gateway     Fraud Engine    Notification
          │
     PostgreSQL
          │
        Kafka

---

# Payment Flow

Customer Clicks Pay

↓

Validate Payment Request

↓

Generate Idempotency Key

↓

Authorize Payment

↓

Payment Gateway

↓

Payment Success

↓

Publish PaymentAuthorized Event

↓

Order Service

↓

Capture Payment

↓

Payment Completed

---

# APIs

## Authorize Payment

POST /api/v1/payments/authorize

---

## Capture Payment

POST /api/v1/payments/capture

---

## Refund Payment

POST /api/v1/payments/refund

---

## Cancel Payment

POST /api/v1/payments/cancel

---

## Payment Status

GET /api/v1/payments/{paymentId}

---

# Database Design

## Payment

| Field | Type |
|--------|------|
| paymentId | UUID |
| orderId | UUID |
| customerId | UUID |
| amount | Decimal |
| currency | String |
| status | Enum |
| paymentMethod | String |
| gatewayTransactionId | String |
| createdDate | Timestamp |

---

# Payment Status

| Status | Description |
|----------|-------------|
| INITIATED | Payment request created |
| AUTHORIZED | Payment authorized |
| CAPTURED | Payment captured |
| FAILED | Authorization failed |
| CANCELLED | Payment cancelled |
| REFUNDED | Refund completed |
| PARTIALLY_REFUNDED | Partial refund processed |

---

# Idempotency

Every payment request includes an **Idempotency Key** to prevent duplicate payment processing.

Example

```
Idempotency-Key:
PAY-ORD-123456
```

If the same request is received multiple times due to retries or network failures, the Payment Service returns the original response instead of initiating another transaction.

Benefits

- Prevent duplicate charges
- Safe retries
- Improved customer experience
- Financial consistency

---

# Payment Gateway Integration

Supported capabilities:

- Authorization
- Capture
- Refund
- Void
- Payment Status
- Webhook Notifications

The Payment Service acts as an abstraction layer, allowing payment providers to be changed without impacting upstream services.

---

# Caching

Redis is used to cache frequently accessed payment information.

Cached Data

- Payment Status
- Supported Payment Methods
- Payment Gateway Configuration

TTL

5 minutes

Sensitive payment information is **never cached**.

---

# Event Publishing

The Payment Service publishes:

- PaymentInitiated
- PaymentAuthorized
- PaymentCaptured
- PaymentFailed
- PaymentCancelled
- RefundInitiated
- RefundCompleted

---

# Event Consumption

Consumes:

- CheckoutCompleted
- OrderCreated
- OrderCancelled
- ReturnCompleted

---

# Failure Handling

## Payment Gateway Timeout

Retry with exponential backoff.

---

## Duplicate Payment Request

Return existing payment using Idempotency Key.

---

## Gateway Unavailable

Return retryable error.

Trigger alert.

---

## Refund Failure

Retry asynchronously.

Notify Operations Team.

---

# Security

- PCI DSS Compliance
- HTTPS
- TLS Encryption
- OAuth2 Authentication
- JWT Validation
- Tokenization
- Secrets Management
- Audit Logging

Card details are never stored within the Payment Service.

---

# Fraud Detection

Payments are validated using:

- Velocity checks
- Duplicate payment detection
- Suspicious IP detection
- Device fingerprinting
- Transaction amount validation

High-risk transactions may require additional verification.

---

# Reconciliation

Daily reconciliation ensures payment records match gateway transactions.

Reconciliation verifies:

- Authorized Payments
- Captured Payments
- Failed Transactions
- Refunds
- Settlement Reports

Discrepancies are logged and investigated automatically.

---

# Monitoring

Business Metrics

- Payment Success Rate
- Authorization Rate
- Refund Rate
- Payment Volume
- Revenue Processed

Technical Metrics

- API Latency
- Gateway Response Time
- Error Rate
- Retry Count
- Timeout Rate

---

# Scalability

The Payment Service is stateless and supports:

- Horizontal Scaling
- Kubernetes HPA
- Multi-Gateway Support
- Retry Queue
- Event-Driven Processing

---

# Design Decisions

| Decision | Reason |
|----------|--------|
| PostgreSQL | ACID compliance for financial transactions |
| Redis | Fast payment status lookup |
| Idempotency | Prevent duplicate charges |
| Kafka | Asynchronous event publishing |
| PCI DSS | Secure payment processing |
| Tokenization | Protect sensitive payment information |

---

# Trade-offs

| Decision | Benefit | Drawback |
|----------|---------|----------|
| REST APIs | Immediate response | Tight coupling during authorization |
| Event-Driven Notifications | Loose coupling | Eventual consistency |
| Tokenization | Improved security | Additional gateway dependency |
| Redis Cache | Faster status lookup | Cache invalidation complexity |

---

# Future Enhancements

- Multi-payment gateway routing
- Smart gateway selection
- AI-powered fraud detection
- Installment payments
- Cryptocurrency support
- Dynamic payment retries
- Cross-border payment optimization

---

# Interview Questions

### Why is idempotency mandatory for payments?

Because payment requests may be retried due to network failures or client timeouts. Idempotency guarantees that the customer is charged only once, even if the same request is received multiple times.

---

### Why doesn't Checkout process payments directly?

Checkout orchestrates the business workflow, while the Payment Service owns all payment-related functionality, ensuring clear separation of concerns and easier maintenance.

---

### What happens if payment is authorized but order creation fails?

The Saga Pattern executes compensating actions. The payment authorization is voided (or refunded if already captured), inventory is released, and the checkout process is marked as failed.

---

### Why don't we store card details?

To reduce security risks and maintain PCI DSS compliance. Sensitive card information is tokenized and securely managed by certified payment gateways.

---

### How do you support multiple payment gateways?

The Payment Service abstracts gateway-specific APIs through a common interface. This allows switching or adding providers without impacting Checkout or Order services.