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


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







