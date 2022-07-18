# 🍜 Case Study #1: Danny's Diner

## Solution

### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT s.customer_id, SUM(price) AS total_sales
FROM sales AS s
JOIN menu AS m
   ON s.product_id = m.product_id
GROUP BY customer_id; 
````

#### Steps:
- Use **SUM** and **GROUP BY** to find out ```total_sales``` contributed by each customer.
- Use **JOIN** to merge ```sales``` and ```menu``` tables as ```customer_id``` and ```price``` are from both tables.


#### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

***

### 2. How many days has each customer visited the restaurant?

````sql
SELECT customer_id, COUNT(DISTINCT(order_date)) AS visit_count
FROM sales
GROUP BY customer_id;
````

#### Steps:
- Use **DISTINCT** and wrap with **COUNT** to find out the ```visit_count``` for each customer.
- If we do not use **DISTINCT** on ```order_date```, the number of days may be repeated. For example, if Customer A visited the restaurant twice on '2021–01–07', then number of days is counted as 2 days instead of 1 day.

#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

***

### 3. What was the first item from the menu purchased by each customer?

````sql
WITH cte AS
(
   SELECT customer_id, order_date, product_name,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY s.order_date) AS rank
   FROM sales AS s
   JOIN menu AS m
      ON s.product_id = m.product_id
)

SELECT customer_id, product_name
FROM cte
WHERE rank = 1
GROUP BY customer_id, product_name;
````

#### Steps:
- Create a temp table ```cte``` and use **Windows function** with **DENSE_RANK** to create a new column ```rank``` based on ```order_date```.
- Instead of **ROW_NUMBER** or **RANK**, use **DENSE_RANK** as ```order_date``` is not time-stamped hence, there is no sequence as to which item is ordered first if 2 or more items are ordered on the same day.
- Subsequently, **GROUP BY** all columns to show ```rank = 1``` only.

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

- Customer A's first orders are curry and sushi.
- Customer B's first order is curry.
- Customer C's first order is ramen.

***

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT
  m.product_name,
  COUNT(s.product_id) AS order_count
FROM sales as s 
JOIN menu as m 
  ON s.product_id = m.product_id
GROUP BY
  m.product_name
ORDER BY order_count DESC
LIMIT 1;
````

#### Steps:
- **COUNT** number of ```product_id``` and **ORDER BY** ```order_count``` by descending order. 
- Then, use **Limit 1** to filter highest number of purchased item.

#### Answer:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |


- Most purchased item on the menu is ramen : 8 times. 

***

### 5. Which item was the most popular for each customer?

````sql
WITH cte AS
(
   SELECT s.customer_id, m.product_name, COUNT(m.product_id) AS order_count,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY COUNT(s.customer_id) DESC) AS rank
   FROM menu AS m
   JOIN sales AS s
      ON m.product_id = s.product_id
   GROUP BY s.customer_id, m.product_name
)

SELECT customer_id, product_name, order_count
FROM cte 
WHERE rank = 1;
````

#### Steps:
- Create a ```cte``` and use dense_rank to ```rank``` the ```order_count``` for each product by descending order for each customer.
- Generate results where product ```rank = 1``` only as the most popular product for each customer.

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customer A and C's favourite item is ramen.
- Customer B enjoys all items on the menu. the ideal customer !

***

### 6. Which item was purchased first by the customer after they became a member?

````sql
WITH member_sales_cte AS 
(
   SELECT s.customer_id, m2.join_date, s.order_date, s.product_id,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY s.order_date) AS rank
   FROM sales AS s
   JOIN members AS m2
      ON s.customer_id = m2.customer_id
   WHERE s.order_date >= m2.join_date
)

SELECT s.customer_id, s.order_date, m.product_name 
FROM member_sales_cte AS s
JOIN menu AS m
   ON s.product_id = m.product_id
WHERE rank = 1;
````

#### Steps:
- Create ```member_sales_cte``` by using dense_rank and partitioning ```customer_id``` by ascending ```order_date```. Then, filter ```order_date``` to be on or after ```join_date```.
- Then, filter table by ```rank = 1``` to show 1st item purchased by each customer.

#### Answer:
| customer_id | order_date  | product_name |
| ----------- | ---------- |----------  |
| A           | 2021-01-07 | curry        |
| B           | 2021-01-11 | sushi        |

- Customer A's first order as member is curry.
- Customer B's first order as member is sushi.

***

### 7. Which item was purchased just before the customer became a member?

````sql
WITH prior_member_purchased_cte AS 
(
   SELECT s.customer_id, m2.join_date, s.order_date, s.product_id,
         DENSE_RANK() OVER(PARTITION BY s.customer_id
         ORDER BY s.order_date DESC) AS rank
   FROM sales AS s
   JOIN members AS m2
      ON s.customer_id = m2.customer_id
   WHERE s.order_date < m2.join_date
)

SELECT s.customer_id, s.order_date, m.product_name 
FROM prior_member_purchased_cte AS s
JOIN menu AS m
   ON s.product_id = m.product_id
WHERE rank = 1;
````

#### Steps:
- Create a ```prior_member_purchased_cte``` to create new column ```rank``` by using dense_rank and partitioning ```customer_id``` by descending ```order_date``` to find out the last ```order_date``` before customer becomes a member.
- Filter ```order_date``` before ```join_date```.

#### Answer:
| customer_id | order_date  | product_name |
| ----------- | ---------- |----------  |
| A           | 2021-01-01 |  sushi        |
| A           | 2021-01-01 |  curry        |
| B           | 2021-01-04 |  sushi        |

- Customer A’s last order before becoming a member is sushi and curry.
- Customer B, it's sushi.

***

### 8. What is the total items and amount spent for each member before they became a member?

````sql
SELECT s.customer_id, COUNT(DISTINCT s.product_id) AS unique_menu_item, 
   SUM(m.price) AS total_sales
FROM sales AS s
JOIN members AS m2
   ON s.customer_id = m2.customer_id
JOIN menu AS m
   ON s.product_id = m.product_id
WHERE s.order_date < m2.join_date
GROUP BY s.customer_id;

````

#### Steps:
- Filter ```order_date``` before ```join_date``` and perform a **COUNT** **DISTINCT** on ```product_id``` and **SUM** the ```total spent``` before becoming member.

#### Answer:
| customer_id | unique_menu_item | total_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 2 |  40       |

Before becoming members,
- Customer A has spent $ 25 on 2 items.
- Customer B has spent $40 on 2 items.

***

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

````sql
WITH price_points_cte AS
(
   SELECT *, 
      CASE
         WHEN product_id = 1 THEN price * 20
         ELSE price * 10
      END AS points
   FROM menu
)

SELECT s.customer_id, SUM(p.points) AS total_points
FROM price_points_cte AS p
JOIN sales AS s
   ON p.product_id = s.product_id
GROUP BY s.customer_id
order by customer_id
````

#### Steps:
Let’s breakdown the question.
- Each $1 spent = 10 points.
- But, sushi (product_id 1) gets 2x points, meaning each $1 spent = 20 points
So, we use CASE WHEN to create conditional statements
- If product_id = 1, then every $1 price multiply by 20 points
- All other product_id that is not 1, multiply $1 by 10 points
Using ```price_points```, **SUM** the ```points```.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Total points for Customer A is 860.
- Total points for Customer B is 940.
- Total points for Customer C is 360.

***

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?

````sql
WITH count_points_cte AS (
    SELECT
      s.customer_id,
      order_date,
      join_date,
      product_name,
      SUM(point) AS point
    FROM
      sales AS s
      JOIN (
        SELECT
          product_id,
          product_name,
          CASE
            WHEN product_name = 'sushi' THEN price * 20
            ELSE price * 10
          END AS point
        FROM
          menu AS m
      ) AS p ON s.product_id = p.product_id
      JOIN members AS mm ON s.customer_id = mm.customer_id
    GROUP BY
      s.customer_id,
      order_date,
      join_date,
      product_name,
      point
  )
SELECT
  customer_id,
  SUM(
    CASE
      WHEN order_date >= join_date
      AND order_date < join_date + (7 * INTERVAL '1 day')
      AND product_name != 'sushi' THEN point * 2
      ELSE point
    END
  ) AS new_points
FROM
  count_points_cte
WHERE
  DATE_PART('month', order_date) = 1
GROUP BY
  1
ORDER BY
  1
````

#### Steps:

-Let's count points as usual: 10 points for each dollar spent on :curry: and :ramen: and 20 points for each dollar spent on :sushi:. 
We add this calculation to the `CTE` using `WITH` statement. Then we use this `CTE` to add extra 10 points for all the purchases of :curry: and :ramen: made by customers on the first week of their membership and return the sum of new points. The points for :sushi: remain the same - 20 points.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1370 |
| B           | 820 |

- Total points for Customer A is 1,370.
- Total points for Customer B is 820.

***
