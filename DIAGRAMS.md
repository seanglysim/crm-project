# CRM Project - Comprehensive Diagrams & Visual Representations

## Table of Contents
1. [Entity Relationship Diagram (ERD)](#1-entity-relationship-diagram-erd)
2. [Data Flow Diagram (DFD)](#2-data-flow-diagram-dfd)
3. [Context Diagram (System Context)](#3-context-diagram-system-context)
4. [Application Architecture](#4-application-architecture)
5. [Request/Response Flow](#5-requestresponse-flow)
6. [Database Schema](#6-database-schema)

---

## 1. Entity Relationship Diagram (ERD)

### Complete ERD with Attributes

```
┌──────────────────────────────────────────┐
│          CUSTOMER TABLE                  │
├──────────────────────────────────────────┤
│ PK  id: INTEGER                          │
│     name: VARCHAR(200)                   │
│     phone: VARCHAR(200)                  │
│     email: VARCHAR(200)                  │
│     date_created: DATETIME               │
└──────────────────────────────────────────┘
           │
           │ 1:N Relationship
           │ (One Customer → Many Orders)
           │
           ▼
┌──────────────────────────────────────────┐
│          ORDER TABLE                     │
├──────────────────────────────────────────┤
│ PK  id: INTEGER                          │
│ FK  customer_id: INTEGER (SET_NULL)      │
│ FK  product_id: INTEGER (SET_NULL)       │
│     date_created: DATETIME               │
│     status: VARCHAR(50) [Pending|        │
│             Out for delivery|Delivered]  │
└──────────────────────────────────────────┘
           │
           │ N:1 Relationship
           │ (Many Orders → One Product)
           │
           ▼
┌──────────────────────────────────────────┐
│          PRODUCT TABLE                   │
├──────────────────────────────────────────┤
│ PK  id: INTEGER                          │
│     name: VARCHAR(200)                   │
│     price: FLOAT                         │
│     category: VARCHAR(50) [Indoor|       │
│               Out Door]                  │
│     description: TEXT                    │
│     date_created: DATETIME               │
└──────────────────────────────────────────┘
```

### Relationship Matrix

```
┌─────────┬─────────┬────────────────┐
│ From    │ To      │ Relationship   │
├─────────┼─────────┼────────────────┤
│ Customer│ Order   │ 1:N (SET_NULL) │
│ Product │ Order   │ 1:N (SET_NULL) │
└─────────┴─────────┴────────────────┘
```

### Primary & Foreign Keys

```
CUSTOMER (customers_customer)
├── id (PrimaryKey)
├── name (CharField)
├── phone (CharField)
├── email (CharField)
└── date_created (DateTimeField)

ORDER (accounts_order)
├── id (PrimaryKey)
├── customer_id (ForeignKey→Customer, null=True)
├── product_id (ForeignKey→Product, null=True)
├── date_created (DateTimeField)
└── status (CharField)

PRODUCT (accounts_product)
├── id (PrimaryKey)
├── name (CharField)
├── price (FloatField)
├── category (CharField)
├── description (TextField)
└── date_created (DateTimeField)
```

---

## 2. Data Flow Diagram (DFD)

### Level 0: System Context

```
            ┌─────────────────────┐
            │  END USERS/BROWSER  │
            └──────────┬──────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
   [HTTP GET]   [HTTP POST]   [HTTP GET]
        │              │              │
        ▼              ▼              ▼
   ┌─────────────────────────────────────┐
   │      DJANGO CRM APPLICATION         │
   │  (Views, Templates, Forms)          │
   └──────────┬────────────────────┬─────┘
              │                    │
              │ ORM Queries        │ HTML Response
              ▼                    ▼
         ┌─────────────┐      Browser
         │ SQLite DB   │
         └─────────────┘
```

### Level 1: Detailed Data Flow

```
                        WEB BROWSER
                            │
         ┌──────────────────┼──────────────────┐
         │                  │                  │
         ▼                  ▼                  ▼
    ┌─────────┐        ┌──────────┐      ┌──────────┐
    │GET /    │        │GET       │      │POST      │
    │Dashboard│        │/products │      │/order/..│
    └────┬────┘        └────┬─────┘      └────┬─────┘
         │                  │                  │
         ▼                  ▼                  ▼
   Django URL Router (urls.py)
         │                  │                  │
         ▼                  ▼                  ▼
   ┌──────────────┐  ┌──────────────┐ ┌────────────────┐
   │dashBoard()   │  │products()    │ │createOrder()   │
   │View Function │  │View Function │ │View Function   │
   └──────┬───────┘  └──────┬───────┘ └────────┬───────┘
          │                 │                  │
          ▼                 ▼                  ▼
   ┌────────────────────────────────────────────────────┐
   │         Django ORM Layer (models.py)              │
   │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
   │  │Customer  │  │Product   │  │Order     │        │
   │  │Manager   │  │Manager   │  │Manager   │        │
   │  └──────────┘  └──────────┘  └──────────┘        │
   └────────┬───────────────────────────────┬──────────┘
            │                               │
            │ SELECT * FROM...              │ INSERT/UPDATE/DELETE
            │ SQL Queries                   │ SQL Commands
            ▼                               ▼
   ┌──────────────────────────────────────────────────┐
   │        SQLite Database                          │
   │  ┌──────────────────────────────────────────┐  │
   │  │ accounts_customer                        │  │
   │  │ accounts_product                         │  │
   │  │ accounts_order                           │  │
   │  └──────────────────────────────────────────┘  │
   └──────────┬───────────────────────────────┬─────┘
              │                               │
              │ Query Results (Row Objects)   │ Confirmation
              ▼                               ▼
   ┌────────────────────────────────────────────────────┐
   │    Django Template Rendering (HTML)               │
   │  ┌──────────────┐  ┌──────────────┐             │
   │  │dashboard.html│  │products.html │             │
   │  │customer.html │  │order_form... │             │
   │  └──────────────┘  └──────────────┘             │
   └────────────────────────┬──────────────────────────┘
                            │
                            │ HTML Response
                            ▼
                       WEB BROWSER
                     (Display & User)
```

### Data Processing Steps

```
REQUEST FLOW:
Browser → URL Route → View Function → ORM Query → Database
              ↓          ↓              ↓
          URL Pattern  Validate    Execute SQL
          Matching     & Process

RESPONSE FLOW:
Database → Query Results → Template Render → HTML Response → Browser
   ↓            ↓               ↓
Return      Convert to      Inject Data
Rows        Objects        & Format
```

### CRUD Operations Data Flow

```
CREATE (POST /order/create/)
┌────────────────┐
│ Form Submitted │
└────────┬───────┘
         ▼
┌────────────────────────────┐
│ Django Form Validation     │
│ (check required fields)    │
└────────────────────────────┘
         │ Valid?
      ┌──┴──┐
      │     │
     Yes   No
      │     │
      ▼     ▼
    Save  Return Form
      ↓    with Errors
    DB
      ↓
  Redirect


READ (GET /customer/<pk>/)
┌─────────────────────────────┐
│ Get Customer ID from URL    │
└──────────┬──────────────────┘
           ▼
    ┌────────────────────┐
    │ Query Customer by  │
    │ ID from Database   │
    └──────────┬─────────┘
               ▼
        ┌──────────────────┐
        │ Query Orders by  │
        │ Customer FK      │
        └──────────┬───────┘
                   ▼
            ┌────────────────────┐
            │ Apply Filter if    │
            │ Query Parameters   │
            └──────────┬─────────┘
                       ▼
            ┌──────────────────────┐
            │ Render Template with │
            │ Context Data         │
            └─────────────────────┘


UPDATE (POST /order/update/<pk>/)
┌──────────────────────────────┐
│ Get Order ID & Form Data     │
└──────────┬───────────────────┘
           ▼
    ┌────────────────────────┐
    │ Fetch Order by ID      │
    └──────────┬─────────────┘
               ▼
        ┌──────────────────────┐
        │ Populate Form with   │
        │ Existing Values      │
        └──────────┬───────────┘
                   ▼
            ┌────────────────────────┐
            │ Validate Updated Form  │
            │ Data                   │
            └──────────┬─────────────┘
                       ▼
                    Save to DB
                       ↓
                  Redirect


DELETE (POST /order/delete/<pk>/)
┌──────────────────────────────┐
│ Get Order ID from URL        │
└──────────┬───────────────────┘
           ▼
    ┌────────────────────────┐
    │ Fetch Order by ID      │
    └──────────┬─────────────┘
               ▼
        ┌──────────────────────┐
        │ Delete from Database │
        └──────────┬───────────┘
                   ▼
        ┌──────────────────────┐
        │ Redirect to Customer │
        │ Detail Page          │
        └──────────────────────┘
```

---

## 3. Context Diagram (System Context)

### High-Level System Boundary

```
┌─────────────────────────────────────────────────────────────┐
│                    CRM SYSTEM BOUNDARY                       │
│                                                               │
│   ┌───────────────────────────────────────────────────────┐ │
│   │          DJANGO WEB APPLICATION                       │ │
│   │  ┌──────────────────────────────────────────────────┐ │ │
│   │  │         URL ROUTING LAYER                        │ │ │
│   │  │  ├─ urls.py: Route HTTP requests to views      │ │ │
│   │  │  └─ Supports: GET, POST methods                │ │ │
│   │  └──────────────────────────────────────────────────┘ │ │
│   │                      ▼                                 │ │
│   │  ┌──────────────────────────────────────────────────┐ │ │
│   │  │    BUSINESS LOGIC LAYER (VIEWS)                 │ │ │
│   │  │  ├─ dashBoard(): Dashboard with statistics      │ │ │
│   │  │  ├─ products(): Product list view               │ │ │
│   │  │  ├─ customer(): Customer detail & orders        │ │ │
│   │  │  ├─ createOrder(): Order creation               │ │ │
│   │  │  ├─ updateOrder(): Order modification           │ │ │
│   │  │  └─ deleteOrder(): Order deletion               │ │ │
│   │  └──────────────────────────────────────────────────┘ │ │
│   │                      ▼                                 │ │
│   │  ┌──────────────────────────────────────────────────┐ │ │
│   │  │    DATA ACCESS LAYER (ORM)                       │ │ │
│   │  │  ├─ Customer.objects: CRUD operations            │ │ │
│   │  │  ├─ Product.objects: CRUD operations             │ │ │
│   │  │  ├─ Order.objects: CRUD operations               │ │ │
│   │  │  └─ OrderFilter: Custom filtering                │ │ │
│   │  └──────────────────────────────────────────────────┘ │ │
│   │                      ▼                                 │ │
│   │  ┌──────────────────────────────────────────────────┐ │ │
│   │  │    PRESENTATION LAYER (TEMPLATES)               │ │ │
│   │  │  ├─ dashboard.html: Statistics & overview        │ │ │
│   │  │  ├─ products.html: Product listing               │ │ │
│   │  │  ├─ customer.html: Customer & order details      │ │ │
│   │  │  ├─ order_form.html: Create/update form          │ │ │
│   │  │  └─ delete_item.html: Confirmation page          │ │ │
│   │  └──────────────────────────────────────────────────┘ │ │
│   │                      ▼                                 │ │
│   └───────────────────────────────────────────────────────┘ │
│                                                               │
│   ┌───────────────────────────────────────────────────────┐ │
│   │          DATA PERSISTENCE LAYER                       │ │
│   │  ┌──────────────────────────────────────────────────┐ │ │
│   │  │  SQLite Database (db.sqlite3)                   │ │ │
│   │  │  ├─ accounts_customer (Customers)               │ │ │
│   │  │  ├─ accounts_product (Products)                 │ │ │
│   │  │  └─ accounts_order (Orders with FK refs)        │ │ │
│   │  └──────────────────────────────────────────────────┘ │ │
│   │                                                         │ │
│   └───────────────────────────────────────────────────────┘ │
│                                                               │
└─────────────────────────────────────────────────────────────┘
             ▲                                        ▲
             │ HTTP Requests                         │ HTTP Responses
             │ (GET, POST)                           │ (HTML Pages)
             │                                        │
    ┌────────┴────────────────────────────────┬──────┴─────────┐
    │                                          │                │
    │                           ┌──────────────┴──────────────┐ │
    │                           │                            │ │
    ▼                           ▼                            ▼ │
┌──────────────────────────────────────────────────────────────────┐
│                       END USERS (EXTERNAL)                       │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │   Admin      │  │   Manager    │  │   Staff      │           │
│  │              │  │              │  │              │           │
│  │ Access via   │  │ Access via   │  │ Access via   │           │
│  │ Web Browser  │  │ Web Browser  │  │ Web Browser  │           │
│  │ (Chrome,    │  │ (Firefox,   │  │ (Safari,    │           │
│  │  Firefox,   │  │  Safari, etc)│  │  Chrome,    │           │
│  │  etc)       │  │              │  │  etc)       │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### System Interactions

```
USER INTERACTIONS:
┌─────────────┐
│ End User    │
│ (Browser)   │
└──────┬──────┘
       │
       ├─ 1️⃣ View Dashboard ──────────► dashBoard() ──► Query all data
       │
       ├─ 2️⃣ Browse Products ────────► products() ──► List all products
       │
       ├─ 3️⃣ View Customer ─────────► customer() ──► Customer + Orders
       │
       ├─ 4️⃣ Create Order ─────────► createOrder() ──► Form + Save
       │
       ├─ 5️⃣ Update Order ────────► updateOrder() ──► Form + Update
       │
       └─ 6️⃣ Delete Order ────────► deleteOrder() ──► Confirm + Delete
```

---

## 4. Application Architecture

### MVC (Model-View-Control) Pattern

```
┌────────────────────────────────────────────────────────────┐
│                   DJANGO MVC ARCHITECTURE                  │
└────────────────────────────────────────────────────────────┘

┌──────────────────┐
│    MODEL LAYER   │
│   (models.py)    │
│                  │
│ ┌──────────────┐ │
│ │ Customer     │ │
│ │ Product      │ │
│ │ Order        │ │
│ └──────────────┘ │
│        ▲         │
│        │         │
└────────┼─────────┘
         │ Define Data
         │ Structure
         ▼
┌────────────────────────────────────────────────────────────┐
│    VIEW LAYER                                              │
│   (views.py)                                               │
│                                                             │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐          │
│  │ dashBoard  │  │ products   │  │ customer   │          │
│  └────────────┘  └────────────┘  └────────────┘          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐          │
│  │createOrder │  │updateOrder │  │deleteOrder │          │
│  └────────────┘  └────────────┘  └────────────┘          │
│                                                             │
│  Responsibilities:                                         │
│  • Fetch data from models                                 │
│  • Process user requests                                  │
│  • Pass data to templates                                 │
└────────────────────────────────────────────────────────────┘
         │                              │
         │ Retrieve Data                │ Pass Context
         │                              │
         ▼                              ▼
┌────────────────────────────────────────────────────────────┐
│    CONTROLLER LAYER                                        │
│   (urls.py - URL Routing)                                 │
│                                                             │
│  http://localhost/ ──────► dashBoard view                │
│  http://localhost/products ──► products view             │
│  http://localhost/customer/1 ──► customer view           │
│  http://localhost/create_order ──► createOrder view      │
└────────────────────────────────────────────────────────────┘
         │                              │
         │ HTTP Request                 │ Route Mapping
         │                              │
         ▼                              ▼
┌────────────────────────────────────────────────────────────┐
│    TEMPLATE LAYER (Presentation)                           │
│   (templates/accounts/*.html)                              │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │dashboard.html│  │products.html │  │customer.html │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │order_form... │  │delete_item...│  │navbar.html   │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
│                                                             │
│  Responsibilities:                                         │
│  • Render HTML pages                                      │
│  • Display model data                                     │
│  • Handle user forms                                      │
└────────────────────────────────────────────────────────────┘
         ▲                              │
         │ Context Data                 │ HTML Response
         │                              │
         └──────────────────────────────►
                                        ▼
                                   WEB BROWSER
```

### Layer Interaction Sequence

```
Request → Router → View → Model → Template → Response

1. User clicks link or submits form
2. Django URL Router (urls.py) matches the request to a view
3. View Function (views.py) is executed
4. View queries the Model (models.py) for data
5. Model executes database queries via ORM
6. Database returns data as Model instances
7. View passes data to Template (templates/*.html)
8. Template renders HTML with the data
9. HTML response sent back to user's browser
```

---

## 5. Request/Response Flow

### Complete Request Lifecycle

```
┌──────────────────────────────────────────────────────────────┐
│                  USER INITIATES REQUEST                      │
│          (Click link, Submit form, Type URL)                 │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
        ┌─────────────────────────────────┐
        │  1. HTTP Request Generated      │
        │  GET /customer/1/ HTTP/1.1      │
        │  Host: localhost:8000           │
        │  Cookie: sessionid=...          │
        └──────────────┬──────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  2. Django URL Resolver              │
        │  (urls.py) Matches URL Pattern       │
        │                                      │
        │  path('customer/<str:pk>/',          │
        │        views.customer, ...)          │
        │                                      │
        │  Extracts: pk=1                      │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  3. View Function Invoked            │
        │  customer(request, pk='1')           │
        │  Location: accounts/views.py         │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  4. Database Queries via ORM         │
        │                                      │
        │  customer = Customer.objects.get     │
        │              (id=pk)                 │
        │  orders = customer.order_set.all()   │
        │                                      │
        │  ↓ Converts to SQL                   │
        │  SELECT * FROM accounts_customer    │
        │  WHERE id = 1;                       │
        │                                      │
        │  SELECT * FROM accounts_order       │
        │  WHERE customer_id = 1;              │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  5. Database Executes Queries        │
        │  SQLite (db.sqlite3)                 │
        │                                      │
        │  Returns row data                    │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  6. ORM Creates Python Objects       │
        │  customer = Customer instance        │
        │  orders = [Order, Order, Order]      │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  7. View Prepares Context            │
        │                                      │
        │  context = {                         │
        │    'customer': customer,             │
        │    'orders': orders,                 │
        │    'total_orders': 5,                │
        │    'filter': orderFilter             │
        │  }                                   │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  8. Template Rendering               │
        │                                      │
        │  render(request,                     │
        │         'accounts/customer.html',    │
        │         context)                     │
        │                                      │
        │  Template injects context data:      │
        │  <h1>{{ customer.name }}</h1>        │
        │  {% for order in orders %}           │
        │    <tr><td>{{ order.id }}</td>...    │
        │  {% endfor %}                        │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  9. HTML Generated                   │
        │                                      │
        │  <!DOCTYPE html>                     │
        │  <html>...                           │
        │  <h1>John Doe</h1>                   │
        │  <table>                             │
        │    <tr><td>Order 1</td>...</tr>      │
        │  </table>                            │
        │  </html>                             │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  10. HTTP Response Sent              │
        │  HTTP/1.1 200 OK                     │
        │  Content-Type: text/html             │
        │  Content-Length: 5432                │
        │                                      │
        │  [HTML body]                         │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  11. Browser Renders Page            │
        │                                      │
        │  Display HTML in browser             │
        │  Load CSS, images                    │
        │  Execute JavaScript                 │
        └──────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────┐
│                  USER SEES PAGE                              │
└──────────────────────────────────────────────────────────────┘
```

### Form Submission Flow (POST Request)

```
┌──────────────────────────────────────────────────────────────┐
│            USER SUBMITS FORM (create_order/)                │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
        ┌──────────────────────────────────────┐
        │  1. Form Data Encoded                │
        │  POST /create_order/ HTTP/1.1        │
        │  Content-Type: application/x-www...  │
        │                                      │
        │  customer_id=5&product_id=3&         │
        │  status=Pending&csrftoken=abc123     │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  2. Django Middleware Processing     │
        │  - CsrfViewMiddleware: Validate CSRF │
        │  - SessionMiddleware: Load session   │
        │  - AuthenticationMiddleware: User    │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  3. URL Routing                      │
        │  Matches: path('create_order/',      │
        │           views.createOrder)         │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  4. View Function Execution          │
        │  createOrder(request)                │
        │                                      │
        │  if request.method == 'POST':        │
        │    form = OrderForm(request.POST)    │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  5. Form Validation                  │
        │  if form.is_valid():                 │
        │    - Check required fields           │
        │    - Validate field types            │
        │    - Check ForeignKey exists         │
        │                                      │
        │    Outcome: VALID or INVALID         │
        └──────────────┬───────────────────────┘
                       │
                 ┌─────┴──────┐
                 │            │
            VALID         INVALID
                 │            │
                 ▼            ▼
        ┌──────────────┐  ┌────────────────┐
        │ 6a. Save     │  │ 6b. Return Form│
        │ to Database  │  │ with Errors    │
        │              │  │                │
        │ form.save()  │  │ Render         │
        │              │  │ template with  │
        │ INSERT INTO  │  │ error messages │
        │ accounts...  │  └────────────────┘
        │              │
        │ 7a. Redirect │
        │ return       │
        │ redirect('/')│
        └──────────────┘
                │
                ▼
        ┌──────────────────────────────────────┐
        │  HTTP Response Sent                  │
        │  302 Found (Redirect)                │
        │  Location: /                         │
        └──────────────┬───────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────────┐
        │  Browser Follows Redirect            │
        │  New GET request to /                │
        └──────────────┬───────────────────────┘
                       │
                       ▼
                  Dashboard View
                   (Success)
```

---

## 6. Database Schema

### SQLite Table Structure

```
SQLite Database: db.sqlite3
│
├─ accounts_customer
│  ├─ Column: id (INTEGER PRIMARY KEY)
│  ├─ Column: name (VARCHAR(200))
│  ├─ Column: phone (VARCHAR(200))
│  ├─ Column: email (VARCHAR(200))
│  └─ Column: date_created (DATETIME)
│
├─ accounts_product
│  ├─ Column: id (INTEGER PRIMARY KEY)
│  ├─ Column: name (VARCHAR(200))
│  ├─ Column: price (REAL)
│  ├─ Column: category (VARCHAR(200))
│  ├─ Column: description (TEXT)
│  └─ Column: date_created (DATETIME)
│
├─ accounts_order
│  ├─ Column: id (INTEGER PRIMARY KEY)
│  ├─ Column: customer_id (INTEGER FOREIGN KEY, NULLABLE)
│  │           └─ References: accounts_customer.id
│  ├─ Column: product_id (INTEGER FOREIGN KEY, NULLABLE)
│  │           └─ References: accounts_product.id
│  ├─ Column: date_created (DATETIME)
│  └─ Column: status (VARCHAR(200))
│
└─ [Django System Tables]
   ├─ auth_user (User management)
   ├─ auth_group (User groups/permissions)
   ├─ django_session (Session storage)
   └─ ... (other Django tables)
```

### Sample Data Example

```
accounts_customer
┌─────┬──────────┬──────────────┬──────────────────┬─────────────────┐
│ id  │ name     │ phone        │ email            │ date_created    │
├─────┼──────────┼──────────────┼──────────────────┼─────────────────┤
│ 1   │ John Doe │ 555-1234     │ john@example.com │ 2024-01-15...   │
│ 2   │ Jane Doe │ 555-5678     │ jane@example.com │ 2024-01-16...   │
│ 3   │ Bob Smith│ 555-9012     │ bob@example.com  │ 2024-01-17...   │
└─────┴──────────┴──────────────┴──────────────────┴─────────────────┘

accounts_product
┌─────┬─────────────┬────────┬──────────┬──────────────────┬─────────────────┐
│ id  │ name        │ price  │ category │ description      │ date_created    │
├─────┼─────────────┼────────┼──────────┼──────────────────┼─────────────────┤
│ 1   │ Indoor Lamp │ 29.99  │ Indoor   │ LED desk lamp... │ 2024-01-10...   │
│ 2   │ Garden Hose │ 19.99  │ Out Door │ 50ft rubber...   │ 2024-01-11...   │
│ 3   │ Office Chair│ 149.99 │ Indoor   │ Ergonomic chair..│ 2024-01-12...   │
└─────┴─────────────┴────────┴──────────┴──────────────────┴─────────────────┘

accounts_order
┌─────┬──────────────┬────────────┬──────────────────┬─────────────────┐
│ id  │ customer_id  │ product_id │ status           │ date_created    │
├─────┼──────────────┼────────────┼──────────────────┼─────────────────┤
│ 1   │ 1            │ 1          │ Delivered        │ 2024-01-20...   │
│ 2   │ 1            │ 3          │ Out for delivery │ 2024-01-25...   │
│ 3   │ 2            │ 2          │ Pending          │ 2024-02-01...   │
│ 4   │ 3            │ 1          │ Delivered        │ 2024-02-05...   │
└─────┴──────────────┴────────────┴──────────────────┴─────────────────┘
```

---

## Summary

This diagram set provides:

1. **ERD (Entity Relationship Diagram)**: Shows how Customer, Product, and Order tables relate to each other
2. **DFD (Data Flow Diagram)**: Shows how data flows through the application from browser to database and back
3. **Context Diagram**: Shows the application as a black box and its interactions with external users
4. **Architecture**: Shows the MVC pattern and layer separation
5. **Request/Response Flow**: Detailed step-by-step lifecycle of HTTP requests
6. **Database Schema**: The actual database structure and sample data

These diagrams help understand how the CRM system works at different levels of abstraction.
