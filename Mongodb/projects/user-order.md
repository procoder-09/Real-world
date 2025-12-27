Great! Let’s design a **User-Order system** that uses **PostgreSQL** for relational data and **MongoDB** for document-based data. I’ll give a detailed, practical plan including **schema design, use cases, queries, and integration ideas**.

---

## **1. Project Overview**

* **Goal:** Build a system to manage users and their orders.
* **Features:**

  * User registration and management
  * Create and track orders
  * Store structured relational data in **PostgreSQL**
  * Store flexible/document-oriented data in **MongoDB**
* **Tech Stack:**

  * Backend: Python (FastAPI) / Node.js (Express)
  * PostgreSQL: Relational data (normalized)
  * MongoDB: Flexible, nested order data, analytics, history

---

## **2. Database Design**

### **A) PostgreSQL (Relational)**

Use PostgreSQL for **core structured data**: users, products, relationships, transactions.

**Tables:**

#### **1. Users Table**

```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### **2. Products Table**

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price NUMERIC(10,2) NOT NULL,
    stock INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### **3. Orders Table**

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(user_id),
    total_amount NUMERIC(10,2) NOT NULL,
    status VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### **4. Order Items Table**

```sql
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id INT REFERENCES products(product_id),
    quantity INT NOT NULL,
    price NUMERIC(10,2) NOT NULL
);
```

✅ **Why PostgreSQL?**

* Relationships: Users → Orders → Order Items
* Ensures **data integrity**
* Supports **joins, transactions**

---

### **B) MongoDB (Document-based)**

Use MongoDB for **order details, logs, analytics, or flexible attributes**.

**Collection:** `order_history`

```json
{
  "_id": ObjectId("..."),
  "order_id": 101,
  "user_id": 1,
  "products": [
    { "product_id": 1, "name": "Laptop", "price": 1000, "quantity": 1 },
    { "product_id": 2, "name": "Mouse", "price": 50, "quantity": 2 }
  ],
  "total_amount": 1100,
  "status": "pending",
  "created_at": ISODate("2025-12-27T08:00:00Z"),
  "events": [
    { "status": "pending", "timestamp": ISODate("2025-12-27T08:00:00Z") },
    { "status": "shipped", "timestamp": ISODate("2025-12-28T10:00:00Z") }
  ]
}
```

✅ **Why MongoDB?**

* Flexible schema for order logs, events, and nested arrays
* Faster read/write for **analytics and history**
* No need for strict relational constraints

---

## **3. Integration Strategy**

### **Workflow**

1. **User creates an order**

   * Insert order in **PostgreSQL** (`orders` + `order_items`)
   * Insert **order details** in MongoDB (`order_history`) for logs

2. **Update order status**

   * Update PostgreSQL for transactional status
   * Push a new event in MongoDB for history

3. **Analytics**

   * Aggregate orders in MongoDB by user, product, or status
   * Example: total orders by product last month

---

### **4. Example Queries**

#### **PostgreSQL Queries**

* Create a new order:

```sql
BEGIN;

INSERT INTO orders(user_id, total_amount) VALUES (1, 150) RETURNING order_id;

INSERT INTO order_items(order_id, product_id, quantity, price)
VALUES (101, 1, 2, 50), (101, 2, 1, 50);

COMMIT;
```

* Get all orders of a user:

```sql
SELECT o.order_id, o.total_amount, o.status, o.created_at,
       json_agg(json_build_object('product_id', oi.product_id, 'quantity', oi.quantity, 'price', oi.price)) AS items
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.user_id = 1
GROUP BY o.order_id;
```

---

#### **MongoDB Queries**

* Insert order history:

```js
db.order_history.insertOne({
  order_id: 101,
  user_id: 1,
  products: [
    { product_id: 1, name: "Laptop", price: 1000, quantity: 1 },
    { product_id: 2, name: "Mouse", price: 50, quantity: 2 }
  ],
  total_amount: 1100,
  status: "pending",
  created_at: new Date(),
  events: [{ status: "pending", timestamp: new Date() }]
});
```

* Update order status and log event:

```js
db.order_history.updateOne(
  { order_id: 101 },
  {
    $set: { status: "shipped" },
    $push: { events: { status: "shipped", timestamp: new Date() } }
  }
);
```

* Aggregate total spent by a user:

```js
db.order_history.aggregate([
  { $match: { user_id: 1 } },
  { $group: { _id: "$user_id", totalSpent: { $sum: "$total_amount" } } }
]);
```

---

## **5. Best Practices**

1. **Use PostgreSQL for transactional data**

   * Ensures ACID compliance for orders and users
2. **Use MongoDB for flexible/nested data**

   * Ideal for order history, events, logs, analytics
3. **Keep MongoDB in sync with PostgreSQL**

   * Use **application-level sync** or **change streams**
4. **Indexing**

   * PostgreSQL: `user_id`, `order_id` for fast joins
   * MongoDB: `order_id`, `user_id`, `status` for queries
5. **Data archiving**

   * Old order history can stay in MongoDB, while PostgreSQL keeps recent transactions

---

## **6. Tech Diagram**

```
      Frontend
         |
      Backend (FastAPI/Express)
         |
    -----------------------
    |                     |
PostgreSQL             MongoDB
(users, orders)       (order_history, logs)
```

* Backend handles syncing and queries across both databases.
* PostgreSQL for **reliable transactions**, MongoDB for **flexible analytics**.

---

✅ This system combines the **strengths of both relational and document-based databases**, making it scalable and maintainable.

---

If you want, I can also **write a ready-to-run backend structure** for this project in **FastAPI + PostgreSQL + MongoDB**, with models, APIs for creating orders, fetching history, and updating status.

Do you want me to do that?
