# Case Study #2 - Pizza Runner
<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">

## Table of Contents
Since this particular case study is quite lengthy, I will provide a table of contents to quickly navigate to different sections!

- [Problem Summary](##problem-summary)
- [Creating Our Database Schema](#creating-our-database-schema)

<a name="problem-summary"></a>
## Problem Summary
Danny has launched Pizza Runner, aiming to revolutionize pizza delivery by combining the 80s retro style with the Uber model. He's recruited "runners" to deliver pizzas and developed a mobile app for ordering. Now, he needs help cleaning his data and applying calculations to optimize operations. We'll be working with datasets within the `pizza_runner` database schema to tackle this challenge.

### SQL Topics Covered:
- Joins
- Aggregate functions
- Data cleaning (string functions, handling NULL values)
- Common table expressions
- Temporary tables

*All information regarding this case study can be found [here](https://8weeksqlchallenge.com/case-study-2/).*

<a name="creating-schema"></a>
## Creating Our Database Schema
Upon opening a new SQL script on MySQL Workbench, we will create the `pizza_runner` database along with the `runners`, `customer_orders`,  `runner_orders`, `pizza_names`, `pizza_recipes` and `pizza_toppings` tables. Then we will populate the tables with the data that Danny has provided us.

```sql
CREATE DATABASE pizza_runner;
USE pizza_runner;

-- Create the runners table
CREATE TABLE runners (
  runner_id INTEGER,
  registration_date DATE
);

-- Insert data into the runners table
INSERT INTO runners (runner_id, registration_date)
VALUES
  (1, '2021-01-01'),
  (2, '2021-01-03'),
  (3, '2021-01-08'),
  (4, '2021-01-15');

-- Create the customer_orders table
CREATE TABLE customer_orders (
  order_id INTEGER,
  customer_id INTEGER,
  pizza_id INTEGER,
  exclusions VARCHAR(4),
  extras VARCHAR(4),
  order_time TIMESTAMP
);

-- Insert data into the customer_orders table
INSERT INTO customer_orders (order_id, customer_id, pizza_id, exclusions, extras, order_time)
VALUES
  ('1', '101', '1', '', '', '2021-01-01 18:05:02'),
  ('2', '101', '1', '', '', '2021-01-01 19:00:52'),
  ('3', '102', '1', '', '', '2021-01-02 23:51:23'),
  ('3', '102', '2', '', NULL, '2021-01-02 23:51:23'),
  ('4', '103', '1', '4', '', '2021-01-04 13:23:46'),
  ('4', '103', '1', '4', '', '2021-01-04 13:23:46'),
  ('4', '103', '2', '4', '', '2021-01-04 13:23:46'),
  ('5', '104', '1', 'null', '1', '2021-01-08 21:00:29'),
  ('6', '101', '2', 'null', 'null', '2021-01-08 21:03:13'),
  ('7', '105', '2', 'null', '1', '2021-01-08 21:20:29'),
  ('8', '102', '1', 'null', 'null', '2021-01-09 23:54:33'),
  ('9', '103', '1', '4', '1, 5', '2021-01-10 11:22:59'),
  ('10', '104', '1', 'null', 'null', '2021-01-11 18:34:49'),
  ('10', '104', '1', '2, 6', '1, 4', '2021-01-11 18:34:49');

-- Create the runner_orders table
CREATE TABLE runner_orders (
  order_id INTEGER,
  runner_id INTEGER,
  pickup_time VARCHAR(19),
  distance VARCHAR(7),
  duration VARCHAR(10),
  cancellation VARCHAR(23)
);

-- Insert data into the runner_orders table
INSERT INTO runner_orders (order_id, runner_id, pickup_time, distance, duration, cancellation)
VALUES
  ('1', '1', '2021-01-01 18:15:34', '20km', '32 minutes', ''),
  ('2', '1', '2021-01-01 19:10:54', '20km', '27 minutes', ''),
  ('3', '1', '2021-01-03 00:12:37', '13.4km', '20 mins', NULL),
  ('4', '2', '2021-01-04 13:53:03', '23.4', '40', NULL),
  ('5', '3', '2021-01-08 21:10:57', '10', '15', NULL),
  ('6', '3', 'null', 'null', 'null', 'Restaurant Cancellation'),
  ('7', '2', '2021-01-08 21:30:45', '25km', '25mins', 'null'),
  ('8', '2', '2021-01-10 00:15:02', '23.4 km', '15 minute', 'null'),
  ('9', '2', 'null', 'null', 'null', 'Customer Cancellation'),
  ('10', '1', '2021-01-11 18:50:20', '10km', '10minutes', 'null');

-- Create the pizza_names table
CREATE TABLE pizza_names (
  pizza_id INTEGER,
  pizza_name TEXT
);

-- Insert data into the pizza_names table
INSERT INTO pizza_names (pizza_id, pizza_name)
VALUES
  (1, 'Meatlovers'),
  (2, 'Vegetarian');

-- Create the pizza_recipes table
CREATE TABLE pizza_recipes (
  pizza_id INTEGER,
  toppings TEXT
);

-- Insert data into the pizza_recipes table
INSERT INTO pizza_recipes (pizza_id, toppings)
VALUES
  (1, '1, 2, 3, 4, 5, 6, 8, 10'),
  (2, '4, 6, 7, 9, 11, 12');

-- Create the pizza_toppings table
CREATE TABLE pizza_toppings (
  topping_id INTEGER,
  topping_name TEXT
);

-- Insert data into the pizza_toppings table
INSERT INTO pizza_toppings (topping_id, topping_name)
VALUES
  (1, 'Bacon'),
  (2, 'BBQ Sauce'),
  (3, 'Beef'),
  (4, 'Cheese'),
  (5, 'Chicken'),
  (6, 'Mushrooms'),
  (7, 'Onions'),
  (8, 'Pepperoni'),
  (9, 'Peppers'),
  (10, 'Salami'),
  (11, 'Tomatoes'),
  (12, 'Tomato Sauce');
```

## Cleaning the Data
Before we move onto answering questions, we notice that some of the data that Danny provided has some problems, particularly in the `customer_orders` and `runner_orders` tables!

When we take a look at the `customer_orders` table, we notice that there are empty and 'null' strings in the `exclusions` and `extras` fields.
| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
|----------|-------------|----------|------------|--------|---------------------|
| 1        | 101         | 1        |            |        | 2021-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2021-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2021-01-02 23:51:23 |
| 3        | 102         | 2        |            | NULL   | 2021-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2021-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2021-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2021-01-04 13:23:46 |
| 5        | 104         | 1        | null       | 1      | 2021-01-08 21:00:29 |
| 6        | 101         | 2        | null       | null   | 2021-01-08 21:03:13 |
| 7        | 105         | 2        | null       | 1      | 2021-01-08 21:20:29 |
| 8        | 102         | 1        | null       | null   | 2021-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2021-01-10 11:22:59 |
| 10       | 104         | 1        | null       | null   | 2021-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2021-01-11 18:34:49 |

Let's handle this by making all empty and 'null' strings consistent by making them all `NULL`. Once we do, we can ensure consistent data for smooth and accurate analysis. To do this, let's use data manipulation language (DML) to set values `NULL` where I found empty or 'null' strings for both relevant fields.
```sql
UPDATE customer_orders
SET exclusions = NULL
WHERE exclusions IN ('null', '');

UPDATE customer_orders
SET extras = NULL
WHERE extras IN ('null', '');
```

Our `customer_orders` table is now clean!
| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
|----------|-------------|----------|------------|--------|---------------------|
| 1        | 101         | 1        | NULL       | NULL   | 2021-01-01 18:05:02 |
| 2        | 101         | 1        | NULL       | NULL   | 2021-01-01 19:00:52 |
| 3        | 102         | 1        | NULL       | NULL   | 2021-01-02 23:51:23 |
| 3        | 102         | 2        | NULL       | NULL   | 2021-01-02 23:51:23 |
| 4        | 103         | 1        | 4          | NULL   | 2021-01-04 13:23:46 |
| 4        | 103         | 1        | 4          | NULL   | 2021-01-04 13:23:46 |
| 4        | 103         | 2        | 4          | NULL   | 2021-01-04 13:23:46 |
| 5        | 104         | 1        | NULL       | 1      | 2021-01-08 21:00:29 |
| 6        | 101         | 2        | NULL       | NULL   | 2021-01-08 21:03:13 |
| 7        | 105         | 2        | NULL       | 1      | 2021-01-08 21:20:29 |
| 8        | 102         | 1        | NULL       | NULL   | 2021-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2021-01-10 11:22:59 |
| 10       | 104         | 1        | NULL       | NULL   | 2021-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2021-01-11 18:34:49 |

When we observe the `runner_orders` table, there are also empty and 'null' strings. In addition, we notice that fields like `distance` and `duration` are `VARCHAR` fields that contain actual numbers we might want to use to make calculations.
| order_id | runner_id | pickup_time         | distance | duration    | cancellation          |
|----------|-----------|---------------------|----------|-------------|-----------------------|
| 1        | 1         | 2021-01-01 18:15:34 | 20km     | 32 minutes  |                       |
| 2        | 1         | 2021-01-01 19:10:54 | 20km     | 27 minutes  |                       |
| 3        | 1         | 2021-01-03 00:12:37 | 13.4km   | 20 mins     | NULL                  |
| 4        | 2         | 2021-01-04 13:53:03 | 23.4     | 40          | NULL                  |
| 5        | 3         | 2021-01-08 21:10:57 | 10       | 15          | NULL                  |
| 6        | 3         | null                | null     | null        | Restaurant Cancellation |
| 7        | 2         | 2021-01-08 21:30:45 | 25km     | 25mins      | null                  |
| 8        | 2         | 2021-01-10 00:15:02 | 23.4 km  | 15 minute   | null                  |
| 9        | 2         | null                | null     | null        | Customer Cancellation |
| 10       | 1         | 2021-01-11 18:50:20 | 10km     | 10minutes   | null                  |

Let's start out doing the same thing we did for the `customer_orders` table by setting values to `NULL` where we find empty and 'null' strings.
```sql
UPDATE runner_orders
SET cancellation = NULL
WHERE cancellation IN ('null', '');

UPDATE runner_orders
SET pickup_time = NULL,
  distance = NULL,
  duration = NULL
WHERE order_id IN (6, 9);
```
Great! Let's move onto the `distance` field. To properly clean `distance`, we must convert this field from `VARCHAR` to `DECIMAL` in order to make calculations. However, we must trim out the unnecessary spaces and units 'km' beforehand.

First, I start off by removing 'km' from the tail of these values where 'km' is found. I use a combination of the `LEFT()` and `LENGTH()` functions to remove 'km'. Then, use `RTRIM()` to remove all the trailing spaces.
```sql
UPDATE runner_orders
SET distance = LEFT(distance, LENGTH(distance) - 2)
WHERE LOWER(distance) LIKE '%km';

UPDATE runner_orders
SET distance = RTRIM(distance);
```

We can do the same approach for our `duration` field. However, I use the `SUBSTRING_INDEX()` function to substring each duration value with units. Then trim all trailing spaces.
```sql
UPDATE runner_orders
SET duration = SUBSTRING_INDEX(duration, 'm', 1)
WHERE duration LIKE '%min%';

UPDATE runner_orders
SET duration = RTRIM(duration);
```

Now that we've cleaned both `distance` and `duration` fields, let's convert them to `DECIMAL(3, 1)` and `INTEGER` respectively. We will also rename them to clarify the units being used.
```sql
ALTER TABLE runner_orders
CHANGE distance distance_km DECIMAL(3,1);

ALTER TABLE runner_orders
CHANGE duration duration_min INT;
```

Our `runner_orders` table is clean, and we are ready to do some analysis!
| order_id | runner_id | pickup_time         | distance_km | duration_min | cancellation          |
|----------|-----------|---------------------|-------------|--------------|-----------------------|
| 1        | 1         | 2021-01-01 18:15:34 | 20.0        | 32           | NULL                  |
| 2        | 1         | 2021-01-01 19:10:54 | 20.0        | 27           | NULL                  |
| 3        | 1         | 2021-01-03 00:12:37 | 13.4        | 20           | NULL                  |
| 4        | 2         | 2021-01-04 13:53:03 | 23.4        | 40           | NULL                  |
| 5        | 3         | 2021-01-08 21:10:57 | 10.0        | 15           | NULL                  |
| 6        | 3         | NULL                | NULL        | NULL         | Restaurant Cancellation |
| 7        | 2         | 2021-01-08 21:30:45 | 25.0        | 25           | NULL                  |
| 8        | 2         | 2021-01-10 00:15:02 | 23.4        | 15           | NULL                  |
| 9        | 2         | NULL                | NULL        | NULL         | Customer Cancellation |
| 10       | 1         | 2021-01-11 18:50:20 | 10.0        | 10           | NULL                  |

## A. Pizza Metrics - Answers and Solutions
### 1. How many pizzas were ordered?
**Answer:**
```sql
SELECT
  COUNT(*) AS num_pizzas
FROM
  customer_orders;
```

**Steps:**
- Simply use the function `COUNT()` to count every row from the `customer_orders` table.

**Output:**
| num_pizzas |
|-------------|
| 14 |

There were 14 pizzas ordered.

### 2. How many unique customer orders were made?
**Answer:**
```sql
SELECT
  COUNT(DISTINCT order_id) AS unique_orders
FROM
  customer_orders;
```
**Steps:**
- Simply use the function `COUNT()` to count every `DISTINCT order_id` from the `customer_orders` table.
  
**Output:**
| unique_orders |
|-------------|
| 10 |

### 3. How many successful orders were delivered by each runner?
**Answer:**
```sql
SELECT
  runner_id,
  COUNT(runner_id) AS successful_orders
FROM
  runner_orders
WHERE
  cancellation IS NULL
GROUP BY
  runner_id;
```

**Steps:**
- Use the **WHERE** clause to filter results where the `cancellation` field is `NULL`. When `cancellation` is `NULL` that means we have a successful delivery.
- We want to group the orders by `runner_id` by using the **GROUP BY** clause, then the aggregate function `COUNT()` to count each successful delivery.

**Output:**
| runner_id | successful_orders |
|------|------|
| 1 | 4 |
| 2 | 3 |
| 3 | 1 |

### 4. How many of each type of pizza was delivered?
**Answer:**
```sql
WITH successful_delivery AS (
  SELECT
    order_id,
    CASE
      WHEN cancellation IS NULL THEN 'Yes'
      ELSE 'No'
    END AS delivery
  FROM
    runner_orders
)
SELECT
  pizza_names.pizza_name,
  COUNT(customer_orders.pizza_id) AS num_deliveries
FROM
  customer_orders
JOIN
  successful_delivery
  ON successful_delivery.order_id = customer_orders.order_id
JOIN
  pizza_names
  ON pizza_names.pizza_id = customer_orders.pizza_id
WHERE
  successful_delivery.delivery = 'Yes'
GROUP BY
  pizza_names.pizza_name;
```

**Steps:**
- Use a common table expression to **JOIN** a 'Yes' or 'No' field onto the `customer_orders` table based on `order_id`. This new field will determine whether the delivery was successful or not for each particular pizza order.
- Use another **JOIN** to combine rows from the `pizza_names` table onto the `customer_orders` table based on `pizza_id` in order to obtain the `pizza_name` of each order.
- Use the **WHERE** clause to filter the results where `successful_delivery.delivery` is 'Yes'.
- Group the results by `pizza_name` using the **GROUP BY** clause. Use the aggregate function `COUNT()` to count the number of successful deliveries for each `pizza_name`.
  
**Output:**
| pizza_name | num_deliveries |
|------------|------|
| Meatlovers | 9 |
| Vegetarian | 3 |

### 5. How many Vegetarian and Meatlovers were ordered by each customer?
**Answer:**
```sql
SELECT
  customer_orders.customer_id,
  pizza_names.pizza_name,
  COUNT(pizza_names.pizza_name) AS num_orders
FROM
  customer_orders
JOIN
  pizza_names
  ON pizza_names.pizza_id = customer_orders.pizza_id
GROUP BY
  customer_orders.customer_id,
  pizza_names.pizza_name
ORDER BY
  customer_orders.customer_id,
  pizza_names.pizza_name DESC;
```

**Steps:**
- Use **JOIN** to combine rows from the `pizza_names` table onto to the `customer_orders` table based on `pizza_id` in order to obtain the `pizza_names` for each order.
- Group the results by `customer_id` and `pizza_name` using the **GROUP BY** clause, then use the aggregate function `COUNT()` to count the number of orders for each customer and pizza name.
- Make sure to use the **ORDER BY** clause to sort our results by `customer_id` and `pizza_name`, obtaining an organized result!

**Output:**
| customer_id | pizza_name | num_orders |
|-------------|------------|------------|
| 101         | Vegetarian | 1          |
| 101         | Meatlovers | 2          |
| 102         | Vegetarian | 1          |
| 102         | Meatlovers | 2          |
| 103         | Vegetarian | 1          |
| 103         | Meatlovers | 3          |
| 104         | Meatlovers | 3          |
| 105         | Vegetarian | 1          |

### 6. What was the maximum number of pizzas delivered in a single order?
**Answer:**
```sql
SELECT
  MAX(num_pizzas)
FROM (
  SELECT
    c.order_id,
    COUNT(c.order_id) AS num_pizzas
  FROM
    customer_orders c
  JOIN
    runner_orders r ON c.order_id = r.order_id
  WHERE
    r.cancellation IS NULL
  GROUP BY
    c.order_id
) AS orders;
```

**Steps:**
- In the subquery, we get a table where count the number of pizzas for each `order_id` from the `customer_orders` table by using the **GROUP BY** clause and the aggregate function `COUNT()`.
    - We also **JOIN** the runner_orders based on `order_id` to then filter successful deliveries through the **WHERE** clause.
- We then use the `MAX()` function to get the highest number of pizzas delievered in a single order.
  
**Output:**
| MAX(num_pizzas) |
|------|
| 3 |

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
**Answer:**
```sql
SELECT
  customer_orders.customer_id,
  SUM(
    CASE
      WHEN customer_orders.exclusions IS NOT NULL
        OR customer_orders.extras IS NOT NULL
        THEN 1
      ELSE 0
    END
  ) AS changed,
  SUM(
    CASE
      WHEN customer_orders.exclusions IS NULL
        AND customer_orders.extras IS NULL
        THEN 1
    ELSE 0
  END
  ) AS not_changed
FROM
  customer_orders 
JOIN
  runner_orders
  ON runner_orders.order_id = customer_orders.order_id
WHERE
  cancellation IS NULL
GROUP BY
  customer_orders.customer_id
ORDER BY
  customer_orders.customer_id;
```

**Steps:**
- Use **JOIN** to combine rows from the `runner_orders` table with the `customer_orders` table based on `order_id` to then filter our results where `cancellation` is `NULL`.
    - This will allow us to deal with only successful deliveries.
- To determine whether there is a change in a pizza, we want to check whether `exclusions` and `extras` are both `NULL`.
  - Create two additional fields in the `SELECT` statement using `CASE` expressions: `changed` and `not_changed`.
  - We will count an order as `changed` when either the `exclusions` or `extras` fields are `NOT NULL`. We will count an order as `not_changed` when both `exclusions` and `extras` fields are `NULL`.
- Use the aggregate function `SUM()` to total each `CASE` expression/new field. Group the results by `customer_id` using the **GROUP BY** clause.
- Sort the results by `customer_id`.

**Output:**
| customer_id | changed | not_changed |
|-------------|------------|------------|
| 101         | 0 | 2          |
| 102         | 0 | 3          |
| 103         | 3 | 0          |
| 104         | 2 | 1          |
| 105         | 1 | 0          |

### 8. How many pizzas were delivered that had both exclusions and extras?
**Answer:**
```sql
SELECT
  SUM(
    CASE
      WHEN customer_orders.exclusions IS NOT NULL
        AND customer_orders.extras IS NOT NULL
        THEN 1
      ELSE 0
    END
  ) AS had_both_exclusions_and_extras
FROM
  customer_orders 
JOIN
  runner_orders
  ON runner_orders.order_id = customer_orders.order_id
WHERE
  cancellation IS NULL
ORDER BY
  customer_orders.customer_id;
 ```

**Steps:**
- The answer will be pretty much the same as the one in the previous question. However, the `SELECT` statement will be different.
- This time we create a different field using the `CASE` expression.
  - Check whether both `exclusions` and `extras` fields are `NOT NULL`, then use `SUM()` to total the cases.

**Output:**
| had_both_exclusions_and_extras |
|-------------|
| 1         |

### 9. What was the total volume of pizzas ordered for each hour of the day?
**Answer:**
```sql
WITH RECURSIVE all_hours AS (
  SELECT
    0 AS num
  UNION ALL
  SELECT
    num + 1
  FROM
    all_hours
  WHERE
    num < 23
)
SELECT
  all_hours.num,
  COUNT(customer_orders.order_id) AS total_pizzas_ordered
FROM
  all_hours
LEFT JOIN
  customer_orders
  ON all_hours.num = HOUR(customer_orders.order_time)
GROUP BY
  all_hours.num
ORDER BY
  all_hours.num;
```

**Steps:**
- Under a common table expression, we use recursion to create a table `all_hours` with all numbered hours in a day (0 through 23).
- In the main query, we will then **JOIN** `all_hours` onto the `customer_orders` table based on `HOUR(customer_orders.order_time)`.
- Group the results by each hour of the day, `all_hours.num` using the **GROUP BY** clause. Then, use the aggregate function `COUNT()` to count each order in each hour.

**Output:**
| num | total_pizzas_ordered |
|-----|----------------------|
| 0   | 0                    |
| 1   | 0                    |
| 2   | 0                    |
| 3   | 0                    |
| 4   | 0                    |
| 5   | 0                    |
| 6   | 0                    |
| 7   | 0                    |
| 8   | 0                    |
| 9   | 0                    |
| 10  | 0                    |
| 11  | 1                    |
| 12  | 0                    |
| 13  | 3                    |
| 14  | 0                    |
| 15  | 0                    |
| 16  | 0                    |
| 17  | 0                    |
| 18  | 3                    |
| 19  | 1                    |
| 20  | 0                    |
| 21  | 3                    |
| 22  | 0                    |
| 23  | 3                    |

### 10. What was the volume of orders for each day of the week?
**Answer:**
First, create a temporary table to then populate it with the days of the week.
```sql
CREATE TEMPORARY TABLE days_of_the_week (
  num INT,
  day VARCHAR(9)
);

INSERT INTO days_of_the_week (num, day)
VALUES (1, 'Sunday'), (2, 'Monday'), (3, 'Tuesday'), (4, 'Wednesday'), (5, 'Thursday'), (6, 'Friday'), (7, 'Saturday');
```

Then we can write our answer query:
```sql
SELECT
  days_of_the_week.day,
  COUNT(customer_orders.order_id) AS num_orders
FROM
  days_of_the_week
LEFT JOIN
  customer_orders
  ON DAYOFWEEK(customer_orders.order_time) = days_of_the_week.num
GROUP BY
  days_of_the_week.day;
```

**Steps:**
- Use **LEFT JOIN** to join our newly created `days_of_the_week` table onto the `customer_orders` table based on `DAYOFWEEK(customer_orders.order_time) = days_of_the_week.num`.
- Group the results by day of the week using the **GROUP BY** clause. Then, use the aggregate function `COUNT()` to count the number of orders in each day of the week.

**Output:**
| day       | num_orders |
|-----------|------------|
| Sunday    | 1          |
| Monday    | 5          |
| Tuesday   | 0          |
| Wednesday | 0          |
| Thursday  | 0          |
| Friday    | 5          |
| Saturday  | 3          |
