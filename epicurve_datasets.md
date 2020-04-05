# Epicurves


```r
library(sars2pack)
```

```
## Loading required package: R0
```

```
## Loading required package: MASS
```

```
## Loading required package: sf
```

```
## Linking to GEOS 3.7.2, GDAL 2.4.2, PROJ 5.2.0
```

```r
library(DT)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following object is masked from 'package:MASS':
## 
##     select
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

## Worldwide datasets


```r
jhu = jhu_data()
DT::datatable(jhu[sample(1:nrow(jhu),500),] %>%
              dplyr::arrange(CountryRegion, date),
              extensions = 'Scroller', options = list(scrollX=TRUE))
```

![](epicurve_datasets_files/figure-latex/unnamed-chunk-2-1.pdf)<!-- --> 

## European datasets


```r
eucov = eu_data_cache_data()
DT::datatable(eucov[sample(1:nrow(eucov),500),] %>%
              dplyr::arrange(country, date),
              extensions = 'Scroller', options = list(scrollX=TRUE))
```

![](epicurve_datasets_files/figure-latex/unnamed-chunk-3-1.pdf)<!-- --> 

## United States datasets

### USAFacts


```r
usa_facts = usa_facts_data()
DT::datatable(usa_facts[sample(1:nrow(usa_facts),500),] %>%
              dplyr::arrange(state, county, date, subset),
              extensions = 'Scroller', options = list(scrollX=TRUE))
```

![](epicurve_datasets_files/figure-latex/us_epicurves-1.pdf)<!-- --> 

### New York Times


```r
nytimes = nytimes_county_data()
DT::datatable(nytimes[sample(1:nrow(nytimes),500),] %>%
              dplyr::arrange(state, county, date, subset),
              extensions = 'Scroller', options = list(scrollX=TRUE))
```

![](epicurve_datasets_files/figure-latex/nyt_epicurves-1.pdf)<!-- --> 
