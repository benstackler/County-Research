# Calculating the California Counties with the Highest WIC Usage per Capita

California WIC usage data taken from [California Open Data Portal](https://data.ca.gov/dataset/wic-participants-by-county)

California county population data taken from [World Population Review](https://worldpopulationreview.com/us-counties/california)

## Creating WIC Table

````sql
create table WIC(county_code int, county_of_residence text,
year_month text, race text, participants int);
````
### Sorting WIC Table by Year 2024 and County of Residence

````sql
select county_of_residence, sum(participants) as total from WIC where 
county_of_residence <> 'Statewide' and year_month = 2024
group by county_of_residence order by total desc;
````
## Creating California County Population Table

````sql
create table cali(population int, density int, county text);
````
## Joining Both Tables and Sorting by Ratio

````sql
select wic.county_of_residence, sum(wic.participants) as total, cali.population, 
CAST(sum(wic.participants) as DECIMAL) / cali.population as ratio
from wic left join cali on wic.county_of_residence = cali.county
where wic.county_of_residence <> 'Statewide' and 
year_month = 2024 group by wic.county_of_residence, cali.population order by ratio desc;
````
