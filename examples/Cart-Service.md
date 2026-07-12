# Cart Service

## Overview

The Cart Service is responsible for managing a customer's shopping cart throughout their shopping journey. It provides APIs to add, update, remove, and retrieve cart items while maintaining real-time pricing, inventory validation, and promotional calculations.

The service acts as a temporary storage for items before checkout and is optimized for high read/write throughput and low latency.

---

# Business Context

The shopping cart is one of the most frequently accessed components in an e-commerce platform. Customers expect instant responses when adding or updating items.

Key business objectives include:

- Fast Add to Cart experience
- Real-time cart updates
- Support guest and registered users
- Low latency (<100ms)
- High availability
- Seamless checkout integration

---

# Responsibilities

The Cart Service is responsible for:

- Create Cart
- Retrieve Cart
- Add Item
- Remove Item
- Update Quantity
- Merge Guest Cart
- Apply Promotions
- Calculate Cart Total
- Validate Inventory
- Save for Later (Optional)

---

# Functional Requirements

- Create a shopping cart
- Add product to cart
- Remove product from cart
- Update quantity
- Retrieve cart details
- Merge guest cart after login
- Apply coupons
- Calculate subtotal
- Calculate taxes
- Calculate shipping estimates
- Calculate discounts

---

# Non-Functional Requirements

| Requirement | Target |
|------------|---------|
| Availability | 99.99% |
| Response Time | <100ms |
| Scalability | Horizontal |
| Durability | High |
| Throughput | High |
| Fault Tolerance | Yes |

---

# High Level Architecture

```text
                    Client
                       │
                API Gateway
                       │
                 Cart Service
                       │
       ┌───────────────┼──────────────┐
       │               │              │
    Redis         Product API    Pricing API
       │               │              │
       └───────────────┼──────────────┘
                       │
                Inventory API
                       │
                  Kafka Events
```

---

# Data Model

## Cart

| Field | Type |
|---------|------|
| cartId | UUID |
| customerId | UUID |
| createdDate | Timestamp |
| updatedDate | Timestamp |

---

## Cart Item

| Field | Type |
|---------|------|
| productId | UUID |
| sku | String |
| quantity | Integer |
| unitPrice | Decimal |
| discount | Decimal |
| totalPrice | Decimal |

---

# APIs

## Create Cart

```
POST /api/v1/cart
```

---

## Get Cart

```
GET /api/v1/cart/{customerId}
```

---

## Add Item

```
POST /api/v1/cart/items
```

Request

```json
{
    "productId":"123",
    "quantity":2
}
```

---

## Update Quantity

```
PUT /api/v1/cart/items/{productId}
```

---

## Remove Item

```
DELETE /api/v1/cart/items/{productId}
```

---

## Apply Coupon

```
POST /api/v1/cart/coupon
```

---

# Database Design

Redis is used as the primary datastore because:

- Extremely low latency
- Temporary data
- High read/write throughput
- Automatic expiration
- Horizontal scalability

Cart expiration:

- Guest Cart : 24 Hours
- Logged-in User : 30 Days

---

# Sequence Flow

Customer clicks Add to Cart

↓

Cart Service

↓

Validate Product

↓

Validate Inventory

↓

Get Current Price

↓

Store Cart in Redis

↓

Publish CartUpdated Event

↓

Return Updated Cart

---

# Event Publishing

The Cart Service publishes the following events:

- CartCreated
- CartUpdated
- CartMerged
- CouponApplied
- CartCheckedOut
- CartExpired

---

# Event Consumption

The Cart Service consumes:

- ProductUpdated
- PriceChanged
- InventoryUpdated
- CustomerLoggedIn

---

# Cache Strategy

Redis acts as both:

- Primary datastore
- Distributed cache

TTL:

- Guest Cart : 24 Hours
- Logged-in User : 30 Days

Frequently accessed carts remain entirely in memory.

---

# Scalability

The Cart Service is completely stateless.

Horizontal scaling is achieved by:

- Kubernetes HPA
- Load Balancer
- Redis Cluster

Only Redis stores session state.

---

# Failure Handling

## Inventory Unavailable

Customer receives validation error.

---

## Redis Failure

Fallback to Redis Replica.

---

## Pricing Service Down

Use last known price.

Flag cart for revalidation during checkout.

---

## Product Removed

Automatically remove unavailable item.

---

# Security

- OAuth2 Authentication
- JWT Validation
- HTTPS
- Input Validation
- Rate Limiting
- Customer owns only their cart

---

# Monitoring

Business Metrics

- Active Carts
- Abandoned Carts
- Cart Conversion Rate
- Average Cart Value

Technical Metrics

- API Latency
- Redis Latency
- Cache Hit Ratio
- Error Rate
- Requests Per Second

---

# Design Decisions

| Decision | Reason |
|------------|---------|
| Redis | Extremely fast read/write operations |
| Stateless Service | Horizontal scalability |
| REST APIs | Simple client integration |
| Event Publishing | Loose coupling |
| JWT | Secure authentication |
| Redis TTL | Automatic cleanup |

---

# Trade-offs

| Choice | Pros | Cons |
|---------|------|------|
| Redis | Very fast | Data loss if persistence is disabled |
| SQL Database | Durable | Higher latency |
| Cache Everything | Faster | Stale data risk |

---

# Future Enhancements

- Multi-device cart synchronization
- AI-powered product recommendations
- Save for Later
- Shared Shopping Cart
- Real-time inventory updates
- Cart analytics
- Cart recovery notifications

---

# Interview Questions

### Why Redis instead of PostgreSQL?

Because cart data is temporary, frequently updated, and requires sub-millisecond access times.

---

### Why doesn't Cart own product information?

Product details belong to the Product Service. Cart stores only product references to avoid data duplication.

---

### What happens if product pricing changes after the item is added to the cart?

The cart displays the latest available pricing. Pricing is revalidated during checkout to ensure the customer is charged the current price.

---

### How do you prevent overselling?

Inventory is validated during checkout, and stock reservation is handled by the Inventory Service.

---

### How would you scale the Cart Service during Black Friday?

Scale the stateless Cart Service horizontally, use a Redis Cluster, enable autoscaling, and monitor cache performance and latency.