# Part 2: Database Design – Inventory Management System (StockFlow)

## Objective

Design a scalable and flexible database schema for a B2B inventory management platform where:
- Companies manage multiple warehouses
- Products can exist in multiple warehouses
- Inventory changes must be tracked over time
- Suppliers provide products
- Some products are bundles made up of other products

The requirements are intentionally incomplete, so assumptions and open questions are documented explicitly.

---

## Core Design Principles

- Normalize data to avoid duplication
- Support many-to-many relationships where required
- Ensure data integrity using constraints
- Allow future scalability (analytics, reporting, auditing)
- Keep schema readable and maintainable

---

## Proposed Database Schema

### 1. Companies

Represents businesses using the platform.

companies (
id BIGINT PRIMARY KEY,
name VARCHAR(255) NOT NULL,
created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
)

---

### 2. Warehouses

Each company can operate multiple warehouses.

warehouses (
id BIGINT PRIMARY KEY,
company_id BIGINT NOT NULL,
name VARCHAR(255) NOT NULL,
location VARCHAR(255),
created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

FOREIGN KEY (company_id) REFERENCES companies(id)
)


---

### 3. Products

Products belong to a company and are independent of warehouses.

products (
id BIGINT PRIMARY KEY,
company_id BIGINT NOT NULL,
name VARCHAR(255) NOT NULL,
sku VARCHAR(100) NOT NULL UNIQUE,
price DECIMAL(10,2) NOT NULL,
is_bundle BOOLEAN NOT NULL DEFAULT FALSE,
low_stock_threshold INT NOT NULL DEFAULT 0,
created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

FOREIGN KEY (company_id) REFERENCES companies(id)
)


**Why this design**
- SKU is globally unique as required
- Product is decoupled from warehouse to support multiple locations
- `low_stock_threshold` supports alert logic

---

### 4. Inventory

Tracks product quantities per warehouse.

inventory (
id BIGINT PRIMARY KEY,
product_id BIGINT NOT NULL,
warehouse_id BIGINT NOT NULL,
quantity INT NOT NULL,
updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

UNIQUE (product_id, warehouse_id),
FOREIGN KEY (product_id) REFERENCES products(id),
FOREIGN KEY (warehouse_id) REFERENCES warehouses(id)
)


**Why this design**
- Composite uniqueness prevents duplicate inventory rows
- Supports same product in multiple warehouses
- Enables easy aggregation and reporting

---

### 5. Inventory Change History

Tracks all inventory movements for auditing and analysis.

inventory_changes (
id BIGINT PRIMARY KEY,
product_id BIGINT NOT NULL,
warehouse_id BIGINT NOT NULL,
old_quantity INT NOT NULL,
new_quantity INT NOT NULL,
change_reason VARCHAR(255),
changed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

FOREIGN KEY (product_id) REFERENCES products(id),
FOREIGN KEY (warehouse_id) REFERENCES warehouses(id)
)

inventory_changes (
id BIGINT PRIMARY KEY,
product_id BIGINT NOT NULL,
warehouse_id BIGINT NOT NULL,
old_quantity INT NOT NULL,
new_quantity INT NOT NULL,
change_reason VARCHAR(255),
changed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

FOREIGN KEY (product_id) REFERENCES products(id),
FOREIGN KEY (warehouse_id) REFERENCES warehouses(id)
)


**Why this design**
- Enables traceability of stock changes
- Useful for debugging, audits, and analytics
- Prevents bloating main inventory table

---

### 6. Suppliers

Suppliers provide products to companies.

suppliers (
id BIGINT PRIMARY KEY,
name VARCHAR(255) NOT NULL,
contact_email VARCHAR(255),
created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
)


---

### 7. Product–Supplier Mapping

Supports multiple suppliers per product.



product_suppliers (
product_id BIGINT NOT NULL,
supplier_id BIGINT NOT NULL,

PRIMARY KEY (product_id, supplier_id),
FOREIGN KEY (product_id) REFERENCES products(id),
FOREIGN KEY (supplier_id) REFERENCES suppliers(id)
)


---

### 8. Product Bundles

Defines bundled products composed of multiple items.



bundle_items (
bundle_id BIGINT NOT NULL,
product_id BIGINT NOT NULL,
quantity INT NOT NULL,

PRIMARY KEY (bundle_id, product_id),
FOREIGN KEY (bundle_id) REFERENCES products(id),
FOREIGN KEY (product_id) REFERENCES products(id)
)


**Why this design**
- Supports recursive product composition
- Allows flexible bundle definitions
- Keeps bundle logic separate from core product data

---

## Indexing Strategy

- `products.sku` → Unique index for fast lookup
- `inventory.product_id`, `inventory.warehouse_id` → Query optimization
- `inventory_changes.changed_at` → Time-based analytics
- `warehouses.company_id` → Company-level filtering

---

## Missing Requirements & Questions for Product Team

1. Can a product have multiple active suppliers at the same time?
2. Should low-stock thresholds vary by warehouse or remain product-level?
3. What qualifies as “recent sales activity”?
4. Are bundles allowed to be sold independently?
5. Should inventory quantities ever be allowed to go negative?
6. Is inventory tracked in real-time or via batch updates?

These answers would influence indexing, constraints, and API behavior.

---

## Scalability & Maintenance Considerations

- Schema supports horizontal scaling via company isolation
- History tables enable analytics without affecting transactional performance
- Design allows future extensions (pricing rules, reorder automation, forecasting)

---

## Conclusion

This schema balances flexibility, correctness, and scalability while aligning with real-world B2B inventory workflows. Assumptions are documented, and the design can evolve as product requirements mature.
