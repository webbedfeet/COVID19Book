---
title: "Exploring the COVID-19 pandemic using sars2pack"
author: "Sean Davis <seandavi@gmail.com> and Vincent J. Carey, stvjc at channing.harvard.edu"
date: "April 05, 2020"
vignette: >
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteIndexEntry{Usage and Data Exploration}
  %\VignetteEncoding{UTF-8}
output:
  BiocStyle::html_document:
    highlight: pygments
    number_sections: yes
    theme: united
    toc: yes
---


```
## Warning: package 'BiocStyle' was built under R version 3.6.2
```

```
## Warning: package 'tibble' was built under R version 3.6.2
```

# Quick start

## Installation


```r
BiocManager::install('seandavi/sars2pack')
```

[sars2pack]: https://github.com/seandavi/sars2pack
[New York Times]: https://github.com/nytimes/covid-19-data
[JHU]: https://github.com/CSSEGISandData/COVID-19/tree/master/csse_covid_19_data/
[USAFacts]:  https://usafacts.org/visualizations/coronavirus-covid-19-spread-map/
[pull request]: https://github.com/seandavi/sars2pack/compare
[issue]: https://github.com/seandavi/sars2pack/issues/new

## COVID-19 resources in this package

The COVID-19 data in this package are, right now, focused toward 
time-series descriptions of confirmed cases, deaths, testing, and
recovered cases. **There is no requirement that this remain the case**.
Contributions of additional data resources or simple accessor functions
will only add to our abilities to use data science and modeling
to understand COVID-19.

*Request for help*: I would be more than happy to accept help with 
defining new data resources. Consider a [pull request] (or an [issue] for
non-programmer types). 

### Epidemic time-series data

- [JHU] : global deaths, confirmed cases, and recovered time series
  data; *does not include fine-level United States data*. See
  `jhu_data()`.
- [New York Times] : United states state and county level deaths,
  confirmed cases time series. See `nytimes_county_data()` and
  `nytimes_state_data`.
- [USAFacts] : Alternative United states state and county level deaths
  and confirmed cases time series

## Additional resources described in this vignette



- 


# Epidemic time series data

Usage of each of the time series datasets follows a similar pattern. 

1. Fetch a tidy `tbl_df` using a function such as `jhu_data()`
2. In the resulting `tbl_df`, the columns `date` (of type `date`) and
   `count` of type `numeric` columns are standard.
3. Additional columns describe locations, subsets of data (such as
   `confirmed`, `deaths`, `recovered`) and vary from dataset to
   dataset. 

Regardless of the original format of the data, the `sars2pack`
datasets are presented as [tidy data] to facilitate `dplyr`, `ggplot`,
and other fluid analysis approaches to apply directly.

[tidy data]: https://cran.r-project.org/web/packages/tidyr/vignettes/tidy-data.html

## Access data

This section briefly introduces how to access the data
resources in this package. Note that many of the functions
below **require a network connection** to get updated data.

### JHU Dataset


```r
jhu = jhu_data()
class(jhu)
```

```
## [1] "tbl_df"     "tbl"        "data.frame"
```

```r
dim(jhu)
```

```
## [1] 38332     7
```

Column names include:


```r
colnames(jhu)
```

```
## [1] "ProvinceState" "CountryRegion" "Lat"           "Long"         
## [5] "date"          "count"         "subset"
```

And a very small subset of the data. 


```r
head(jhu,3)
```

```
## # A tibble: 3 x 7
##   ProvinceState CountryRegion   Lat  Long date       count subset   
##   <chr>         <chr>         <dbl> <dbl> <date>     <dbl> <chr>    
## 1 <NA>          Afghanistan      33    65 2020-01-22     0 confirmed
## 2 <NA>          Afghanistan      33    65 2020-01-23     0 confirmed
## 3 <NA>          Afghanistan      33    65 2020-01-24     0 confirmed
```


### USAFacts Dataset




```r
usa_facts = usa_facts_data()
class(usa_facts)
```

```
## [1] "tbl_df"     "tbl"        "data.frame"
```

```r
dim(usa_facts)
```

```
## [1] 434792      7
```

Column names include:


```r
colnames(usa_facts)
```

```
## [1] "county_fips" "county"      "state"       "state_fips"  "subset"     
## [6] "date"        "count"
```

And a very small subset of the data. 


```r
head(usa_facts,3)
```

```
## # A tibble: 3 x 7
##   county_fips county                state state_fips subset    date       count
##         <dbl> <chr>                 <chr>      <dbl> <chr>     <date>     <dbl>
## 1           0 Statewide Unallocated AL             1 confirmed 2020-01-22     0
## 2           0 Statewide Unallocated AL             1 confirmed 2020-01-23     0
## 3           0 Statewide Unallocated AL             1 confirmed 2020-01-24     0
```


### NYTimes datasets




```r
nytimes_state = nytimes_state_data()
class(nytimes_state)
```

```
## [1] "tbl_df"     "tbl"        "data.frame"
```

```r
dim(nytimes_state)
```

```
## [1] 3658    5
```

Column names include:


```r
colnames(nytimes_state)
```

```
## [1] "date"   "state"  "fips"   "count"  "subset"
```

And a very small subset of the data. 


```r
head(nytimes_state,3)
```

```
## # A tibble: 3 x 5
##   date       state      fips  count subset   
##   <date>     <chr>      <chr> <dbl> <chr>    
## 1 2020-01-21 Washington 00053     1 confirmed
## 2 2020-01-22 Washington 00053     1 confirmed
## 3 2020-01-23 Washington 00053     1 confirmed
```



```r
nytimes_county = nytimes_county_data()
class(nytimes_county)
```

```
## [1] "tbl_df"     "tbl"        "data.frame"
```

```r
dim(nytimes_county)
```

```
## [1] 66502     6
```

```r
colnames(nytimes_county)
```

```
## [1] "date"   "county" "state"  "fips"   "count"  "subset"
```

# Use cases

## Basic data exploration

In this section, we will be using a combination of [dplyr] and
[ggplot2] to explore the COVID-19 global data from JHU. For details on
this dataset, see the help using `?jhu_data`.

The next line of code will do a (set of) network calls to fetch the
most up-to-date dataset from the JHU github repository.


```r
jhu = jhu_data()
head(jhu,3)
```

```
## # A tibble: 3 x 7
##   ProvinceState CountryRegion   Lat  Long date       count subset   
##   <chr>         <chr>         <dbl> <dbl> <date>     <dbl> <chr>    
## 1 <NA>          Afghanistan      33    65 2020-01-22     0 confirmed
## 2 <NA>          Afghanistan      33    65 2020-01-23     0 confirmed
## 3 <NA>          Afghanistan      33    65 2020-01-24     0 confirmed
```

We now want to ask a series of questions about the dataset. 

- **How many records are in the dataset?**


```r
nrow(jhu)
```

```
## [1] 38332
```

- **How many different countries are represented?**


```r
length(unique(jhu$CountryRegion))
```

```
## [1] 181
```

Most records have no listing for `ProvinceState` column. Let's look at
a few of those to get an idea of what is there when not empty:

- **What is in the `ProvinceState` column?**

To answer this question, we will be using `dplyr`, so some familiarity
with that package will be helpful to follow this code.


```r
jhu %>%
    dplyr::filter(!is.na(ProvinceState)) %>%
    dplyr::select(ProvinceState, CountryRegion) %>%
    unique() %>%
    head(10)
```

```
## # A tibble: 10 x 2
##    ProvinceState                CountryRegion
##    <chr>                        <chr>        
##  1 Australian Capital Territory Australia    
##  2 New South Wales              Australia    
##  3 Northern Territory           Australia    
##  4 Queensland                   Australia    
##  5 South Australia              Australia    
##  6 Tasmania                     Australia    
##  7 Victoria                     Australia    
##  8 Western Australia            Australia    
##  9 Alberta                      Canada       
## 10 British Columbia             Canada
```

We still have not looked at the most valuable information, the `date`
and `count` columns in any detail.

- **What is the current count of confirmed cases by country, ordered
  by highest count down?**

There is a lot to unpack in the next code block, but the results are
quite useful. We will use the [DT] package to make the dataset searchable
and sortable.


```r
library(DT)
latest_jhu_data = jhu %>%
    dplyr::filter(subset=='confirmed' & is.na(ProvinceState)) %>%
    dplyr::group_by(CountryRegion) %>%
    dplyr::slice(which.max(date)) %>%
    dplyr::arrange(desc(count))
DT::datatable(latest_jhu_data, rownames=FALSE)
```

![](sars2pack_files/figure-latex/global_latest_date-1.pdf)<!-- --> 

**Note**: I included a little `is.na` in the filtering above to remove
records where country data are split out over subparts. We revisit
this below.

[DT]: https://rstudio.github.io/DT/

The data here could be usefully displayed as a graph as well.


```r
par(las=2, mar=c(8,5,5,1))
barplot(count ~ CountryRegion, xlab = '',
        data=head(latest_jhu_data,10),
        main='Confirmed cases, top 10 countries')
```

![](sars2pack_files/figure-latex/latest_jhu_bargraph-1.pdf)<!-- --> 

We note here that China is not shown. That is because we limited the
data to only rows that had empty ProvinceState records. To add those
records back in, we sum all the China rows (and those of other
countries like Australia, etc.) by country and then perform similar
work to produce a final plot.


```r
latest_jhu_data = jhu %>%
    dplyr::filter(subset=='confirmed') %>%
    dplyr::select(-c(ProvinceState,Lat,Long)) %>%
    dplyr::group_by(CountryRegion,date) %>%
    dplyr::summarize(count = sum(count)) %>%
    dplyr::slice(which.max(date)) %>%
    dplyr::arrange(desc(count))
par(las=2, mar=c(8,5,5,1))
barplot(count ~ CountryRegion, xlab = '',
        data=head(latest_jhu_data,10),
        main='Confirmed cases, top 10 countries')
```

![](sars2pack_files/figure-latex/unnamed-chunk-2-1.pdf)<!-- --> 



## Visualize time series data

Up to now, we have ignored the time series aspects of the data and
have sliced the dataset by country. In this section, we will be using
dplyr and ggplot2 to visualize disease infection and deaths over time.

- **How have the cases in Italy changed over time?**


```r
library(ggplot2)
italy_cc_ts = jhu %>%
    dplyr::filter(CountryRegion == 'Italy' & subset=='confirmed')
ggplot(italy_cc_ts,aes(x=date, y=count)) +
    geom_line() +
    ggtitle('Confirmed cases') 
```

![](sars2pack_files/figure-latex/italy_ts_1-1.pdf)<!-- --> 

- **How do the confirmed cases in China, US, Italy, Spain, Germany,
  and Russia compare over time?**

We have to play the same game of summing all values by country and
date. Here, we filter the countries to be in a list of countries.


```r
countries_of_interest = c('China','US','Italy','Spain','Germany','Russia')
library(ggplot2)
cc_ts = jhu %>%
    dplyr::group_by(CountryRegion,date) %>%
    dplyr::filter(CountryRegion %in% countries_of_interest & subset=='confirmed') %>%
    dplyr::summarize(count = sum(count))
head(cc_ts)
```

```
## # A tibble: 6 x 3
## # Groups:   CountryRegion [1]
##   CountryRegion date       count
##   <chr>         <date>     <dbl>
## 1 China         2020-01-22   548
## 2 China         2020-01-23   643
## 3 China         2020-01-24   920
## 4 China         2020-01-25  1406
## 5 China         2020-01-26  2075
## 6 China         2020-01-27  2877
```

To make the plot, we use the ggplot2 grouping and coloring to provide
curves for each country on the same axis.


```r
ggplot(cc_ts,aes(x=date, y=count, group=CountryRegion)) +
    geom_line(aes(color=CountryRegion)) +
    ggtitle('Confirmed cases') 
```

![](sars2pack_files/figure-latex/cc_ts_plot-1.pdf)<!-- --> 

Changing to log scale can give a sense of the "exponentialness" of
these data. Also, to remove zeros from the data (which cause problems
when taking logs), we can filter data to include only values
>=50. Note that ggplot2 will "do the right thing".


```r
cc_ts %>%
    dplyr::filter(count>=50) %>%
    ggplot(aes(x=date, y=count, group=CountryRegion)) +
    geom_line(aes(color=CountryRegion)) +
    ggtitle('Confirmed cases') +
    scale_y_log10()
```

![](sars2pack_files/figure-latex/cc_ts_plot_log-1.pdf)<!-- --> 

Consider the following questions based on the figure:

- What does the slope of the lines in this plot represent?
- What is the difference between China and other countries? What does
  this difference mean in terms of how the disease is spreading?
- What does each
- Pick an arbitrary level on the y-axis and look at the dates
  associated with each country's curve with respect to that
  level. What do differences along the x-axis tell us about where the
  countries are with respect to disease process?


