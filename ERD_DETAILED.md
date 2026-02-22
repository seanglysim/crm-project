# Entity Relationship Diagram (ERD) - Complete Documentation

## Overview

This document provides a comprehensive Entity Relationship Diagram (ERD) for the Django CRM system. The ERD illustrates how the three main entities (Customer, Product, Order) are related and structured within the database.

---

## 1. Basic ERD Visual

### Simple Entity Relationship Structure

```
                    ┌─────────────────┐
                    │    CUSTOMER     │
                    ├─────────────────┤
                    │ id (PK)         │
                    │ name            │
                    │ phone           │
                    │ email           │
                    │ date_created    │
                    └─────┬───────────┘
                          │
                          │ 1:N Relationship
                          │ (One customer → Many orders)
                          │
                    ┌─────▼───────────┐         ┌──────────────┐
                    │     ORDER       │◄────────│   PRODUCT    │
                    ├─────────────────┤  1:N    ├──────────────┤
                    │ id (PK)         │         │ id (PK)      │
                    │ customer_id (FK)│─┐       │ name         │
                    │ product_id (FK) │─┼──────►│ price        │
                    │ date_created    │ │       │ category     │
                    │ status          │ │       │ description  │
                    └─────────────────┘ │       │ date_created │
                                        │       └──────────────┘
                                        │
                          Relationship: 1:N (One product → Many orders)
```

---

## 2. Detailed Entity Specifications

### 2.1 CUSTOMER Entity

**Purpose**: Store information about customers/clients

**Primary Key**: `id` (Auto-incrementing Integer)

| Field | Type | Nullable | Default | Description |
|-------|------|----------|---------|-------------|
| `id` | Integer | NO | AUTO | Unique customer identifier |
| `name` | String(200) | YES | NULL | Full name of customer |
| `phone` | String(200) | YES | NULL | Contact phone number |
| `email` | String(200) | YES | NULL | Email address |
| `date_created` | DateTime | YES | NOW | Timestamp when customer record created |

**Relationships**:
- `ORDER.customer_id` → `CUSTOMER.id` (One-to-Many)

**Example Data**:
```
id | name           | phone        | email              | date_created
1  | John Doe       | 555-0123     | john@example.com   | 2024-01-15 10:30:00
2  | Jane Smith     | 555-0124     | jane@example.com   | 2024-01-16 11:45:00
3  | Bob Johnson    | 555-0125     | bob@example.com    | 2024-01-17 09:15:00
```

---

### 2.2 PRODUCT Entity

**Purpose**: Store product information for sale

**Primary Key**: `id` (Auto-incrementing Integer)

| Field | Type | Nullable | Choices | Description |
|-------|------|----------|---------|-------------|
| `id` | Integer | NO | - | Unique product identifier |
| `name` | String(200) | YES | - | Product name |
| `price` | Float | YES | - | Product price |
| `category` | String(200) | YES | 'Indoor', 'Out Door' | Product category |
| `description` | Text | NO | - | Detailed description |
| `date_created` | DateTime | YES | - | Record creation timestamp |

**Relationships**:
- `ORDER.product_id` → `PRODUCT.id` (One-to-Many)

**Example Data**:
```
id | name        | price  | category  | description                    | date_created
1  | Chair       | 50.00  | Indoor    | Comfortable office chair       | 2024-01-10 08:00:00
2  | Table       | 100.00 | Indoor    | Solid wood conference table    | 2024-01-10 08:15:00
3  | Desk Lamp   | 35.00  | Indoor    | LED desk lamp with USB port    | 2024-01-11 09:30:00
4  | Garden Bench| 150.00 | Out Door  | Teak wood outdoor bench        | 2024-01-12 10:45:00
5  | Plant Pot   | 20.00  | Out Door  | Ceramic outdoor planter       | 2024-01-12 11:00:00
```

---

### 2.3 ORDER Entity

**Purpose**: Store order information linking customers to products

**Primary Key**: `id` (Auto-incrementing Integer)

| Field | Type | Nullable | FK Reference | Choices | Description |
|-------|------|----------|--------------|---------|-------------|
| `id` | Integer | NO | - | - | Unique order identifier |
| `customer_id` | Integer | YES | CUSTOMER.id | - | Reference to customer |
| `product_id` | Integer | YES | PRODUCT.id | - | Reference to product |
| `date_created` | DateTime | YES | - | - | Order creation timestamp |
| `status` | String(200) | YES | - | 'Pending', 'Out for delivery', 'Delivered' | Order status |

**Foreign Key Constraints**:
- `customer_id` → `CUSTOMER.id` with `ON DELETE SET NULL`
- `product_id` → `PRODUCT.id` with `ON DELETE SET NULL`

**Example Data**:
```
id | customer_id | product_id | date_created           | status
1  | 1          | 1          | 2024-01-18 14:30:00   | Delivered
2  | 1          | 2          | 2024-01-18 14:35:00   | Pending
3  | 2          | 3          | 2024-01-19 09:00:00   | Out for delivery
4  | 3          | 4          | 2024-01-19 10:15:00   | Pending
5  | 2          | 5          | 2024-01-20 11:30:00   | Delivered
```

---

## 3. Relationship Types and Cardinality

### 3.1 Customer ↔ Order (One-to-Many)

```
┌──────────────┐          ┌──────────────┐
│  CUSTOMER    │ 1 ─────N │    ORDER     │
│   (1 side)   │  1 to N  │   (N side)   │
└──────────────┘          └──────────────┘
```

**Explanation**:
- **One** customer can place **Many** orders
- **Each** order belongs to **Exactly One** customer (or NULL if customer deleted)
- **Implementation**: Foreign key `customer_id` in ORDER table

**Query Examples**:
```python
# Django ORM
customer = Customer.objects.get(id=1)
orders = customer.order_set.all()  # Get all orders by this customer

order = Order.objects.get(id=1)
customer = order.customer  # Get the customer who placed this order
```

**SQL Relationship**:
```sql
-- All orders for customer with id=1
SELECT * FROM accounts_order 
WHERE customer_id = 1;

-- Count orders per customer
SELECT customer_id, COUNT(*) as order_count 
FROM accounts_order 
GROUP BY customer_id;
```

---

### 3.2 Product ↔ Order (One-to-Many)

```
┌──────────────┐          ┌──────────────┐
│  PRODUCT     │ 1 ─────N │    ORDER     │
│   (1 side)   │  1 to N  │   (N side)   │
└──────────────┘          └──────────────┘
```

**Explanation**:
- **One** product can be ordered **Many** times
- **Each** order references **Exactly One** product (or NULL if product deleted)
- **Implementation**: Foreign key `product_id` in ORDER table

**Query Examples**:
```python
# Django ORM
product = Product.objects.get(id=1)
orders = product.order_set.all()  # Get all orders for this product

order = Order.objects.get(id=1)
product = order.product  # Get the product in this order
```

**SQL Relationship**:
```sql
-- All orders for product with id=1
SELECT * FROM accounts_order 
WHERE product_id = 1;

-- Count sales per product
SELECT product_id, COUNT(*) as times_ordered 
FROM accounts_order 
GROUP BY product_id;
```

---

## 4. Foreign Key Constraints & Cascading

### 4.1 Cascade Behavior

Both foreign keys use `SET_NULL` behavior:

```python
# Django Model Definition
class Order(models.Model):
    customer = models.ForeignKey(
        Customer, 
        on_delete=models.SET_NULL,  # ← SET_NULL behavior
        null=True
    )
    product = models.ForeignKey(
        Product, 
        on_delete=models.SET_NULL,  # ← SET_NULL behavior
        null=True
    )
```

### 4.2 Deletion Scenarios

**Scenario 1: Customer Deleted**
```
BEFORE:
Order 1: customer_id=5, product_id=2, status='Pending'

IF Customer(id=5) is deleted:

AFTER (SET_NULL):
Order 1: customer_id=NULL, product_id=2, status='Pending'
↑ Order survives with NULL customer reference
```

**Scenario 2: Product Deleted**
```
BEFORE:
Order 2: customer_id=1, product_id=7, status='Delivered'

IF Product(id=7) is deleted:

AFTER (SET_NULL):
Order 2: customer_id=1, product_id=NULL, status='Delivered'
↑ Order survives with NULL product reference
```

### 4.3 Referential Integrity

The database ensures:
- Cannot create Order with non-existent customer_id (unless NULL)
- Cannot create Order with non-existent product_id (unless NULL)
- Invalid foreign key values are rejected

---

## 5. Complete Entity Structure Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    DATABASE SCHEMA                          │
└─────────────────────────────────────────────────────────────┘

┌──────────────────────────┐
│     CUSTOMER TABLE       │
├──────────────────────────┤
│ id                 [PK]  │  ◄─────┐
│ name                     │        │
│ phone                    │        │
│ email                    │        │  1:N Relationship
│ date_created             │        │
└──────────────────────────┘        │
                                    │
                    ┌───────────────┴─────────────────┐
                    │                                 │
          ┌─────────▼──────────────┐      ┌──────────▼─────────────┐
          │   ORDER TABLE          │      │   PRODUCT TABLE        │
          ├────────────────────────┤      ├────────────────────────┤
          │ id            [PK]     │      │ id              [PK]   │
          │ customer_id   [FK]─────┼──────► (fk to CUSTOMER)      │
          │ product_id    [FK]─────┼──────► id (references)       │
          │ date_created           │      │ name                   │
          │ status                 │      │ price                  │
          └────────────────────────┘      │ category               │
                                         │ description            │
                                         │ date_created           │
                                         └────────────────────────┘

Legend:
[PK] = Primary Key
[FK] = Foreign Key
──┬── = One-to-Many relationship (1:N)
  │
```

---

## 6. Data Integrity Rules

### 6.1 Domain Constraints

| Field | Rule | Validation |
|-------|------|-----------|
| Customer.name | Required | Max 200 chars |
| Customer.phone | Optional | Max 200 chars |
| Customer.email | Optional | Max 200 chars |
| Product.price | Required | Float value |
| Product.category | Limited Choice | 'Indoor' or 'Out Door' |
| Order.status | Limited Choice | 'Pending' or 'Out for delivery' or 'Delivered' |

### 6.2 Referential Integrity

| Constraint | Rule | Effect |
|-----------|------|--------|
| Order.customer_id → Customer.id | Foreign Key | Must exist or be NULL |
| Order.product_id → Product.id | Foreign Key | Must exist or be NULL |
| Customer.id | Primary Key | Must be unique and NOT NULL |
| Product.id | Primary Key | Must be unique and NOT NULL |
| Order.id | Primary Key | Must be unique and NOT NULL |

---

## 7. Sample Data Relationships

### Example 1: Simple Purchase

```
Customer (id=1): John Doe
├─ Order (id=1): 
│  ├─ Customer: John Doe (id=1)
│  ├─ Product: Chair (id=1)
│  └─ Status: Delivered
│
└─ Order (id=2):
   ├─ Customer: John Doe (id=1)
   ├─ Product: Table (id=2)
   └─ Status: Pending
```

### Example 2: Multiple Customers Ordering Same Product

```
Product (id=3): Desk Lamp

Orders for Desk Lamp:
├─ Order (id=10): Customer Jane Smith (id=2) → Delivered
├─ Order (id=15): Customer Bob Johnson (id=3) → Pending
└─ Order (id=20): Customer John Doe (id=1) → Out for delivery
```

### Example 3: Orphaned Order (After Customer Deletion)

```
BEFORE: Customer deletion
Order (id=5): customer_id=2, product_id=1

AFTER: Customer(id=2) deleted (SET_NULL applied)
Order (id=5): customer_id=NULL, product_id=1
┌─ Notice: Customer reference lost, but order data preserved
```

---

## 8. Useful Queries for Understanding Data

### Count by Relationship

```sql
-- How many orders does each customer have?
SELECT c.id, c.name, COUNT(o.id) as order_count
FROM accounts_customer c
LEFT JOIN accounts_order o ON c.id = o.customer_id
GROUP BY c.id, c.name
ORDER BY order_count DESC;

-- How many times has each product been ordered?
SELECT p.id, p.name, COUNT(o.id) as times_ordered
FROM accounts_product p
LEFT JOIN accounts_order o ON p.id = o.product_id
GROUP BY p.id, p.name
ORDER BY times_ordered DESC;

-- What's the total value of orders per customer?
SELECT c.id, c.name, SUM(p.price) as total_spent
FROM accounts_customer c
LEFT JOIN accounts_order o ON c.id = o.customer_id
LEFT JOIN accounts_product p ON o.product_id = p.id
GROUP BY c.id, c.name
ORDER BY total_spent DESC;
```

### Find Anomalies

```sql
-- Orders with missing customer (SET_NULL in action)
SELECT * FROM accounts_order 
WHERE customer_id IS NULL;

-- Orders with missing product (SET_NULL in action)
SELECT * FROM accounts_order 
WHERE product_id IS NULL;

-- Customers with no orders
SELECT c.* FROM accounts_customer c
LEFT JOIN accounts_order o ON c.id = o.customer_id
WHERE o.id IS NULL;
```

---

## 9. Django ORM Examples

### Querying Relationships

```python
# Get customer and their orders
from accounts.models import Customer, Order, Product

customer = Customer.objects.get(id=1)
customer_orders = customer.order_set.all()

# Get order and related customer + product
order = Order.objects.get(id=1)
print(order.customer.name)
print(order.product.name)

# Filter orders by status
pending_orders = Order.objects.filter(status='Pending')

# Orders by specific customer
john_orders = Order.objects.filter(customer__name='John Doe')

# Products ordered by specific customer
products_bought_by_john = Product.objects.filter(
    order__customer__name='John Doe'
).distinct()
```

### Aggregation

```python
from django.db.models import Count, Sum

# Total orders per customer
customer_orders = Customer.objects.annotate(
    total_orders=Count('order')
)

# Average order price per customer
from django.db.models import Avg
avg_price = Order.objects.values('customer__name').annotate(
    avg_price=Avg('product__price')
)
```

---

## 10. Index Recommendations

For performance optimization, consider adding indexes:

```python
# In models.py
class Order(models.Model):
    customer = models.ForeignKey(Customer, on_delete=models.SET_NULL, 
                                 null=True, db_index=True)  # ← Add index
    product = models.ForeignKey(Product, on_delete=models.SET_NULL,
                               null=True, db_index=True)   # ← Add index
    status = models.CharField(max_length=200, null=True, db_index=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['customer_id', 'status']),  # Composite index
            models.Index(fields=['product_id', 'date_created']),
        ]
```

---

## 11. Migration Guide

### Creating the Database Schema

```bash
# Create initial migrations
python manage.py makemigrations accounts

# Apply migrations to database
python manage.py migrate accounts
```

### Adding a New Field

Example: Add `quantity` field to Order

```python
# models.py
class Order(models.Model):
    # ... existing fields ...
    quantity = models.IntegerField(default=1)  # New field
```

```bash
python manage.py makemigrations accounts
python manage.py migrate accounts
```

---

## Summary

This Entity Relationship Diagram defines:

1. **Three Main Entities**: Customer, Product, Order
2. **Two One-to-Many Relationships**: Customer→Order, Product→Order
3. **Referential Integrity**: Foreign keys with SET_NULL cascade behavior
4. **Primary Keys**: Auto-incrementing integer IDs for each entity
5. **Data Constraints**: Choice fields for category and status

The design enables efficient tracking of customer orders and product sales while maintaining data integrity and supporting common CRM operations like customer history, product popularity analysis, and order status tracking.

