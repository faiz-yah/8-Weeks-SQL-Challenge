# Case Study #1 - Danny's Diner

The [link](https://8weeksqlchallenge.com/case-study-1/) to the challenge.

My choice of DBMS was **PostgreSQL.**

## My solutions

### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT customer_id, SUM(price) AS Price
FROM dannys_diner.sales 
JOIN dannys_diner.menu 
ON sales.product_id = menu.product_id
GROUP BY customer_id
ORDER BY customer_id ASC
````

### 2.How many days has each customer visited the restaurant?

````sql
SELECT customer_id, COUNT(order_date)
FROM dannys_diner.sales 
GROUP BY customer_id
````

### 3. What was the first item from the menu purchased by each customer?
````sql
WITH CTE_table AS (
  SELECT
  sales.customer_id, 
  sales.product_id, 
  product_name, 
  order_date,
  DENSE_RANK() OVER(
    PARTITION BY sales.customer_id
    ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
)

SELECT customer_id, product_name, order_date
FROM CTE_table_2
WHERE rank = 1
GROUP BY customer_id, product_name, order_date
````

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
SELECT menu.product_name , count(sales.product_id) AS amount
FROM dannys_diner.sales
JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY menu.product_name, sales.product_id
ORDER BY amount DESC
LIMIT 1
````

### 5.Which item was the most popular for each customer?
````sql
WITH CTE_table AS (
SELECT s.customer_id, s.product_id, m.product_name, count(s.product_id) AS order_count,
DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY count(s.product_id) desc) AS rank
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m
ON s.product_id = m.product_id
GROUP BY s.customer_id, s.product_id, m.product_name)


SELECT customer_id, product_name, order_count, rank
FROM CTE_table
WHERE rank = 1
````

### 6. Which item was purchased first by the customer after they became a member?
````sql
WITH CTE_table AS (
SELECT mem.customer_id, sal.order_date, sal.product_id,
DENSE_RANK() OVER(PARTITION BY mem.customer_id ORDER BY order_date) AS rank
FROM dannys_diner.members mem
JOIN dannys_diner.sales sal
ON mem.customer_id = sal.customer_id
WHERE order_date >= join_date)


SELECT customer_id, order_date, product_name
FROM CTE_table as t
JOIN dannys_diner.menu as m
ON t.product_id = m.product_id
WHERE rank = 1
ORDER BY customer_id
````

### 7. Which item was purchased just before the customer became a member?
````sql
WITH CTE_table AS (

SELECT mem.customer_id, order_date, product_id,
DENSE_RANK() OVER(PARTITION BY mem.customer_id ORDER BY order_date DESC) AS rank
FROM dannys_diner.sales sal
JOIN dannys_diner.members mem
ON sal.customer_id = mem.customer_id
WHERE order_date <= join_date)


SELECT customer_id, order_date, product_name, t.product_id
FROM CTE_table as t
JOIN dannys_diner.menu as m
ON t.product_id = m.product_id
WHERE rank = 1
ORDER BY customer_id

````

### 8. What is the total items and amount spent for each member before they became a member?
````sql
SELECT s.customer_id, count(s.product_id) as total_items, sum(price) as total_amount
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON s.product_id = m.product_id
JOIN dannys_diner.members mem
ON s.customer_id = mem.customer_id
AND order_date < join_date
GROUP BY s.customer_id
ORDER BY customer_id
````
