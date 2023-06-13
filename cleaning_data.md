---Here is a link sharing all the notes I took during Cleaning Data. The issues found below, as well as the queries used to solve the issues, are detailed in the notes.---

---https://docs.google.com/document/d/1nUb6Jd6mmsi-2q-qvwA2EkfDGv1XQnX9yHOPxjoc4SA/edit?usp=sharing---

What issues will you address by cleaning the data?
1. Duplicate Records
2. Data Format Issue
3. Missing Values
4. Invalid Column
5. Data Logic Issue
6. Category Chaos

Queries:

Below, provide the SQL queries you used to clean your data.

---1. Duplicate Records---

---(Visitid is an example of this issue)---

DELETE FROM all_sessions WHERE visitid IN (
    SELECT visitid FROM all_sessions
    GROUP BY visitid
    HAVING COUNT(visitid) > 1
)

---2. Data Format Issue---

alter table all_sessions add column newtime time
UPDATE all_sessions SET newtime = '00:00'::time + time * '1 second'::interval
ALTER TABLE all_sessions DROP COLUMN time
ALTER TABLE all_sessions RENAME COLUMN newtime TO time

---3. Missing Values---

UPDATE all_sessions
SET column_name = COALESCE(column_name, default_value)
WHERE column_name IS NULL

---For some columns, I will set null to 0 because they use 1 or 0 to express yes or no---
---For some columns, I will set null to 'N/A' or 'Not set'---
---the default_value is dependent on the datatype of the column.---

---4. Invalid Column---

ALTER TABLE all_sessions DROP COLUMN column_name

---5. Data Logic Issue---

---units_sold should be a positive value.---

DELETE FROM analytics WHERE units_sold < 0

---6. Category Chaos---

---v2productcategory has many categories with 'Home' beginning and no 'Home' beginning two versions.---

UPDATE all_sessions
SET v2productcategory = REPLACE(v2productcategory, 'Home/', '')
WHERE v2productcategory LIKE 'Home/%';

---v2productcategory has a category names '${escCatTitle}' should be fixed as 'CatTitle'.---

UPDATE all_sessions
SET v2productcategory = 'CatTitle'
WHERE v2productcategory = '${escCatTitle}'









