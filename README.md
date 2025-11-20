# 🏠: Comparing County Property Taxes and Real Estate Values

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
![desc](https://github.com/benstackler/County-Research/blob/benstackler-images2/Screen%20Shot%202025-11-20%20at%204.38.57%20PM.png)
![asc](https://github.com/benstackler/County-Research/blob/benstackler-images2/Screen%20Shot%202025-11-20%20at%204.39.07%20PM.png)
