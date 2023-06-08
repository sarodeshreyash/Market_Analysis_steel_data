# Market_Analysis_steel_data

```
# Q 1) How many transactions were completed during each marketing campaign?**

select campaign_name, count(transaction_id) as no_of_transactions
from transactions t
join marketing_campaigns m on t.purchase_date between m.start_date and m.end_date
group by campaign_name;

/* 2. Which product had the highest sales quantity? */
with cte as
(select t.product_id, product_name, sum(quantity) as total_sold_quantity
from transactions t  
inner join sustainable_clothing s  on t.product_id = s.product_id
group by t.product_id, product_name
order by total_sold_quantity desc)

select * 
from cte
where total_sold_quantity in (select max(total_sold_quantity) from cte);


/* Q 3  What is the total revenue generated from each marketing campaign? */

select campaign_name, CAST(SUM(t.quantity * s.price) AS NUMERIC(10, 2)) as total_revenue
from transactions t
join sustainable_clothing s on t.product_id = s.product_id
join marketing_campaigns m on t.purchase_date between m.start_date and end_date
group by campaign_name;

/* Q 4 What is the top-selling product category based on the total revenue generated? */

SELECT s.category, CAST(SUM(t.quantity * s.price) AS NUMERIC(10, 2)) AS revenue_generated
FROM sustainable_clothing s
JOIN transactions t ON t.product_id = s.product_id
GROUP BY s.category
order by revenue_generated desc
limit 1;

/* 5. Which products had a higher quantity sold compared to the average quantity sold? */

select t.product_id, s.product_name, t.quantity
from transactions t
join sustainable_clothing s on t.product_id = s.product_id
where t.quantity > (SELECT AVG(quantity) FROM transactions);


/* 6. What is the average revenue generated per day during the marketing campaigns? */

select t.purchase_date, (CAST(avg(t.quantity * s.price) AS NUMERIC(10, 2))) AS avg_revenue_generated
FROM sustainable_clothing s
JOIN transactions t ON t.product_id = s.product_id
group by 1
order by avg_revenue_generated desc;

/* 7. What is the percentage contribution of each product to the total revenue? */

SELECT s.product_id, s.product_name, CAST((SUM(t.quantity * s.price) / total_revenue) * 100 AS NUMERIC(10, 2)) AS contribution_percentage
FROM sustainable_clothing s
JOIN transactions t ON t.product_id = s.product_id
CROSS JOIN (SELECT SUM(t.quantity * s.price) AS total_revenue 
			FROM sustainable_clothing s 
			JOIN transactions t ON t.product_id = s.product_id) AS revenue_total
GROUP BY s.product_id, s.product_name, total_revenue
order by contribution_percentage desc;

/* 8. Compare the average quantity sold during marketing campaigns to outside the marketing campaigns */

with cte as
(select avg(quantity) as avg_qty_during_campaign
from transactions t
join sustainable_clothing s on t.product_id=s.product_id
join marketing_campaigns m on t.purchase_date between m.start_date and m.end_date
	and t.product_id=m.product_id),

cte2 as
(select avg(quantity) as total_avg_qty
from transactions t
join sustainable_clothing s on t.product_id=s.product_id)

select total_avg_qty, avg_qty_during_campaign, 
	total_avg_qty-avg_qty_during_campaign as avg_qty_outside_campaign
from cte, cte2;

/* Q 9 Compare the revenue generated by products inside the marketing campaigns to outside the campaigns */

WITH cte AS (
  SELECT round(sum(quantity * price)::numeric, 2) AS rev_during_campaign
  FROM transactions t
  JOIN sustainable_clothing s ON t.product_id = s.product_id
  JOIN marketing_campaigns m ON t.purchase_date BETWEEN m.start_date AND m.end_date
    AND t.product_id = m.product_id
),
cte2 AS (
  SELECT round(sum(quantity * price)::numeric, 2) AS total_revenue
  FROM transactions t
  JOIN sustainable_clothing s ON t.product_id = s.product_id
)

SELECT cte2.total_revenue, cte.rev_during_campaign, 
       cte2.total_revenue - cte.rev_during_campaign AS rev_outside_campaign
FROM cte, cte2;

/* 10 Rank the products by their average daily quantity sold */

with cte as
(select product_name, avg(quantity) as avg_sold_qty 
from transactions t
join sustainable_clothing s on t.product_id=s.product_id 
group by 1)

select product_name, avg_sold_qty, 
	dense_rank() over(order by avg_sold_qty) as avg_rank 
from cte;
```
