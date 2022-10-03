# Case_Study_2_Pizza_Runner_8_Week_SQL_Challenge


<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193538874-129146a5-0c38-4edd-b1d7-11628c16219c.png" height="500"/>
</p>


## Introduction

This is analysis is the solving of 2nd case study the popular online 8 Week SQL Challenge by Danny Ma. 

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

## Problem Statement

The problem is to address how the business is going and what can be optimized by analyzing the data. 

## Data

Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.

He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193540532-8213fbab-ed4d-4e81-b080-ac7b9ccabf5c.png"/>
</p>

## Questions
they are broken up by area of focus including: * Pizza Metrics * Runner and Customer Experience * Ingredient Optimisation * Pricing and Ratings * Bonus DML Challenges 

Before starting to answer the questions, some data cleaning is in order. Specifically:
    - null values and data types in the customer_orders table
    - null values and data types in the runner_orders table
    - Alter data type in pizza_names table
    
First, we'll clean up exclusions and extras in the customer_orders by createting a TEMP TABLE customer_orders1 and using CASE WHEN    

~~~~sql
SELECT 
order_id, 
customer_id, 
pizza_id, 
CASE 
  WHEN exclusions IS null OR exclusions LIKE 'null' THEN ' '
  ELSE exclusions
  END AS exclusions,
CASE 
  WHEN extras IS NULL or extras LIKE 'null' THEN ' '
  ELSE extras 
  END AS extras, 
order_time
INTO 
customer_orders1
FROM 
pizza_runner.customer_orders;
~~~~

Secondly, we'll clean the runner_orders table with CASE WHEN and TRIM and create a TEMP TABLE runner_orders1.
In short:
    - pickup_time — Remove nulls and replace with ‘ ‘
    - distance — Remove ‘km’ and nulls
    - duration — Remove ‘minutes’ and nulls
    - cancellation — Remove NULL and null and replace with ‘ ‘

~~~~sql
SELECT 
order_id, 
runner_id,
CASE 
  WHEN pickup_time LIKE 'null' THEN ' '
  ELSE pickup_time 
  END AS pickup_time,
CASE 
  WHEN distance LIKE 'null' THEN ' '
  WHEN distance LIKE '%km' THEN TRIM('km' from distance) 
  ELSE distance END AS distance,
CASE 
  WHEN duration LIKE 'null' THEN ' ' 
  WHEN duration LIKE '%mins' THEN TRIM('mins' from duration) 
  WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)        
  WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)       
  ELSE duration END AS duration,
CASE 
  WHEN cancellation IS NULL or cancellation LIKE 'null' THEN ''
  ELSE cancellation END AS cancellation
INTO 
runner_orders1
FROM 
pizza_runner.runner_orders;
~~~~

Lastly, we have to to alter some data to their correct data types:

~~~~sql
ALTER TABLE 1runner_orders
ALTER COLUMN pickup_time DATETIME,
ALTER COLUMN distance FLOAT, 
ALTER COLUMN duration INT;

ALTER TABLE pizza_runner.pizza_names
ALTER COLUMN pizza_name NVARCHAR (50);
~~~~

Now we are ready to go.

### A. Pizza Metrics
#### 01. How many pizzas were ordered?

~~~~sql
SELECT
  COUNT (pizza_id) AS total_pizzas
FROM
  customer_orders1;
~~~~

<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193543741-e16f0eb3-64e1-48f4-8644-bf37993c162d.png"/>
</p>

#### 02. How many unique customer orders were made?

~~~~sql
    SELECT
  COUNT (DISTINCT(order_id)) AS unique_orders
FROM
  customer_orders1;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193545013-efab6bbb-a09e-436c-b698-d4644ae17bbf.png"/>
</p>

#### 03. How many successful orders were delivered by each runner?

~~~~sql
  SELECT
  runner_id,
  count(DISTINCT (order_id)) AS succesful_orders
FROM
  runner_orders1
WHERE
 cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
GROUP BY
  runner_id
ORDER BY
  succesful_orders DESC;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193545583-b1e014f6-5d79-4697-bc4b-aac597bd3c28.png"/>
</p>

#### 04. How many of each type of pizza was delivered?

~~~~sql
SELECT 
customer_orders1.pizza_id,
pizza_names.pizza_name,
COUNT (customer_orders1.pizza_id) AS n_of_orders
FROM 
customer_orders1
INNER JOIN  pizza_runner.pizza_names ON customer_orders1.pizza_id=pizza_names.pizza_id
INNER JOIN  runner_orders1 ON customer_orders1.order_id=runner_orders1.order_id
WHERE 
runner_orders1.cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
GROUP BY 
pizza_name,customer_orders1.pizza_id;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193546701-2927fa6d-d134-4e0d-a83e-d0167424298b.png"/>
</p>

#### 05. How many Vegetarian and Meatlovers were ordered by each customer?

~~~~sql
SELECT 
customer_orders1.customer_id, 
SUM (CASE WHEN customer_orders1.pizza_id =1 THEN 1 ELSE 0 END) AS meatlovers,
SUM (CASE WHEN customer_orders1.pizza_id =2 THEN 1 ELSE 0 END) AS vegetarian
FROM
customer_orders1
INNER JOIN pizza_runner.pizza_names on customer_orders1.pizza_id=pizza_names.pizza_id
INNER JOIN runner_orders1 on customer_orders1.order_id=runner_orders1.order_id
WHERE 
runner_orders1.cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
GROUP BY 
customer_orders1.customer_id
ORDER BY 
customer_id;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193547814-b5196328-26a3-4b13-b09a-aab5e5898985.png"/>
</p>

#### 06. What was the maximum number of pizzas delivered in a single order?

~~~~sql
WITH pizza_counter AS 
(SELECT
customer_orders1.order_id,
COUNT (customer_orders1.pizza_id) AS pizza_count,
RANK () OVER (ORDER BY COUNT (customer_orders1.pizza_id) DESC) AS count_rank
FROM
customer_orders1
INNER JOIN pizza_runner.pizza_names on customer_orders1.pizza_id=pizza_names.pizza_id
INNER JOIN runner_orders1 on customer_orders1.order_id=runner_orders1.order_id
WHERE
runner_orders1.cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
GROUP BY customer_orders1.order_id
)

SELECT
pizza_count 
FROM
pizza_counter 
WHERE
count_rank=1
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193548660-3d543a69-f236-47ea-83a0-57ce65ef4c12.png"/>
</p>

#### 07. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?


~~~~sql
SELECT
customer_id,
SUM (CASE WHEN (exclusions <> ' ') OR (extras <> ' ') THEN 1 ELSE 0 END) AS at_least_1_change,
SUM (CASE WHEN (exclusions = ' ') AND (extras   = ' ') THEN 1 ELSE 0 END) AS no_changes
FROM
customer_orders1
INNER JOIN runner_orders1 ON runner_orders1.order_id=customer_orders1.order_id
WHERE
runner_orders1.cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
GROUP BY 
customer_id;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193549167-984964ab-8a86-429c-852e-39834da14f6b.png"/>
</p>

#### 08. How many pizzas were delivered that had both exclusions and extras?

~~~~sql
SELECT
customer_id,
SUM (CASE WHEN (exclusions <> ' ') AND (extras <> ' ') THEN 1 ELSE 0 END) AS exclusion_and_extra
FROM 
customer_orders1
INNER JOIN runner_orders1 on runner_orders1.order_id=customer_orders1.order_id
WHERE
runner_orders1.cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
GROUP BY
customer_id;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193549854-dccf7391-77b6-450b-93f7-a04225159138.png"/>
</p>

#### 09. What was the total volume of pizzas ordered for each hour of the day?
~~~~sql
SELECT 
DATEPART(HOUR, [order_time]) AS hour, 
COUNT(order_id) AS number_of_pizzas
FROM 
customer_orders1
GROUP BY 
DATEPART(HOUR, [order_time]);
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193550333-4a054661-fc8e-4316-8bfd-7dbb46610cb3.png"/>
</p>

#### 10. What was the volume of orders for each day of the week?
~~~~sql
SELECT 
DATENAME(WEEKDAY,[order_time]) AS day_of_the_week,
COUNT(customer_orders1.order_id) AS number_of_pizzas
FROM 
customer_orders1
INNER JOIN runner_orders1 on runner_orders1.order_id=customer_orders1.order_id
WHERE
runner_orders1.cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
GROUP BY
DATENAME(WEEKDAY,[order_time]), DATEPART(dw,[order_time]) 
ORDER BY
DATEPART(dw,[order_time]) ;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193550864-16566053-584f-4402-a4ed-62990b2db9f1.png"/>
</p>

### B. Runner and Customer Experience
#### 01. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

~~~~sql
SELECT 
DATEPART(WEEK, registration_date) AS registration_week,
COUNT(runner_id) AS runner_signup
FROM 
pizza_runner.runners
GROUP BY 
DATEPART(WEEK, registration_date);
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193594527-fb99de0c-7f8f-4a24-879f-5b0c2d69b788.png"/>
</p>

#### 02. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
~~~~sql
WITH diff_time AS (
SELECT
customer_orders1.order_time,
runner_orders1.pickup_time,
DATEDIFF (MINUTE, customer_orders1.order_time,runner_orders1.pickup_time) AS pickup_minutes
FROM 
customer_orders1
INNER JOIN runner_orders1 ON customer_orders1.order_id=runner_orders1.order_id
WHERE runner_orders1.distance <> 0
)

SELECT
ROUND (AVG (pickup_minutes),3) AS avg_time
FROM
diff_time
WHERE
pickup_minutes >1;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193595896-b59a5da2-13ac-4d6d-b19b-a450060e5261.png"/>
</p>


#### 03. Is there any relationship between the number of pizzas and how long the order takes to prepare?
~~~~sql
SELECT
customer_orders1.order_id,
COUNT (customer_orders1.order_id) AS pizza_order,
customer_orders1.order_time,
runner_orders1.pickup_time,
DATEDIFF(MINUTE, customer_orders1.order_time,runner_orders1.pickup_time) AS prep_time_minutes
FROM 
customer_orders1
INNER JOIN runner_orders1 on customer_orders1.order_id=runner_orders1.order_id
WHERE 
runner_orders1.distance <> 0
GROUP BY 
customer_orders1.order_id, customer_orders1.order_time,runner_orders1.pickup_time
ORDER BY 
COUNT (customer_orders1.order_id);
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193597736-d3d33a4a-6761-4e94-842b-bbcbd01ab499.png"/>
</p>

#### 04. What was the average distance travelled for each customer?

~~~~sql
SELECT 
customer_orders1.customer_id,
ROUND (AVG(runner_orders1.distance),1) AS average_distance
FROM
runner_orders1
INNER JOIN customer_orders1 ON runner_orders1.order_id=customer_orders1.order_id
WHERE
distance <> 0
GROUP BY customer_orders1.customer_id;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193598160-53d9c775-5ec2-4e10-bf6d-c2a9659cf03b.png"/>
</p>

#### 05. What was the difference between the longest and shortest delivery times for all orders?
~~~~sql
WITH max_and_min AS
( 
SELECT
MAX(duration) AS max_duration,
MIN (duration) AS min_duration
FROM
runner_orders1
WHERE
distance <> 0
)

SELECT
max_duration - min_duration as diff_time
FROM
max_and_min;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193614327-b6ef5785-0ec2-417b-8799-6866ab2dac4f.png"/>
</p>

#### 06. What was the average speed for each runner for each delivery and do you notice any trend for these values?
~~~~sql
SELECT
runner_id,
order_id,
distance,
duration,
ROUND (AVG (distance/duration*60),1) AS average_speed,
DATEPART(HOUR, [pickup_time]) AS hour_of_the_day
from runner_orders1
where distance <> 0
GROUP by runner_id, order_id, distance,duration, pickup_time
ORDER by runner_id;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193614778-a70b2ac3-0295-4d37-bd21-b52e407f13c0.png"/>
</p>

More or less, it looks like as the hours go by the speed increases, probably because there is less traffic. Runner n.2 has a lot of variation, maybe there's a special reason?

#### 07. What is the successful delivery percentage for each runner?
~~~~sql
SELECT
runner_id,
ROUND(
100 * SUM(CASE WHEN pickup_time <> ' ' THEN 1 ELSE 0 END) /
COUNT(*), 0
  ) AS success_percentage
FROM 
runner_orders1
GROUP BY 
runner_id
ORDER BY 
runner_id;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193615896-4bb479cb-f3f5-4a90-a123-c417bcd1cbcb.png"/>
</p>

### C. Ingredient Optimisation
#### 01. What are the standard ingredients for each pizza?
This one is a but tricky as we have to do some transformations: first we have to cast some data types, then extract all values as single values and put everything back togheter.
~~~~sql
--casting data type for string manipulation
WITH converting AS (
SELECT 
pizza_id,
CAST (toppings AS NVARCHAR (50)) AS toppings
FROM 
pizza_runner.pizza_recipes
),
--using cross applying string split to extract all values as single values
temp AS (
SELECT 
pizza_id, toppings, value
FROM 
converting  
CROSS APPLY STRING_SPLIT(toppings, ',')
  ),

--combining everything togheter
combining AS (
SELECT
pizza_names.pizza_name,
temp.value,
CAST (pizza_toppings.topping_name AS NVARCHAR (50)) AS toppings
FROM
pizza_runner.pizza_names
INNER JOIN temp ON pizza_names.pizza_id=temp.pizza_id
INNER JOIN pizza_runner.pizza_toppings ON pizza_toppings.topping_id=temp.value)
---putting all toppings back in a single row
SELECT 
pizza_name,
STRING_AGG(combining.toppings,',' ) AS toppings
FROM
combining 
GROUP BY pizza_name;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193617674-b4903913-f196-4997-9616-2cf4e6f82b04.png"/>
</p>

#### 02. What was the most commonly added extra?
This is quite similar to the previous one. I'll just create a TEMP TABLE for the toppings.
~~~~sql
WITH converting AS (
SELECT 
pizza_id,
CAST (toppings AS NVARCHAR (50)) AS toppings
FROM pizza_runner.pizza_recipes
),
--using cross applying string split to extract all values as single values
temp AS (
SELECT pizza_id, toppings, value
FROM converting  
CROSS APPLY STRING_SPLIT(toppings, ',')
),

--combining everything togheter
topping AS (SELECT
pizza_names.pizza_name,
temp.value,
CAST (pizza_toppings.topping_name AS NVARCHAR (50)) AS toppings
from pizza_runner.pizza_names
INNER JOIN temp ON pizza_names.pizza_id=temp.pizza_id
INNER JOIN pizza_runner.pizza_toppings ON pizza_toppings.topping_id=temp.value)

--creating a temp table for toppings
select 
pizza_name,
TRIM (value) as value,
toppings
into toppings1
from topping;

--applying the same transformation to the extras
WITH temp_extras AS (
SELECT extras, value AS extra_name
FROM customer_orders1  
    CROSS APPLY STRING_SPLIT(extras, ',')
)
--joining the results
SELECT
toppings1.toppings,
count (temp_extras.extra_name) AS n_of_ordered_extras
from temp_extras
INNER join toppings1 on toppings1.value=temp_extras.extra_name
where extra_name <> ' '
Group by toppings1.toppings, extra_name
ORDER BY count (temp_extras.extra_name) DESC;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193618921-1498f166-dc48-4163-8155-0c7d98604e46.png"/>
</p>


#### 03. What was the most common exclusion?
~~~~sql
WITH t_exclusions AS (
SELECT 
exclusions, value AS exclusion_name
FROM 
customer_orders1  
CROSS APPLY STRING_SPLIT(exclusions, ',')
),

temp_exclusions AS (
SELECT
exclusions,
TRIM (exclusion_name) AS exclusion_name
FROM
t_exclusions
)

SELECT 
temp_exclusions.exclusion_name,
COUNT (temp_exclusions.exclusion_name) n_of_exclusions,
toppings1.toppings AS name_of_topping
FROM 
toppings1
FULL JOIN temp_exclusions ON toppings1.value=temp_exclusions.exclusion_name
WHERE temp_exclusions.exclusion_name <> ' '
GROUP BY temp_exclusions.exclusion_name, toppings1.toppings
ORDER BY COUNT (temp_exclusions.exclusion_name) DESC;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193621134-761975c5-ba7f-434e-b23d-28978623bb2a.png"/>
</p>


### D. Pricing and Ratings
#### 01. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

~~~~sql
SELECT
SUM(
CASE 
WHEN pizza_id = 1 THEN 12 
WHEN pizza_id=2 THEN 10
END) AS total_revenue
FROM
customer_orders1;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193622190-9be91275-4ba3-47f5-8d39-a6688b974f19.png"/>
</p>

#### 02. What if there was an additional $1 charge for any pizza extras? + Add cheese is $1 extra

~~~~sql
WITH temp AS (
SELECT 
REPLACE (customer_orders1.extras, ',', '') AS n_of_extras,
customer_orders1.pizza_id,
runner_orders1.distance
FROM
customer_orders1
INNER JOIN runner_orders1 ON customer_orders1.order_id=runner_orders1.order_id
WHERE
runner_orders1.distance<>0 
),

temp_2 AS (
SELECT 
LEN (REPLACE(n_of_extras,' ','')) AS n_of_extras,
pizza_id 
FROM
temp)

SELECT
SUM(
CASE 
WHEN pizza_id = 1 THEN 12 
WHEN pizza_id=2 THEN 10
END) + SUM (n_of_extras) AS total_revenue 
FROM temp_2;
~~~~
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193631732-7289563b-af47-4333-a04f-7ce32c4b8171.png"/>
</p>

#### 0
~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>

#### 0
~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>

#### 0
~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>

#### 0
~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>

#### 0
~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>
