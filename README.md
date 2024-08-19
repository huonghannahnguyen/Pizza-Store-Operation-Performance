# Pizza-Store-Operation-Performance
Pizza Store Operation Performance


-- Link to the dataset: https://www.kaggle.com/datasets/shilongzhuang/pizza-sales
-- Link to Tableau Dashboard: https://public.tableau.com/app/profile/huong6399/viz/PizzaStoreOperationPerformance/Story1
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
![image](https://github.com/user-attachments/assets/949ec29e-ea7c-4252-81a6-cd9780a917d0)
-----------------------------------------------------
-- Analysis
-- Quantity, Sales & Orders by Pizza

	SELECT name, SUM(quantity) AS quantity, SUM(total_price::decimal) AS sales,
		category, ingredients, COUNT(distinct order_id) AS number_of_order
	FROM pizzastore
	GROUP BY name, category, ingredients
![image](https://github.com/user-attachments/assets/5139167b-95a8-4131-a27c-e89afa917522)
-- Quantity and Sales by Day in week

	WITH a AS (SELECT TO_CHAR(order_date, 'IW') AS week, TO_CHAR(order_date, 'DAY') AS day, 
			SUM(quantity) AS quantity, SUM(total_price::DECIMAL) AS revenue
		   FROM pizzastore
		   GROUP BY week, day)

	SELECT day, AVG(quantity) AS quantity, AVG(revenue) AS revenue
	FROM a
	GROUP BY day
 ![image](https://github.com/user-attachments/assets/90440f4d-1cbe-4ddc-a08d-73352a1e4ad0)
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
 ![image](https://github.com/user-attachments/assets/75c48d2c-cd8d-43a0-8179-db796cab0190)
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
![image](https://github.com/user-attachments/assets/64085fb9-42ca-4231-bd1c-4e06b7d5950a)
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
 ![image](https://github.com/user-attachments/assets/a1df052e-8bb2-4581-9fea-66dab3e77f46)
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
![image](https://github.com/user-attachments/assets/d9916f8f-7a7a-4445-8c3d-49439ef5ec54)
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
 ![image](https://github.com/user-attachments/assets/a8fdd02e-4fd1-492e-b7d5-8bc590fc17cd)
-- Total sales, quantity and order

	SELECT SUM(quantity) AS total_quantity, SUM(total_price::decimal) AS total_sales, 
		COUNT(DISTINCT order_id) AS total_orders
	FROM pizzastore
![image](https://github.com/user-attachments/assets/3bef222f-30ee-4e2b-82a6-414182af0de7)
-- Quantity and sales per order

	WITH a AS (SELECT order_id, SUM(quantity) AS total_quantity, SUM(total_price::decimal) AS total_price
	FROM pizzastore
	GROUP BY order_id)

	SELECT category, AVG(total_quantity) AS average_quantity_per_order, AVG(total_price) AS average_value_per_order
	FROM a
![image](https://github.com/user-attachments/assets/f330d62b-89c9-4f36-9b04-89111b2d9e02)
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
 ![image](https://github.com/user-attachments/assets/8788d697-6296-4947-9c95-63c860416121)
-- Number of order by day

	WITH a AS (SELECT TO_CHAR(order_date, 'IW') AS week, TO_CHAR(order_date, 'DAY') AS day, 
			COUNT(distinct order_id) AS total_order
		   FROM pizzastore
		   GROUP BY week, day)

	SELECT day, AVG(total_order) AS avg_order
	FROM a
	GROUP BY day
![image](https://github.com/user-attachments/assets/743e232e-d995-4ef3-845c-e6cad745b231)
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
![image](https://github.com/user-attachments/assets/72b4b60c-dfa9-403c-9c1f-4d9389002df1)
-- Quantity by category

	SELECT category, SUM(quantity)
	FROM pizzastore
	GROUP BY category
 ![image](https://github.com/user-attachments/assets/aac7ae8c-207e-4067-8e76-bffa1b11577e)
-- Quantity and number of order by pizza size

	SELECT category, size, SUM(quantity) AS quantity, COUNT(distinct order_id) AS order
	FROM pizzastore
	GROUP BY category, size
![image](https://github.com/user-attachments/assets/0a7d6d48-d916-4202-b040-acf6aa7f06fd)
-- Sales over time

	SELECT order_date, SUM(total_price::decimal) AS sales
	FROM pizzastore
	GROUP BY order_date
![image](https://github.com/user-attachments/assets/97284abf-1e11-4db0-ad89-08c1532cd917)

![image](https://github.com/user-attachments/assets/953b7e0e-fc12-4006-9ef7-d987924a5b82)

