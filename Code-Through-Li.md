---
layout: page
title: Code-Through
subtitle: tidycensus Package in Simple Steps 
---

<br>

# Introduction to tidycensus package 

We have already learned the fundamental to API and how to use API to  query census data. Also, we have learned the tidy form of data that is handy for data analysts to perform tons of possible analyses. This code-through will introduce the basic of the package called tidycensus. The big merit working with this library is to have the tidy form returned for census data. 


**Note:**

- tidycensus is usually used along with the package "tidyverse", so be sure to install and load **THEM both** before calling any functions. 
- the API key is for personal use. Please obtain yours from [here](http://api.census.gov/data/key_signup.html). Be sure to **ACTIVATE** by clicking the link sent to your email.



![](/assets/img/1.png)


---

![](/assets/img/2.png)


<center>
 
 **YOUR KEY IS NOW READY TO WORK!** 

</center>

---

<br>

## Load library

```r
library( tidycensus )   # census data in tidy format 
library( tidyverse )    # tidy operations
library( ggplot2 )      # nice plots    
library( pander )       # nice tables

# replay with your API KEY 
census_api_key( "69b310b6c76339cfd5da39f8c83c351045d1a14b" )
```


---

<br>

## Function：get_decennial()
One of the two major functions in tidycensus is **get_decennial()**. It connects the 2000 and 2010 decennial US Census APIs, Note: to access to all census datasets please consider [censusapi](https://github.com/hrecht/censusapi) package.


### Examples

```r

# get median age by state in 2000
age.00 <- get_decennial( geography = "state", variables = "P013001", year = 2000 )

# check the data we have queried
head( age.00, 10 ) %>% pander()

```
![](/assets/img/git-1-1.PNG)

The function returns a tibble with four columns by default: 

- **GEOID**: an identifier for the geographical unit associated with the row; 
- **NAME**: a descriptive name of the geographical unit; 
- **variable**: the Census variable represented in the row; 
- **value**: the value of the variable for that unit. 

By default, tidycensus functions return **tidy data frames** in which columns represent variables and rows represent observations. Therefore, we can plot the data directly:

```r

age.00 %>% ggplot( aes( x = value, y = reorder( NAME, value ) ) ) + # order the age based on the sates
  xlab( "Median ages" ) +
  ylab( "States" ) +
  geom_point()

```
![](/assets/img/git-1-2.PNG)

<br>

However, if you want to have a wide data frame (for better reports) with Census variable names in the columns, set output = "wide" in the function call.

```r

# get median age by state in 2010 and return in wide form
age.10.wide <- get_decennial( geography = "state", variables = "P013001", 
                             year = 2010, output = "wide" )

# check the data we have queried
head( age.10.wide, 10 )  %>% pander()

```

![](/assets/img/git-1-3.PNG)


```r
# print the median age for Arizona and California in 2010 and return in wide form
get_decennial( geography = "state", state = c( "Arizona", "California" ), 
               variables = "P013001", year = 2010, output = "wide" ) %>% 
  pander() 

```

![](/assets/img/git-1-4.PNG)

---

<br>

## Variables

As you may note in the previous example, the get_decennial() requires knowing the variables ID. To search for variables, **load_variables()** function can help. This function takes two **required** arguments: 

1. year of the Census or end year of the ACS sample, 
2. the dataset - one of "sf1", "sf3", or "acs5". 


### Examples

```r

# set cache = TRUE to store the result on computer for future access
# search the variables
variable.00 <- load_variables( 2000, "sf3", cache = TRUE )

# get the first 10 variables 
head( variable.00, 10 ) %>% pander() 

view( variable.00 )

```
![](/assets/img/git-1-5.PNG)

So, we can ask for the housing units ( name == "H001001" ) for 2000 by supplying the variables argument with "H001001." 

```r

# get housing units by state in 2000
housing.00 <- get_decennial( geography = "state", variables = "H001001", year = 2000 )

# check the data we have queried
head( housing.00, 10 ) %>% pander()

```
![](/assets/img/git-1-6.PNG)

Similarly, because of the tidy format returned, We can plot the result directly using ggplot2 functions:  

```r

housing.00 %>% ggplot( aes( x = value, y = reorder( NAME, value ) ) ) + # order the age based on the sates
  xlab( "Totoal housing units" ) +
  ylab( "States" ) +
  geom_col()

```

![](/assets/img/git-1-7.PNG)

### Summary Files (sf)

Census data is presented in **4 separated Summary Files (sf1 to sf4)**. Each differs in data resolution, the kind of information it contains, and whether they are derived from a total count (100%) or a sample or the population:


 
![](/assets/img/5.PNG)

<center>
 
 Note: at the moment, access to the 1990 (SF1 and SF3) and 2000 (SF3) APIs has been suspended.

</center>

As we have noted, the variables starts with semantic meanings in categorical ways. For 2020 Census, for example, the summary file 1 dataset contains variables (tables):

- 177 population tables (identified with a ‘‘P’’) and 58 housing tables (identified with an ‘‘H’’) shown down to the block level; 
- 82 population tables (identified with a ‘‘PCT’’) and 4 housing tables (identified with an “HCT”) shown down to the census tract level; 
- 10 population tables (identified with a “PCO”) shown down to the county level, for a total of 331 tables. 

<br>

---

## Geography 

Census surveys record data in various level. The census follows **the hierarchy:**

<center>
 
**USA -> State -> County -> Census Tract -> Census Block Group -> Census Block**

</center>


To query data from different levels, we can supply an argument to the required **geography** parameter. Arguments are formatted as consumed by the Census API, and specified in the table below: 


![](/assets/img/3.PNG) 

![](/assets/img/4.PNG) 


**Note:**

- Not all geographies are available for all surveys, all years, and all variables. Most Census geographies are supported in tidycensus at the moment; 
- If state or county is in bold face in “Available by”, you are required to supply a state and/or county for the given geography.

<br>

### Examples

```r

# get housing units by counties in 2000
get_decennial( geography = "county", variables = "H001001", year = 2000 ) %>% 
  head() %>% pander()
  
```

![](/assets/img/git-1-8.PNG)

```r

# get housing units in the US in 2000
get_decennial( geography = "us", variables = "H001001", year = 2000 ) %>% 
  head() %>% pander()
  
```

  ![](/assets/img/git-1-9.PNG)

```r

# get housing units by regions in 2000
get_decennial( geography = "region", variables = "H001001", year = 2000 ) %>% 
  head() %>% pander()
  
```
![](/assets/img/git-1-10.PNG)

Similarly, we can ask for data at different levels for median age in 2000 

```r

# print the median age for counties in 2010 
get_decennial( geography = "county", variables = "P013001", year = 2000 ) %>% 
  head() %>% pander()

```


![](/assets/img/git-1-11.PNG)

```r

# print the median age for counties in Arizona and California in 2010 
get_decennial( geography = "county", state = c( "Arizona", "California" ), 
               variables = "P013001", year = 2010 ) %>% pander() 
```
![](/assets/img/git-1-12.PNG)

**NOTE: only the first 19 rows are shown in the table above.** 

<br>

---

# Sources 

- https://walker-data.com/tidycensus/articles/basic-usage.html
- https://www.census.gov/data/datasets/2010/dec/summary-file-1.html
- https://projects.ncsu.edu/woodlands//current_pdfs/teleconf_data_sources.pdf

---

<br>

<center> 
 
 *This is the end of this tutorial, and many thanks for your attention!* 
 
</center>
