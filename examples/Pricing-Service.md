# Pricing Service

## Overview

The Pricing Service is responsible for calculating the final selling price of products by applying business rules such as base pricing, promotions, discounts, coupons, customer-specific pricing, and taxes. It acts as the single source of truth for pricing information across the e-commerce platform.

The service provides real-time pricing for product listing pages, product detail pages, shopping carts, and checkout while ensuring pricing consistency across all customer touchpoints.

---

# Business Context

Pricing directly influences customer purchasing decisions and revenue. The service must support complex pricing rules while maintaining low latency and high availability.

Business Objectives

- Real-time price calculation
- Consistent pricing across all channels
- Support promotions and coupons
- Dynamic pricing capability
- High scalability
- Low response time

---

# Responsibilities

The Pricing Service is responsible for:

- Base Price Management
- Promotional Pricing
- Customer-specific Pricing
- Coupon Validation
- Discount Calculation
- Tax Calculation
- Price Rule Evaluation
- Price History
- Publish Pricing Events

---

# Functional Requirements

- Get Product Price
- Calculate Final Price
- Apply Promotions
- Apply Coupons
- Calculate Taxes
- Customer-specific Pricing
- Bulk Price Retrieval
- Scheduled Promotions
- Price Updates

---

# Non Functional Requirements

| Requirement | Target |
|------------|---------|
| Availability | 99.99% |
| Latency | <50ms |
| Scalability | Horizontal |
| Throughput | High |
| Cache Hit Ratio | >90% |

---

# High Level Architecture

                    Client
                       │
                 API Gateway
                       │
                Pricing Service
                       │
        ┌──────────────┼───────────────┐
        │              │               │
   PostgreSQL       Redis Cache      Kafka
        │
 Promotion Engine

---

# Pricing Components

Final Price is calculated using:

```
Final Price

=

Base Price

-

Product Discount

-

Coupon Discount

+

Taxes

+

Shipping Charges
```

---

# APIs

## Get Product Price

```http
GET /api/v1/pricing/{productId}
```

---

## Bulk Pricing

```http
POST /api/v1/pricing/bulk
```

---

## Apply Coupon

```http
POST /api/v1/pricing/coupon
```

---

## Calculate Cart Pricing

```http
POST /api/v1/pricing/cart
```

---

# Database Design

## Product Price

| Field | Type |
|--------|------|
| productId | UUID |
| basePrice | Decimal |
| currency | String |
| effectiveDate | Timestamp |

---

## Promotion

| Field | Type |
|--------|------|
| promotionId | UUID |
| promotionType | Enum |
| discount | Decimal |
| startDate | Timestamp |
| endDate | Timestamp |

---

## Coupon

| Field | Type |
|--------|------|
| couponCode | String |
| discountType | Enum |
| discountValue | Decimal |
| expiryDate | Timestamp |

---

# Pricing Flow

Customer Views Product

↓

Pricing Service

↓

Redis Cache

↓

Cache Miss

↓

Database

↓

Promotion Engine

↓

Tax Engine

↓

Final Price

↓

Update Cache

↓

Return Response

---

# Caching

Pricing is one of the most frequently accessed datasets.

Redis is used to cache:

- Product Prices
- Promotion Rules
- Coupon Rules
- Tax Configuration

Cache Strategy

Cache Aside Pattern

TTL

5 Minutes

Cache Invalidation

- Price Update
- Promotion Start
- Promotion End
- Coupon Update

---

# Idempotency

Price retrieval APIs are naturally idempotent.

Administrative operations such as price updates use an Idempotency Key to prevent duplicate processing.

Example

```
PUT /pricing/product

Idempotency-Key:
PRICE-12345
```

Benefits

- Safe retries
- Prevent duplicate price updates
- Consistent pricing

---

# Dynamic Pricing

The service supports:

- Flash Sale Pricing
- Customer Segment Pricing
- Loyalty Discounts
- Seasonal Promotions
- Bulk Purchase Discounts
- Marketplace Pricing

---

# Event Publishing

Publishes:

- PriceUpdated
- PromotionCreated
- PromotionExpired
- CouponCreated
- CouponExpired

---

# Event Consumption

Consumes:

- ProductCreated
- ProductUpdated
- InventoryLow
- CustomerSegmentUpdated

---

# Failure Handling

## Promotion Engine Down

Return base price.

Log warning.

---

## Redis Down

Read from PostgreSQL.

Rebuild cache.

---

## Coupon Validation Failed

Ignore coupon.

Return validation message.

---

## Tax Service Down

Retry.

Fallback to cached tax rules if applicable.

---

# Security

- OAuth2
- JWT
- HTTPS
- RBAC
- Audit Logging

Only authorized administrators can modify pricing.

---

# Monitoring

Business Metrics

- Promotion Usage
- Coupon Usage
- Average Discount
- Revenue Impact
- Price Update Frequency

Technical Metrics

- API Latency
- Cache Hit Ratio
- Error Rate
- Requests Per Second
- Database Response Time

---

# Scalability

Pricing Service is stateless.

Supports:

- Horizontal Scaling
- Redis Cluster
- Read Replicas
- Kubernetes HPA

Frequently accessed prices are served directly from Redis.

---

# Design Decisions

| Decision | Reason |
|----------|--------|
| PostgreSQL | Strong consistency |
| Redis | Low-latency price retrieval |
| Promotion Engine | Centralized business rules |
| Cache Aside | Reduce database load |
| Kafka | Event-driven synchronization |

---

# Trade-offs

| Decision | Benefit | Drawback |
|----------|---------|----------|
| Redis Cache | Fast responses | Cache invalidation complexity |
| Rule Engine | Flexible pricing | Increased implementation complexity |
| Dynamic Pricing | Business flexibility | Higher computational cost |

---

# Future Enhancements

- AI-driven Dynamic Pricing
- Competitor Price Matching
- Personalized Promotions
- Real-time Surge Pricing
- Multi-currency Pricing
- Regional Pricing Rules

---

# Interview Questions

### Why separate Pricing from Product Service?

Product Service manages product information, while Pricing Service owns all pricing logic, promotions, coupons, and tax calculations. This separation allows pricing rules to evolve independently without affecting the product catalog.

---

### Why cache pricing?

Pricing APIs are among the highest-traffic APIs in an e-commerce platform. Caching significantly reduces database load and improves response times while maintaining acceptable freshness through cache invalidation events.

---

### How do you handle a price change while a customer has an item in the cart?

The latest price is fetched during checkout. If the price has changed, the customer is informed and the cart total is recalculated before payment authorization.

---

### Why isn't the final price stored in the database?

The final selling price depends on several dynamic factors such as promotions, coupons, customer segment, taxes, and shipping. It is calculated at runtime to ensure accuracy.

---

### How would you scale Pricing Service during Black Friday?

- Scale stateless service instances horizontally.
- Use a Redis Cluster for price caching.
- Cache promotion and tax rules.
- Preload flash sale pricing into cache.
- Monitor cache hit ratio, API latency, and promotion engine performance.