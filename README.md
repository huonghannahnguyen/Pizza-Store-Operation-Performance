# Pizza-Store-Operation-Performance
Pizza Store Operation Performance
Link to the dataset: https://www.kaggle.com/datasets/shilongzhuang/pizza-sales
------------------------------------------------------
-- Create table pizzastore in postgreSQL

	CREATE TABLE pizzastore (order_details_id int, order_id int, pizza_id varchar, 
		quantity int, order_date date, order_time time, unit_price varchar, total_price varchar,
		size text, category text, ingredients varchar, name text);
------------------------------------------------------
-- Clean data
-- Update unit_price to correct number format

	UPDATE pizzastore
	SET unit_price = REPLACE(unit_price,',','.'), 
		total_price = REPLACE(total_price,',','.')

	UPDATE pizzastore
	SET unit_price = CAST(unit_price AS decimal),
		total_price = CAST(total_price AS decimal)

-----------------------------------------------------
-- Analysis
-- Quantity, Sales & Orders by Pizza

	SELECT name, SUM(quantity) AS quantity, SUM(total_price::decimal) AS sales,
		category, ingredients, COUNT(distinct order_id) AS number_of_order
	FROM pizzastore
	GROUP BY name, category, ingredients

-- Quantity and Sales by Day in week

	WITH a AS (SELECT TO_CHAR(order_date, 'IW') AS week, TO_CHAR(order_date, 'DAY') AS day, 
			SUM(quantity) AS quantity, SUM(total_price::DECIMAL) AS revenue
		   FROM pizzastore
		   GROUP BY week, day)

	SELECT day, AVG(quantity) AS quantity, AVG(revenue) AS revenue
	FROM a
	GROUP BY day
 
-- Quantity and Sales by Hour

	WITH a AS (SELECT TO_CHAR(order_date, 'IW') AS week, TO_CHAR(order_date, 'DAY') AS day,
			DATE_TRUNC('hour', order_time) AS time,
			SUM(quantity) AS quantity, SUM(total_price::DECIMAL) AS revenue
		   FROM pizzastore
		   GROUP BY week, day, time),

	b AS (SELECT week, time, AVG(quantity) AS quantity, AVG(revenue) AS revenue
	FROM a
	GROUP BY week, time)

	SELECT time, AVG(quantity) AS quantity, AVG(revenue) AS revenue
	FROM b
	GROUP BY time
	ORDER BY time
 
-- Quantity of cheese used by cheese category 

	WITH a AS (SELECT TRIM(regexp_split_to_table(ingredients,',')) AS ingredients
	FROM pizzastore),

	b AS (SELECT DISTINCT ingredients, COUNT(ingredients) AS quantity_ordered
	FROM a
	GROUP BY ingredients
	ORDER BY quantity_ordered DESC)

	SELECT ingredients, quantity_ordered
	FROM b
	WHERE ingredients LIKE '%Cheese'

-- Quantity of sauce used by sauce category

	WITH a AS (SELECT TRIM(regexp_split_to_table(ingredients,',')) AS ingredients
	FROM pizzastore),

	b AS (
	SELECT DISTINCT ingredients, COUNT(ingredients) AS quantity_ordered
	FROM a
	GROUP BY ingredients
	ORDER BY quantity_ordered DESC)

	SELECT ingredients, quantity_ordered
	FROM b
	WHERE ingredients LIKE '%Sauce' 
 	
-- Quantity of meat used by meat category

	WITH a AS (SELECT TRIM(regexp_split_to_table(ingredients,',')) AS ingredients
	FROM pizzastore),

	b AS (SELECT DISTINCT ingredients, COUNT(ingredients) AS quantity_ordered
	FROM a
	GROUP BY ingredients
	ORDER BY quantity_ordered DESC)

	SELECT ingredients, quantity_ordered
	FROM b
	WHERE LOWER(ingredients) LIKE ANY (ARRAY ['%pepperoni%', '%capocollo%', '%bacon%', 
		'%anchovies%', '%pancetta%','%prosciutto%', '%chicken%','%ham%','%salami%', 
		'%sausage%','%prosciutto%','%beef%'])
  
-- Quantity of veggies & herbs used by veggies & herbs category

	WITH a AS (SELECT TRIM(regexp_split_to_table(ingredients,',')) AS ingredients
	FROM pizzastore),

	b AS (SELECT DISTINCT ingredients, COUNT(ingredients) AS quantity_ordered
	FROM a
	GROUP BY ingredients
	ORDER BY quantity_ordered DESC),

	c AS (
	SELECT ingredients, quantity_ordered
	FROM b
	WHERE LOWER(ingredients) LIKE ANY (ARRAY ['%garlic%','%tomatoes%','%onions%','%peppers%',
		'%spinach%','%mushrooms%','%olives%','%pineapple%','%cilantro%','%corn%','%zucchini%',
		'%peperoncini%','%arugula%','%oregano%','%eggplant%','%pears%','%thyme%','%artichoke%'
	])),

	d AS (SELECT REPLACE(ingredients,'Artichokes','Artichoke') AS ingredients, quantity_ordered
	FROM c)

	SELECT DISTINCT ingredients, SUM(quantity_ordered) AS quantity_ordered
	FROM d
	GROUP BY ingredients
	ORDER BY quantity_ordered DESC
 
-- Total sales, quantity and order

	SELECT SUM(quantity) AS total_quantity, SUM(total_price::decimal) AS total_sales, 
		COUNT(DISTINCT order_id) AS total_orders
	FROM pizzastore

-- Quantity and sales per order

	WITH a AS (SELECT order_id, SUM(quantity) AS total_quantity, SUM(total_price::decimal) AS total_price
	FROM pizzastore
	GROUP BY order_id)

	SELECT category, AVG(total_quantity) AS average_quantity_per_order, AVG(total_price) AS average_value_per_order
	FROM a
 
-- Sales, quantity and order per day

	WITH a AS (SELECT category, order_date, SUM(quantity) AS total_quantity, 
			SUM(total_price::decimal) AS total_sales, 
			COUNT(DISTINCT order_id) AS total_order
		   FROM pizzastore
		   GROUP BY order_date, category)

	SELECT category, AVG(total_quantity) AS avg_quantity, AVG(total_sales) AS avg_sales, 
		AVG(total_order) AS avg_order
	FROM a
	GROUP BY category
 
-- Number of order by day

	WITH a AS (SELECT TO_CHAR(order_date, 'IW') AS week, TO_CHAR(order_date, 'DAY') AS day, 
			COUNT(distinct order_id) AS total_order
		   FROM pizzastore
		   GROUP BY week, day)

	SELECT day, AVG(total_order) AS avg_order
	FROM a
	GROUP BY day
 
-- Number of order by hour

	WITH a AS (SELECT TO_CHAR(order_date, 'IW') AS week, TO_CHAR(order_date, 'DAY') AS day,
			DATE_TRUNC('hour', order_time) AS time, COUNT(distinct order_id) AS total_order
		   FROM pizzastore
		   GROUP BY week, day, time),

	b AS (SELECT week, time, AVG(total_order) AS avg_order
	FROM a
	GROUP BY week, time)

	SELECT time, AVG(avg_order) AS avg_order
	FROM b
	GROUP BY time
	ORDER BY time
 
-- Quantity by category

	SELECT category, SUM(quantity)
	FROM pizzastore
	GROUP BY category
 
-- Quantity and number of order by pizza size

	SELECT category, size, SUM(quantity) AS quantity, COUNT(distinct order_id) AS order
	FROM pizzastore
	GROUP BY category, size
 
-- Sales over time

	SELECT order_date, SUM(total_price::decimal) AS sales
	FROM pizzastore
	GROUP BY order_date
