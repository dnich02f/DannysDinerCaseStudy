**Schema (PostgreSQL v13)**

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

    WITH first_item_cte AS 
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
        FROM first_item_cte
        WHERE ranked = 1
        ;

| customer_id | product_name | order_date               | join_date                | ranked |
| ----------- | ------------ | ------------------------ | ------------------------ | ------ |
| A           | sushi        | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z | 1      |
| A           | curry        | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z | 1      |
| B           | sushi        | 2021-01-04T00:00:00.000Z | 2021-01-09T00:00:00.000Z | 1      |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/dtT4rRi6ZySgc4YLLe5uBq/0)
