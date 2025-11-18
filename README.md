````sql
create table WIC(county_code int, county_of_residence text,
year_month text, race text, participants int);
````

````sql
select county_of_residence, sum(participants) as total from WIC where 
county_of_residence <> 'Statewide' and year_month = 2024
group by county_of_residence order by total desc;
````

````sql
create table cali(population int, density int, county text);
````

````sql
select wic.county_of_residence, sum(wic.participants) as total, cali.population, 
CAST(sum(wic.participants) as DECIMAL) / cali.population as ratio
from wic left join cali on wic.county_of_residence = cali.county
where wic.county_of_residence <> 'Statewide' and 
year_month = 2024 group by wic.county_of_residence, cali.population order by ratio desc;
````
