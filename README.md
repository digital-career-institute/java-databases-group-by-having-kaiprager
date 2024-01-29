# Java- Databases: Group By & Having

1. Select all orders that have total price greater than 300.

2. Select only employees that have more than 100 orders.

3. Select the latest order (order_id, customer_id, order_date) for each customer. Include only orders that were made after 1998-05-05.

4. Select shipper company name and calculate how many orders have each.

5. Select the most expensive order.

6. Select an order that have the most items in it.

7. Select orders that was shipped by company with id = 1 and contains more than 4 items.

8. Select category id, category name and sumarize quantity of products for each category. Take into consideratation only categories with id: 1,2,3 and list only those where sum of products is bigger than 6000.

9. Make a report on the 10 best selling products with categories in 1997.

10. Select the least popular categories of products for orders made in 1997.




SELECT od.order_id, p.product_name, SUM(od.unit_price * od.quantity) AS total_price
	FROM order_details od
	JOIN products p ON od.product_id = p.product_id
	GROUP BY od.order_id, p.product_name
	HAVING SUM(od.unit_price * od.quantity) > 300;
	
SELECT e.employee_id, e.last_name, e.first_name, COUNT(DISTINCT o.order_id) AS total_orders
	FROM employees e
	JOIN orders o ON e.employee_id = o.employee_id
	GROUP By e.employee_id, e.last_name, e.first_name, o.order_id
	HAVING (o.order_id) > 100;
	
SELECT e.employee_id, e.last_name, e.first_name, total_orders
	FROM employees e
	JOIN (SELECT employee_id, COUNT(order_id) AS total_orders
	FROM orders
	GROUP BY employee_id
	HAVING COUNT(order_id) > 100)
	subquery ON e.employee_id = subquery.employee_id;
	
SELECT MAX(o.order_id) AS latest_order_id, o.customer_id, c.company_name, c.contact_name,
		MAX(order_date) AS latest_order_date
		FROM orders o
		LEFT JOIN customers c ON o.customer_id = c.customer_id
		WHERE order_date > '1998-05-05'
		GROUP BY o.customer_id, c.company_name, c.contact_name
		HAVING MAX(order_date) IS NOT NULL;
		
SELECT s.shipper_id, s.company_name, COUNT(o.order_id) AS number_of_orders FROM shippers s
	JOIN orders o ON s.shipper_id = o.ship_via
	GROUP BY s.shipper_id, s.company_name;
	
SELECT order_id, customer_id, order_date, total_price 
	FROM(SELECT o.order_id, o.customer_id, o.order_date, SUM(od.unit_price * od.quantity) AS total_price
		FROM orders o
		JOIN order_details od ON o.order_id = od.order_id
		GROUP BY o.order_id, o.customer_id, o.order_date
		ORDER BY
		total_price DESC
		LIMIT 1
		) AS most_expansive_order;
		
SELECT o.order_id, o.customer_id, o.order_date, COUNT(od.product_id) AS total_items
	FROM orders o
	JOIN order_details od ON o.order_id = od.order_id
	GROUP BY o.order_id, o.customer_id, o.order_date
	HAVING
    COUNT(o.product_id) = (SELECT MAX(item_count)
        FROM (SELECT o1.order_id, COUNT(od1.product_id) AS item_count
            FROM orders o1
            JOIN order_details od1 ON o1.order_id = od1.order_id
            GROUP BY o1.order_id) AS counts);
			
SELECT o.order_id, o.customer_id, o.order_date, s.company_name, COUNT(od.product_id) AS total_items
	FROM orders o
	JOIN shippers s ON o.ship_via = s.shipper_id
	JOIN order_details od ON o.order_id = od.order_id
	WHERE o.ship_via = 1
	GROUP BY o.order_id, o.customer_id, o.order_date, s.company_name
	HAVING COUNT(od.product_id) > 4;
	
SELECT c.category_id, c.category_name, SUM(od.quantity) AS total_quantity 
	FROM categories c
	JOIN products p ON c.category_id = p.category_id
	JOIN order_details od ON p.product_id = od.product_id
	WHERE c.category_id IN (1, 2, 3)
	GROUP BY c.category_id, c.category_name
	HAVING SUM(od.quantity) > 6000;
	
SELECT p.product_id, p.product_name, c.category_id, c.category_name, SUM(od.quantity) AS total_quantity_sold
	FROM orders o
	JOIN order_details od ON o.order_id = od.order_id
	JOIN products p ON od.product_id = p.product_id
	JOIN categories c ON p.category_id = c.category_id
	WHERE EXTRACT(YEAR FROM o.order_date) = 1997
	GROUP BY p.product_id, p.product_name, c.category_id, c.category_name
	HAVING SUM(od.quantity) > 100
	ORDER BY total_quantity_sold DESC LIMIT 10;
	
SELECT p.product_id, p.product_name, c.category_id, c.category_name, SUM(od.quantity) AS total_quantity_sold
	FROM orders o
	JOIN order_details od ON o.order_id = od.order_id
	JOIN products p ON od.product_id = p.product_id
	JOIN categories c ON p.category_id = c.category_id
	WHERE EXTRACT(YEAR FROM o.order_date) = 1997
	GROUP BY p.product_id, p.product_name, c.category_id, c.category_name
	HAVING SUM(od.quantity) < 100
	ORDER BY total_quantity_sold LIMIT 10;
