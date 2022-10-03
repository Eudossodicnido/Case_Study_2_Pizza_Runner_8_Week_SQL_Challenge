# Case_Study_2_Pizza_Runner_8_Week_SQL_Challenge


<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193538874-129146a5-0c38-4edd-b1d7-11628c16219c.png" height="500"/>
</p>


### Introduction

This is analysis is the solving of 2nd case study the popular online 8 Week SQL Challenge by Danny Ma. 

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

### Problem Statement

The problem is to address how the business is going and what can be optimized by analyzing the data. 

### Data

Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.

He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/193540532-8213fbab-ed4d-4e81-b080-ac7b9ccabf5c.png"/>
</p>

### Questions
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

#### A. Pizza Metrics
### 01. How many pizzas were ordered?

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




~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>


~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>


~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>


~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>


~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>


~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>


~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>


~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>


~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>


~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>


~~~~sql
~~~~
<p align="center">
  <img src=""/>
</p>
