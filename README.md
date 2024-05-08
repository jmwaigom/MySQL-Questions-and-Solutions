# MySQL-Questions-and-Solutions

## Question 1
You are working on a data analysis project at Deloitte where you need to analyze a dataset containing information
about various cities. Your task is to calculate the population density of these cities, rounded to the nearest integer, and identify the cities with the minimum and maximum densities.
The population density should be calculated as (Population / Area).

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

<img width="585" alt="Screenshot 2024-05-07 at 11 30 39 AM" src="https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/a8a34805-795c-468d-8e0b-b5a794e31c68">

## Question 2
Find the email activity rank for each user. Email activity rank is defined by the total number of emails sent. The user with the highest number of emails sent will have a rank of 1, and so on. Output the user, total emails, and their activity rank. Order records by the total emails in descending order. Sort users with the same number of emails in alphabetical order.
In your rankings, return a unique value (i.e., a unique rank) even if multiple users have the same number of emails. For tie breaker use alphabetical order of the user usernames.

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
![Ans2](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/5ae4948c-1af9-4d72-9c3a-2eb25139f129)

## Question 3
Find the top 10 users that have traveled the greatest distance. Output their id, name and a total distance traveled\
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
![ans3](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/2dbe273c-2826-4fa7-8f83-84cd3ea79b90)

## Question 4
Given a table of purchases by date, calculate the month-over-month percentage change in revenue. The output should include the year-month date (YYYY-MM) and percentage change, rounded to the 2nd decimal point, and sorted from the beginning of the year to the end of the year.
The percentage change column will be populated from the 2nd month forward and can be calculated as ((this month's revenue - last month's revenue) / last month's revenue)*100.\
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
![ans4](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/99f8af92-0546-4b50-86bf-f1752ba3a078)

## Question 5
Write a query that identifies cities with higher than average home prices when compared to the national average. Output the city names.\
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
![ans5](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/7bdba71a-a915-4351-8e6f-716357943f1b)

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
![ans6](https://github.com/jmwaigom/MySQL-Questions-and-Solutions/assets/155841258/dbb05497-5614-4112-852b-14bbc23f302a)

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









