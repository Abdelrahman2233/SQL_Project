
# SQL_Project

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

![2](https://github.com/user-attachments/assets/28ff51e6-dcb3-437a-a1c6-d12217bd2959)

## 📚Table of Contents
- [The Problem](#The-Problem)

- [The Approach](#The-Approach)

- [The Answers](#The-Answers)

## The Problem
Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.

He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

## The Approach 
1st : Fully Understand and grasp the business problem with it's full dimensions

2nd : obersve the Entity Relationship Diagram 

3rd : Assess the data quality

4th : Start the Analysis
## Entity Relationship Diagram

![Screenshot 2024-09-29 175753](https://github.com/user-attachments/assets/55f10a14-def8-4118-9cdf-2d93421f80ab)

## The Answers
#### 1.How many pizzas were ordered?

````sql
SELECT 
    COUNT(*) AS number_of_orders
FROM
    customer_orders;
````
![1](https://github.com/user-attachments/assets/298b7721-2a2b-4847-95e7-f0cc94f69db7)

#### 2.How many unique customer orders were made?
````sql
SELECT 
    COUNT(DISTINCT order_id) AS unique_orders
FROM
    customer_orders;
````

![22](https://github.com/user-attachments/assets/003898f4-6ae7-4370-98dd-94f5306c647d)

#### 3.How many successful orders were delivered by each runner?
````sql
SELECT 
    runner_id, COUNT(order_id) AS successful_orders
FROM
    runner_orders
WHERE
    cancellation = ''
GROUP BY runner_id;
````

![3](https://github.com/user-attachments/assets/1dc2cc90-65dd-4584-be4f-a17969d64ea6)

#### 4.How many of each type of pizza was delivered?
````sql
SELECT 
    p.pizza_name, COUNT(c.order_id) AS Delivered
FROM
    pizza_names AS p
        JOIN
    customer_orders AS c ON p.pizza_id = c.pizza_id
GROUP BY p.pizza_name;
````

![4](https://github.com/user-attachments/assets/1c9cafdf-e606-4862-9c9e-12a8378dc1b0)

#### 5.How many Vegetarian and Meatlovers were ordered by each customer?
````sql
SELECT 
    p.pizza_name, c.customer_id, COUNT(c.order_id) AS Delivered
FROM
    pizza_names AS p
        JOIN
    customer_orders AS c ON p.pizza_id = c.pizza_id
GROUP BY p.pizza_name , c.customer_id
ORDER BY c.customer_id;
````
![5](https://github.com/user-attachments/assets/6114c43b-f008-420e-9d4e-5231ec7ed037)


#### 6.What was the maximum number of pizzas delivered in a single order?
````sql
SELECT 
    r.order_id, COUNT(c.pizza_id) AS pizzas
FROM
    customer_orders AS c
        JOIN
    runner_orders AS r ON c.order_id = r.order_id
GROUP BY r.order_id
ORDER BY pizzas DESC
LIMIT 1;
````
![6](https://github.com/user-attachments/assets/660d2d90-e231-44b4-b167-52a86b9db5cc)

#### 7.For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
````sql
WITH cte_changes AS ( 
  SELECT c.customer_id, COUNT(c.order_id) AS changes
  FROM customer_orders AS c 
  JOIN runner_orders AS r 
  ON c.order_id = r.order_id
  WHERE r.pickup_time IS NOT NULL
  AND (c.exclusions != '' OR c.extras != '')
  GROUP BY c.customer_id
),
cte_no_changes AS ( 
  SELECT c.customer_id, COUNT(c.order_id) AS no_changes
  FROM customer_orders AS c 
  JOIN runner_orders AS r 
  ON c.order_id = r.order_id
  WHERE r.pickup_time IS NOT NULL
  AND (c.exclusions = '' AND c.extras = '')
  GROUP BY c.customer_id
)

SELECT COALESCE(c1.customer_id, c2.customer_id) AS customer_id, 
       COALESCE(c1.changes, 0) AS changes, 
       COALESCE(c2.no_changes, 0) AS no_changes
FROM cte_changes AS c1
LEFT JOIN cte_no_changes AS c2
ON c1.customer_id = c2.customer_id

UNION

SELECT COALESCE(c1.customer_id, c2.customer_id) AS customer_id, 
       COALESCE(c1.changes, 0) AS changes, 
       COALESCE(c2.no_changes, 0) AS no_changes
FROM cte_no_changes AS c2
LEFT JOIN cte_changes AS c1
ON c1.customer_id = c2.customer_id;
````
![7](https://github.com/user-attachments/assets/984da392-f6b6-4971-a5be-59f601cd2936)

####  8.What was the total volume of pizzas ordered for each hour of the day?
````sql
SELECT 
    HOUR(order_time) AS order_hour,
    COUNT(order_id) AS total_pizzas
FROM
    customer_orders
GROUP BY HOUR(order_time)
ORDER BY order_hour;
````
![8](https://github.com/user-attachments/assets/ced028b4-ac26-4164-95f5-a9041ae2035e)

#### 9.What was the volume of orders for each day of the week?
````sql
SELECT 
    DAYNAME(DATE_ADD(order_time, INTERVAL 2 DAY)) AS day_of_week,
    COUNT(order_id) AS total_pizzas_ordered
FROM
    customer_orders
GROUP BY DAYNAME(DATE_ADD(order_time, INTERVAL 2 DAY)) , DAYOFWEEK(DATE_ADD(order_time, INTERVAL 2 DAY))
ORDER BY DAYOFWEEK(DATE_ADD(order_time, INTERVAL 2 DAY));
````
![9](https://github.com/user-attachments/assets/f2616885-47e0-4665-a6b2-ee677f94d802)

