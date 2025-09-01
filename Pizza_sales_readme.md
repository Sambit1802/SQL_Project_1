### This project is based on a database pizzahut which contains 4 tables, naming orders, order_details, pizzas, pizza_types 
and 13 questions to solve from basic level to intermediate to advance.
(Data source, questions and guidance are from wsCube Tech youtube video and description)

1. **Retrieve the total number of orders placed.**
```sql
SELECT 
COUNT(order_id) AS Total_orders
FROM orders;
```


2. **Calculate the total revenue generated from pizza sales.**
```sql
SELECT ROUND(SUM(od.quantity * p.price), 2) AS Total_revenue
FROM order_details AS od
LEFT JOIN pizzas AS p ON od.pizza_id = p.pizza_id;
```


3. **Identify the highest-priced pizza.**
```sql
SELECT pt.name, p.price
FROM pizza_types AS pt
LEFT JOIN pizzas AS p ON pt.pizza_type_id = p.pizza_type_id
GROUP BY pt.name , p.price
ORDER BY p.price DESC
LIMIT 1;
```



4. **Identify the most common pizza size ordered.**
```sql
SELECT p.size, SUM(od.quantity) AS total_quantity
FROM order_details AS od
LEFT JOIN pizzas AS p ON od.pizza_id = p.pizza_id
GROUP BY p.size
ORDER BY total_quantity DESC
LIMIT 1;
```


5. **List the top 5 most ordered pizza types along with their quantities.**
```sql
SELECT pt.name, pt.category, SUM(od.quantity) AS Total_quantity
FROM pizzas AS p
RIGHT JOIN order_details AS od ON p.pizza_id = od.pizza_id
RIGHT JOIN pizza_types AS pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name , pt.category
ORDER BY Total_quantity DESC
LIMIT 5;
```


# (6) Find the total quantity of each pizza category ordered.

SELECT 
    pt.category, SUM(od.quantity) AS Total_quantity
FROM
    pizzas AS p
        RIGHT JOIN
    order_details AS od ON p.pizza_id = od.pizza_id
        RIGHT JOIN
    pizza_types AS pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category
ORDER BY Total_quantity DESC;



# (7) Determine the distribution of orders by hour of the day.

SELECT 
    HOUR(order_time) AS hour, COUNT(order_id) AS order_count
FROM
    orders
GROUP BY hour
ORDER BY hour;



# (8) Join relevant tables to find the category-wise distribution of pizzas.

SELECT 
    category, COUNT(name) AS count
FROM
    pizza_types
GROUP BY category;



# (9) Group the orders by date and calculate the average number of pizzas ordered per day.

SELECT 
    ROUND(AVG(quantity), 0) AS avg_orders
FROM
    (SELECT 
        o.order_date, SUM(od.quantity) AS quantity
    FROM
        orders AS o
    LEFT JOIN order_details AS od ON o.order_id = od.order_id
    GROUP BY o.order_date) AS sub;



# (10) Determine the top 3 most ordered pizza types based on revenue.

SELECT 
    pt.category,
    ROUND(SUM(od.quantity * p.price), 0) AS total_revenue
FROM
    order_details AS od
        LEFT JOIN
    pizzas AS p ON od.pizza_id = p.pizza_id
        LEFT JOIN
    pizza_types AS pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category
ORDER BY total_revenue DESC
LIMIT 3;



# (11) Calculate the percentage contribution of each pizza type to total revenue.

SELECT 
    pt.category,
    ROUND(SUM(od.quantity * p.price) / (SELECT 
                    ROUND(SUM(p.price * od.quantity), 2) AS total_revenue
                FROM
                    pizzas AS p
                        RIGHT JOIN
                    order_details AS od ON p.pizza_id = od.pizza_id) * 100,
            2) AS percentage
FROM
    order_details AS od
        LEFT JOIN
    pizzas AS p ON od.pizza_id = p.pizza_id
        LEFT JOIN
    pizza_types AS pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category
ORDER BY percentage DESC;



# (12) Analyze the cumulative revenue generated over time.

SELECT 
    order_date, ROUND(SUM(revenue) OVER(ORDER BY order_date),2) AS cum_revenue
FROM
    (SELECT 
        o.order_date, SUM(od.quantity * p.price) AS revenue
    FROM
		order_details AS od
            LEFT JOIN
        pizzas AS p ON od.pizza_id = p.pizza_id
            LEFT JOIN
        orders AS o ON od.order_id = o.order_id
    GROUP BY o.order_date) AS sub



# (13) Determine the top 3 most ordered pizza types based on revenue for each pizza category

SELECT name, category, revenue
FROM
(SELECT category,name,revenue, RANK() OVER(PARTITION BY category ORDER BY revenue DESC) AS rn
FROM
(SELECT 
    pt.name,
    pt.category,
    ROUND(SUM(od.quantity* p.price), 0) AS revenue
FROM
    pizzas AS p
        LEFT JOIN
    order_details AS od ON od.pizza_id = p.pizza_id
        LEFT JOIN
    pizza_types AS pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name , pt.category) AS sub) AS Rank_Table
