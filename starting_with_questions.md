Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
---The Highest Level City---

select city, sum(totaltransactionrevenue) as Citytotaltransactionrevenues
from public.all_sessions
where city <> 'not available in demo dataset'
group by city
order by sum(totaltransactionrevenue) desc
limit 1

---The Highest Level Country---

select country, sum(totaltransactionrevenue) as countrytotaltransactionrevenues
from public.all_sessions
where country <> 'not available in demo dataset'
group by country
order by sum(totaltransactionrevenue) desc
limit 1

Answer:
City: "San Francisco"	1564320000.00
Country: "United States"	13154170000.00


**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
---Avg for each City---

select city, avg(productquantity) as AvgCityordersquantity
from public.all_sessions
where city <> 'not available in demo dataset' and productquantity is not null
group by city
order by avg(productquantity) desc

---Avg for each Country---

select country, avg(productquantity) as AvgCountryordersquantity
from public.all_sessions
where city <> 'not available in demo dataset' and productquantity is not null
group by country
order by avg(productquantity) desc


Answer:
City:
"Madrid"	10.0000000000000000
"Salem"	8.0000000000000000
"Atlanta"	4.0000000000000000
"Houston"	2.0000000000000000
"New York"	1.1666666666666667
"Dallas"	1.00000000000000000000
"Detroit"	1.00000000000000000000
"Dublin"	1.00000000000000000000
"Los Angeles"	1.00000000000000000000
"Mountain View"	1.00000000000000000000
"Palo Alto"	1.00000000000000000000
"San Francisco"	1.00000000000000000000
"San Jose"	1.00000000000000000000
"Seattle"	1.00000000000000000000
"(not set)"	1.00000000000000000000
"Sunnyvale"	1.00000000000000000000
"Ann Arbor"	1.00000000000000000000
"Bengaluru"	1.00000000000000000000
"Chicago"	1.00000000000000000000
"Columbus"	1.00000000000000000000

Country:
"Spain"	10.0000000000000000
"United States"	1.4000000000000000
"India"	1.00000000000000000000
"Ireland"	1.00000000000000000000


**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

---In order to have a clearer understanding of the number of orders placed for the product category in the group, all the lower-level categories are merged into the first level in the v2productcategory (except for Home, which accounts for too much and is not representative at all)---

SQL Queries:

---For City----

select city, v2productcategory, count(v2productcategory) as orderedtimes
from all_sessions
where city <> '(not set)' 
and city <> 'not available in demo dataset'
and v2productcategory <> '(not set)' 
and productquantity >0
group by city, v2productcategory
order by city, count(v2productcategory) desc

---For Country---

select country, v2productcategory, count(v2productcategory) as orderedtimes
from all_sessions
where country <> '(not set)' 
and country <> 'not available in demo dataset'
and v2productcategory <> '(not set)' 
and productquantity >0
group by country, v2productcategory
order by country, count(v2productcategory) desc


Answer:

City:
It can be seen from the few records of confirmed order purchases that 'Nest-USA' and 'Apparel' are two relatively popular product categories.

Country:
The United States is the country where the trend can be seen most. It is obvious that the category of top3 is:
"United States" "Nest-USA" 16
"United States" "Apparel" 8
"United States" "CatTitle" 5


**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**
---When I ranked cities by sales, I found that there were only a few cities with visual differences in rankings:  "Mountain View" "New York" "San Francisco" "Sunnyvale" "San Jose" "Los Angeles" "Chicago" "Seattle" "Austin" "Palo Alto". So I take these ten cities as an example to check the best-selling products.---

SQL Queries:

----City rank-----

WITH RankedProducts AS (
    SELECT 
        al.city,
        p.name,
        COUNT(p.name) AS topselling,
        ROW_NUMBER() OVER (PARTITION BY al.city ORDER BY COUNT(p.name) DESC) AS rank
    FROM all_sessions as al
    JOIN products as p
    ON al.productsku = p.sku
    WHERE al.city IN ('Mountain View', 'New York', 'San Francisco', 'Sunnyvale', 'San Jose', 'Los Angeles', 'Chicago', 'Seattle', 'Austin', 'Palo Alto')
    GROUP BY al.city, p.name
)
SELECT
    city,
    name,
    topselling
FROM RankedProducts
WHERE rank <= 5
ORDER BY city, topselling DESC;

-----Country Rank-----

WITH RankedProducts AS (
    SELECT 
        al.country, 
        p.name, 
        COUNT(p.name) AS topselling,
        ROW_NUMBER() OVER (PARTITION BY al.country ORDER BY COUNT(p.name) DESC) AS rank
    FROM all_sessions as al
    JOIN products as p
    ON al.productsku = p.sku
    GROUP BY al.country, p.name
    HAVING COUNT(p.name) > 5
)
SELECT
    country,
    name,
    topselling
FROM RankedProducts
WHERE rank <= 3
ORDER BY country, topselling DESC;

Answer:

City：
It can be seen that the sales rankings of different cities are different, but most of them have products related to "Men's Short Sleeve" in Top05.

Country:
It can be seen that in most countries, the best-selling products are clothing and hats.



**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

WITH CityRevenue AS (
    SELECT
        al.city,
        SUM(a.revenue) AS CityRevenue,
        SUM(a.revenue) / (SELECT SUM(revenue) FROM analytics WHERE revenue IS NOT NULL) * 100 AS CityPercentage
    FROM all_sessions al
    JOIN analytics a USING(fullvisitorid)
    WHERE a.revenue IS NOT NULL
    GROUP BY al.city
),

CountryRevenue AS (
    SELECT
        al.country,
        SUM(a.revenue) AS CountryRevenue,
        SUM(a.revenue) / (SELECT SUM(revenue) FROM analytics WHERE revenue IS NOT NULL) * 100 AS CountryPercentage
    FROM all_sessions al
    JOIN analytics a USING(fullvisitorid)
    WHERE a.revenue IS NOT NULL
    GROUP BY al.country
)

SELECT
    'City' AS GroupType,
    city AS Name,
    CityRevenue AS Revenue,
    CityPercentage AS PercentageOfTotal
FROM CityRevenue
UNION ALL
SELECT
    'Country' AS GroupType,
    country AS Name,
    CountryRevenue AS Revenue,
    CountryPercentage AS PercentageOfTotal
FROM CountryRevenue
ORDER BY GroupType, Revenue DESC;

Answer:
group type   name        revenue                    percentage of total
"City"	"not available in demo dataset"	83094339938.00	7.11985906306852182600
"City"	"Mountain View"	10549489996.00	0.90392296292159643800
"City"	"San Bruno"	4230990000.00	0.36252833249206915600
"City"	"New York"	3485789992.00	0.29867658236424644900
"City"	"Sunnyvale"	3039259992.00	0.26041608628468028600
"City"	"Chicago"	1383630000.00	0.11855501352780357500
"City"	"Charlotte"	1161389993.00	0.09951259103313075000
"City"	"Kirkland"	1103520000.00	0.09455405601801189700
"City"	"San Francisco"	987449998.00	0.08460870888237434300
"City"	"Los Angeles"	942999999.00	0.08080005322099387700
"City"	"Jersey City"	933850000.00	0.08001604430587611400
"City"	"Austin"	861549996.00	0.07382108759614862000
"City"	"Seattle"	796899997.00	0.06828161424993793700
"City"	"Palo Alto"	709000000.00	0.06074998705666452300
"City"	"Milpitas"	494300000.00	0.04235362285205821400
"City"	"Toronto"	487670000.00	0.04178553764164116800
"City"	"Ann Arbor"	442219999.00	0.03789119776508913700
"City"	"Salem"	346020000.00	0.02964839283687878500
"City"	"San Jose"	239740000.00	0.02054189266144535000
"City"	"Cambridge"	165420000.00	0.014173854525971009109300
"City"	"Fremont"	124000000.00	0.010624821431631030888400
"City"	"Atlanta"	83000000.00	0.007111775635688512610800
"City"	"South San Francisco"	82420000.00	0.007062078890282496498600
"City"	"Denver"	41980000.00	0.003597016158869924812100
"City"	"Yokohama"	30880000.00	0.002645923272651340595400
"City"	"Zurich"	16990000.00	0.001455771904221058183800
"Country"	"United States"	115229279900.00	9.87331066639460299800
"Country"	"Canada"	487670000.00	0.04178553764164116800
"Country"	"Germany"	69980000.00	0.005996169385367254367500
"Country"	"Japan"	30880000.00	0.002645923272651340595400
"Country"	"Switzerland"	16990000.00	0.001455771904221058183800

Some Conclusions by results:

1. The majority of revenue is generated in the United States, accounting for approximately 9.87% of total revenue.

2. Among the cities, 'Mountain View' is the second-highest revenue-generating city, contributing around 0.90% of total revenue. It's worth noting that the top revenue-generating 'city' is marked as "not available in demo dataset", which suggests that the data for the highest revenue-generating city is not disclosed and that the undisclosed city contributes around 7.12% of the total revenue.

3. Other cities like 'San Bruno', 'New York', and 'Sunnyvale' also have significant contributions, with 0.36%, 0.30%, and 0.26% of total revenue, respectively.

4. Outside of the United States, Canada is the highest revenue-generating country, contributing around 0.042% of total revenue. This is significantly lower compared to the revenue generated in the United States.

5. Cities such as 'Chicago', 'Charlotte', 'Kirkland', 'San Francisco', 'Los Angeles' and others are also contributing, but with smaller shares in total revenue.

6. There are also revenues coming from countries like Germany, Japan, and Switzerland, but their contribution to the total revenue is very small.



