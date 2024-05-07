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















