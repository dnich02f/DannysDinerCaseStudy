**Dannys Diner SQL Case Study** 
\
*Special Thanks to Danny Ma for creating this challenge*

![DannysDinerCaseStudyResized2](https://user-images.githubusercontent.com/90117717/154757160-f683ee49-c386-4911-907e-6bb386692bf8.png)

Schema and Queries created with PostgreSQL v13

    CREATE SCHEMA dannys_diner;
    SET search_path = dannys_diner;
    
    CREATE TABLE sales (
      "customer_id" VARCHAR(1),
      "order_date" DATE,
      "product_id" INTEGER
    );
    
    INSERT INTO sales
      ("customer_id", "order_date", "product_id")
    VALUES
      ('A', '2021-01-01', '1'),
      ('A', '2021-01-01', '2'),
      ('A', '2021-01-07', '2'),
      ('A', '2021-01-10', '3'),
      ('A', '2021-01-11', '3'),
      ('A', '2021-01-11', '3'),
      ('B', '2021-01-01', '2'),
      ('B', '2021-01-02', '2'),
      ('B', '2021-01-04', '1'),
      ('B', '2021-01-11', '1'),
      ('B', '2021-01-16', '3'),
      ('B', '2021-02-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-07', '3');
     
    
    CREATE TABLE menu (
      "product_id" INTEGER,
      "product_name" VARCHAR(5),
      "price" INTEGER
    );
    
    INSERT INTO menu
      ("product_id", "product_name", "price")
    VALUES
      ('1', 'sushi', '10'),
      ('2', 'curry', '15'),
      ('3', 'ramen', '12');
      
    
    CREATE TABLE members (
      "customer_id" VARCHAR(1),
      "join_date" DATE
    );
    
    INSERT INTO members
      ("customer_id", "join_date")
    VALUES
      ('A', '2021-01-07'),
      ('B', '2021-01-09');

---

**Query #1** 

 What is the total amount each customer spent at Dannys Diner?


    SELECT
    	t1.customer_id,
        sum(t2.price)
    FROM
    	dannys_diner.sales t1
    JOIN dannys_diner.menu t2 on (t1.product_id = t2.product_id)
    GROUP BY t1.customer_id
    ORDER BY t1.customer_id
    ;

| customer_id | sum |
| ----------- | --- |
| A           | 76  |
| B           | 74  |
| C           | 36  |

---
**Query #2**

How many days has each customer visited Dannys Diner?

    SELECT
    	customer_id,
        count( distinct order_date) as "NumberOfVisits"
    FROM
    	dannys_diner.sales
    GROUP BY customer_id
    ORDER BY "NumberOfVisits" desc
    ;

| customer_id | NumberOfVisits |
| ----------- | -------------- |
| B           | 6              |
| A           | 4              |
| C           | 2              |

---
**Query #3**

 What was the first item from the menu purchased by each customer?

    WITH CTE_FIRST_ITEM AS
    (
    SELECT 
    	t1.customer_id,
        t2.product_name,
        t1.order_date,
      	DENSE_RANK() OVER (PARTITION BY t1.customer_id ORDER BY t1.order_date) as rank
    FROM 
    	dannys_diner.sales t1
    JOIN dannys_diner.menu t2 on (t1.product_id = t2.product_id)
    )
    Select distinct *
    FROM CTE_FIRST_ITEM
    WHERE rank = 1
    ;

| customer_id | product_name | order_date               | rank |
| ----------- | ------------ | ------------------------ | ---- |
| A           | curry        | 2021-01-01T00:00:00.000Z | 1    |
| A           | sushi        | 2021-01-01T00:00:00.000Z | 1    |
| B           | curry        | 2021-01-01T00:00:00.000Z | 1    |
| C           | ramen        | 2021-01-01T00:00:00.000Z | 1    |

---
**Query #4**

What is the most purchased item on the menu and how many times was it purchased by all customers?

    SELECT
    	t2.product_name,
        count(t1.product_id) as number_of_purchases
    FROM 
    	dannys_diner.sales t1
        JOIN dannys_diner.menu t2 on (t1.product_id = t2.product_id)
    GROUP BY t2.product_name
    LIMIT 1
    ;

| product_name | number_of_purchases |
| ------------ | ------------------- |
| ramen        | 8                   |

---
**Query #5**

Which item was the most popular for each customer?


    WITH pop_item AS
    (
    SELECT
    	t1.customer_id,
        t2.product_name,
        count(t2.product_name) as times_ordered,
      	DENSE_RANK() OVER (PARTITION BY t1.customer_id ORDER BY count(t1.customer_id) desc) as ranked
    FROM 
    	dannys_diner.sales t1
        JOIN dannys_diner.menu t2 on (t1.product_id = t2.product_id)
    GROUP BY t1.customer_id, t2.product_name
    )
     
    SELECT customer_id, product_name, times_ordered
    FROM pop_item
    WHERE ranked = 1
    ;

| customer_id | product_name | times_ordered |
| ----------- | ------------ | ------------- |
| A           | ramen        | 3             |
| B           | ramen        | 2             |
| B           | curry        | 2             |
| B           | sushi        | 2             |
| C           | ramen        | 3             |

---
**Query #6**

Which item was purchased first by the customer after they became a member?

    WITH first_item_cte AS 
    (
    SELECT
    	t1.customer_id,
        t2.product_name,
        t1.order_date,
        t3.join_date,
        dense_rank () OVER (PARTITION BY t1.customer_id ORDER BY t1.order_date asc) as ranked
    FROM 
    	dannys_diner.sales t1
        JOIN dannys_diner.menu t2 on (t1.product_id = t2.product_id)
        JOIN dannys_diner.members t3 on (t1.customer_id = t3.customer_id)
    WHERE t1.order_date >= t3.join_date
    )
    SELECT * 
    FROM first_item_cte
    WHERE ranked = 1
    ;

| customer_id | product_name | order_date               | join_date                | ranked |
| ----------- | ------------ | ------------------------ | ------------------------ | ------ |
| A           | curry        | 2021-01-07T00:00:00.000Z | 2021-01-07T00:00:00.000Z | 1      |
| B           | sushi        | 2021-01-11T00:00:00.000Z | 2021-01-09T00:00:00.000Z | 1      |

---
**Query #7**

Which item was purchased just before the customer became a member?

    WITH prior_item_cte AS 
    (
    SELECT
        t1.customer_id,
        t2.product_name,
        t1.order_date,
        t3.join_date,
       dense_rank () OVER (PARTITION BY t1.customer_id ORDER BY t1.order_date desc) as ranked
    FROM 
        dannys_diner.sales t1
         JOIN dannys_diner.menu t2 on (t1.product_id = t2.product_id)
         JOIN dannys_diner.members t3 on (t1.customer_id = t3.customer_id)
    WHERE t1.order_date < t3.join_date
     )
    SELECT * 
    FROM prior_item_cte
    WHERE ranked = 1
    ;

| customer_id | product_name | order_date               | join_date                | ranked |
| ----------- | ------------ | ------------------------ | ------------------------ | ------ |
| A           | sushi        | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z | 1      |
| A           | curry        | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z | 1      |
| B           | sushi        | 2021-01-04T00:00:00.000Z | 2021-01-09T00:00:00.000Z | 1      |

---
**Query #8**

What is the total items and amount spent for each member before they became a member?

    SELECT 
    	t1.customer_id,
        count(t1.product_id) as total_items,
        sum(t2.price) as amount_spent
    FROM 
    	dannys_diner.sales t1
        JOIN dannys_diner.menu t2 on (t1.product_id = t2.product_id)
        JOIN dannys_diner.members t3 on (t1.customer_id = t3.customer_id)
    WHERE
    	t1.order_date < t3.join_date
    GROUP BY 
    	t1.customer_id
    ;

| customer_id | total_items | amount_spent |
| ----------- | ----------- | ------------ |
| B           | 3           | 40           |
| A           | 2           | 25           |

---
**Query #9**

If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

    WITH price_points_cte AS
    (
    SELECT 
    	*,
        CASE
        	WHEN t2.product_name = 'sushi' THEN t2.price * 20
            ELSE t2.price * 10
        END as points_earned
    FROM 
    	dannys_diner.menu t2
    )
    SELECT 
    	t1.customer_id,
    	sum(t2.points_earned)
    FROM
    	price_points_cte t2
        JOIN dannys_diner.sales t1 on(t1.product_id = t2.product_id)
    GROUP BY
    	t1.customer_id
    ORDER BY 
    	2 desc
    ;

| customer_id | sum |
| ----------- | --- |
| B           | 940 |
| A           | 860 |
| C           | 360 |

---
**Query #10**

In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

    WITH first_week_cte AS 
    (
      SELECT
      	t1.customer_id,
      	t1.order_date,
      	t2.product_id,
      	t2.product_name,
      	t2.price,
      	t3.join_date,
      	CASE
      		WHEN t1.order_date BETWEEN t3.join_date AND (t3.join_date + 6) AND t2.product_name = 'sushi' THEN t2.price * 2 * 20
      		WHEN t1.order_date BETWEEN t3.join_date AND (t3.join_date + 6) THEN t2.price * 2 * 10
      		WHEN t2.product_name = 'sushi' THEN t2.price * 20
      		ELSE t2.price * 1 * 10
      	END as point_bonus
      	
      FROM
      	dannys_diner.sales t1
      	JOIN dannys_diner.menu t2 on (t1.product_id = t2.product_id)
      	JOIN dannys_diner.members t3 on (t1.customer_id = t3.customer_id)
    )
    
    SELECT
    	customer_id,
        sum(point_bonus)
    FROM 
    	first_week_cte
    GROUP BY
        customer_id
    ;

| customer_id | sum  |
| ----------- | ---- |
| A           | 1370 |
| B           | 1140 |

---
**BONUS: Query #11**

The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL. 
--CREATE TABLE joined_v1 AS


    SELECT 
    	t1.customer_id, 
        t1.order_date, 
        t2.product_name, 
        t2.price,
          CASE
          	WHEN t3.join_date > t1.order_date THEN 'N'
           	WHEN t3.join_date <= t1.order_date THEN 'Y'
           ELSE 'N'
          END as member
    FROM 
    	dannys_diner.sales t1
    	LEFT JOIN dannys_diner.menu t2 on t1.product_id = t2.product_id
    LEFT JOIN dannys_diner.members t3 on t1.customer_id = t3.customer_id
    ;

| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |

---
**BONUS: Query #12**

Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program. Let rank the table we just created.

    WITH ranked_joined_cte AS
    (
    SELECT 
    	t1.customer_id, 
        t1.order_date, 
        t2.product_name, 
        t2.price,
          CASE
          	WHEN t3.join_date > t1.order_date THEN 'N'
           	WHEN t3.join_date <= t1.order_date THEN 'Y'
           ELSE 'N'
          END as member
    FROM 
    	dannys_diner.sales t1
    	LEFT JOIN dannys_diner.menu t2 on t1.product_id = t2.product_id
    LEFT JOIN dannys_diner.members t3 on t1.customer_id = t3.customer_id
    )
    
    SELECT 
    	*, 
        CASE
                WHEN member = 'N' then NULL
     	ELSE RANK () OVER(PARTITION BY customer_id, member
      	ORDER BY order_date) END as ranked
    FROM ranked_joined_cte
    ;

| customer_id | order_date               | product_name | price | member | ranked |
| ----------- | ------------------------ | ------------ | ----- | ------ | ------ |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |        |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |        |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      | 1      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      | 2      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |        |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |        |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |        |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      | 1      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      | 2      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      | 3      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |        |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |        |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |        |

---

# Recommendations and Insights #
* Members spent more than 2 times as much money as non members
* Members visited 2 - 3 times more than non members
* Customers whose first purchase included Curry became members 
* Ramen was the most purchased item on the menu for a total of 8 times! Tasty!
* Customers who became members ordered Sushi the meal before they became members
* Customers who became members ordered 2 items each before they became members
* Customer A and C's most popular order was Ramen whereas Customer B ordered the same amount of Sushi, Ramen, and Curry. It's too tough to choose from!

**Danny should feature Ramen at the top of the menu, and run specials to convert casual diners into loyal members!**

