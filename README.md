# :baby_bottle: Calculating the California Counties with the Highest WIC Usage per Capita

What if we want to calcualte the California counties with the **highest WIC rate** (calculated as WIC participants divided by total county population)? We would need some way to combine county population estimates with WIC participant estimates, all grouped by county. 

Luckily, we have access to the [California Open Data Portal](https://data.ca.gov/dataset/wic-participants-by-county), which contains information on 2024 WIC usage by county. Similarly, [World Population Review](https://worldpopulationreview.com/us-counties/california) contains 2025 California county populations (straight from U.S. Census). 

After cleaning data in both CSV files we are ready to begin our SQL Work.

## Creating WIC Table

````sql
create table WIC(county_code int, county_of_residence text,
year_month text, race text, participants int);
````
Importantly, we want to ensure that our WIC table variable are correct before importing the WIC data. I imported year_month as text because we already have aggregated year rows, so we don't actually need to convert it to a date format (saving us time). The rest of the columns can be formatted in either integer or text format.

However, we quickly notice that **some of our participant data is null** for certain rows. This is because the table is broken down by race/ethncitiy and certain counties have 0 participants classified as a certain race/ethnicity. We fix this by updating the table to convert null values to 0.

````sql
UPDATE WIC
SET participants = 0
WHERE participants IS NULL;
````
This will ensure that any rows lacking participant data are displayed as '0' instead of null.

### Sorting WIC Table by Year 2024 and County of Residence

````sql
select county_of_residence, sum(participants) as total from WIC where 
county_of_residence <> 'Statewide' and year_month = 2024
group by county_of_residence order by total desc;
````
Here we are ensuring that we're **only querying for 2024 data**, and that we're excluding rows where the counties are combined into a nationwide measurement. We want to **sum participants** so that all race metrics are summed, and we **group at the county level** to ensure that our results are listed by county. 

Unsuprisingly, Los Angeles has the **total most WIC participants** (as it has the highest total population of any county). But this begs the question: Which county has the **highest WIC usage ratio per capita**? Los Angeles' ratio could be expected to be lower than other counties because it has such a large population, and includes certain neighborhoods which are among the wealthiest in the nation.

To find out, we must incorporate **total county population data.**

## Creating California County Population Table

````sql
create table cali(population int, density int, county text);
````
We are importing an extremely simple table, so our table command reflect only three columns. While we don't need population density (population / sq miles), it is interesting to have in our back pocket.

## Joining Both Tables and Sorting by Ratio

Our question primarily involves WIC data, so we will **use the WIC table as our primary table**. We are doing a **left join** on the California County Population table, using county name as the join key. 

However, we still want to display our aggregate participant values, so we include a **sum(participants)** aggregate function within the 'Select' command. We have to incorporate a 'group by' command now that we've used an aggregate command, so we will first be **grouping by county**. Then we want to have **county population** as a point of comparison, so we will include that as our second 'group by' column. 

Finally, we want our **per capita WIC usage rate**, (defined as WIC participants / county population), so we include a CAST function to ensure that when we divide our two integer columns, we get the corresponding ratio, and not a result of '0.'

````sql
select wic.county_of_residence, sum(wic.participants) as total, cali.population, 
CAST(sum(wic.participants) as DECIMAL) / cali.population as ratio
from wic left join cali on wic.county_of_residence = cali.county
where wic.county_of_residence <> 'Statewide' and 
year_month = 2024 group by wic.county_of_residence, cali.population order by ratio desc;
````
