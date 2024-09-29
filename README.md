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
[Question 25](#question-25) : Average Customers Per City\
[Question 26](#question-26) : Top Monthly Sellers\
[Question 27](#question-27) : Algorithm Performance\
[Question 28](#question-28) : Naive Forecasting\
[Question 29](#question-29) : Risky Projects\
[Question 30](#question-30) : Premium vs  Freemium\
[Questoin 31](#question-31) : SMS Confirmations From Users\
[Question 32](#question-32) : Spotify Penetration Analysis\
[Question 33](#question-33) : Meta/Facebook Matching Users Pairs\
[Question 34](#question-34) : Above Average But Not At The Top

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

## Question 26
You are provided with a transactional dataset from Amazon that contains detailed information about sales across different products and marketplaces. Your task is to list the top 3 sellers in each product category for January.

The output should contain 'seller_id' , 'total_sales' ,'product_category' , 'market_place', and 'month'.

Table: sales_data
![Qn26](https://github.com/user-attachments/assets/36f3bee6-ec07-49a6-9008-e5d759e348b8)

### Solution

```
-- For each product category, rank the top 3 sellers

select
    sub.seller_id,
    sub.total_sales,
    sub.product_category,
    sub.market_place,
    sub.month
from (
    select
        seller_id,
        total_sales,
        product_category,
        market_place,
        month,
        rank() over(partition by product_category order by total_sales desc) as sales_ranking
    from sales_data
    where month = '2024-01'
    ) as sub
where sub.sales_ranking < 4;

```
![Ans26](https://github.com/user-attachments/assets/878d4b72-009d-467c-9c42-1a11eab40f94)


## Question 27
Meta/Facebook is developing a search algorithm that will allow users to search through their post history. You have been assigned to evaluate the performance of this algorithm.


We have a table with the user's search term, search result positions, and whether or not the user clicked on the search result.


Write a query that assigns ratings to the searches in the following way:
•	If the search was not clicked for any term, assign the search with rating=1
•	If the search was clicked but the top position of clicked terms was outside the top 3 positions, assign the search a rating=2
•	If the search was clicked and the top position of a clicked term was in the top 3 positions, assign the search a rating=3


As a search ID can contain more than one search term, select the highest rating for that search ID. Output the search ID and its highest rating.


Example: The search_id 1 was clicked (clicked = 1) and its position is outside of the top 3 positions (search_results_position = 5), therefore its rating is 2.

Table: fb_search_events
![Qn27](https://github.com/user-attachments/assets/0029f212-8585-4597-ab06-abaf6e76682f)

### Solution 

```
/*
Creating a CTE with two tables. Table1 uses case statement to create a new rating column

Table 2 ranks the ratings, partitioning the search_id column to single out a search_id
with the highest rating
*/

with table1 as (
    select
        *,
        (case
            when clicked = 0 then 1 
            when clicked = 1 and search_results_position > 3 then 2 
            when clicked = 1 and search_results_position <= 3 then 3 
            else 0
        end) as rating
    from fb_search_events
    ),
    
    table2 as (
    select
        *,
        rank() over(partition by search_id order by rating desc) as rating_rank
    from table1
    )

/* Filtering out search IDs with the highest rating. Using DISTINCT to remove duplicate
search IDs that have the same rating
*/

select distinct
    search_id,
    rating as highest_rating
from table2
where rating_rank = 1

```
![Ans27](https://github.com/user-attachments/assets/6205683a-d4d9-48c4-b84b-5c63ced9445d)

## Question 28
Some forecasting methods are extremely simple and surprisingly effective. Naïve forecast is one of them; we simply set all forecasts to be the value of the last observation. Our goal is to develop a naïve forecast for a new metric called "distance per dollar" defined as the (distance_to_travel/monetary_cost) in our dataset and measure its accuracy.


To develop this forecast,  sum "distance to travel"  and "monetary cost" values at a monthly level before calculating "distance per dollar". This value becomes your actual value for the current month. The next step is to populate the forecasted value for each month. This can be achieved simply by getting the previous month's value in a separate column. Now, we have actual and forecasted values. This is your naïve forecast. Let’s evaluate our model by calculating an error matrix called root mean squared error (RMSE). RMSE is defined as sqrt(mean(square(actual - forecast)). Report out the RMSE rounded to the 2nd decimal spot.

Table: uber_request_logs
![Qn28](https://github.com/user-attachments/assets/e711114a-13ff-4f83-b81a-a9078f0cd920)

### Solution

```
with maintable as (
    select
        request_month,
        distance_to_travel/monetary_cost as actual_distance_per_dollar,
        lag(distance_to_travel/monetary_cost) over(order by request_month) as forecast_distance_per_dollar
    from (
        select
            extract(month from request_date) as request_month,
            sum(distance_to_travel) as distance_to_travel,
            sum(monetary_cost) as monetary_cost
        from uber_request_logs
        group by extract(month from request_date)
        order by 1
        ) as sub
        )

select
    sqrt(avg(power(actual_distance_per_dollar - forecast_distance_per_dollar,2))) as RMSE
from maintable

```
![Ans28](https://github.com/user-attachments/assets/8508a656-ffa8-445e-8791-41f019712b6b)

## Question 29

Identify projects that are at risk for going overbudget. A project is considered to be overbudget if the cost of all employees assigned to the project is greater than the budget of the project.


You'll need to prorate the cost of the employees to the duration of the project. For example, if the budget for a project that takes half a year to complete is $10K, then the total half-year salary of all employees assigned to the project should not exceed $10K. Salary is defined on a yearly basis, so be careful how to calculate salaries for the projects that last less or more than one year.


Output a list of projects that are overbudget with their project name, project budget, and prorated total employee expense (rounded to the next dollar amount).


HINT: to make it simpler, consider that all years have 365 days. You don't need to think about the leap years.

Tables: linkedin_projects, linkedin_emp_projects, linkedin_employees

![Qn29a](https://github.com/user-attachments/assets/61de2367-a88d-4641-bc47-476b08cbdef6)
![Qn29b](https://github.com/user-attachments/assets/818d1aa6-76e1-4184-808a-259c34619dc0)
![Qn29c](https://github.com/user-attachments/assets/d11192a8-6f5e-4758-a410-96ed0f975fcf)

### Solution
```
/*
Overbudget scenario: Cost of all employees in the project > budget of the project
(For each project, find out how much is assigned and total employee cost)

- For each project, find out its duration and total employee salary for that duration
*/
with table1 as (
    select
        id as project_id,
        title as project_name,
        budget,
        (end_date::date - start_date::date) as project_duration_day
    from linkedin_projects
    order by 3 desc -- All projects last less than a year
    ),

-- This query returns all the necessary columns needed for analysis
    table2 as (
    select 
        t1.project_id as project_id,
        t1.project_name as project_name,
        t1.budget as budget,
        le.salary as salary,
        t1.project_duration_day as project_duration_days,
        le.first_name || ' ' || le.last_name as employee_name
        from table1 as t1
    inner join linkedin_emp_projects as lep
    using(project_id)
    inner join linkedin_employees as le
    on lep.emp_id = le.id
    order by t1.project_name
    ),

    table3 as (
    select
        project_name,
        budget,
        sum(salary) * project_duration_days/365 as total_prorated_employee_cost,
        project_duration_days
    from table2
    group by project_name, budget,project_duration_days
    order by project_duration_days desc
    )

select
    project_name,
    budget,
    ceiling(total_prorated_employee_cost) as total_employee_expense
from table3
where total_prorated_employee_cost > budget

```
![Ans29](https://github.com/user-attachments/assets/cc6cb73f-ac24-4204-85c3-731c81ec6fec)

## Question 30

Find the total number of downloads for paying and non-paying users by date. Include only records where non-paying customers have more downloads than paying customers. The output should be sorted by earliest date first and contain 3 columns date, non-paying downloads, paying downloads. Hint: In Oracle you should use "date" when referring to date column (reserved keyword).

Tables: ms_user_dimension, ms_acc_dimension, ms_download_facts

![Qn30a](https://github.com/user-attachments/assets/43fee856-f66e-4edd-a66a-06e098473d3e)
![Qn30b](https://github.com/user-attachments/assets/e39a1a8b-d48e-4281-8aa6-14c12a41cf6e)
![Qn30c](https://github.com/user-attachments/assets/ca3f2a88-0c2f-4230-b265-51761f16e696)

### Solution
```
/*
-- There are paying and non-paying users
-- For each date, find the total number of downloads for both paying and non-paying users
-- Output only records where non-paying customers have more downloads
-- Output columns: date (ascending order),non-paying downloads, paying downloads
*/

-- This code joins all three tables and returns a table with only the necessary columns
with table1 as ( 
    select 
        mdf.date as date,
        mud.user_id as user_id,
        mad.acc_id as acc_id,
        mdf.downloads as downloads,
        mad.paying_customer as paying_or_not_paying_customer
    from ms_download_facts as mdf
    inner join ms_user_dimension as mud
    using(user_id)
    inner join ms_acc_dimension as mad
    on mud.acc_id = mad.acc_id
    order by date
    ),

/*
This table (table2) groups by date, create two columns for paying and non-paying customers
and finds their total downloads for each date
*/

    table2 as (
    select
        date,
        sum(case when paying_or_not_paying_customer = 'yes' then downloads end) as paying_customers,
        sum(case when paying_or_not_paying_customer = 'no' then downloads end) as non_paying_customers
    from table1
    group by date
    )

/*
This code filters for records where non-paying users have more downloads than paying users
*/

select *
from table2 
where paying_customers - non_paying_customers < 0
order by date

```
![Ans30](https://github.com/user-attachments/assets/2d433d6e-92b4-493b-b2d4-66a64b509566)

## Question 31
Meta/Facebook sends SMS texts when users attempt to 2FA (2-factor authenticate) into the platform to log in. In order to successfully 2FA they must confirm they received the SMS text message. Confirmation texts are only valid on the date they were sent.


Unfortunately, there was an ETL problem with the database where friend requests and invalid confirmation records were inserted into the logs, which are stored in the 'fb_sms_sends' table. These message types should not be in the table.


Fortunately, the 'fb_confirmers' table contains valid confirmation records so you can use this table to identify SMS text messages that were confirmed by the user.


Calculate the percentage of confirmed SMS texts for August 4, 2020. Be aware that there are multiple message types, the ones you're interested in are messages with type equal to 'message'.

Tables: fb_sms_sends, fb_confirmers

![Qn31a](https://github.com/user-attachments/assets/114acbed-84fb-4095-8e31-4687ecbe49ee)
![Qn31b](https://github.com/user-attachments/assets/edf79ac5-342f-45f9-9eb3-4e8167d52f25)

### Solution
```
/*
Confirmation texts are only valid on the date they were sent
Objective: Calculate % confirmation for Aug 4, 2020
*/

-- This query joins the two tables, filtering out confirmation and friend_request message types

with table1 as (
    select
        fs.ds as date_sent,
        fc.date as date_received,
        fs.type as type
    from fb_sms_sends as fs
    left join fb_confirmers as fc -- This type of join ensures that all send dates appear, but if not received, then nulls appear for receiving_date
    using(phone_number)
    where fs.type not in ('confirmation', 'friend_request') 
    order by date_sent
    ),

-- This code returns records for August 4, 2020 only

    table2 as (
    select *
    from table1 
    where date_sent = '2020-08-04'
    )

-- This code calculates percentage of confirmed texts by dividing confirmed texts on 2020-08-04 by total rows for the date

select 
    sum(case when date_received = '2020-08-04' then 1 else 0 end)/count(*)::float * 100 as percent_confirmed
from table2

```
![Ans31](https://github.com/user-attachments/assets/11877b59-975d-4bee-9db9-99959087b3ca)

## Question 32
Market penetration is an important metric for understanding Spotify's performance and growth potential in different regions.
You are part of the analytics team at Spotify and are tasked with calculating the active user penetration rate in specific countries.


For this task, 'active_users' are defined based on the  following criterias:


last_active_date: The user must have interacted with Spotify within the last 30 days.
•    sessions: The user must have engaged with Spotify for at least 5 sessions.
•    listening_hours: The user must have spent at least 10 hours listening on Spotify.


Based on the condition above, calculate the active 'user_penetration_rate' by using the following formula.


•    Active User Penetration Rate = (Number of Active Spotify Users in the Country / Total users in the Country)


Total Population of the country is based on both active and non-active users.
​
The output should contain 'country' and 'active_user_penetration_rate' rounded to 2 decimals.


Let's assume the current_day is 2024-01-31.

Table: penetration_analysis

![Qn32](https://github.com/user-attachments/assets/c0e44a62-6479-4c9e-ba6a-4673e2ad0bea)

### Solution
```
-- This CTE groups by country and counts number of actice and inactive users

with table1 as (
    select
        country,
        sum(case 
                when sessions >=5 and listening_hours >=10 and datediff('2024-01-31'::date,last_active_date) <= 30 
                then 1 else 0
                end) as n_active_users,
        count(*) as total_users
    from penetration_analysis
    group by country
    )
    
-- This query calculates the percentage of active users by dividing active users by total users in a country 

select
    country,
    round(1.0 * n_active_users/total_users,2) as  active_user_penetration_rate
from table1

```
![Ans32](https://github.com/user-attachments/assets/2667f5bc-f9b5-4689-9452-acdaa59ac78f)

## Question 33
Find matching pairs of Meta/Facebook employees such that they are both of the same nation, different age, same gender, and at different seniority levels.
Output ids of paired employees.

Table: facebook_employees
![Qn33](https://github.com/user-attachments/assets/9be12682-f895-4d10-918e-d8e084ab30ea)

### Solution
```
select
    a.id as emp1,
    b.id as emp2
from facebook_employees as a
inner join facebook_employees as b
on 
    a.location = b.location and 
    a.id != b.id and a.age != b.age 
    and a.gender = b.gender 
    and a.is_senior != b.is_senior
where a.id is not null and b.id is not null;

```
![Ans33](https://github.com/user-attachments/assets/695ef555-77d9-4b17-b69e-9e248513f59c)

## Question 34
Find all people who earned more than the average in 2013 for their designation but were not amongst the top 5 earners for their job title. Use the totalpay column to calculate total earned and output the employee name(s) as the result.

Table: sf_public_salaries
![Qn34](https://github.com/user-attachments/assets/50e2f7f6-f001-43bf-a367-ac8fd82c6c49)

### Solution 
```

-- This cte returns a table which shows average per jobtitle and payrank of each employee per department in 2013

with maintable as (
    select
        jobtitle,
        employeename, 
        totalpay,
        avg(totalpay) over(partition by jobtitle) as avg_dept_salary,
        rank() over(partition by jobtitle order by totalpay desc) as dept_payrank
    from sf_public_salaries
    where year = 2013
    )
/*
This query filters out employee(s) whose totalpay is greater than their jobtitle average but not amongst the top 5 earners
within their jobtitle 
*/

select employeename
from maintable
where totalpay > avg_dept_salary and dept_payrank > 5;

```
![Ans34](https://github.com/user-attachments/assets/9b4ed6b0-cc30-49f6-b827-41d16eacb381)


