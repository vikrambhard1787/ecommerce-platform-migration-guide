# Search Service

## Overview

The Search Service provides fast and intelligent product discovery capabilities across the e-commerce platform. It enables customers to search products using keywords, categories, filters, facets, autocomplete, and sorting options.

Unlike the Product Service, the Search Service does not own product master data. Instead, it maintains a search-optimized index in Elasticsearch that is continuously synchronized through domain events published by the Product Service.

---

# Business Context

Product search is one of the highest traffic areas of an e-commerce platform. Customers expect relevant search results within milliseconds while supporting advanced search capabilities.

Business Objectives

- Fast Product Search
- Intelligent Product Discovery
- Search Suggestions
- Faceted Search
- Typo Tolerance
- High Availability
- Low Latency

---

# Responsibilities

The Search Service is responsible for:

- Product Search
- Full-Text Search
- Auto Complete
- Search Suggestions
- Category Search
- Brand Search
- Faceted Search
- Product Ranking
- Sorting
- Search Analytics

---

# Functional Requirements

- Search Products
- Search by SKU
- Search by Brand
- Search by Category
- Filter Results
- Sort Results
- Pagination
- Auto Suggest
- Search History
- Trending Searches

---

# Non Functional Requirements

| Requirement | Target |
|------------|---------|
| Availability | 99.99% |
| Response Time | <100 ms |
| Scalability | Horizontal |
| Throughput | Very High |
| Search Accuracy | High |

---

# High Level Architecture

                     Client
                        │
                  API Gateway
                        │
                  Search Service
                        │
                Elasticsearch
                        ▲
                        │
                     Kafka
                        ▲
                        │
                 Product Service

---

# Search APIs

## Search Products

GET /api/v1/search?q=laptop

---

## Auto Complete

GET /api/v1/search/suggestions?q=iph

---

## Search by Category

GET /api/v1/search?category=Mobiles

---

## Search by Brand

GET /api/v1/search?brand=Samsung

---

## Search with Filters

GET /api/v1/search?q=laptop&brand=Dell&price=50000-100000

---

# Elasticsearch Indexing

The Search Service maintains a dedicated Elasticsearch index optimized for fast product retrieval.

The Product Service remains the source of truth while Elasticsearch acts as the search read model.

## Indexed Fields

- Product ID
- SKU
- Product Name
- Description
- Brand
- Category
- Attributes
- Keywords
- Price
- Rating
- Availability
- Image URL

---

# Index Synchronization

Whenever a product changes, the Product Service publishes a domain event.

Example Flow

```
Product Updated

↓

Kafka

↓

Search Service

↓

Transform Document

↓

Update Elasticsearch Index
```

Published Events

- ProductCreated
- ProductUpdated
- ProductDeleted
- CategoryUpdated

The Search Service consumes these events and updates Elasticsearch asynchronously.

---

# Index Mapping

Example Elasticsearch document

```json
{
  "productId": "12345",
  "sku": "SKU123",
  "name": "Apple iPhone 17",
  "brand": "Apple",
  "category": "Mobiles",
  "price": 89999,
  "rating": 4.8,
  "available": true
}
```

---

# Search Features

Supported capabilities include:

- Full Text Search
- Fuzzy Search
- Prefix Search
- Auto Complete
- Synonyms
- Faceted Search
- Relevance Scoring
- Category Filters
- Brand Filters
- Price Filters

---

# Cache Strategy

Frequently executed search queries are cached.

Technology

- Redis

Cached Data

- Trending Searches
- Search Suggestions
- Popular Queries

TTL

5 Minutes

Search results themselves are primarily served from Elasticsearch.

---

# Idempotency

Search APIs are read-only and inherently idempotent.

Indexing operations use the Product ID as the Elasticsearch document ID, ensuring repeated indexing requests update the existing document rather than creating duplicates.

---

# Failure Handling

## Elasticsearch Unavailable

Return graceful error.

Retry indexing.

---

## Kafka Failure

Replay missed events.

Resume indexing.

---

## Product Update Failure

Retry indexing.

Dead Letter Queue (DLQ) for failed events.

---

# Event Publishing

Publishes

- SearchPerformed
- SearchSuggestionClicked
- ZeroResultSearch

---

# Event Consumption

Consumes

- ProductCreated
- ProductUpdated
- ProductDeleted
- CategoryUpdated

---

# Monitoring

Business Metrics

- Search Volume
- Top Searches
- Zero Result Searches
- Click Through Rate
- Search Conversion Rate

Technical Metrics

- Query Latency
- Elasticsearch Response Time
- Indexing Rate
- Kafka Consumer Lag
- Failed Index Updates
- Error Rate

---

# Scalability

The Search Service supports:

- Horizontal Scaling
- Elasticsearch Cluster
- Multiple Shards
- Replicas
- Read Scaling
- Kubernetes HPA

Search traffic is distributed across Elasticsearch nodes for high throughput.

---

# Design Decisions

| Decision | Reason |
|----------|--------|
| Elasticsearch | Optimized for full-text search |
| Kafka | Event-driven synchronization |
| Product Service as SSOT | Single source of truth |
| Redis | Cache suggestions and popular searches |
| Async Indexing | Loose coupling |

---

# Trade-offs

| Decision | Benefit | Drawback |
|----------|---------|----------|
| Elasticsearch | Extremely fast search | Eventual consistency |
| Event-Driven Indexing | Decoupled architecture | Slight indexing delay |
| Separate Search Service | Independent scaling | Additional operational complexity |

---

# Future Enhancements

- Semantic Search
- AI-powered Search
- Personalized Search Results
- Voice Search
- Image Search
- Vector Search
- Search Analytics Dashboard

---

# Interview Questions

### Why separate Search from Product Service?

Search workloads are read-intensive and require specialized indexing, ranking, and text search capabilities. Elasticsearch is optimized for these operations, whereas Product Service is responsible for maintaining product master data.

---

### Why use Elasticsearch instead of SQL?

Relational databases are not optimized for full-text search, fuzzy matching, faceted filtering, or relevance scoring. Elasticsearch provides these capabilities while delivering significantly lower query latency.

---

### How is Elasticsearch kept synchronized?

The Product Service publishes domain events whenever products are created, updated, or deleted. The Search Service consumes these events through Kafka and updates the Elasticsearch index asynchronously.

---

### What happens if Elasticsearch indexing fails?

The indexing request is retried. Failed events are moved to a Dead Letter Queue (DLQ) for investigation and replay. Product updates are not blocked because the Product Service remains the source of truth.

---

### Why is eventual consistency acceptable?

Search is a read-only capability. A delay of a few seconds before a product appears in search results is acceptable, whereas blocking product updates to wait for search indexing would reduce system availability and increase coupling.