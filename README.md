# 🍕 Pizza Sales Analysis - SQL Project

A SQL-based analysis of pizza restaurant sales data, progressing from basic aggregation queries to advanced window functions and ranking logic. Built to practice real-world business analytics using SQL.

## 📌 Objective

To analyze pizza sales data using SQL - evaluating order volume, revenue and sales trends across categories, sizes and time of day - in order to identify best-selling items, peak order times and revenue drivers, and translate raw transactional data into actionable business recommendations.

## 🛠 Tools Used

- **MySQL** — querying and analysis
- **MySQL Workbench** — query execution and result visualization
- **Canva** — presentation design for LinkedIn/Instagram carousel

## 🗃 Database Schema

The dataset consists of four related tables:

| Table | Columns |
|---|---|
| `orders` | order_id, order_date, order_time |
| `order_details` | order_details_id, order_id, pizza_id, quantity |
| `pizzas` | pizza_id, pizza_type_id, size, price |
| `pizza_types` | pizza_type_id, name, category, ingredients |

**Relationships:** `pizza_types` → `pizzas` → `order_details` ← `orders`

## 📁 Repository Structure
pizza-sales-sql-analysis/
│
├── Pizza_Sales_Analysis.pdf # Full carousel with queries, results & insights
└── README.md

## 📊 Key Insights

- Large pizzas and the Classic category dominate overall orders
- Lunch hour (12–1 PM) sees peak order volume
- Chicken-based pizzas generate the highest revenue despite lower order counts - a higher price point per unit
- Classic category contributes the largest share (27%) to total revenue
- Revenue is fairly evenly split across categories (24–27% each) -no single category carries the business

## 🧾 SQL Queries

### Basic

**Q1. Total number of orders placed**
```sql
SELECT COUNT(order_id) AS total_orders
FROM orders;
```

**Q2. Total revenue generated from pizza sales**
```sql
SELECT ROUND(SUM(order_details.quantity * pizzas.price)) AS total_sales
FROM order_details
JOIN pizzas ON pizzas.pizza_id = order_details.pizza_id;
```

**Q3. Highest-priced pizza**
```sql
SELECT pizza_types.name, pizzas.price
FROM pizza_types
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;
```

**Q4. Most common pizza size ordered**
```sql
SELECT pizzas.size, COUNT(order_details.order_details_id) AS order_count
FROM pizzas
JOIN order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY order_count DESC
LIMIT 1;
```

### Intermediate

**Q5. Top 5 most ordered pizza types by quantity**
```sql
SELECT pizza_types.name, SUM(order_details.quantity) AS quantity
FROM pizza_types
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC
LIMIT 5;
```

**Q6. Total quantity ordered per pizza category**
```sql
SELECT pizza_types.category, SUM(order_details.quantity) AS quantity
FROM pizza_types
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY quantity DESC;
```

**Q7. Distribution of orders by hour of the day**
```sql
SELECT HOUR(order_time) AS hour, COUNT(order_id) AS order_count
FROM orders
GROUP BY HOUR(order_time);
```

**Q8. Category-wise distribution of pizza types**
```sql
SELECT category, COUNT(name)
FROM pizza_types
GROUP BY category;
```

**Q9. Average number of pizzas ordered per day**
```sql
SELECT ROUND(AVG(quantity)) FROM
(SELECT orders.order_date, SUM(order_details.quantity) AS quantity
 FROM orders
 JOIN order_details ON orders.order_id = order_details.order_id
 GROUP BY orders.order_date) AS order_quantity;
```

**Q10. Top 3 pizza types by revenue**
```sql
SELECT pizza_types.name, SUM(order_details.quantity * pizzas.price) AS revenue
FROM pizza_types
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;
```

**Q11. Percentage contribution of each category to total revenue**
```sql
SELECT pizza_types.category,
ROUND((SUM(order_details.quantity*pizzas.price) /
 (SELECT ROUND(SUM(order_details.quantity*pizzas.price))
  FROM order_details
  JOIN pizzas ON pizzas.pizza_id = order_details.pizza_id)
 ) * 100) AS revenue
FROM pizza_types
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY revenue DESC;
```

### Advanced

**Q12. Cumulative revenue generated over time** *(window function — running total)*
```sql
SELECT order_date, SUM(revenue) OVER(ORDER BY order_date) AS cum_revenue
FROM (SELECT orders.order_date,
      SUM(order_details.quantity * pizzas.price) AS revenue
      FROM order_details
      JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
      JOIN orders ON orders.order_id = order_details.order_id
      GROUP BY orders.order_date) AS sales;
```

**Q13. Top 3 pizza types by revenue within each category** *(window function — RANK() + PARTITION BY)*
```sql
SELECT name, revenue FROM
(SELECT category, name, revenue,
 RANK() OVER(PARTITION BY category ORDER BY revenue DESC) AS rn
 FROM (SELECT pizza_types.category, pizza_types.name,
       SUM(order_details.quantity*pizzas.price) AS revenue
       FROM pizza_types
       JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
       JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
       GROUP BY pizza_types.category, pizza_types.name) AS a
) AS b
WHERE rn <= 3;
```

## 🔗 Connect

If you found this useful or have suggestions, feel free to connect or reach out!

**Anushka Singhal**
[https://www.linkedin.com/in/anushka-singhal-036785355]
