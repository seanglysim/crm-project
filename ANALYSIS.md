# CRM Project Analysis

## Project Overview
This is a Django-based Customer Relationship Management (CRM) system designed for managing customers, products, and orders. The application provides a dashboard with order tracking, customer management, and product catalog features.

---

## 1. Entity Relationship Diagram (ERD)

```
┌─────────────────┐
│    CUSTOMER     │
├─────────────────┤
│ id (PK)         │
│ name            │
│ phone           │
│ email           │
│ date_created    │
└─────────────────┘
        │
        │ (1:N)
        │ FK customer_id
        ▼
┌─────────────────┐         ┌─────────────────┐
│     ORDER       │◄────────│    PRODUCT      │
├─────────────────┤  (N:1)  ├─────────────────┤
│ id (PK)         │         │ id (PK)         │
│ customer_id (FK)│─────────│ name            │
│ product_id (FK) │         │ price           │
│ date_created    │         │ category        │
│ status          │         │ description     │
└─────────────────┘         │ date_created    │
                            └─────────────────┘
```

### Entity Definitions

**CUSTOMER**
- Primary Key: `id` (auto-increment)
- Fields:
  - `name` (CharField, max 200)
  - `phone` (CharField, max 200)
  - `email` (CharField, max 200)
  - `date_created` (DateTime, auto)
- Properties: `orders` (count of related orders)

**PRODUCT**
- Primary Key: `id` (auto-increment)
- Fields:
  - `name` (CharField, max 200)
  - `price` (FloatField)
  - `category` (CharField, choices: 'Indoor', 'Out Door')
  - `description` (TextField)
  - `date_created` (DateTime, auto)

**ORDER**
- Primary Key: `id` (auto-increment)
- Foreign Keys:
  - `customer_id` → CUSTOMER (N:1, SET_NULL)
  - `product_id` → PRODUCT (N:1, SET_NULL)
- Fields:
  - `date_created` (DateTime, auto)
  - `status` (CharField, choices: 'Pending', 'Out for delivery', 'Delivered')

### Relationships
- **Customer → Order**: One-to-Many (1:N)
  - One customer can have multiple orders
  - Accessed via `customer.order_set.all()`
  
- **Product → Order**: One-to-Many (1:N)
  - One product can be ordered multiple times
  - Foreign Key: `product_id` in Order table

---

## 2. Data Flow Diagram (DFD)

```
                              ┌─────────────────────┐
                              │   USER/BROWSER      │
                              └──────────┬──────────┘
                                         │
                    ┌────────────────────┼────────────────────┐
                    │                    │                    │
                    ▼                    ▼                    ▼
            ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
            │  Dashboard   │      │  Products    │      │  Customer    │
            │   View       │      │   View       │      │   View       │
            └──────┬───────┘      └──────┬───────┘      └──────┬───────┘
                   │                     │                      │
        ┌──────────┴─────────┐          │              ┌───────┴────────┐
        │                    │          │              │                │
        ▼                    ▼          ▼              ▼                ▼
   ┌─────────┐         ┌──────────┐  ┌────────┐  ┌─────────────┐  ┌──────────┐
   │Dashboard│         │Order Mgmt│  │Product │  │Order Filter │  │Customer  │
   │Manager  │         │Views     │  │View    │  │Views        │  │Detail    │
   │Process  │         │(CRUD)    │  │        │  │             │  │View      │
   └────┬────┘         └────┬─────┘  └───┬────┘  └─────┬───────┘  └────┬─────┘
        │                   │            │             │               │
        │  ┌────────────────┼────────────┼─────────────┼───────────────┤
        │  │                │            │             │               │
        ▼  ▼                ▼            ▼             ▼               ▼
    ┌────────────────────────────────────────────────────────────────────────┐
    │              DJANGO ORM (models.py, filters.py)                        │
    │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
    │  │  Customer    │  │  Product     │  │  Order       │               │
    │  │  Manager     │  │  Manager     │  │  Manager     │               │
    │  └──────────────┘  └──────────────┘  └──────────────┘               │
    └────────────────┬───────────────────────────────────────────────────────┘
                     │
                     │ SQL Queries
                     ▼
    ┌──────────────────────────────────┐
    │      SQLite Database             │
    │  ┌────────────────────────────┐  │
    │  │  accounts_customer table   │  │
    │  ├────────────────────────────┤  │
    │  │ id | name | phone | email  │  │
    │  └────────────────────────────┘  │
    │                                  │
    │  ┌────────────────────────────┐  │
    │  │  accounts_product table    │  │
    │  ├────────────────────────────┤  │
    │  │ id | name | price | cat..  │  │
    │  └────────────────────────────┘  │
    │                                  │
    │  ┌────────────────────────────┐  │
    │  │  accounts_order table      │  │
    │  ├────────────────────────────┤  │
    │  │ id | customer_id | prod... │  │
    │  └────────────────────────────┘  │
    └──────────────────────────────────┘
```

### Data Flow Steps

1. **User Request**: Browser sends HTTP request to Django URLs
2. **URL Routing**: `urls.py` routes to appropriate view function
3. **View Processing**: 
   - `dashBoard()` - aggregates statistics
   - `products()` - fetches all products
   - `customer()` - fetches customer & filtered orders
   - `createOrder()`, `updateOrder()`, `deleteOrder()` - CRUD operations
4. **ORM Layer**: Django ORM translates to SQL queries
5. **Database Operations**: SQLite executes queries
6. **Response**: Data is rendered in HTML templates and returned to browser

### Key Data Flows

**Dashboard View Flow:**
```
User visits / → dashBoard() → Query orders (last 5) 
             → Query customers count
             → Query order statistics (delivered, pending)
             → Render dashboard.html with context
```

**Create Order Flow:**
```
User clicks Create → GET: show empty form
User submits → POST: validate form
             → Save to database
             → Redirect to home
```

**Update Order Flow:**
```
User clicks Update → GET: fetch order, show pre-filled form
User submits → POST: validate, update order
             → Redirect to customer page
```

---

## 3. Context Diagram (System Context Diagram)

```
┌─────────────────────────────────────────────────────────────────┐
│                      CRM System Boundary                         │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │    ┌─────────────────────────────────────────────────┐   │  │
│  │    │          Django CRM Application                 │   │  │
│  │    │  (Views, Forms, Filters, Templates)             │   │  │
│  │    │                                                  │   │  │
│  │    │  • Dashboard                                     │   │  │
│  │    │  • Product Management                            │   │  │
│  │    │  • Customer Management                           │   │  │
│  │    │  • Order Management (CRUD)                       │   │  │
│  │    │  • Order Filtering & Tracking                    │   │  │
│  │    └─────────────────────────────────────────────────┘   │  │
│  │                         │                                  │  │
│  │                         ▼                                  │  │
│  │    ┌─────────────────────────────────────────────────┐   │  │
│  │    │         SQLite Database                         │   │  │
│  │    │  (Customer, Product, Order tables)              │   │  │
│  │    └─────────────────────────────────────────────────┘   │  │
│  │                                                            │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
     ▲                                                    ▲
     │                                                    │
     │ HTTP Requests/Responses                           │
     │ (GET, POST, PUT, DELETE)                          │
     │                                                    │
┌────┴────────────────────────────────────────────────────┴───────┐
│                                                                   │
│                         END USERS                                │
│  (Admin, Staff, Managers via Web Browser)                       │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

### System Components

**External Actors:**
- **End Users**: Access the CRM through a web browser to manage customers, products, and orders

**Internal Systems:**
1. **Django Application Layer**
   - URL Routing: Maps HTTP requests to views
   - Views: Handle business logic
   - Forms: Validate and process user input
   - Filters: Filter orders by criteria
   - Templates: Render HTML responses

2. **Data Persistence Layer**
   - SQLite Database: Stores all application data

### System Responsibilities

| Component | Responsibility |
|-----------|---|
| **Dashboard** | Provide overview of orders, customers, metrics |
| **Product Management** | Display product catalog with pricing |
| **Customer Management** | Manage customer profiles and view order history |
| **Order Management** | Full CRUD operations for orders |
| **Order Filtering** | Filter and search orders by status |
| **Order Tracking** | Track order status (Pending → Out for delivery → Delivered) |

### User Interactions

1. **View Dashboard**: See order statistics and customer overview
2. **Manage Products**: Browse product catalog
3. **Manage Customers**: View customer profiles and order history
4. **Create/Update/Delete Orders**: Manage order lifecycle
5. **Filter Orders**: Search orders by status or criteria

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| **Backend** | Django 4.2.26 (Python web framework) |
| **Database** | SQLite (development) |
| **Frontend** | HTML Templates, Bootstrap CSS |
| **ORM** | Django ORM |
| **Additional Libraries** | django-widget-tweaks, django-filter |

---

## Project Structure Summary

```
crm/
├── accounts/
│   ├── models.py           ← Data models (Customer, Product, Order)
│   ├── views.py            ← Business logic (views)
│   ├── forms.py            ← Form validation
│   ├── filters.py          ← Order filtering
│   ├── urls.py             ← URL routing
│   ├── admin.py            ← Django admin configuration
│   ├── templates/          ← HTML templates
│   │   └── accounts/
│   │       ├── dashboard.html
│   │       ├── customer.html
│   │       ├── products.html
│   │       ├── order_form.html
│   │       └── ...
│   └── migrations/         ← Database schema migrations
│
├── crm/
│   ├── settings.py         ← Django configuration
│   ├── urls.py             ← Root URL configuration
│   └── wsgi.py             ← WSGI application
│
└── manage.py               ← Django management script
```

---

## Data Model Details

### Customer Model
```python
Customer
├── id: Integer (Primary Key)
├── name: String(200)
├── phone: String(200)
├── email: String(200)
├── date_created: DateTime (Auto)
└── orders (Property): Count of related Order objects
```

### Product Model
```python
Product
├── id: Integer (Primary Key)
├── name: String(200)
├── price: Float
├── category: Choice ('Indoor' | 'Out Door')
├── description: Text
└── date_created: DateTime (Auto)
```

### Order Model
```python
Order
├── id: Integer (Primary Key)
├── customer_id: Foreign Key → Customer (SET_NULL)
├── product_id: Foreign Key → Product (SET_NULL)
├── date_created: DateTime (Auto)
└── status: Choice ('Pending' | 'Out for delivery' | 'Delivered')
```

---

## URL Routing Map

| URL Pattern | View Function | Method | Description |
|------------|---|--------|---|
| `/` | `dashBoard()` | GET | Dashboard with statistics |
| `/products/` | `products()` | GET | Product listing |
| `/customer/<pk>/` | `customer()` | GET | Customer detail & orders |
| `/order/create/` | `createOrder()` | GET, POST | Create new order |
| `/order/update/<pk>/` | `updateOrder()` | GET, POST | Update order |
| `/order/delete/<pk>/` | `deleteOrder()` | GET, POST | Delete order |

---

## Error Handling & Security Notes

### Current Implementation Issues:
1. **Foreign Key Handling**: Uses `SET_NULL` which may leave orphaned orders if customer is deleted
2. **Form Validation**: Limited validation in forms
3. **Authorization**: No user authentication/authorization (admin only)
4. **SQL Injection**: Protected by Django ORM parameterized queries
5. **CSRF Protection**: Should be enabled in production settings

### Recommendations:
1. Add user authentication and role-based access control
2. Implement proper error handling and user feedback
3. Add transaction management for order creation
4. Implement audit logging for order changes
5. Add data validation and sanitization
