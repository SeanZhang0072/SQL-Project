Question 1: 
What are the top 3 cities where visitors are most likely to make a purchase?

SQL Queries:
WITH city_purchasers AS (
    SELECT city, COUNT(DISTINCT fullvisitorid) AS purchasers
    FROM all_sessions al
    JOIN analytics a USING (fullvisitorid)
    WHERE a.revenue IS NOT NULL
    GROUP BY city
),
city_visitors AS (
    SELECT city, COUNT(DISTINCT fullvisitorid) AS visitors
    FROM all_sessions
    GROUP BY city
)
SELECT cp.city, (purchasers::decimal / visitors) * 100 AS purchase_probability
FROM city_purchasers cp
JOIN city_visitors cv ON cp.city = cv.city
ORDER BY purchase_probability DESC
LIMIT 3;


Answer: 
"South San Francisco"	25.00000000000000000000
"Milpitas"	22.22222222222222222200
"Jersey City"	20.00000000000000000000


Question 2: 
Which product is the most popular and sells the best?

SQL Queries:
SELECT p.name, SUM(a.units_sold) AS total_units_sold
FROM products p
JOIN all_sessions s ON p.SKU = s.productSKU
JOIN analytics a ON s.fullVisitorId = a.fullvisitorId
WHERE a.units_sold IS NOT NULL
GROUP BY p.name
ORDER BY total_units_sold DESC
LIMIT 1;

Answer:
" Collapsible Pet Bowl"	11328

Question 3: 
From which city do visitors contribute the most to the site's transaction-generated revenue?

SQL Queries:
SELECT s.city, SUM(s.totalTransactionRevenue) AS total_revenue
FROM all_sessions s
WHERE s.totalTransactionRevenue IS NOT NULL AND city <> 'not available in demo dataset'
GROUP BY s.city
ORDER BY total_revenue DESC
LIMIT 1;

Answer:
"San Francisco"	1564320000.00


Question 4: 
List products with more than 100 in stock and more than 40% sold in stock

SQL Queries:
SELECT p.name, p.stockLevel, COALESCE(ss.total_ordered, 0) AS total_sales
FROM products p
LEFT JOIN sales_by_sku ss ON p.SKU = ss.productSKU
WHERE p.stockLevel > 100 AND COALESCE(ss.total_ordered, 0) > p.stockLevel * 0.4
ORDER BY p.stockLevel DESC;

Answer:
" Blackout Cap"	415	189
" Leather Journal-Black"	354	250
"Android 17oz Stainless Steel Sport Bottle"	283	167
" Leather Journal"	181	82


Question 5: 
In which country do users view the most pages on average?

SQL Queries:
SELECT s.country, AVG(a.pageviews) AS average_pageviews
FROM all_sessions s
JOIN analytics a ON s.fullVisitorId = a.fullvisitorId
GROUP BY s.country
ORDER BY average_pageviews DESC
LIMIT 1;

Answer:
"Hong Kong"	20.5304659498207885
