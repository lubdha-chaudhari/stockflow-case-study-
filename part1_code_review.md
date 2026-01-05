# Part 1: Code Review & Debugging

## Original Problem
The given API endpoint compiles successfully but fails to behave correctly in production environments.

---

## Issues Identified

### 1. No Input Validation
- `request.json` is accessed directly without validation.
- Missing or malformed JSON causes runtime errors.

**Impact:**  
API crashes (500 errors), unstable behavior in production.

---

### 2. SKU Uniqueness Not Enforced
- Business rule requires SKU to be unique across the platform.
- Code does not check or handle duplicates.

**Impact:**  
Duplicate SKUs lead to incorrect inventory tracking and operational failures.

---

### 3. Product Incorrectly Tied to a Single Warehouse
- Product model includes `warehouse_id`.
- Requirement states products can exist in multiple warehouses.

**Impact:**  
Invalid data model, inability to scale inventory across warehouses.

---

### 4. No Transaction Safety
- Two separate commits are used.
- Partial failures lead to inconsistent data.

**Impact:**  
Product may exist without inventory records.

---

### 5. Floating-Point Price Handling
- Price is directly assigned.
- Floating-point precision errors may occur.

**Impact:**  
Incorrect pricing and billing discrepancies.

---

### 6. No Error Handling or Rollback
- No try/except blocks.
- Database session is not rolled back on failure.

**Impact:**  
Corrupt or partial data stored in the database.

---

## Corrected Implementation

```python
from decimal import Decimal
from sqlalchemy.exc import IntegrityError

@app.route('/api/products', methods=['POST'])
def create_product():
    data = request.get_json()

    if not data:
        return {"error": "Invalid JSON body"}, 400

    required_fields = ['name', 'sku', 'price']
    for field in required_fields:
        if field not in data:
            return {"error": f"{field} is required"}, 400

    try:
        product = Product(
            name=data['name'],
            sku=data['sku'],
            price=Decimal(str(data['price']))
        )

        db.session.add(product)
        db.session.flush()  # Obtain product.id safely

        if 'warehouse_id' in data and 'initial_quantity' in data:
            inventory = Inventory(
                product_id=product.id,
                warehouse_id=data['warehouse_id'],
                quantity=data['initial_quantity']
            )
            db.session.add(inventory)

        db.session.commit()

        return {
            "message": "Product created successfully",
            "product_id": product.id
        }, 201

    except IntegrityError:
        db.session.rollback()
        return {"error": "SKU must be unique"}, 409

    except Exception:
        db.session.rollback()
        return {"error": "Internal server error"}, 500

Why This Fix Works

Ensures atomic operations

Preserves data integrity

Supports multiple warehouses

Prevents duplicate SKUs

Handles production failures gracefully
