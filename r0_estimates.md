# Modeling Replication rate



## Background

TODO: PARAPHRASE!!!!!

### What is $R_0$?

$R_0$ is pronounced “R naught.” It’s a mathematical term that indicates how contagious an infectious disease is. It’s also referred to as the reproduction number. As an infection spreads to new people, it reproduces itself.

$R_0$ tells you the average number of people who will catch a disease from one contagious person. It specifically applies to a population of people who were previously free of infection and haven’t been vaccinated. If a disease has an $R_0$ of 18, a person who has the disease will transmit it to an average of 18 other people, as long as no one has been vaccinated against it or is already immune to it in their community.

### What do $R_0$ values mean?

Three possibilities exist for the potential spread or decline of a disease, depending on its $R_0$ value:

- If $R_0$ is less than 1, each existing infection causes less than one new infection. In this case, the disease will decline and eventually die out.
- If $R_0$ equals 1, each existing infection causes one new infection. The disease will stay alive and stable, but there won’t be an outbreak or an epidemic.
- If $R_0$ is more than 1, each existing infection causes more than one new infection. The disease will spread between people, and there may be an outbreak or epidemic.

Importantly, a disease’s $R_0$ value only applies when everyone in a population is completely vulnerable to the disease. This means:

- no one has been vaccinated
- no one has had the disease before
- there’s no way to control the spread of the disease

This combination of conditions is rare nowadays thanks to advances in medicine. Many diseases that were deadly in the past can now be contained and sometimes cured. For example, in 1918 there was a worldwide outbreak of the swine flu that killed 50 million people. According to a review article published in BMC Medicine, the $R_0$ value of the 1918 pandemic was estimated to be between 1.4 and 2.8. But when the swine flu, or H1N1 virus, came back in 2009, its $R_0$ value was between 1.4 and 1.6, report researchers in the journal Science. The existence of vaccines and antiviral drugs made the 2009 outbreak much less deadly.

### How is the $R_0$ of a disease calculated?

The following factors are taken into account to calculate the $R_0$ of a disease:

- *Infectious period*: Some diseases are contagious for longer periods than others. For example, according to the Centers for Disease Control and Prevention, adults with the flu are typically contagious for up to eight days, while children can be contagious for up to two weeks. The longer the infectious period of a disease, the more likely an infected person is to spread the disease to other people. A long period of infectiousness will contribute to a higher $R_0$ value.

- *Contact rate*: If a person who’s infected with a contagious disease comes into contact with many people who aren’t infected or vaccinated, the disease will spread more quickly. If that person remains at home, in a hospital, or otherwise quarantined while they’re contagious, the disease will spread more slowly. A high contact rate will contribute to a higher $R_0$ value.

- *Mode of transmission*:
The diseases that spread most quickly and easily are the ones that can travel through the air, such as the flu or measles. Physical contact with an infected person isn’t necessary for the transmission of such conditions. You can catch the flu from breathing near someone who has the flu, even if you never touch them.

In contrast, diseases that are transmitted through bodily fluids, such as Ebola or HIV, aren’t as easy to catch or spread. This is because you need to come into contact with infected blood, saliva, or other bodily fluids to contract them. Airborne illnesses tend to have a higher $R_0$ value than those spread through contact.


## Simulated epidemic model

Following code conveyed by John Mallery, we have the following
approach for estimating $R_0$ using a single realization of
an epidemic simulation.

Note that there can be failures of `estimate.R` for certain
inputs.  We are working on that.


```r
library(R0)
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```

```r
# Generating an epidemic with given parameters
mGT <- generation.time("gamma", c(3,1.5))
set.seed(5432)  # always initialize when simulating!
mEpid <- sim.epid(epid.nb=1, GT=mGT, epid.length=30, 
     family="poisson", R0=1.67, peak.value=500)
mEpid <- mEpid[,1]
# Running estimations
est <- estimate.R(epid=mEpid, GT=mGT, methods=c("EG","ML","TD"), begin=1, end=30)
```

```
## Waiting for profiling to be done...
```

```
## Warning in est.R0.TD(epid = c(1, 0, 1, 0, 1, 0, 2, 1, 2, 1, 7, 2, 3, 4, :
## Simulations may take several minutes.
```

```
## Warning in est.R0.TD(epid = c(1, 0, 1, 0, 1, 0, 2, 1, 2, 1, 7, 2, 3, 4, : Using
## initial incidence as initial number of cases.
```

We modified the plotting function in *[R0](https://CRAN.R-project.org/package=R0)* which
was calling `dev.new` too often.  Use `plot2`.


```r
par(mfrow=c(2,2))
plot2(est)
```

![](r0_estimates_files/figure-latex/lksim-1.pdf)<!-- --> 

The plotfit2 function is also useful.  These fits
look identical but they are not.


```r
par(mfrow=c(2,2))
plotfit2(est)
```

![](r0_estimates_files/figure-latex/lksim2-1.pdf)<!-- --> 

## Real data examples

Now we extract information from the time-series table and
obtain estimates of $R_0$ under exponential growth.

### Hubei Province

We are able to use exponential growth and time-dependent models
with this data, using generation time model from a
recent [Annals of Internal Medicine](https://annals.org/aim/fullarticle/2762808/incubation-period-coronavirus-disease-2019-covid-19-from-publicly-reported) paper.

The incidence data probably need smoothing, and the time-dependent
model has unreasonable fluctuations.


```r
dates = lubridate::as_date(mdy(names(mar19df)[-c(1:4)]))
hubdat = as.numeric(get_series(province="Hubei", country="China", 
    dataset=sars2pack::mar19df))
names(hubdat) = dates
mGT <- generation.time("gamma", c(5.8, 0.95)) # from DOI 10.7326/M20-0504
mGT <- generation.time("gamma", c(3.96, 4.75)) # from DOI 10.7326/M20-0504
hubdat.filt = trim_leading_values(c(hubdat[1], diff(hubdat)))
est.EG <- estimate.R(epid=hubdat.filt, GT=mGT, 
    methods=c("EG", "TD"), begin=1L, end=as.integer(length(hubdat.filt)))
```

```
## Waiting for profiling to be done...
```

```
## Warning in est.R0.TD(epid = c(`2020-01-22` = 444, `2020-01-23` = 0, `2020-01-24`
## = 105, : Simulations may take several minutes.
```

```
## Warning in est.R0.TD(epid = c(`2020-01-22` = 444, `2020-01-23` = 0, `2020-01-24`
## = 105, : Using initial incidence as initial number of cases.
```

```r
est.EG
```

```
## Reproduction number estimate using  Exponential Growth  method.
## R :  0.8190473[ 0.8164334 , 0.821658 ]
## 
## Reproduction number estimate using  Time-Dependent  method.
## 2.020789 0 3.0142 3.134995 3.32356 3.865543 1.596743 0 1.878637 2.079345 ...
```

```r
par(mfrow=c(2,2), mar=c(5,3,2,2))
plot2(est.EG)
plotfit2(est.EG)
```

![](r0_estimates_files/figure-latex/dohub-1.pdf)<!-- --> 

### Italy

For Italy, only the EG model seems to work, with the
Annals of Internal Medicine generation time model.  It
fits the data reasonably well, but the data seems to include
a reporting gap.


```r
itdat = as.numeric(get_series(province="", country="Italy", 
    dataset=sars2pack::mar19df))
names(itdat) = dates
itdat.filt = trim_leading_values(c(itdat[1], diff(itdat)))
est.EG <- estimate.R(epid=itdat.filt, GT=mGT, 
    methods=c("EG"), begin=1L, end=as.integer(length(itdat.filt)))
```

```
## Waiting for profiling to be done...
```

```r
est.EG
```

```
## Reproduction number estimate using  Exponential Growth  method.
## R :  1.968466[ 1.957161 , 1.979874 ]
```

```r
par(mfrow=c(2,2), mar=c(5,3,2,2))
plot2(est.EG, main="Italy")
plotfit2(est.EG, main="Italy")
```

![](r0_estimates_files/figure-latex/doit-1.pdf)<!-- --> 

### New York City


```r
nyt = nytimes_county_data() %>%
    dplyr::filter(county=='New York City' & subset=='confirmed') %>%
    dplyr::arrange(date)
nytdat = nyt$count
# do we need to chop zeros off? Seems like not.
nytdat.filt = c(nytdat[1], diff(nytdat))
est <- estimate.R(epid=nytdat.filt, GT=mGT, 
                  methods=c("EG","TD","ML"), begin=1L, end=as.integer(length(nytdat.filt)))
```

We can also use the package *[EpiEstim](https://CRAN.R-project.org/package=EpiEstim)* to perform time-dependent $R_0$ calculations.


```r
library(EpiEstim)
```

```
## 
## Attaching package: 'EpiEstim'
```

```
## The following object is masked from 'package:sars2pack':
## 
##     estimate_R
```

```r
epiestim = EpiEstim::estimate_R(nytdat.filt, method = "parametric_si",
                                config = EpiEstim::make_config(list(
                                    mean_si = 3.96, std_si = 4.75)))
```

```
## Default config will estimate R on weekly sliding windows.
##     To change this change the t_start and t_end arguments.
```

```r
plot(epiestim)
```

![](r0_estimates_files/figure-latex/unnamed-chunk-3-1.pdf)<!-- --> 

```
## TableGrob (3 x 1) "arrange": 3 grobs
##       z     cells    name           grob
## incid 1 (1-1,1-1) arrange gtable[layout]
## R     2 (2-2,1-1) arrange gtable[layout]
## SI    3 (3-3,1-1) arrange gtable[layout]
```
