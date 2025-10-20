# **MySQL-coding-question**


### Question 1: Total Sales Revenue by Product
SELECT  p.id AS product_id,p.name AS product_name,
    SUM(oi.quantity * oi.price) AS total_revenue
    FROM products p
    JOIN order_items oi ON p.id = oi.product_id
    GROUP BY p.id, p.name
    ORDER BY total_revenue DESC;

## Explanation

oi.quantity * oi.price gives the total price per item.

We SUM() them per product.

Sorting by total_revenue DESC gives highest-selling products first.


### Question 2: Top Customers by Spending
SELECT  c.id AS customer_id,c.name AS customer_name,
    SUM(oi.quantity * oi.price) AS total_spent
    FROM customers c
    JOIN orders o ON c.id = o.customer_id
    JOIN order_items oi ON o.id = oi.order_id
    GROUP BY c.id, c.name
    ORDER BY total_spent DESC
    LIMIT 5;

## Explanation

Joins customers → orders → order_items.

Calculates each customer’s total spend.

Sorted by top spenders and limited to 5.


### Question 3: Average Order Value per Customer
SELECT  c.id AS customer_id,c.name AS customer_name,
    (SUM(oi.quantity * oi.price) / COUNT(DISTINCT o.id)) AS avg_order_value
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
GROUP BY c.id, c.name
HAVING COUNT(DISTINCT o.id) > 0
ORDER BY avg_order_value DESC;


## Explanation
Total spending ÷ number of orders = average order value.

HAVING ensures we only include customers who have at least one order.


### Question 4: Recent Orders (Last 30 Days)
SELECT o.id AS order_id,c.name AS customer_name,o.order_date,o.status AS order_status
    FROM orders o
    JOIN customers c ON o.customer_id = c.id
    WHERE o.order_date >= DATE_SUB(NOW(), INTERVAL 30 DAY)
    ORDER BY o.order_date DESC;

## Explanation
Uses DATE_SUB(NOW(), INTERVAL 30 DAY) to filter recent orders.

Shows order and customer details.

### Question 5: Running Total of Customer Spending
WITH customer_orders AS (
    SELECT 
        o.id AS order_id,
        o.customer_id,
        o.order_date,
        SUM(oi.quantity * oi.price) AS order_total
    FROM orders o
    JOIN order_items oi ON o.id = oi.order_id
    GROUP BY o.id, o.customer_id, o.order_date
)
SELECT 
    co.customer_id,
    co.order_id,
    co.order_date,
    co.order_total,
    SUM(co.order_total) OVER (
        PARTITION BY co.customer_id 
        ORDER BY co.order_date
    ) AS running_total
FROM customer_orders co
ORDER BY co.customer_id, co.order_date;


## Explanation

The CTE (customer_orders) calculates each order’s total.

SUM() OVER (PARTITION BY ...) creates a running total per customer over time.


### Question 6: Product Review Summary
SELECT 
    p.id AS product_id,
    p.name AS product_name,
    AVG(r.rating) AS avg_rating,
    COUNT(r.id) AS total_reviews
FROM products p
LEFT JOIN reviews r ON p.id = r.product_id
GROUP BY p.id, p.name
ORDER BY avg_rating DESC, total_reviews DESC;


## Explanation
LEFT JOIN ensures products with no reviews still appear.

Uses AVG(r.rating) and COUNT(r.id) to summarize.

### Question 7: Customers Without Orders
SELECT 
    c.id AS customer_id,
    c.name AS customer_name
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;

## Explanation
LEFT JOIN + WHERE o.id IS NULL → finds customers who have never placed orders.


### Question 8: Update Last Purchased Date
UPDATE products p
SET p.last_purchased = (
    SELECT MAX(o.order_date)
    FROM order_items oi
    JOIN orders o ON oi.order_id = o.id
    WHERE oi.product_id = p.id
);


## Explanation
Subquery gets the latest order date for each product.

Updates last_purchased field accordingly.


### Question 9: Transaction Scenario
START TRANSACTION;

-- 1. Deduct stock
UPDATE products 
SET stock = stock - 2 
WHERE id = 1;

-- 2. Insert into orders
INSERT INTO orders (customer_id, order_date, status) 
VALUES (3, NOW(), 'Placed');

SET @order_id = LAST_INSERT_ID();

-- 3. Insert into order_items
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (@order_id, 1, 2, 499.00);

-- 4. Update last_purchased
UPDATE products 
SET last_purchased = NOW()
WHERE id = 1;

COMMIT;


## Explanation
Uses transaction control.

If any step fails → use ROLLBACK;

Ensures atomicity (all or nothing).


### Question 10: Query Optimization & Indexing
Suppose we optimize Question 2 (Top Customers by Spending).

Using EXPLAIN:
EXPLAIN SELECT 
    c.id, c.name, SUM(oi.quantity * oi.price)
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
GROUP BY c.id, c.name;

Optimization Ideas:
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);

Avoid unnecessary DISTINCT or subqueries.

Use proper JOINs instead of nested IN clauses.

### Question 11: Query Optimization Challenge

Unoptimized Query:
Uses nested subqueries and IN, which is slow for large datasets.

Optimized Query:
SELECT 
    c.id AS customer_id,
    c.name,
    SUM(oi.quantity * oi.price) AS total_spent
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
GROUP BY c.id, c.name
ORDER BY total_spent DESC;

