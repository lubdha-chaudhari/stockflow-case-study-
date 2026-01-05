# Part 3: Low-Stock Alerts API

## Endpoint
GET /api/companies/{company_id}/alerts/low-stock

---

## Assumptions

- Recent sales = activity within last 30 days
- Low-stock threshold stored in `products.low_stock_threshold`
- Products without recent sales are ignored
- One primary supplier per product for alerting
- Sales data exists in `order_items`

---

## Implementation (Flask Example)

```python
@app.route('/api/companies/<int:company_id>/alerts/low-stock', methods=['GET'])
def low_stock_alerts(company_id):
    alerts = []

    inventories = db.session.query(Inventory)\
        .join(Product)\
        .join(Warehouse)\
        .filter(Warehouse.company_id == company_id)\
        .all()

    for inv in inventories:
        product = inv.product

        if inv.quantity >= product.low_stock_threshold:
            continue

        if not product.has_recent_sales():
            continue

        supplier = product.suppliers[0] if product.suppliers else None

        alerts.append({
            "product_id": product.id,
            "product_name": product.name,
            "sku": product.sku,
            "warehouse_id": inv.warehouse.id,
            "warehouse_name": inv.warehouse.name,
            "current_stock": inv.quantity,
            "threshold": product.low_stock_threshold,
            "days_until_stockout": product.estimate_stockout_days(),
            "supplier": {
                "id": supplier.id,
                "name": supplier.name,
                "contact_email": supplier.contact_email
            } if supplier else None
        })

    return {
        "alerts": alerts,
        "total_alerts": len(alerts)
    }

---

## Edge Cases Handled

- **Products without suppliers**  
  Alerts are still generated, but supplier details are returned as `null` to avoid breaking the response structure.

- **Warehouses with sufficient stock**  
  Products whose current stock is above the configured threshold are excluded early for efficiency.

- **Products with no recent sales activity**  
  These products are ignored to prevent unnecessary alerts for inactive items.

- **Multiple warehouses per company**  
  Inventory is evaluated per warehouse to ensure accurate, location-specific alerts.

---

## Why This Approach

- **Readable and maintainable logic**  
  The API uses clear conditional checks, making it easy to understand and modify.

- **Easy to extend**  
  The design supports future enhancements such as background jobs, caching, or notification services.

- **Aligned with real-world inventory workflows**  
  Alerts are actionable, contextual, and include supplier information for faster reordering decisions.
