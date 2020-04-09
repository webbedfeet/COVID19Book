





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

*Request for help*: We are looking for additional data resources. Consider a [pull request] (or an [issue] for non-programmer types) with suggestions. 

# TODO: give background on datasets

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
## [1] 60450     7
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
## [1] 3988    5
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
## [1] 81440     6
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
## [1] 60450
```

- **How many different countries are represented?**


```r
length(unique(jhu$CountryRegion))
```

```
## [1] 184
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

<!--html_preserve--><div id="htmlwidget-48583d059be3f60e33d8" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-48583d059be3f60e33d8">{"x":{"filter":"none","data":[[null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null],["US","Spain","Italy","Germany","France","Iran","United Kingdom","Turkey","Belgium","Switzerland","Netherlands","Brazil","Portugal","Austria","Korea, South","Israel","Russia","Sweden","Norway","Ireland","India","Chile","Denmark","Czechia","Poland","Romania","Ecuador","Peru","Pakistan","Japan","Malaysia","Philippines","Luxembourg","Indonesia","Saudi Arabia","Mexico","Serbia","United Arab Emirates","Finland","Thailand","Panama","Qatar","Dominican Republic","Colombia","Greece","South Africa","Argentina","Ukraine","Singapore","Iceland","Algeria","Egypt","Croatia","Morocco","New Zealand","Iraq","Estonia","Moldova","Slovenia","Belarus","Lithuania","Hungary","Armenia","Kuwait","Bahrain","Azerbaijan","Bosnia and Herzegovina","Cameroon","Kazakhstan","Diamond Princess","Slovakia","Tunisia","North Macedonia","Bulgaria","Latvia","Lebanon","Andorra","Uzbekistan","Cyprus","Costa Rica","Cuba","Afghanistan","Uruguay","Oman","Burkina Faso","Albania","Cote d'Ivoire","Taiwan*","Jordan","Niger","Ghana","Honduras","Malta","San Marino","Nigeria","Mauritius","Kyrgyzstan","West Bank and Gaza","Vietnam","Montenegro","Senegal","Bangladesh","Georgia","Bolivia","Sri Lanka","Kosovo","Congo (Kinshasa)","Kenya","Venezuela","Guinea","Brunei","Djibouti","Paraguay","Cambodia","Rwanda","Trinidad and Tobago","El Salvador","Madagascar","Guatemala","Monaco","Liechtenstein","Togo","Barbados","Jamaica","Mali","Ethiopia","Uganda","Congo (Brazzaville)","Bahamas","Zambia","Guyana","Gabon","Eritrea","Guinea-Bissau","Liberia","Haiti","Benin","Tanzania","Burma","Libya","Angola","Antigua and Barbuda","Maldives","Syria","Equatorial Guinea","Mozambique","Mongolia","Namibia","Dominica","Fiji","Laos","Saint Lucia","Sudan","Eswatini","Grenada","Somalia","Saint Kitts and Nevis","Seychelles","Zimbabwe","Chad","Suriname","MS Zaandam","Nepal","Belize","Central African Republic","Holy See","Malawi","Saint Vincent and the Grenadines","Cabo Verde","Sierra Leone","Botswana","Mauritania","Nicaragua","Bhutan","Gambia","Sao Tome and Principe","Western Sahara","Burundi","Papua New Guinea","South Sudan","Timor-Leste"],[37.0902,40,43,51,46.2276,32,55.3781,38.9637,50.8333,46.8182,52.1326,-14.235,39.3999,47.5162,36,31,60,63,60.472,53.1424,21,-35.6751,56.2639,49.8175,51.9194,45.9432,-1.8312,-9.19,30.3753,36,2.5,13,49.8153,-0.7893,24,23.6345,44.0165,24,64,15,8.538,25.3548,18.7357,4.5709,39.0742,-30.5595,-38.4161,48.3794,1.2833,64.9631,28.0339,26,45.1,31.7917,-40.9006,33,58.5953,47.4116,46.1512,53.7098,55.1694,47.1625,40.0691,29.5,26.0275,40.1431,43.9159,3.848,48.0196,0,48.669,34,41.6086,42.7339,56.8796,33.8547,42.5063,41.3775,35.1264,9.7489,22,33,-32.5228,21,12.2383,41.1533,7.54,23.7,31.24,17.6078,7.9465,15.2,35.9375,43.9424,9.082,-20.2,41.2044,31.9522,16,42.5,14.4974,23.685,42.3154,-16.2902,7,42.602636,-4.0383,-0.0236,6.4238,9.9456,4.5353,11.8251,-23.4425,11.55,-1.9403,10.6918,13.7942,-18.7669,15.7835,43.7333,47.14,8.6195,13.1939,18.1096,17.570692,9.145,1,-4.0383,25.0343,-15.4167,5,-0.8037,15.1794,11.8037,6.4281,18.9712,9.3077,-6.369,21.9162,26.3351,-11.2027,17.0608,3.2028,34.802075,1.5,-18.665695,46.8625,-22.9576,15.415,-17.7134,19.85627,13.9094,12.8628,-26.5225,12.1165,5.1521,17.357822,-4.6796,-20,15.4542,3.9193,0,28.1667,13.1939,6.6111,41.9029,-13.254308,12.9843,16.5388,8.460555,-22.3285,21.0079,12.8654,27.5142,13.4432,0.18636,24.2155,-3.3731,-6.315,6.877,-8.874217],[-95.7129,-4,12,9,2.2137,53,-3.436,35.2433,4,8.2275,5.2913,-51.9253,-8.2245,14.5501,128,35,90,16,8.4689,-7.6921,78,-71.543,9.5018,15.473,19.1451,24.9668,-78.1834,-75.0152,69.3451,138,112.5,122,6.1296,113.9213,45,-102.5528,21.0059,54,26,101,-80.7821,51.1839,-70.1627,-74.2973,21.8243,22.9375,-63.6167,31.1656,103.8333,-19.0208,1.6596,30,15.2,-7.0926,174.886,44,25.0136,28.3699,14.9955,27.9534,23.8813,19.5033,45.0382,47.75,50.55,47.5769,17.6791,11.5021,66.9237,0,19.699,9,21.7453,25.4858,24.6032,35.8623,1.5218,64.5853,33.4299,-83.7534,-80,65,-55.7658,57,-1.5616,20.1683,-5.5471,121,36.51,8.0817,-1.0232,-86.2419,14.3754,12.4578,8.6753,57.5,74.7661,35.2332,108,19.3,-14.4524,90.3563,43.3569,-63.5887,81,20.902977,21.7587,37.9062,-66.5897,-9.6966,114.7277,42.5903,-58.4438,104.9167,29.8739,-61.2225,-88.8965,46.8691,-90.2308,7.4167,9.55,0.8248,-59.5432,-77.2975,-3.996166,40.4897,32,21.7587,-77.3963,28.2833,-58.75,11.6094,39.7823,-15.1804,-9.4295,-72.2852,2.3158,34.8888,95.956,17.228331,17.8739,-61.7964,73.2207,38.996815,10,35.529562,103.8467,18.4904,-61.371,178.065,102.495496,-60.9789,30.2176,31.4659,-61.679,46.1996,-62.782998,55.492,30,18.7322,-56.0278,0,84.25,-59.5432,20.9394,12.4534,34.301525,-61.2872,-23.0418,-11.779889,24.6849,10.9408,-85.2072,90.4336,-15.3101,6.613081,-12.8858,29.9189,143.9555,31.307,125.727539],["2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08","2020-04-08"],[429052,148220,139422,113296,112950,64586,60733,38226,23403,23280,20549,16170,13141,12942,10384,9404,8672,8419,6086,6074,5916,5546,5402,5312,5205,4761,4450,4342,4263,4257,4119,3870,3034,2956,2932,2785,2666,2659,2487,2369,2249,2210,2111,2054,1884,1845,1715,1668,1623,1616,1572,1560,1343,1275,1210,1202,1185,1174,1091,1066,912,895,881,855,823,822,804,730,727,712,682,628,617,593,577,576,564,545,526,502,457,444,424,419,414,400,384,379,358,342,313,312,299,279,276,273,270,263,251,248,244,218,211,210,189,184,180,179,167,164,135,135,119,117,110,107,93,93,87,81,78,70,63,63,59,55,53,45,40,39,37,34,33,33,31,27,26,25,22,21,19,19,19,19,18,17,16,16,15,15,15,14,14,12,12,12,11,11,11,10,10,9,9,8,8,8,8,8,7,7,6,6,6,5,4,4,4,3,2,2,1],["confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed","confirmed"]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th>ProvinceState<\/th>\n      <th>CountryRegion<\/th>\n      <th>Lat<\/th>\n      <th>Long<\/th>\n      <th>date<\/th>\n      <th>count<\/th>\n      <th>subset<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":[2,3,5]}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

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

<img src="sars2pack_files/figure-html/latest_jhu_bargraph-1.png" width="672" />

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

<img src="sars2pack_files/figure-html/unnamed-chunk-2-1.png" width="672" />



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

<img src="sars2pack_files/figure-html/italy_ts_1-1.png" width="672" />

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

<img src="sars2pack_files/figure-html/cc_ts_plot-1.png" width="672" />

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

<img src="sars2pack_files/figure-html/cc_ts_plot_log-1.png" width="672" />

Consider the following questions based on the figure:

- What does the slope of the lines in this plot represent?
- What is the difference between China and other countries? What does
  this difference mean in terms of how the disease is spreading?
- What does each
- Pick an arbitrary level on the y-axis and look at the dates
  associated with each country's curve with respect to that
  level. What do differences along the x-axis tell us about where the
  countries are with respect to disease process?


