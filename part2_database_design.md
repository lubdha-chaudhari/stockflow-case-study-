

# Part 2: Database Design

## Objective
Design a scalable inventory management schema for a B2B SaaS platform with incomplete requirements.

---

## Proposed Schema

### Companies
companies (
id PK,
name,
created_at
)

shell
Copy code

### Warehouses
warehouses (
id PK,
company_id FK,
name,
location
)

shell
Copy code

### Products
products (
id PK,
company_id FK,
name,
sku UNIQUE,
price DECIMAL(10,2),
is_bundle BOOLEAN,
low_stock_threshold INT
)

shell
Copy code

### Inventory
inventory (
id PK,
product_id FK,
warehouse_id FK,
quantity INT,
UNIQUE(product_id, warehouse_id)
)

shell
Copy code

### Inventory History
inventory_changes (
id PK,
product_id FK,
warehouse_id FK,
old_quantity,
new_quantity,
changed_at
)

shell
Copy code

### Suppliers
suppliers (
id PK,
name,
contact_email
)

shell
Copy code

### Product–Supplier Mapping
product_suppliers (
product_id FK,
supplier_id FK
)

shell
Copy code

### Product Bundles
bundle_items (
bundle_id FK,
product_id FK,
quantity
)

yaml
Copy code

---

## Design Decisions

- Inventory separated from Product to support multiple warehouses
- Inventory history enables auditing and analytics
- Unique constraints prevent duplicate inventory rows
- Join tables allow flexible supplier and bundle relationships

---

## Missing Requirements / Questions for Product Team

- Can products have multiple suppliers?
- Should low-stock thresholds vary per warehouse?
- What defines “recent sales activity”?
- Are bundles sellable independently?
- Should inventory ever go negative?

---

## Scalability Considerations

- Index on `sku`, `product_id`, and `warehouse_id`
- History tables allow time-based analysis without bloating core tables
