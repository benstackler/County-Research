# 🏠 Comparing County Property Taxes and Real Estate Values

What if we want to calculate effective tax rate using property taxes and hom values, sorted by state?

**Source:** [Tax Foundation County Tax Data](https://taxfoundation.org/data/all/state/property-taxes-by-state-county/)

## Creating Property Tax and Real Estate Value Table

````sql
create table county_tax(state text, county text,
house_value integer, property_tax integer);
````

### Raw County Data
![raw](https://github.com/benstackler/County-Research/blob/benstackler-images/Screen%20Shot%202025-11-18%20at%203.31.43%20PM.png)

Home values and property taxes are grouped by county and state, but what if we want to calculate the effective tax rate and order from highest to lowest? 

````sql
select state, county, CAST(COALESCE(property_tax, 0) as DECIMAL) / NULLIF(house_value, 0) * 100 as tax_rate from county_tax
WHERE property_tax <> 0 AND house_value <> 0
order by CAST(COALESCE(property_tax, 0) as DECIMAL) / NULLIF(house_value, 0) * 100 asc;
````
There are several things that need to be accounted for here. Firstly, we need to use the CAST function on our integer variables to convert them to decimals in order to ensure that our division function reflects its actual quotient. However, we also need to use the COALESCE function to convert null values to 0 (in case any property tax valuesare listed as nulls). In our denominator, we have to convert any 0 values to null (so we do not divide by zero).

We also add in a WHERE function to avoid returning null results (rows with 0 values or null values for both property tax and house value). We then want to order by new county_tax column.

### Highest Effective Property Tax Rate Counties
![desc](https://github.com/benstackler/County-Research/blob/benstackler-images2/Screen%20Shot%202025-11-20%20at%204.38.57%20PM.png)

### Lowest Effective Property Tax Rate Counties
![asc](https://github.com/benstackler/County-Research/blob/benstackler-images2/Screen%20Shot%202025-11-20%20at%204.39.07%20PM.png)

## Creating New View to Find Average State Effective Property Tax Rate
What if we want to calculate a simple average state effective tax rate for each state? We already have a query that gives us the tax rate breakdown by county. We can save this query as a new view, then write a secondary query to select from this view, grouping by state. 

````sql
CREATE view county_averages as
select state, county, CAST(COALESCE(property_tax, 0) as DECIMAL) / NULLIF(house_value, 0) * 100 as tax_rate from county_tax
WHERE property_tax <> 0 AND house_value <> 0
order by CAST(COALESCE(property_tax, 0) as DECIMAL) / NULLIF(house_value, 0) * 100 asc;
````
Let's query this new view and group by state, averaging the effective tax rate of each county within the state.

````sql
select state, AVG(tax_rate) as average_tax from county_averages 
group by state order by AVG(tax_rate) asc;
````
Creating a new view means that we were able to utilize a pretty simple query to group by state. We also didn't have to worry about any zero or null values, because we had previously filtered those out in our first query.
### States with Highest Average Property Tax Rate
![asc](https://github.com/benstackler/County-Research/blob/benstackler-images3/Screen%20Shot%202025-11-21%20at%208.50.11%20AM.png)

### States with Lowest Average Property Tax Rate
![asc](https://github.com/benstackler/County-Research/blob/benstackler-images3/Screen%20Shot%202025-11-21%20at%208.50.26%20AM.png)

### Tableau Visual: States by Average County Property Tax Rate
![tableau](https://github.com/benstackler/County-Research/blob/benstackler-images3/Screen%20Shot%202025-11-21%20at%209.01.46%20AM.png)
