# Case Study #1 - Danny's Diner
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## Problem Summary
Danny, a lover of Japanese food, opened a restaurant called Danny's Diner in 2021, specializing in sushi, curry, and ramen. However, the restaurant lacks the expertise to leverage its basic operational data effectively. They need assistance in analyzing customer patterns, spending habits, and menu preferences to enhance customer experience and potentially expand their loyalty program. Danny provided sample datasets including `sales`, `menu`, and `members` information for analysis, hoping to derive insights and optimize business decisions.

### SQL Tools Demonstrated
- Joins
- Aggregate functions
- Window functions
- Subqueries

*All information regarding this case study can be found [here](https://8weeksqlchallenge.com/case-study-1/).*

## Creating Our Database Schema
Upon opening a new SQL script on MySQL Workbench, we will create the `dannys_diner` database along with the `sales`, `menu`, and `members` tables.

```sql
CREATE DATABASE dannys_diner;
USE dannys_diner;

CREATE TABLE members (
    customer_id VARCHAR(1),
    join_date TIMESTAMP
);

CREATE TABLE menu (
    product_id INT,
    product_name VARCHAR(5),
    price INT NOT NULL
);

CREATE TABLE sales (
    customer_id VARCHAR(1),
    order_date DATE,
    product_id INT
);
```

Then, we will populate these tables using the datasets Danny provided.

```sql
INSERT INTO members
  (customer_id, join_date)
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');

INSERT INTO menu
  (product_id, product_name, price)
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');

INSERT INTO sales
(customer_id, order_date, product_id)
VALUES
  ('A', '2021-01-01', 1),
  ('A', '2021-01-01', 2),
  ('A', '2021-01-07', 2),
  ('A', '2021-01-10', 3),
  ('A', '2021-01-11', 3),
  ('A', '2021-01-11', 3),
  ('B', '2021-01-01', 2),
  ('B', '2021-01-02', 2),
  ('B', '2021-01-04', 1),
  ('B', '2021-01-11', 1),
  ('B', '2021-01-16', 3),
  ('B', '2021-02-01', 3),
  ('C', '2021-01-01', 3),
  ('C', '2021-01-01', 3),
  ('C', '2021-01-07', 3);
```

## Case Study Questions with Solutions
1. What is the total amount each customer spent at the restaurant?
```sql
SELECT
  customer_id,
  SUM(price) AS total_spent
FROM
  sales
JOIN
  menu ON sales.product_id = menu.product_id
GROUP BY
  customer_id;
```

2. How many days has each customer visited the restaurant?

3. What was the first item from the menu purchased by each customer?

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

5. Which item was the most popular for each customer?

6. Which item was purchased first by the customer after they became a member?

7. Which item was purchased just before the customer became a member?

8. What is the total items and amount spent for each member before they became a member?

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
