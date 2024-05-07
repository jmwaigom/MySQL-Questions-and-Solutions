# MySQL-Questions-and-Solutions

### Question 1
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

### Question 2
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

### Question 3
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


























