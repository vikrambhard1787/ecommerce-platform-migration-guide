# Inventory Allocation

Inventory is allocated based on configurable business rules:

- Nearest warehouse
- Highest available stock
- Store pickup preference
- Fulfillment center priority
- Delivery SLA
- Shipping cost optimization

This enables intelligent order fulfillment while reducing shipping costs and improving delivery times.

# Inventory Retrieval

The Inventory Service provides real-time inventory availability for products across warehouses, fulfillment centers, and stores.

Inventory information is consumed by:

- Product Detail Page
- Search Service
- Cart Service
- Checkout Service
- Order Service
- OMS
- Store Locator

## APIs

### Get Inventory by SKU

```http
GET /api/v1/inventory/{sku}
```

Response

```json
{
    "sku":"SKU123",
    "availableQuantity":120,
    "reservedQuantity":15,
    "warehouse":"WH-01",
    "status":"IN_STOCK"
}
```

---

### Get Inventory for Multiple SKUs

```http
POST /api/v1/inventory/bulk
```

Request

```json
{
    "skus":[
        "SKU101",
        "SKU102",
        "SKU103"
    ]
}
```

---

## Inventory Status

Every SKU maintains one of the following inventory states.

| Status | Description |
|----------|-------------|
| IN_STOCK | Inventory available for purchase |
| LOW_STOCK | Quantity below threshold |
| OUT_OF_STOCK | No sellable inventory |
| RESERVED | Inventory reserved during checkout |
| BACKORDER | Inventory expected from supplier |
| PREORDER | Available for future release |
| DISCONTINUED | Product no longer sold |

Status changes generate events for downstream systems.

---

# Inventory Reservation

Inventory is **never deducted** when customers add products to their cart.

Inventory is reserved only during checkout.

Example Flow

```
Checkout

↓

Reserve Inventory

↓

Payment

↓

Order Created

↓

Inventory Deducted
```

If payment fails

```
Release Reservation

↓

Inventory Available Again
```

---

# Idempotency

Inventory operations are idempotent to prevent duplicate stock reservations caused by retries or network failures.

Every reservation request includes an **Idempotency Key**.

Example

```
POST /inventory/reserve

Idempotency-Key:
ORD-12345
```

If the same request is received again, the previously processed response is returned instead of creating another reservation.

Benefits

- Prevent duplicate reservations
- Safe retries
- Prevent overselling
- Simplifies failure recovery

---

# Inventory Cache

Inventory availability is requested thousands of times per second.

To reduce database load, frequently accessed inventory is cached.

Technology

- Redis

Cache Contents

- Available Quantity
- Inventory Status
- Warehouse Availability

TTL

30–60 seconds

Cache Strategy

Cache Aside Pattern

```
Request

↓

Redis

↓

Hit

↓

Return

↓

Miss

↓

Database

↓

Populate Cache
```

Inventory update events automatically invalidate cache entries.

---

# Inventory Status Updates

Inventory is updated whenever a business event occurs.

Examples

| Event | Action |
|---------|---------|
| Order Created | Reserve Inventory |
| Payment Successful | Deduct Inventory |
| Payment Failed | Release Reservation |
| Order Cancelled | Release Inventory |
| Return Initiated | Add Inventory |
| Warehouse Update | Refresh Quantity |
| Supplier Delivery | Increase Stock |

Every update publishes an InventoryUpdated event.

---

# Events Published

- InventoryReserved
- InventoryReleased
- InventoryUpdated
- InventoryLow
- InventoryOutOfStock

---

# Events Consumed

- OrderCreated
- PaymentFailed
- OrderCancelled
- ReturnCompleted
- SupplierInventoryUpdated

---

# Scalability

The Inventory Service is stateless and scales horizontally.

Frequently accessed inventory is cached using Redis, while transactional inventory updates are persisted in PostgreSQL.

For high-demand products, inventory can be partitioned by warehouse or fulfillment center to distribute load.

---

# Design Decisions

| Decision | Reason |
|------------|---------|
| PostgreSQL | ACID transactions for stock updates |
| Redis | Fast inventory lookup |
| Idempotency | Prevent duplicate reservations |
| Kafka | Event-driven inventory updates |
| Database Locking | Prevent race conditions |
| Cache Aside | Reduce database load |

---

# Interview Questions

### Why isn't inventory deducted when a customer adds an item to the cart?

Adding an item to the cart does not guarantee a purchase. Reserving inventory at this stage would unnecessarily block stock and reduce product availability for other customers. Inventory is reserved only during checkout.

---

### How do you prevent overselling?

Overselling is prevented through inventory reservation during checkout, database locking or optimistic concurrency control, idempotent reservation requests, and event-driven inventory updates.

---

### Why cache inventory?

Inventory is one of the most frequently accessed datasets in an e-commerce platform. Caching significantly reduces database load and improves response times while keeping inventory reasonably fresh through short TTLs and event-based cache invalidation.

---

### What happens if Redis is unavailable?

The service falls back to the primary database. Although response times may increase, inventory accuracy is maintained. Once Redis becomes available, the cache is repopulated automatically.