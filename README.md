
# Retail Database Project (PostgreSQL-DBeaver)

This project contains a complete relational database for a retail business, including schema creation, synthetic datasets, loading scripts, an ERD diagram, and few analytical SQL queries.  
It is designed as a data engineering and SQL analytics portfolio project.

---

## 📁 Project Structure

- Schema/  → SQL scripts to create tables and relationships.
- Data/ → Synthetic CSV datasets (1000+ rows).
- Scripts/ → Data loading script + analytical SQL.
- Diagrams/ → ER diagrams in Mermaid and PNG.


---

## 🗄️ Database Overview

The database models a simple retail company with:

- **Customers**
- **Stores**
- **Employees**
- **Products**
- **Orders**
- **Order Items**

The schema supports:
- Sales analytics  
- Churn & retention analysis  
- Customer lifetime value (LTV) analysis  
- Store and employee performance  



## 🛠️ Tech Stack

- PostgreSQL (pgAdmin 4)
- Mermaid (for diagrams)
- SQL (data modeling + analytics)



## 🧱 Tables and relationships

### Create_tables.sql

```sql
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(120) UNIQUE,
    phone VARCHAR(20),
    city VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE stores (
    store_id SERIAL PRIMARY KEY,
    store_name VARCHAR(100),
    city VARCHAR(100),
    opened_at DATE
);

CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    store_id INT REFERENCES stores(store_id),
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    hire_date DATE
);

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price NUMERIC(10,2)
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    store_id INT REFERENCES stores(store_id),
    employee_id INT REFERENCES employees(employee_id),
    order_date TIMESTAMP DEFAULT NOW()
);

CREATE TABLE order_items (
    item_id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(order_id),
    product_id INT REFERENCES products(product_id),
    quantity INT,
    unit_price NUMERIC(10,2)
);
```

### Constraints_indexes.sql

```sql
-- Useful indexes
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_store ON orders(store_id);
CREATE INDEX idx_order_items_product ON order_items(product_id);
CREATE INDEX idx_customers_email ON customers(email);
```


## 📥 Load_data.sql
'C:/postgres_projects/retail_db/data/

```sql
COPY customers FROM 'C:/postgres_projects/retail_db/data/customers.csv' CSV HEADER;
COPY stores FROM 'C:/postgres_projects/retail_db/data/stores.csv' CSV HEADER;
COPY employees FROM ''C:/postgres_projects/retail_db/data/employees.csv' CSV HEADER;
COPY products FROM 'C:/postgres_projects/retail_db/data/products.csv' CSV HEADER;
COPY orders FROM 'C:/postgres_projects/retail_db/data/orders.csv' CSV HEADER;
COPY order_items FROM 'C:/postgres_projects/retail_db/data/order_items.csv' CSV HEADER;
```

## 📈 Analysis_queries.sql

```sql
-- Revenue by store
SELECT s.store_name, SUM(oi.quantity * oi.unit_price) AS revenue
FROM order_items oi
JOIN orders o ON oi.order_id = o.order_id
JOIN stores s ON o.store_id = s.store_id
GROUP BY s.store_name
ORDER BY revenue DESC;

-- Top customers by lifetime value
SELECT c.customer_id, c.first_name, c.last_name,
       SUM(oi.quantity * oi.unit_price) AS ltv
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id
ORDER BY ltv DESC
LIMIT 20;

-- Monthly sales
SELECT DATE_TRUNC('month', order_date) AS month,
       SUM(oi.quantity * oi.unit_price) AS revenue
FROM order_items oi
JOIN orders o ON o.order_id = oi.order_id
GROUP BY month
ORDER BY month;

-- Employee performance
SELECT e.employee_id, e.first_name, e.last_name,
       COUNT(o.order_id) AS total_orders
FROM employees e
LEFT JOIN orders o ON o.employee_id = e.employee_id
GROUP BY e.employee_id
ORDER BY total_orders DESC;
```

## 📊 diagrams/erd

<img src="/images/Logical_model.png" alt="ER Diagram" style="width: 50%; height: auto;">