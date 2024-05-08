# Case Study #1 - Danny's Diner
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## Problem Summary
Danny, a lover of Japanese food, opened a restaurant called Danny's Diner in 2021, specializing in sushi, curry, and ramen. However, the restaurant lacks the expertise to leverage its basic operational data effectively. They need assistance in analyzing customer patterns, spending habits, and menu preferences to enhance customer experience and potentially expand their loyalty program. Danny provided sample datasets including `sales`, `menu`, and `members` information for analysis, hoping to derive insights and optimize business decisions.

### SQL Tools Demonstrated
*I want to preface by saying that I had not known about CTE's while doing this case study!* :sweat_smile:
- Joins
- Aggregate functions
- Window functions
- Subqueries
- Common table expressions

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
### 1. What is the total amount each customer spent at the restaurant?
**Answer:**
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

**Steps:**
- Use **JOIN** to combine rows from the `sales` table with matching rows from the `menu` table based on `product_id` in order to obtain price information for each sale.
- Use the **GROUP BY** clause to group the result set by `customer_id`.
- Within the **SELECT** statement, use the `SUM()` function to aggregate the price column, adding up the prices of all sales transactions for each customer.

**Output:**
| customer_id | total_spent |
|-------------|-------------|
| A | 76 |
| B | 74 |
| C | 36 |

Customer A has spent a total of 76 dollars, customer B has spent a total of 74 dollars, and customer C has spent a total of 36 dollars.

### 2. How many days has each customer visited the restaurant?
**Answer:**
```sql
SELECT
    customer_id,
    COUNT(DISTINCT order_date) AS visits_in_days
FROM
    sales
GROUP BY
    customer_id;
```

**Steps:**
- Use the **GROUP BY** clause to group the result set by `customer_id`.
- Within the **SELECT** statement, use the `COUNT()` function to count the unique order dates for each customer.

**Output:**
| customer_id | visits_in_days |
|-------------|-------------|
| A | 4 |
| B | 6 |
| C | 2 |

Customer A has made 4 visits, customer B has made 6 visits, and customer C has made 2 visits.

### 3. What was the first item from the menu purchased by each customer?
**Answer:**
```sql
SELECT
    ranked_items.customer_id,
    menu.product_name
FROM (
    SELECT
        *,
        RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS rnk
    FROM
        sales
    ) AS ranked_items
JOIN
    menu ON menu.product_id = ranked_items.product_id
WHERE
    rnk = 1;
```

**Steps:**
- Within the **FROM** clause, the subquery uses the window function `RANK()` to order each item each customer ordered by date from the `sales` table.
- Use **JOIN** to combine rows from the `menu` table onto the subquery (aliased as "ranked_items") based on `product_id`. This will give us the name of the food for each order.
- Use the **WHERE** clause to filter the results such that we obtain the first ever order for each customer.

**Output:**
| customer_id | product_name |
|-------------|-------------|
| A | sushi |
| A | curry |
| B | curry |
| C | ramen |
| C | ramen |

Customer A's first order was both sushi and curry, customer B's first order was curry, and customer C's first order was two bowls of ramen.

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
**Answer:**
```sql
SELECT
    menu.product_name,
    COUNT(sales.product_id) AS times_purchased
FROM
    sales
JOIN
    menu ON menu.product_id = sales.product_id
GROUP BY
    menu.product_name
ORDER BY
    times_purchased DESC
;
```

**Steps:**
- Use **JOIN** to combine rows from the `menu` table onto `sales` based on `product_id` in order to obtain the name of the food for each order.
- Use the aggregate function `COUNT()` to count the number of sales, then use the **GROUP BY** clause to group the result set by `product_name`.
- Use the **ORDER BY** clause along with the keyword **DESC** to specify the sorting order. We will want the product with the highest number on sales at the very top.

**Output:**
| product_name | times_purchased |
|-------------|-------------|
| ramen | 8 |
| curry | 4 |
| sushi | 3 |

Customers had ordered ramen 8 times, which makes it the most purchased item on the menu.

### 5. Which item was the most popular for each customer?
**Answer:**
```sql
WITH ranked_products AS (
    SELECT
        s.customer_id,
        s.product_name,
        RANK() OVER(PARTITION BY s.customer_id ORDER BY s.times_purchased) AS rnk
    FROM (
        SELECT
            sales.customer_id,
            menu.product_name,
            COUNT(menu.product_name) AS times_purchased
        FROM
            sales
        JOIN
            menu ON menu.product_id = sales.product_id
        GROUP BY
            sales.customer_id,
            menu.product_name
        ORDER BY
            sales.customer_id,
            times_purchased DESC
    ) AS s
)
SELECT
    *
FROM
    ranked_products
WHERE
    rnk = 1;
```

**Steps:**
- Within a common table expression, use the window function `RANK()` to order the menu items by the number of times purchased for each customer.
    - The subquery in the **FROM** clause, which will use **GROUP BY** to group the CTE by `customer_id` and `product_name`, allows us to count how many of each item was purchased by each customer.
- In the outer query, use the **WHERE** clause to filter the results to determine which menu item was most purchased for each customer.

**Output:**
| customer_id | product_name | rnk |
|-------------|-------------|-------------|
| A | sushi | 1 |
| B | sushi | 1 |
| B | curry | 1 |
| B | ramen | 1 |
| C | ramen | 1 |

For customer A, sushi was their most popular item. For customer B, all three menu items were equally as popular. For customer C, ramen was their most popular item.

### 6. Which item was purchased first by the customer after they became a member?

### 7. Which item was purchased just before the customer became a member?

### 8. What is the total items and amount spent for each member before they became a member?

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
