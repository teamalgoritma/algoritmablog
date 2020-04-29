---
title: Age-Period-Cohort (APC) Analysis
author: Shelloren
github: https://github.com/AltruiMetavasi
date: '2019-03-28'
slug: age-period-cohort-apc-analysis
categories:
  - R
tags:
  - APC
  - time series
description: ''
featured: 'banner_age.jpg'
featuredalt: ''
featuredpath: 'date'
linktitle: ''
type: post
---



# Background

## Disclaimer

The following coursebook is produced by the team at [Algoritma](https://algorit.ma) for its Data Science Academy internal training. The coursebook is intended for a restricted audience only, i.e. the individuals and organizations having received this coursebook directly from the training organization. It may not be reproduced, distributed, translated or adapted in any form outside these individuals and organizations without permission. 

## Libraries and Setup

You will need install the package to do APC analysis if it's not already downloaded onto your machine.


```r
library(broom)
library(tidyverse)
```

## Training Objectives

The primary objective of this course is to provide a comprehensive introduction to the science of APC analysis. The syllabus covers:

- **APC Analysis**
- Workflow   
- APC in R  

# APC Analysis

Age-period-cohort (APC) analysis is widely used for capturing the characteristics in time-varying elements, and mostly used in epidemiology and demographic fields or we could say this is identification problem. In particular, APC analysis discerns three types of time varying phenomena[^1]: 

* **Age effects**:

    Age effect is variations linked to biological and social processes of aging specific to individuals. They include physiologic changes and accumulation of social experiences linked to aging, but unrelated to the time period or birth cohort to which an individual belongs. 
    
* **Period effects**:

    Period effect is the result from external factors that equally affect all age groups at a particular calendar time. It could arise from a range of environmental, social and economic factors e.g. war, famine, economic crisis. Methodological changes in outcome definitions, classifications, or method of data collection could also lead to period effects in data.

* **Cohort effects**:

    Cohort effect is variations resulting from the unique experience/exposure of a group of subjects (cohort) as they move across time.  In epidemiology, a cohort effect is conceptualized as an interaction or effect modification due to a period effect that is differentially experienced through age-specific exposure or susceptibility to that event or cause.

    In contrast to this conceptualization of cohort effect as an effect modification in epidemiology, sociological literature consider cohort effect as a structural factor representing the sum of all unique exposures experienced by the cohort from birth. In this case, age and period effect are conceived as confounders of cohort effect and APC analysis aims to disentangle the independent effect of age, period and cohort. Most of the APC analysis strategies are based on the sociological model of cohort effect, conceptualize independent effect of age, period and cohort effect.

**Identification problem in APC:** APC analysis aims at describing and estimating the independent effect of age, period and cohort on the outcome under study. The different strategies used aims to partition variance into the unique components attributable to age, period, and cohort effects. However, there is a major impediment to independently estimating age, period, and cohort effects by modeling the data which is know as the “identification problem” in APC. This is due to the exact linear dependency among age, period, and cohort: `\(Period – Age = Cohort\)`; that is, given the calendar year and age, one can determine the cohort (birth year). The presence of perfectly collinear predictors (age, period and cohort) in a regression model will produce a singular non-identifiable design matrix, from which it is statistically impossible to estimate unique estimates for the three effects.

  Age-Period-Cohort will be exist together at once, we saperate them to see which more important driving the societal changes. But those sometimes could be dificult because three of them were collinear.


## Conventional solutions to APC
  
Turn APC with regression in order to estimate parameters (trends and deviations). The parameters are combined to produce functions that describe relationships between the observed number of suicides and age, calendar period, and birth cohort.
 
A popular approach to resolving the identification problem was by using constraint based regression analysis (Constrained Coefficients GLIM estimator (CGLIM)). In this strategy additional constrains are placed on one of the categories of at least one predictor to simultaneously estimate the age period and cohort effect. Thus assuming some categories of age groups, cohorts or time periods have identical effects on the dependent variable it becomes possible to estimate independent effect of age period and cohort[^3]

However, the results from this analysis will depend on constrains chosen by the investigator based on external information. The validity of the constraints chosen will depend on the theoretical preconception about the categories of parameter that are identical, is often subjective and there is no empirical way to confirm the validity of the chosen constraints [^4].

### APC with `lm`

I've prepared the following data originally made available in [kaggle](https://www.kaggle.com/russellyates88/suicide-rates-overview-1985-to-2016). We'll read the data into our workspace:


```r
# load data
suicides <- read_csv("data_input/suicides.csv")

# quick check
glimpse(suicides)
```

```
#> Observations: 27,820
#> Variables: 6
#> $ suicides <dbl> 21, 16, 14, 1, 9, 1, 6, 4, 1, 0, 0, 0, 2, 17, 1, 14, 4, 8,...
#> $ age      <dbl> 20, 44, 20, 75, 30, 75, 44, 30, 64, 10, 64, 10, 75, 20, 75...
#> $ period   <dbl> 1987, 1987, 1987, 1987, 1987, 1987, 1987, 1987, 1987, 1987...
#> $ cohort   <dbl> 1967, 1943, 1967, 1912, 1957, 1912, 1943, 1957, 1923, 1977...
#> $ sex      <chr> "male", "male", "female", "male", "male", "female", "femal...
#> $ country  <chr> "Albania", "Albania", "Albania", "Albania", "Albania", "Al...
```

The data have 27.820 observation and 5 variables, here some description to get insight of the data:  

- `sucides`: number of suicides
- `age`: _self explanatory_
- `period`: current year when the data is collected
- `cohort`: birth year of each age group during that `period`
- `sex`: gender indicator
- `country`: country indicator

We need to readjust APC variable to start with:


```r
# readjust apc variable
suicides <- suicides %>%
  mutate(age_squared = age ^ 2) %>% 
  mutate_at(vars(period, cohort), funs(as.factor(.))) %>% 
  select(suicides, age, age_squared, period, cohort, everything())

suicides
```

```
#> # A tibble: 27,820 x 7
#>    suicides   age age_squared period cohort sex    country
#>       <dbl> <dbl>       <dbl> <fct>  <fct>  <chr>  <chr>  
#>  1       21    20         400 1987   1967   male   Albania
#>  2       16    44        1936 1987   1943   male   Albania
#>  3       14    20         400 1987   1967   female Albania
#>  4        1    75        5625 1987   1912   male   Albania
#>  5        9    30         900 1987   1957   male   Albania
#>  6        1    75        5625 1987   1912   female Albania
#>  7        6    44        1936 1987   1943   female Albania
#>  8        4    30         900 1987   1957   female Albania
#>  9        1    64        4096 1987   1923   male   Albania
#> 10        0    10         100 1987   1977   female Albania
#> # ... with 27,810 more rows
```

Now we have our data that contained APC variables. For the next step we applied linear regression to extract coefficient of APC with equation:

`$$y_{ij} = \alpha_{ij} + \beta_{ij}Age + \beta_{ij}Age^2 + e_{ij}$$`

where `\(i\)` is representing period, and `\(j\)` representing cohort.


```r
apc_lm <- lm(suicides ~ age + age_squared + period + cohort, suicides)

summary(apc_lm)
```

```
#> 
#> Call:
#> lm(formula = suicides ~ age + age_squared + period + cohort, 
#>     data = suicides)
#> 
#> Residuals:
#>     Min      1Q  Median      3Q     Max 
#>  -579.9  -278.9  -145.3     0.8 21762.0 
#> 
#> Coefficients: (1 not defined because of singularities)
#>               Estimate Std. Error t value Pr(>|t|)    
#> (Intercept) -339.22638   86.84338  -3.906  9.4e-05 ***
#> age           30.40751    2.32352  13.087  < 2e-16 ***
#> age_squared   -0.32299    0.02171 -14.878  < 2e-16 ***
#> period1986   -15.26465   76.16068  -0.200   0.8411    
#> period1987   -38.53047   78.43160  -0.491   0.6232    
#> period1988    -6.43966   77.80014  -0.083   0.9340    
#> period1989    66.94468   73.26358   0.914   0.3609    
#> period1990    60.13425   74.21306   0.810   0.4178    
#> period1991    67.48220   71.88165   0.939   0.3478    
#> period1992   117.89746   75.17127   1.568   0.1168    
#> period1993   147.71811   76.42130   1.933   0.0533 .  
#> period1994   138.08179   73.26564   1.885   0.0595 .  
#> period1995    66.26804   58.33994   1.136   0.2560    
#> period1996    31.27579   65.47891   0.478   0.6329    
#> period1997     4.76593   74.90887   0.064   0.9493    
#> period1998    19.28488   73.47197   0.262   0.7930    
#> period1999    61.33906   63.39151   0.968   0.3332    
#> period2000    44.19804   69.85120   0.633   0.5269    
#> period2001    36.71037   69.35817   0.529   0.5966    
#> period2002    89.40211   72.32410   1.236   0.2164    
#> period2003   114.10428   73.45058   1.553   0.1203    
#> period2004   103.85482   71.23562   1.458   0.1449    
#> period2005    53.98893   57.77302   0.935   0.3501    
#> period2006    12.47963   70.09375   0.178   0.8587    
#> period2007   -11.43421   76.46974  -0.150   0.8811    
#> period2008     0.25884   77.20994   0.003   0.9973    
#> period2009     6.49784   67.83942   0.096   0.9237    
#> period2010    -0.99290   72.96721  -0.014   0.9891    
#> period2011    -1.10585   74.49438  -0.015   0.9882    
#> period2012    37.47225   76.92286   0.487   0.6262    
#> period2013    60.65137   77.25632   0.785   0.4324    
#> period2014    64.04916   76.78528   0.834   0.4042    
#> period2015    81.00665   61.20244   1.324   0.1857    
#> period2016  -176.96659   95.17267  -1.859   0.0630 .  
#> cohort1911    22.61882  148.70351   0.152   0.8791    
#> cohort1912    44.08371  146.19982   0.302   0.7630    
#> cohort1913    26.93923  147.81696   0.182   0.8554    
#> cohort1914   -24.12898  143.26606  -0.168   0.8663    
#> cohort1915   -19.81134  138.07605  -0.143   0.8859    
#> cohort1916   -28.98740  136.20152  -0.213   0.8315    
#> cohort1917   -82.84906  136.98890  -0.605   0.5453    
#> cohort1918  -114.37741  137.04714  -0.835   0.4040    
#> cohort1919  -111.87468  133.66927  -0.837   0.4026    
#> cohort1920   -56.80810  122.40780  -0.464   0.6426    
#> cohort1921    -7.46818  105.86601  -0.071   0.9438    
#> cohort1922    19.71152  117.59522   0.168   0.8669    
#> cohort1923    18.66899  116.22700   0.161   0.8724    
#> cohort1924   -22.24166  111.82124  -0.199   0.8423    
#> cohort1925   -13.71874  112.07531  -0.122   0.9026    
#> cohort1926   -13.01545  110.40762  -0.118   0.9062    
#> cohort1927   -36.82096  109.98751  -0.335   0.7378    
#> cohort1928   -62.24102  110.61311  -0.563   0.5736    
#> cohort1929   -59.50693  109.51221  -0.543   0.5869    
#> cohort1930   -27.45295  103.07426  -0.266   0.7900    
#> cohort1931    12.35088  101.39101   0.122   0.9030    
#> cohort1932    47.60027  105.34566   0.452   0.6514    
#> cohort1933    51.11593  107.56988   0.475   0.6347    
#> cohort1934    41.37847  102.76022   0.403   0.6872    
#> cohort1935    21.78125  101.07994   0.215   0.8294    
#> cohort1936    21.87097  102.64412   0.213   0.8313    
#> cohort1937     2.80057  102.45819   0.027   0.9782    
#> cohort1938   -28.18272  103.14675  -0.273   0.7847    
#> cohort1939   -36.82257  102.73980  -0.358   0.7200    
#> cohort1940   -35.64866   97.49467  -0.366   0.7146    
#> cohort1941    10.99553   90.49327   0.122   0.9033    
#> cohort1942    35.36708  105.19654   0.336   0.7367    
#> cohort1943    51.70791  107.13860   0.483   0.6294    
#> cohort1944    45.48532  107.37955   0.424   0.6719    
#> cohort1945    56.59088  101.43700   0.558   0.5769    
#> cohort1946    64.81355  102.18740   0.634   0.5259    
#> cohort1947    74.93599  101.21464   0.740   0.4591    
#> cohort1948    60.15492  103.81245   0.579   0.5623    
#> cohort1949    53.42971  103.83011   0.515   0.6068    
#> cohort1950    64.52937  101.63502   0.635   0.5255    
#> cohort1951   105.37591   90.35037   1.166   0.2435    
#> cohort1952   162.65568  109.16427   1.490   0.1362    
#> cohort1953   193.34415  119.11312   1.623   0.1046    
#> cohort1954   187.23643  117.66962   1.591   0.1116    
#> cohort1955    67.65765   93.95205   0.720   0.4715    
#> cohort1956    82.22594  101.94472   0.807   0.4199    
#> cohort1957    73.24009  100.42288   0.729   0.4658    
#> cohort1958    49.52042  102.82867   0.482   0.6301    
#> cohort1959    28.79550  101.93464   0.282   0.7776    
#> cohort1960    12.82476   98.64709   0.130   0.8966    
#> cohort1961    25.46981   92.78207   0.275   0.7837    
#> cohort1962    23.88102   99.43981   0.240   0.8102    
#> cohort1963    21.96044  101.38432   0.217   0.8285    
#> cohort1964    22.59980   99.85004   0.226   0.8209    
#> cohort1965    35.39070   84.69657   0.418   0.6761    
#> cohort1966    52.19842   93.33271   0.559   0.5760    
#> cohort1967    54.75169   94.88410   0.577   0.5639    
#> cohort1968    29.30687   95.96987   0.305   0.7601    
#> cohort1969   -16.64207   93.55918  -0.178   0.8588    
#> cohort1970   -13.15350   93.01375  -0.141   0.8875    
#> cohort1971    -9.63131   85.70810  -0.112   0.9105    
#> cohort1972   -94.85714   99.17764  -0.956   0.3389    
#> cohort1973  -129.92048  102.45307  -1.268   0.2048    
#> cohort1974  -122.17291   98.75499  -1.237   0.2160    
#> cohort1975   -33.97887   84.09864  -0.404   0.6862    
#> cohort1976     0.05253   95.53073   0.001   0.9996    
#> cohort1977    26.80063   98.50027   0.272   0.7856    
#> cohort1978     8.19420   98.95543   0.083   0.9340    
#> cohort1979   -28.14974   94.99078  -0.296   0.7670    
#> cohort1980   -18.22349   95.74013  -0.190   0.8490    
#> cohort1981   -19.22080   93.68167  -0.205   0.8374    
#> cohort1982   -61.15087   98.26652  -0.622   0.5338    
#> cohort1983   -89.94060   99.31063  -0.906   0.3651    
#> cohort1984   -84.16653   96.09542  -0.876   0.3811    
#> cohort1985   -35.15202   83.97337  -0.419   0.6755    
#> cohort1986    23.02419  101.33499   0.227   0.8203    
#> cohort1987    52.56698  106.34666   0.494   0.6211    
#> cohort1988    42.80289  106.57370   0.402   0.6880    
#> cohort1989    16.05302  102.94645   0.156   0.8761    
#> cohort1990    26.81317  104.40628   0.257   0.7973    
#> cohort1991    33.33100  102.69174   0.325   0.7455    
#> cohort1992   -12.38976  107.04939  -0.116   0.9079    
#> cohort1993   -42.05786  108.45276  -0.388   0.6982    
#> cohort1994   -37.15501  105.55691  -0.352   0.7248    
#> cohort1995    -5.11853   93.50591  -0.055   0.9563    
#> cohort1996    66.76549  116.98898   0.571   0.5682    
#> cohort1997    88.59953  124.53294   0.711   0.4768    
#> cohort1998    76.83842  125.30561   0.613   0.5397    
#> cohort1999    71.26135  121.94267   0.584   0.5590    
#> cohort2000    78.44310  123.67906   0.634   0.5259    
#> cohort2001    79.48047  123.21747   0.645   0.5189    
#> cohort2002    40.79277  127.30436   0.320   0.7486    
#> cohort2003    18.08633  128.90684   0.140   0.8884    
#> cohort2004    15.46515  127.77105   0.121   0.9037    
#> cohort2005          NA         NA      NA       NA    
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 889.6 on 27692 degrees of freedom
#> Multiple R-squared:  0.03193,	Adjusted R-squared:  0.02749 
#> F-statistic: 7.193 on 127 and 27692 DF,  p-value: < 2.2e-16
```

Use package [broom](https://cran.r-project.org/web/packages/broom/vignettes/broom.html) to constructs a data frame that summarizes the model’s statistical findings.
  

```r
apc_tidy <- tidy(apc_lm)

apc_tidy
```

```
#> # A tibble: 128 x 5
#>    term        estimate std.error statistic  p.value
#>    <chr>          <dbl>     <dbl>     <dbl>    <dbl>
#>  1 (Intercept) -339.      86.8      -3.91   9.40e- 5
#>  2 age           30.4      2.32     13.1    5.11e-39
#>  3 age_squared   -0.323    0.0217  -14.9    7.18e-50
#>  4 period1986   -15.3     76.2      -0.200  8.41e- 1
#>  5 period1987   -38.5     78.4      -0.491  6.23e- 1
#>  6 period1988    -6.44    77.8      -0.0828 9.34e- 1
#>  7 period1989    66.9     73.3       0.914  3.61e- 1
#>  8 period1990    60.1     74.2       0.810  4.18e- 1
#>  9 period1991    67.5     71.9       0.939  3.48e- 1
#> 10 period1992   118.      75.2       1.57   1.17e- 1
#> # ... with 118 more rows
```

From tidied coefficients, we need to tidy our results, starting from period and cohort first:


```r
pc_tidy <- apc_tidy %>%
  mutate(
    key = str_replace_all(term, "[:digit:]", ""), 
    value = str_replace_all(term, "[:alpha:]", "") %>% as.numeric() 
  ) %>% # seperate digit number and alphabet of term 
  select(key, value, estimate) %>% 
  filter(key == "period" | key == "cohort") %>% 
  spread(key, value)

pc_tidy
```

```
#> # A tibble: 125 x 3
#>    estimate cohort period
#>       <dbl>  <dbl>  <dbl>
#>  1   -177.      NA   2016
#>  2   -130.    1973     NA
#>  3   -122.    1974     NA
#>  4   -114.    1918     NA
#>  5   -112.    1919     NA
#>  6    -94.9   1972     NA
#>  7    -89.9   1983     NA
#>  8    -84.2   1984     NA
#>  9    -82.8   1917     NA
#> 10    -62.2   1928     NA
#> # ... with 115 more rows
```

Then we can extract and tidy each coefficient using code in the chunk below:


```r
age_coef <- apc_tidy %>% 
  filter(term == "age") %>% 
  pull(estimate)

age_squared_coef <- apc_tidy %>% 
  filter(term == "age_squared") %>% 
  pull(estimate)

period_coef <- pc_tidy %>% 
  select(x = period, y = estimate) %>% 
  drop_na()

cohort_coef <- pc_tidy %>% 
  select(x = cohort, y = estimate) %>% 
  drop_na()
```

Now we can calculate the age effect, then combine all the coefficient together:


```r
apc_all <- suicides %>%
  mutate(age_effect = (age * age_coef) + (age_squared * age_squared_coef)) %>% 
  select(x = age, y = age_effect) %>% 
  distinct() %>% 
  mutate(group = "Age Effect") %>% 
  bind_rows(mutate(period_coef, group = "Period Effect")) %>% 
  bind_rows(mutate(cohort_coef, group = "Cohort Effect")) %>% 
  select(group, x, y) %>% 
  arrange(x)

apc_all
```

```
#> # A tibble: 131 x 3
#>    group             x     y
#>    <chr>         <dbl> <dbl>
#>  1 Age Effect       10 272. 
#>  2 Age Effect       20 479. 
#>  3 Age Effect       30 622. 
#>  4 Age Effect       44 713. 
#>  5 Age Effect       64 623. 
#>  6 Age Effect       75 464. 
#>  7 Cohort Effect  1911  22.6
#>  8 Cohort Effect  1912  44.1
#>  9 Cohort Effect  1913  26.9
#> 10 Cohort Effect  1914 -24.1
#> # ... with 121 more rows
```

The last but not least, we can visualize the estimated coefficient to get some insight regarding our APC decomposition:


```r
ggplot(apc_all, aes(x, y)) +
  geom_line() +
  facet_wrap(~ group, scales = "free", ncol = 1)
```

<img src="/blog/2019-03-28-age-period-cohort-apc-analysis_files/figure-html/unnamed-chunk-8-1.png" width="672" style="display: block; margin: auto;" />

From plot above we can identify Age, Period and Cohort effect of suicides. But it's not capturing all of our the data dynamics, lets try it to more specific cases from the data. We filltered the data by sex and country:


```r
suicides_male_us <- suicides %>% 
  filter(
    sex == "male",
    country == "United States"
  )

suicides_male_us
```

```
#> # A tibble: 186 x 7
#>    suicides   age age_squared period cohort sex   country      
#>       <dbl> <dbl>       <dbl> <fct>  <fct>  <chr> <chr>        
#>  1     2177    75        5625 1985   1910   male  United States
#>  2     5302    64        4096 1985   1921   male  United States
#>  3     5134    30         900 1985   1955   male  United States
#>  4     6053    44        1936 1985   1941   male  United States
#>  5     4267    20         400 1985   1965   male  United States
#>  6      205    10         100 1985   1975   male  United States
#>  7     2340    75        5625 1986   1911   male  United States
#>  8     5500    64        4096 1986   1922   male  United States
#>  9     5450    30         900 1986   1956   male  United States
#> 10     6450    44        1936 1986   1942   male  United States
#> # ... with 176 more rows
```

Now we have our data that contained APC variables of `male` in `United States`. For the next step we applied linear regression to extract coefficient of APC from the data :


```r
apc_lm <- lm(suicides ~ age + age_squared + period + cohort, suicides_male_us)

summary(apc_lm)
```

```
#> 
#> Call:
#> lm(formula = suicides ~ age + age_squared + period + cohort, 
#>     data = suicides_male_us)
#> 
#> Residuals:
#>     Min      1Q  Median      3Q     Max 
#> -2474.8  -378.9     0.0   439.4  2474.8 
#> 
#> Coefficients: (1 not defined because of singularities)
#>               Estimate Std. Error t value Pr(>|t|)    
#> (Intercept) -5925.2423  1716.8546  -3.451  0.00104 ** 
#> age           433.4174    47.4850   9.127 7.01e-13 ***
#> age_squared    -4.3385     0.4684  -9.262 4.19e-13 ***
#> period1986   -234.8724  1515.8754  -0.155  0.87740    
#> period1987   -377.2245  1590.8747  -0.237  0.81339    
#> period1988   -114.9018  1562.8255  -0.074  0.94164    
#> period1989    332.2516  1468.4095   0.226  0.82178    
#> period1990    399.7556  1534.8937   0.260  0.79543    
#> period1991    329.7095  1471.5206   0.224  0.82348    
#> period1992    912.2598  1591.6599   0.573  0.56872    
#> period1993   1023.7514  1593.4089   0.642  0.52304    
#> period1994    787.2440  1522.2825   0.517  0.60699    
#> period1995    478.5536  1170.8616   0.409  0.68423    
#> period1996   -471.4333  1372.8457  -0.343  0.73252    
#> period1997   -606.9015  1555.5320  -0.390  0.69783    
#> period1998   -413.7255  1535.9278  -0.269  0.78859    
#> period1999    220.0764  1299.5798   0.169  0.86611    
#> period2000    238.8850  1451.0640   0.165  0.86980    
#> period2001    435.6227  1441.6270   0.302  0.76358    
#> period2002   1223.3076  1531.3070   0.799  0.42757    
#> period2003   1498.8270  1533.8367   0.977  0.33247    
#> period2004   1457.0321  1471.2956   0.990  0.32607    
#> period2005    827.0042  1150.5815   0.719  0.47512    
#> period2006    331.3115  1484.2611   0.223  0.82414    
#> period2007    260.8998  1593.2016   0.164  0.87048    
#> period2008    565.8727  1603.4460   0.353  0.72541    
#> period2009   1053.6846  1402.4886   0.751  0.45546    
#> period2010   1262.4640  1529.5100   0.825  0.41247    
#> period2011   1440.0666  1551.2895   0.928  0.35703    
#> period2012   1983.7952  1616.2291   1.227  0.22454    
#> period2013   2322.9728  1611.8220   1.441  0.15481    
#> period2014   2446.7770  1590.2011   1.539  0.12923    
#> period2015   2279.9186  1191.5563   1.913  0.06055 .  
#> cohort1911    397.8724  2687.2179   0.148  0.88280    
#> cohort1912    732.2245  2720.6192   0.269  0.78876    
#> cohort1913    485.9018  2694.6122   0.180  0.85752    
#> cohort1914    -47.2516  2631.0384  -0.018  0.98573    
#> cohort1915     76.2444  2658.8832   0.029  0.97722    
#> cohort1916    166.2905  2612.8067   0.064  0.94947    
#> cohort1917   -516.2598  2672.5255  -0.193  0.84749    
#> cohort1918   -551.7514  2663.7545  -0.207  0.83662    
#> cohort1919   -335.2440  2611.8202  -0.128  0.89830    
#> cohort1920    -75.5536  2413.0484  -0.031  0.97513    
#> cohort1921   1069.7292  1916.9792   0.558  0.57894    
#> cohort1922   1343.8996  2223.3576   0.604  0.54786    
#> cohort1923   1369.4876  2229.7009   0.614  0.54144    
#> cohort1924    710.9253  2139.2092   0.332  0.74082    
#> cohort1925    491.4443  2147.4871   0.229  0.81978    
#> cohort1926    301.3235  2152.1535   0.140  0.88913    
#> cohort1927    -73.9960  2140.7908  -0.035  0.97254    
#> cohort1928   -657.0308  2175.1135  -0.302  0.76366    
#> cohort1929   -703.8791  2143.2927  -0.328  0.74376    
#> cohort1930   -363.1115  2005.5606  -0.181  0.85695    
#> cohort1931     35.5800  1982.0142   0.018  0.98574    
#> cohort1932    584.7793  2085.7695   0.280  0.78018    
#> cohort1933    433.0270  2136.2842   0.203  0.84007    
#> cohort1934     79.0331  2048.7028   0.039  0.96936    
#> cohort1935   -389.7576  2015.1358  -0.193  0.84730    
#> cohort1936   -512.4632  2057.2377  -0.249  0.80415    
#> cohort1937   -591.6963  2059.5317  -0.287  0.77489    
#> cohort1938   -943.1276  2079.7766  -0.453  0.65187    
#> cohort1939  -1059.2894  2062.1911  -0.514  0.60940    
#> cohort1940   -898.4627  1881.1383  -0.478  0.63469    
#> cohort1941    813.6172  1744.7670   0.466  0.64270    
#> cohort1942   1497.8998  2076.5481   0.721  0.47355    
#> cohort1943   1863.2817  2130.2349   0.875  0.38529    
#> cohort1944   1923.1338  2122.3509   0.906  0.36855    
#> cohort1945   1745.1513  2019.2109   0.864  0.39094    
#> cohort1946   2048.0095  2065.0405   0.992  0.32537    
#> cohort1947   2233.7313  2029.5127   1.101  0.27553    
#> cohort1948   2013.0919  2107.2405   0.955  0.34331    
#> cohort1949   2142.7572  2099.5614   1.021  0.31162    
#> cohort1950   2568.6088  2050.3064   1.253  0.21522    
#> cohort1951   3032.8832  1753.2899   1.730  0.08889 .  
#> cohort1952   4709.6467  2392.6906   1.968  0.05373 .  
#> cohort1953   4781.1150  2472.7116   1.934  0.05797 .  
#> cohort1954   4859.9390  2461.9013   1.974  0.05306 .  
#> cohort1955   2934.2540  1807.8481   1.623  0.10991    
#> cohort1956   3395.2859  2035.1210   1.668  0.10055    
#> cohort1957   3538.5931  2032.5711   1.741  0.08691 .  
#> cohort1958   3268.5893  2070.1321   1.579  0.11970    
#> cohort1959   2807.2529  2047.1809   1.371  0.17548    
#> cohort1960   2854.3984  2009.2820   1.421  0.16069    
#> cohort1961   3308.9354  1882.3955   1.758  0.08396 .  
#> cohort1962   3290.5066  2045.8841   1.608  0.11310    
#> cohort1963   3496.4666  2082.0656   1.679  0.09838 .  
#> cohort1964   3697.2339  2049.6374   1.804  0.07636 .  
#> cohort1965   3551.8804  1665.9913   2.132  0.03718 *  
#> cohort1966   3802.9070  1895.8336   2.006  0.04946 *  
#> cohort1967   3697.3129  1932.0752   1.914  0.06052 .  
#> cohort1968   3309.5705  1952.2085   1.695  0.09529 .  
#> cohort1969   2631.5262  1901.7046   1.384  0.17164    
#> cohort1970   2513.8206  1906.0440   1.319  0.19231    
#> cohort1971   2644.8762  1702.5491   1.553  0.12566    
#> cohort1972    931.5493  2117.8954   0.440  0.66166    
#> cohort1973    786.0437  2124.1419   0.370  0.71267    
#> cohort1974   1009.6949  2043.0762   0.494  0.62300    
#> cohort1975   1647.3423  1676.2587   0.983  0.32974    
#> cohort1976   2065.1929  1953.7135   1.057  0.29479    
#> cohort1977   2202.2702  2014.7840   1.093  0.27881    
#> cohort1978   1931.1130  2022.3138   0.955  0.34352    
#> cohort1979   1347.1907  1949.3224   0.691  0.49221    
#> cohort1980   1402.4933  1982.2140   0.708  0.48202    
#> cohort1981   1372.7286  1926.4556   0.713  0.47892    
#> cohort1982    791.4073  2047.8569   0.386  0.70055    
#> cohort1983    558.6778  2059.7818   0.271  0.78716    
#> cohort1984    735.1771  1985.3703   0.370  0.71249    
#> cohort1985   1173.3694  1685.9935   0.696  0.48919    
#> cohort1986   2455.1676  2162.1592   1.136  0.26075    
#> cohort1987   2538.6076  2227.8080   1.140  0.25910    
#> cohort1988   2333.0332  2235.9342   1.043  0.30101    
#> cohort1989   1766.2263  2166.0241   0.815  0.41811    
#> cohort1990   1765.9323  2206.4778   0.800  0.42672    
#> cohort1991   1657.2621  2164.6230   0.766  0.44696    
#> cohort1992    963.5554  2261.0318   0.426  0.67154    
#> cohort1993    646.2068  2281.6992   0.283  0.77800    
#> cohort1994    694.7022  2207.9443   0.315  0.75415    
#> cohort1995   1236.6453  1884.6583   0.656  0.51427    
#> cohort1996   1846.6071  2613.5257   0.707  0.48262    
#> cohort1997   1895.0188  2670.1223   0.710  0.48068    
#> cohort1998   1613.0459  2679.0297   0.602  0.54941    
#> cohort1999   1146.2341  2629.8720   0.436  0.66453    
#> cohort2000    949.4546  2667.0024   0.356  0.72311    
#> cohort2001    785.8521  2640.8477   0.298  0.76707    
#> cohort2002    267.1235  2710.4500   0.099  0.92183    
#> cohort2003    -46.0542  2732.6571  -0.017  0.98661    
#> cohort2004   -144.8584  2692.9176  -0.054  0.95728    
#> cohort2005          NA         NA      NA       NA    
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 1577 on 59 degrees of freedom
#> Multiple R-squared:  0.9105,	Adjusted R-squared:  0.7194 
#> F-statistic: 4.765 on 126 and 59 DF,  p-value: 3.789e-10
```

Again, from tidied coefficients, we need to tidy our results:


```r
apc_tidy <- tidy(apc_lm)

pc_tidy <- apc_tidy %>%
  mutate(
    key = str_replace_all(term, "[:digit:]", ""), 
    value = str_replace_all(term, "[:alpha:]", "") %>% as.numeric() 
  ) %>% # seperate digit number and alphabet of term 
  select(key, value, estimate) %>% 
  filter(key == "period" | key == "cohort") %>% 
  spread(key, value)

age_coef <- apc_tidy %>% 
  filter(term == "age") %>% 
  pull(estimate)

age_squared_coef <- apc_tidy %>% 
  filter(term == "age_squared") %>% 
  pull(estimate)

period_coef <- pc_tidy %>% 
  select(x = period, y = estimate) %>% 
  drop_na()

cohort_coef <- pc_tidy %>% 
  select(x = cohort, y = estimate) %>% 
  drop_na()
```

Now we can calculate the age effect, then combine all the coefficient together:


```r
apc_all_male_us <- suicides_male_us %>%
  mutate(age_effect = (age * age_coef) + (age_squared * age_squared_coef)) %>% 
  select(x = age, y = age_effect) %>% 
  distinct() %>% 
  mutate(group = "Age Effect") %>% 
  bind_rows(mutate(period_coef, group = "Period Effect")) %>% 
  bind_rows(mutate(cohort_coef, group = "Cohort Effect")) %>% 
  select(group, x, y) %>% 
  arrange(x)

apc_all_male_us
```

```
#> # A tibble: 130 x 3
#>    group             x       y
#>    <chr>         <dbl>   <dbl>
#>  1 Age Effect       10  3900. 
#>  2 Age Effect       20  6933. 
#>  3 Age Effect       30  9098. 
#>  4 Age Effect       44 10671. 
#>  5 Age Effect       64  9968. 
#>  6 Age Effect       75  8102. 
#>  7 Cohort Effect  1911   398. 
#>  8 Cohort Effect  1912   732. 
#>  9 Cohort Effect  1913   486. 
#> 10 Cohort Effect  1914   -47.3
#> # ... with 120 more rows
```

The last we visualize the estimated coefficient to get some insight regarding our APC decomposition:


```r
ggplot(apc_all_male_us, aes(x, y)) +
  geom_line() +
  facet_wrap(~ group, scales = "free", ncol = 1)
```

<img src="/blog/2019-03-28-age-period-cohort-apc-analysis_files/figure-html/unnamed-chunk-13-1.png" width="672" style="display: block; margin: auto;" />

*dive deeper*

Create APC analysis, try to compare the APC of `female` in `United States` with the APC we get above:


```r
suicides_female_us <- suicides %>% 
  filter(
    sex == "female",
    country == "United States"
  )

suicides_female_us
```



# Annotations

[^1]: Yang Y, Schulhofer‐Wohl S, Fu WJ, Land KC. The Intrinsic Estimator for Age‐Period‐Cohort Analysis: What It Is and How to Use It1. American Journal of Sociology 2008;113(6):1697-736.
[^2]: Yang, Yang, and Kenneth C. Land. Age-period-cohort analysis: new models, methods, and empirical applications. CRC Press, 2013
[^3]: Mason, Karen Oppenheim, et al. “Some methodological issues in cohort analysis of archival data.” American sociological review (1973): 242-258
[^4]: Keyes KM, Utz RL, Robinson W, Li G. What is a cohort effect? Comparison of three statistical methods for modeling cohort effects in obesity prevalence in the United States, 1971-2006. Soc Sci Med 2010;70(7):1100-8
