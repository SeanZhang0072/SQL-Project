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

SQL Queries:

Answer:



Question 3: 

SQL Queries:

Answer:



Question 4: 

SQL Queries:

Answer:



Question 5: 

SQL Queries:

Answer:
