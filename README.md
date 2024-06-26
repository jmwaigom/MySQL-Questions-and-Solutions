# MySQL Problems and Solutions

![M1bswVs7zkuTaBacgsKxDCB4](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/52ae7224-ea1d-418e-a6e3-244b5b0f55f5)

## Overview
The questions in this project were sourced from Stratascratch, a platform that hosts past interview questions for MySQL and other languages such as Python. Below are a few select questions
that I have attempted. Some of the concepts that I have demonstrated include CTEs, subqueries, window functions and joins. For each question, I have included a snapshot of the table(s) used, 
a solution code and a snapshot of the result/outcome of the query (in green background).  

### Table of Contents
[Question 1](#question-1) : Average Age of Claims by Gender \
[Question 2](#question-2) : Pending Claims\
[Question 3](#question-3) : Distances Traveled\
[Question 4](#question-4) : Monthly Percentage Difference\
[Question 5](#question-5) : Cities With The Most Expensive Homes\
[Question 6](#question-6) : Revenue Over Time\
[Question 7](#question-7) : Product Transaction Count\
[Question 8](#question-8) : Ranking Hosts By Beds\
[Question 9](#question-9) : Find the number of employees who received the bonus and who didn't\
[Question 10](#question-10) : Days At Number One\
[Question 11](#question-11) : Find the number of inspections for each risk category by inspection type\
[Question 12](#question-12) : Find the top 5 least paid employees for each job title\
[Question 13](#question-13) : Employees Without Benefits\
[Question 14](#question-14) : Find the top 5 highest paid and top 5 least paid employees in 2012\
[Question 15](#question-15) : Worst Businesses\
[Question 16](#question-16) : Facilities With Lots Of Inspections\
[Question 17](#question-17) : Find the variance and the standard deviation of scores that have grade A\
[Question 18](#question-18) : 3rd Most Reported Health Issues\
[Question 19](#question-19) : Ad Performance Rating\
[Question 20](#question-20) : Best Selling Item\
[Question 21](#question-21) : Make a pivot table to find the highest payment in each year for each employee\
[Question 22](#question-22) : Population Density\
[Question 23](#question-23) : Activity Rank\
[Question 24](#question-24) : Manager of the Largest Department\
[Question 25](#question-25) : Average Customers Per City

## Question 1
You have been asked to calculate the average age by gender of people who filed more than 1 claim in 2021.
The output should include the gender and average age rounded to the nearest whole number.

Tables: cvs_claims, cvs_accounts
![Qn22a](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/68086a99-7b07-4608-9ec9-99eeda3f5375)
![Qn22b](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/444dcc9d-13c7-4e60-9319-b9892b0528a4)

### Solution

```
with maintable as (
    select
        a.account_id,
        age,
        gender,
        count(*) as number_of_claims
    from cvs_claims as a
    inner join cvs_accounts as b
    using(account_id)
    where year(date_submitted) = '2021'
    group by a.account_id, age, gender
    having count(*) > 1
    )

select
    gender,
    avg(age) as avg_age
from maintable
group by gender

```
![ans22](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/5017535d-d417-4445-94b7-3b4cc8835949)

## Question 2
Count how many claims submitted in December 2021 are still pending. A claim is pending when it has neither an acceptance nor rejection date.

Table: cvs_claims

![Qn23](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/d8bd41bd-5d7a-4f86-a34b-aea0c4afb4ba)

### Solution
```
select
    count(*) as total_pending_claims_Dec_2021
from cvs_claims 
where date_accepted is null
    and date_rejected is null
    and date_format(date_submitted, '%Y-%m') = '2021-12'

```
![Ans23](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/745332ea-eaba-4e31-8619-c0c0e20491bf)

## Question 3
Find the top 10 users that have traveled the greatest distance. Output their id, name and a total distance traveled

Tables: lyft_rides_log, lyft_users
![Qn3a](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/c951cf37-e570-4761-9642-dada0cc2efa7)
![Qn3b](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/f7a6b954-ff5f-4962-a9c7-4e769d7d80fe)

### Solution
```
-- Top 10 users
-- Group by id, name, total distance traveled

select
    lr.user_id,
    lu.name,
    sum(lr.distance) as total_distance_traveled
from lyft_rides_log as lr
inner join lyft_users as lu
on lr.user_id = lu.id
group by lr.user_id, lu.name
order by total_distance_traveled desc
limit 10

```
![answ3](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/f727325d-5636-4ac2-821c-d74387ee3208)

## Question 4
Given a table of purchases by date, calculate the month-over-month percentage change in revenue. The output should include the year-month date (YYYY-MM) and percentage change, rounded to the 2nd decimal point, and sorted from the beginning of the year to the end of the year.
The percentage change column will be populated from the 2nd month forward and can be calculated as ((this month's revenue - last month's revenue) / last month's revenue)*100.

Table: sf_transactions
![Qn4](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/25ebf660-bb2e-4fb5-83f1-c712f9490990)

### Solution 
```
with maintable1 as (
    select 
        date_format(created_at, '%Y-%m') as formatted_date,
        sum(value) as total_amount
    from sf_transactions
    group by date_format(created_at, '%Y-%m')
    order by created_at
    ),

    maintable2 as (
    select
        formatted_date,
        total_amount as current_amount,
        lag(total_amount) over(order by formatted_date) as prev_month_amount
    from maintable1
    )

select
    formatted_date,
    round((current_amount - prev_month_amount)/prev_month_amount * 100,2) as MoM_change
from maintable2
group by formatted_date

```
![answ4b](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/27953935-d891-48f2-a799-25e26bfe5cfb)

## Question 5
Write a query that identifies cities with higher than average home prices when compared to the national average. Output the city names.

Table: zillow_transactions

![Qn5](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/4f5bedf7-4422-4cc6-88f0-52dabc8db22f)

### Solution 
```
-- Compare city average to national average
-- Output: City names

select
    city 
from (
    select
        city,
        avg(mkt_price) as city_avg_mkt_price
    from zillow_transactions
    group by city
    ) as sub
where sub.city_avg_mkt_price > (
    select
        avg(mkt_price) as national_avg_mkt_price
    from zillow_transactions
    )

```
![answ5](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/2763d5ea-6417-4ca4-9810-2cd2a0fe8847)

## Question 6
Find the 3-month rolling average of total revenue from purchases given a table with users, their purchase amount, and date purchased. Do not include returns which are represented by negative purchase values. Output the year-month (YYYY-MM) and 3-month rolling average of revenue, sorted from earliest month to latest month.


A 3-month rolling average is defined by calculating the average total revenue from all user purchases for the current month and previous two months. The first two months will not be a true 3-month rolling average since we are not given data from last year. Assume each month has at least one purchase.

Table: amazon_purchases
![Qn6](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/b4d324e4-a9a9-410c-8587-0693c2bc90c9)

### Solution
```
select
    sub.date,
    avg(sub.monthly_avg_purchase_amt) over (order by sub.date rows between 2 preceding and current row) as 3_mos_rolling_avg
 from (
    select
        date_format(created_at, '%Y-%m') as date,
        sum(purchase_amt) as monthly_avg_purchase_amt
    from amazon_purchases
    where purchase_amt > 0
    group by date_format(created_at, '%Y-%m')
    order by date
    ) as sub

```
![answ6](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/78eca7ee-6b2d-46f4-8c5c-8ebcbb8aed40)


## Question 7
Find the number of transactions that occurred for each product. Output the product name along with the corresponding number of transactions and order records by the product id in ascending order. You can ignore products without transactions.

Tables: excel_sql_inventory_data, excel_sql_transaction_data
![Qn7a](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/7faac4f3-26bd-48b2-a5aa-56092488bbe6)
![Qn7b](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/84144be8-16da-4970-8efa-0860c46fbb89)

### Solution
```
-- Group by product
-- Count number of transactions
-- Output: Product name, count of transactions
-- Ignore products without transactions (inner join)

select
    a.product_name,
    count(*) as number_of_transactions
from excel_sql_inventory_data as a
inner join excel_sql_transaction_data as b
using(product_id)
group by product_name
order by a.product_id;

```
![ans7](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/53d520ec-56fb-491a-afb3-50913055b4b3)

## Question 8
Rank each host based on the number of beds they have listed. The host with the most beds should be ranked 1 and the host with the least number of beds should be ranked last. Hosts that have the same number of beds should have the same rank but there should be no gaps between ranking values. A host can also own multiple properties.
Output the host ID, number of beds, and rank from highest rank to lowest.

Table: airbnb_apartments
![Qn8](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/180c3583-0cda-4ee8-a542-7d9de53ae9d4)

### Solution
```
select
    host_id,
    sum(n_beds) as total_beds,
    dense_rank() over(order by sum(n_beds) desc) as ranking
from airbnb_apartments
group by host_id;

```
![ans8](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/e9edbc16-33ed-40ba-9318-5df77bc6c5e5)

## Question 9
Find the number of employees who received the bonus and who didn't. Bonus values in employee table are corrupted so you should use  values from the bonus table. Be aware of the fact that employee can receive more than one bonus.
Output value inside has_bonus column (1 if they had bonus, 0 if not) along with the corresponding number of employees for each.

Tables: employee, bonus
![Qn9a](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/da858df5-9365-43ff-85c9-f0c82dab716e)
![Qn9b](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/531bbd6f-6555-439d-bb03-4b7da9c544d8)

### Solution
```
-- Output: 1 for has_bonus and 0 for no_bonus (Group by these rows)
-- Count of employees for each value/row above

select
    sub.has_bonus,
    count(sub.id) as n_employee
from (
    select
        e.id,
        sum(b.bonus_amount) as total_bonus,
        (case when sum(b.bonus_amount) is null then 0 else 1 end) as has_bonus
    from employee as e
    left join bonus as b
    on e.id = b.worker_ref_id
    group by e.id
    ) as sub
group by sub.has_bonus

```
![Screenshot 2024-05-07 155837](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/1a91e70e-d2a7-4882-b840-583908b3c21c)

## Question 10
Find the number of days a US track has stayed in the 1st position for both the US and worldwide rankings on the same day. Output the track name and the number of days in the 1st position. Order your output alphabetically by track name.
If the region 'US' appears in dataset, it should be included in the worldwide ranking.

Tables: spotify_daily_rankings_2017_us, spotify_worldwide_daily_song_ranking
![Qn10a](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/aaa86e60-0b09-4dc1-a61b-ee74a2037de8)
![Qn10b](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/83b7d8fb-2963-4f1a-8c3f-fa7f14b80f58)

### Solution
```
-- Track has been #1
-- For both Worldwide and US 
-- Has to be same day
-- Order by track name alphabetically

select distinct
    a.trackname,
    count(*) as n_days
from spotify_daily_rankings_2017_us as a
left join spotify_worldwide_daily_song_ranking as b
on a.trackname = b.trackname and a.date = b.date
where b.position = 1
group by a.trackname
order by a.trackname;

```
![ans10](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/7d481071-b148-4d64-80f6-79c93550d687)

## Question 11
Find the number of inspections that resulted in each risk category per each inspection type.
Consider the records with no risk category value belongs to a separate category.
Output the result along with the corresponding inspection type and the corresponding total number of inspections per that type. The output should be pivoted, meaning that each risk category + total number should be a separate column.
Order the result based on the number of inspections per inspection type in descending order.

Table: sf_restaurant_health_violations
![Qn11](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/82403741-1b89-41cd-b94f-503b081b3b4f)

### Solution
```
select
    *,
    low_risk + moderate_risk + high_risk + no_risk as total_inspections
from (
    select
        inspection_type,
        sum(case when risk_category = 'Low Risk' then 1 else 0 end) as low_risk,
        sum(case when risk_category = 'Moderate Risk' then 1 else 0 end) as moderate_risk,
        sum(case when risk_category = 'High Risk' then 1 else 0 end) as high_risk,
        sum(case when risk_category is null then 1 else 0 end) as no_risk
    from sf_restaurant_health_violations
    group by inspection_type
    ) as sub
order by total_inspections desc;

```
![ans11](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/f3c73e64-5006-4ecd-86b6-e9c9847c7e7f)

## Question 12
Find the top 5 least paid employees for each job title.
Output the employee name, job title and total pay with benefits for the first 5 least paid employees. Avoid gaps in ranking.

Table: sf_public_salaries
![Qn12](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/837d7b90-cd0f-4fe4-bcb7-02e74977a053)

### Solution
```
select
    sub.employeename,
    sub.jobtitle,
    sub.totalpaybenefits
from (
    select
        employeename,
        jobtitle,
        totalpaybenefits,
        dense_rank() over(partition by jobtitle order by totalpaybenefits) as payrank
    from sf_public_salaries
    ) as sub
where payrank <= 5

```
![ans12](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/acdac8fb-4dd9-4056-bc9e-c91e863834e4)

## Question 13
Find the ratio between the number of employees without benefits to total employees. Output the job title, number of employees without benefits, total employees relevant to that job title, and the corresponding ratio. Order records based on the ratio in ascending order.

Table: sf_public_salaries
![Qn13](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/49de0577-b740-40a3-b9bb-382839b3707c)

### Solution
```
-- ratio =  employees without benefits/total employees
-- Output: Jobtitle, # employees without benefits, total employees, ratio

with maintable as (
    select  
        jobtitle,
        sum(case when benefits is null or benefits = 0 then 1 else 0 end) as n_employees_without_benefits,
        count(*) as total_employees
    from sf_public_salaries
    group by jobtitle
    ) 

select
    *,
    n_employees_without_benefits/total_employees as ratio
from maintable
order by ratio

```
![image](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/fee9a9eb-2116-4b09-b7ff-9a4deccda980)

## Question 14
Find the top 5 highest paid and top 5 least paid employees in 2012.
Output the employee name along with the corresponding total pay with benefits.
Sort records based on the total payment with benefits in ascending order.

Table: sf_public_salaries
![Qn15](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/007734f5-f37c-4181-bc8b-dd1cde1c5e53)

### Solution
```
-- Create a CTE which performs UNION on two tables based on common columns
-- Table 1 contains top 10 and Table 2 contains Bottom 10

with maintable as (
    select
        a.employeename,
        a.total_overall_pay
    from (
        select
            employeename,
            sum(totalpaybenefits) as total_overall_pay,
            rank() over(order by sum(totalpaybenefits) desc) as payrank
        from sf_public_salaries
        where year = 2012
        group by employeename
        order by total_overall_pay desc
        ) as a
    where payrank <= 5
    union
    select
        b.employeename,
        b.total_overall_pay
    from (
        select
            employeename,
            sum(totalpaybenefits) as total_overall_pay,
            rank() over(order by sum(totalpaybenefits) asc) as payrank
        from sf_public_salaries
        where year = 2012
        group by employeename
        order by total_overall_pay 
        ) as b
    where payrank <= 5
    )

select
    employeename,
    total_overall_pay
from maintable
order by total_overall_pay

```
![ans15](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/41c0e087-4b67-4a94-aaed-e61f877efc9c)

## Question 15
For every year, find the worst business in the dataset. The worst business has the most violations during the year. You should output the year, business name, and number of violations.

Table: sf_restaurant_health_violations
![Qn14](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/8b091da4-a8be-4ab3-90e7-18d54645e4df)

### Solution
```
-- Find number of violations per business per year (for every business)
-- Pick out the one with the most violations for every year

with maintable as (
    select
        year(inspection_date) as inspection_year,
        business_name,
        count(violation_id) as n_violations,
        rank() over(partition by year(inspection_date) order by count(violation_id) desc) as violations_ranking
    from sf_restaurant_health_violations
    group by year(inspection_date), business_name
    order by inspection_year
    )

select
    inspection_year,
    business_name,
    n_violations
from maintable
where violations_ranking = 1

```
![ans14](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/65b1ec92-0ee9-493a-bb3b-6f08dab3a15a)

## Question 16
Find the facility that got the highest number of inspections in 2017 compared to other years. Compare the number of inspections per year and output only facilities that had the number of inspections greater in 2017 than in any other year.
Each row in the dataset represents an inspection. Base your solution on the facility name and activity date fields.

Table: los_angeles_restaurant_health_inspections
![Qn16](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/a9d09e23-3259-43dd-988c-82f153549abf)

### Solution
```
-- Use subquery in the FROM to create a table/view with n_inspections for each year
-- From that table, compare 2017 to other years. 
-- Logically, number of inspections in 2017 MUST BE greater than 2015 AND 2016 AND 2018
select
    facility_name
from(
    select
        facility_name,
        sum(case when year(activity_date) = 2015 then 1 else 0 end) as `2015`,
        sum(case when year(activity_date) = 2016 then 1 else 0 end) as `2016`,
        sum(case when year(activity_date) = 2017 then 1 else 0 end) as `2017`,
        sum(case when year(activity_date) = 2018 then 1 else 0 end) as `2018`
    from los_angeles_restaurant_health_inspections
    group by facility_name
    order by `2015` + `2016` + `2017` + `2018` desc
    ) as sub
where `2017`> `2015` and
    `2017` > `2016` and
    `2017` > `2018`

```
![ans16](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/9bbc6fdc-7977-4aaf-9579-8e1141ae8d74)

## Question 17
Find the variance of scores that have grade A using the formula AVG((X_i - mean_x) ^ 2).
Output the result along with the corresponding standard deviation.

Table: los_angeles_restaurant_health_inspections
![Q17](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/5cbcfa1d-652c-4537-b35d-c84cf5d384b1)

### Solution
```
-- Standard deviation is square root of variance

select
    avg(power((score - avg_score),2)) as var,
    sqrt(avg(power((score - avg_score),2))) as std
from (
    select
        score,
        avg(score) over() as avg_score
    from los_angeles_restaurant_health_inspections
    where grade = 'A'
    order by score
    ) as x

```
![ans17](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/aa1494e4-5d76-429f-9beb-e72cf35172d6)

## Question 18
Each record in the table is a reported health issue and its classification is categorized by the facility type, size, risk score which is found in the pe_description column.


If we limit the table to only include businesses with Cafe, Tea, or Juice in the name, find the 3rd most common category (pe_description). Output the name of the facilities that contain 3rd most common category.

Table: los_angeles_restaurant_health_inspections
![Qn18](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/2a9dfbeb-e022-4d88-bb47-99c131ce3a30)

### Solution
```
/*
First half of this CTE extracts distinct facility name and pe_description
ony for facilities with 'Cafe', 'Tea' or 'Juice' in their name
*/

with table1 as (
    select
        distinct facility_name,
        pe_description
    from los_angeles_restaurant_health_inspections
    where facility_name like '%cafe%' or
        facility_name like '%tea%' or
        facility_name like '%juice%'
        ),
  
  /*
  This second half of the CTE groups all facilities by description, it counts
  all facilities in each description and ranks according to total facilities
  in each description
  */
  
    table2 as (
    select
        pe_description,
        count(*) as total_facilities,
        rank() over(order by count(*) desc) as rank_of_totals
    from los_angeles_restaurant_health_inspections
    where facility_name like '%cafe%' or
        facility_name like '%tea%' or
        facility_name like '%juice%'
    group by pe_description
    )
    
/*
This query joins tables 1 and 2 on pe_description column and extracts
facilities with rank = 3 (3rd most reported health issues)
*/

select
    t1.facility_name
from table1 as t1
inner join table2 as t2
using(pe_description)
where rank_of_totals = 3

```
![ans18](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/a7e8ec04-da76-4bf1-8269-84dc7f147f0a)

## Question 19
Following a recent advertising campaign, the marketing department wishes to classify its efforts based on the total number of units sold for each product.


You have been tasked with calculating the total number of units sold for each product and categorizing ad performance based on the following criteria for items sold:


Outstanding: 30+\
Satisfactory: 20 - 29\
Unsatisfactory: 10 - 19\
Poor: 1 - 9


Your output should contain the product ID, total units sold in descending order, and its categorized ad performance.

Table: marketing_campaign
![Qn19](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/ed5df6aa-5ba7-407f-8e7d-635190db3442)

### Solution
```
select
    product_id,
    sum(quantity) as total_units_sold,
    (case when sum(quantity) >= 30 then 'Outstanding'
        when sum(quantity) between 20 and 29 then 'Satisfactory'
        when sum(quantity) between 10 and 19 then 'Unsatisfactory'
        else 'Poor'
    end) as category
from marketing_campaign
group by product_id
order by total_units_sold desc;

```
![Screenshot 2024-05-14 114613](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/2f4131bf-6a97-47af-90ac-547a8016c876)

## Question 20
Find the best selling item for each month (no need to separate months by year) where the biggest total invoice was paid. The best selling item is calculated using the formula (unitprice * quantity). Output the month, the description of the item along with the amount paid.

Table: online_retail
![Qn20](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/5ce455b0-1fb8-4f06-bd2f-fedc4a5eced6)

### Solution
```
with maintable as (
    select
        month(invoicedate) as month,
        description,
        sum(quantity * unitprice) as total_amount,
        rank() over(partition by month(invoicedate) order by sum(quantity * unitprice) desc) as monthly_sales_ranking
    from online_retail
    group by month(invoicedate), description
    order by month
    )

select
    month,
    description,
    total_amount
from maintable
where monthly_sales_ranking = 1
order by month

```
![ans20](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/b6a0fb2e-dacf-425f-8bfb-4d673d550454)

## Question 21
Make a pivot table to find the highest payment in each year for each employee.
Find payment details for 2011, 2012, 2013, and 2014.
Output payment details along with the corresponding employee name.
Order records by the employee name in ascending order

Table: sf_public_salaries
![Qn21](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/bd4d5a04-e116-43cd-8cfb-a26e519515c8)

### Solution
```
select
    employeename,
    max(case when year = 2011 then totalpay else 0 end) as `2011`,
    max(case when year = 2012 then totalpay else 0 end) as `2012`,
    max(case when year = 2013 then totalpay else 0 end) as `2013`,
    max(case when year = 2014 then totalpay else 0 end) as `20114`
from sf_public_salaries
group by employeename
order by employeename;

```
![Ans21](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/18e69358-3026-4631-a4e8-3a7dd737b2ea)

## Question 22
You are working on a data analysis project at Deloitte where you need to analyze a dataset containing information
about various cities. Your task is to calculate the population density of these cities, rounded to the nearest integer, and identify the cities with the minimum and maximum densities.
The population density should be calculated as (Population / Area). The output should contain 'city', 'country', 'density'.

Table: cities_population

<img width="679" alt="Screenshot 2024-05-07 at 11 21 57 AM" src="https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/f564417f-c293-4cd8-9177-dccc7c9d1b91">

### Solution

```
/*
objectives:
-- Calculate population density rounded to nearest integer
-- Identify cities with minimum and maximum densities
-- Population density = Population/Area
-- Group by City, Country, Density
*/

select
    city,
    country,
    population_density as density 
from (
select 
    country,
    city,
    round(sum(population)/area) as population_density,
    rank() over(order by (sum(population)/area) desc) as pop_density_ranking
from cities_population
where area > 0
group by country, city
order by population_density desc
) as sub
where pop_density_ranking in (1,7)

```
![Deloitte](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/9da411c9-80ef-4262-838f-ad32c2aed4e7)

## Question 23
Find the email activity rank for each user. Email activity rank is defined by the total number of emails sent. The user with the highest number of emails sent will have a rank of 1, and so on. Output the user, total emails, and their activity rank. Order records by the total emails in descending order. Sort users with the same number of emails in alphabetical order.
In your rankings, return a unique value (i.e., a unique rank) even if multiple users have the same number of emails. For tie breaker use alphabetical order of the user usernames.

Table: google_gmail_emails
![Qn2](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/e61abefc-2d42-44d6-a6de-1d0771f4fe9d)

### Solution
```
-- Ranking (use row_number for unique ranking) 
-- Output user, total emails and ranking
-- Order by total emails in desc order
-- For users with same number of emails, order alphabetically

with maintable as (
    select
        from_user as user,
        count(*) as n_emails
    from google_gmail_emails
    group by from_user
    order by n_emails desc, user
    )
    
select
    user,
    n_emails,
    row_number() over(order by n_emails desc, user asc) as total_emails_ranking
from maintable

```
![goole](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/c27d0629-f513-4cad-89f7-0547d805dc87)

## Question 24
Given a list of a company's employees, find the name of the manager from the largest department. Manager is each employee that contains word "manager" under their position.  Output their first and last name.

Table: az_employees
![Qn24](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/3f38f9a1-58df-40ea-a974-fdc7d983516b)

### Solution 
```
-- Find number of employees per department
-- The largest department is the one with the most employees. Output its manager's name

with maintable1 as (
    select
        first_name,
        last_name,
        department_name,
        position,
        count(id) over(partition by department_name) as n_employees
    from az_employees
    order by n_employees desc
    )
    
    select
        first_name,
        last_name
    from maintable1
    where n_employees  = (
                        select max(n_employees)
                        from maintable1
                        )
        and position like '%manager%'
    
```
![image](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/c03c5087-1789-48c3-8a03-7e7eb08fb41f)

## Question 25
Write a query that will return all cities with more customers than the average number of  customers of all cities that have at least one customer. For each such city, return the country name,  the city name, and the number of customers

Tables: linkedin_customers, linkedin_city, linkedin_country
![Qn25a](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/b1f226a9-e573-4033-8596-e761973584ae)
![Qn25b](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/887be50f-60a2-4273-8aea-4abb94d23a45)
![Qn25c](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/7a9288ee-4551-4975-9f86-6ffb267086f8)

### Solution
```
-- Finding cities with more customers than the average number of customers of all cities that have >= 1 customer
-- Find the average number of customers for cities that have >=1 customer 

with maintable as (
    select
        ci.city_name as city,
        co.country_name as country,
        cu.business_name as customer
    from linkedin_city as ci
    left join linkedin_customers as cu
    on ci.id = cu.city_id
    left join linkedin_country as co
    on ci.country_id = co.id
    ),
    
    subtable1 as (
    select
        city,
        count(customer) as n_customer
    from maintable
    where customer is not null
    group by city
    )

select
    country,
    city,
    count(customer) as n_customers
from maintable
group by country, city
having count(customer) > (
                        select
                            avg(n_customer) as avg_number_of_customers_of_all_cities
                        from subtable1)

```
![Ans25](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/e17de4aa-0dd2-4053-8d2c-7dfabde528f4)









