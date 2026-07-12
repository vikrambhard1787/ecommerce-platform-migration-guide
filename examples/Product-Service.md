# Product Service

## Overview

The Product Service is responsible for managing product master data and providing a centralized source of product information across the e-commerce platform. It maintains product details, categories, attributes, images, specifications, and product lifecycle information while exposing APIs for product discovery and retrieval.

The service acts as the **Single Source of Truth (SSOT)** for product information and integrates with Pricing, Inventory, Search, and Content Management services.

---

# Business Context

Product information is one of the most frequently accessed datasets in an e-commerce platform. Customers expect accurate, consistent, and up-to-date product information across all channels.

Business Objectives

- Centralized product catalog
- Consistent product information
- Fast product retrieval
- Support millions of products
- Easy product onboarding
- Support multiple categories
- Enable omnichannel commerce

---

# Responsibilities

The Product Service is responsible for:

- Product Management
- Product Details
- Product Categories
- Product Attributes
- Product Images
- Product Variants (SKU)
- Product Lifecycle
- Product Metadata
- Product Search Metadata
- Product Publishing

---

# Functional Requirements

- Create Product
- Update Product
- Delete Product
- Get Product Details
- Get Product by SKU
- List Products
- Manage Categories
- Manage Product Variants
- Upload Images
- Manage Product Attributes

---

# Non Functional Requirements

| Requirement | Target |
|------------|---------|
| Availability | 99.99% |
| Response Time | <100ms |
| Scalability | Horizontal |
| Durability | High |
| Consistency | Strong |
| Throughput | High |

---

# High Level Architecture

                    Client
                       │
                 API Gateway
                       │
                Product Service
                       │
      ┌──────────────┬──────────────┐
      │              │              │
 PostgreSQL      Redis Cache     Kafka
      │
 Elasticsearch

---

# Product Domain Model

## Product

| Field | Type |
|--------|------|
| productId | UUID |
| sku | String |
| name | String |
| description | Text |
| brand | String |
| categoryId | UUID |
| status | Enum |
| createdDate | Timestamp |

---

## Category

| Field | Type |
|--------|------|
| categoryId | UUID |
| categoryName | String |
| parentCategoryId | UUID |

---

## Product Variant

| Field | Type |
|--------|------|
| variantId | UUID |
| sku | String |
| color | String |
| size | String |
| barcode | String |

---

## Product Image

| Field | Type |
|--------|------|
| imageId | UUID |
| productId | UUID |
| imageUrl | String |
| displayOrder | Integer |

---

# Product Status

| Status | Description |
|----------|-------------|
| DRAFT | Product under creation |
| ACTIVE | Available for sale |
| INACTIVE | Not visible |
| DISCONTINUED | No longer sold |
| ARCHIVED | Historical record |

---

# APIs

## Create Product

```http
POST /api/v1/products
```

---

## Update Product

```http
PUT /api/v1/products/{productId}
```

---

## Get Product

```http
GET /api/v1/products/{productId}
```

---

## Get Product by SKU

```http
GET /api/v1/products/sku/{sku}
```

---

## List Products

```http
GET /api/v1/products
```

Supports:

- Pagination
- Sorting
- Filtering

---

## Delete Product

```http
DELETE /api/v1/products/{productId}
```

---

# Product Retrieval Flow

Customer Requests Product

↓

Redis Cache

↓

Cache Hit

↓

Return Product

↓

Cache Miss

↓

PostgreSQL

↓

Update Cache

↓

Return Response

---

# Caching

Redis is used to cache frequently accessed product information.

Cached Data

- Product Details
- Categories
- Product Variants
- Product Images

Cache Strategy

Cache Aside Pattern

TTL

30 Minutes

Cache Invalidation

- Product Updated
- Product Deleted
- Category Updated
- Image Updated

---

# Search Integration

The Product Service is **not responsible for search**.

Instead, product updates are published as events.

The Search Service consumes these events and indexes products into Elasticsearch.

Example Flow

Product Updated

↓

Kafka

↓

Search Service

↓

Elasticsearch Index Updated

---

# Idempotency

Administrative APIs support idempotent operations.

Example

```
POST /products

Idempotency-Key:
PRODUCT-10001
```

Duplicate requests return the existing resource instead of creating multiple products.

Benefits

- Safe retries
- Prevent duplicate products
- Reliable integrations

---

# Event Publishing

The Product Service publishes:

- ProductCreated
- ProductUpdated
- ProductDeleted
- CategoryCreated
- CategoryUpdated
- ProductActivated
- ProductDeactivated

---

# Event Consumption

Consumes:

- BrandUpdated
- CategoryHierarchyUpdated
- MediaUploaded

---

# Failure Handling

## Database Failure

Retry transaction.

Return appropriate error.

---

## Redis Failure

Read directly from PostgreSQL.

Repopulate cache.

---

## Kafka Failure

Retry publishing.

Persist event for later replay.

---

## Search Service Down

Product update succeeds.

Search index synchronization occurs asynchronously.

---

# Security

- OAuth2
- JWT Authentication
- HTTPS
- RBAC
- Audit Logging

Only authorized catalog administrators can create or modify products.

---

# Monitoring

Business Metrics

- Products Created
- Products Updated
- Active Products
- Product Views

Technical Metrics

- API Latency
- Cache Hit Ratio
- Error Rate
- Database Response Time
- Kafka Publish Rate

---

# Scalability

The Product Service is stateless.

Supports:

- Horizontal Scaling
- Kubernetes HPA
- Read Replicas
- Redis Cluster
- Database Partitioning

---

# Design Decisions

| Decision | Reason |
|----------|--------|
| PostgreSQL | Strong consistency for product master data |
| Redis | Fast product retrieval |
| Kafka | Event-driven synchronization |
| Elasticsearch | Dedicated product search |
| Cache Aside | Reduced database load |
| Database per Service | Independent ownership |

---

# Trade-offs

| Decision | Benefit | Drawback |
|----------|---------|----------|
| PostgreSQL | Strong consistency | More difficult horizontal scaling |
| Redis | Fast reads | Cache invalidation complexity |
| Elasticsearch | Powerful search | Eventual consistency |
| Event-Driven Sync | Loose coupling | Delayed indexing |

---

# Future Enhancements

- AI Product Recommendations
- Product Similarity
- Product Versioning
- Multi-language Catalog
- Multi-region Catalog
- Product Comparison
- Product Videos
- Digital Assets Management

---

# Interview Questions

### Why is Product Service separated from Search Service?

The Product Service owns product master data, while the Search Service is optimized for full-text search and filtering using Elasticsearch. Separating them allows each service to scale independently and use technologies best suited to their responsibilities.

---

### Why doesn't Product Service own pricing?

Pricing changes frequently due to promotions, customer segments, and campaigns. Keeping pricing in a dedicated Pricing Service allows pricing rules to evolve independently without affecting product management.

---

### Why isn't inventory stored in Product Service?

Inventory is transactional data that changes frequently. Separating it into the Inventory Service prevents excessive updates to product records and enables independent scaling of stock management.

---

### Why cache product details?

Product pages receive a high volume of read requests. Caching significantly reduces database load and improves page response times while maintaining acceptable data freshness.

---

### How do you ensure Search remains up to date?

Whenever a product is created, updated, or deleted, the Product Service publishes domain events. The Search Service consumes these events and updates the Elasticsearch index asynchronously, ensuring eventual consistency between the product catalog and search index.