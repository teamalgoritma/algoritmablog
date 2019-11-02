---
title: Introduction to tidymodels
author: R. Dimas Bagas Herlambang
github: https://github.com/bagasbgy
date: '2019-10-06'
slug: tidymodels
categories:
  - R
tags:
  - tidymodels
  - tidyverse
  - Machine Learning
description: ''
featured: ''
featuredalt: ''
featuredpath: ''
linktitle: '' 
type: post
---



The following presentation is produced by the team at [Algoritma](https://algorit.ma) for its internal training This presentation is intended for a restricted audience only. It may not be reproduced, distributed, translated or adapted in any form outside these individuals and organizations without permission.

# Outline

## Why `tidymodels` Matters?

* Things we think we're doing it right
* Things we never think we could do it better

## Setting the Cross-Validation Scheme using `rsample`

* Rethinking: Why we need validation?
* Tidy way to `rsample`-ing your dataset

## Data Preprocess using `recipes`

* Rethinking: How we should treat train and test?
* Reproducible preprocess `recipes`

## Model Fitting using `parsnip`

* Rethinking: How many machine learning packages you used?
* One vegeta.. I mean package to rule them all: `parsnip`

## Model Evaluation using `yardstick`

* Rethinking: How we measure the goodness of our model?
* It's always good to bring your own `yardstick`


# Why `tidymodels` Matters?

## Things we think we're doing it right

### Sample splitting


```r
# import libs
library(plotly)
library(randomForest)
library(ranger)
library(tidyverse)
library(tidymodels)
```


```r
# import additional libs
library(lubridate)

# prepare example datasets
attrition <- read_csv("data_input/attrition.csv")
```


```r
# set seed
set.seed(100)

# train rows
in_train <- sample(1:nrow(attrition), nrow(attrition) * 0.8)

# check target distribution in train
prop.table(table(attrition$attrition[in_train]))
```

```
#> 
#>        no       yes 
#> 0.8367347 0.1632653
```

```r
# check target distribution in test
prop.table(table(attrition$attrition[-in_train]))
```

```
#> 
#>        no       yes 
#> 0.8469388 0.1530612
```


### Numeric scaling


```r
# scale age in full dataset
age_scaled <- scale(attrition$age)

# check mean and standard deviation
attr(age_scaled, "scaled:center")
```

```
#> [1] 36.92381
```

```r
attr(age_scaled, "scaled:scale")
```

```
#> [1] 9.135373
```

```r
# scale age in train dataset
age_train_scaled <- scale(attrition$age[in_train])

# check mean and standard deviation
attr(age_train_scaled, "scaled:center")
```

```
#> [1] 37.0085
```

```r
attr(age_train_scaled, "scaled:scale")
```

```
#> [1] 9.070969
```

```r
# scale age in test dataset
age_test_scaled <- scale(attrition$age[-in_train])

# check mean and standard deviation
attr(age_test_scaled, "scaled:center")
```

```
#> [1] 36.58503
```

```r
attr(age_test_scaled, "scaled:scale")
```

```
#> [1] 9.396712
```

## Things we never think we could do it better


### How we see model performance


```
#> Confusion Matrix and Statistics
#> 
#>           Reference
#> Prediction   no  yes
#>        no  1036  197
#>        yes  197   40
#>                                           
#>                Accuracy : 0.732           
#>                  95% CI : (0.7085, 0.7545)
#>     No Information Rate : 0.8388          
#>     P-Value [Acc > NIR] : 1               
#>                                           
#>                   Kappa : 0.009           
#>                                           
#>  Mcnemar's Test P-Value : 1               
#>                                           
#>             Sensitivity : 0.16878         
#>             Specificity : 0.84023         
#>          Pos Pred Value : 0.16878         
#>          Neg Pred Value : 0.84023         
#>              Prevalence : 0.16122         
#>          Detection Rate : 0.02721         
#>    Detection Prevalence : 0.16122         
#>       Balanced Accuracy : 0.50450         
#>                                           
#>        'Positive' Class : yes             
#> 
```

### How we use Receiver Operating Curve

<img src="/blog/2019-10-06-tidymodels_files/figure-html/unnamed-chunk-4-1.png" width="672" style="display: block; margin: auto;" />


# Setting the Cross-Validation Scheme using `rsample`

## Rethinking: Why we need validation?

## Tidy way to `rsample`-ing your dataset


[`rsample`](https://tidymodels.github.io/rsample/) is part of `tidymodels` that could help us in splitting or resampling or machine learning dataset.

There are so many splitting and resampling approach provided by `rsample`--as you could see in its [full function references page](https://tidymodels.github.io/rsample/reference/index.html). In this introduction, we will use two most general function:

* [`initial_split()`](https://tidymodels.github.io/rsample/reference/initial_split.html):
    Simple train and test splitting.
* [`vfold_cv()`](https://tidymodels.github.io/rsample/reference/vfold_cv.html):
    k-fold splitting, with optional repetition argument.


### Initial splitting

Initial random sampling for splitting train and test could be done using `initial_split()`:


```r
# set seed
set.seed(100)

# create initial split
splitted <- initial_split(attrition, prop = 0.8)

# check train dataset
training(splitted)
```

```
#> # A tibble: 1,177 x 35
#>    attrition   age business_travel daily_rate department distance_from_h~
#>    <chr>     <dbl> <chr>                <dbl> <chr>                 <dbl>
#>  1 yes          41 travel_rarely         1102 sales                     1
#>  2 no           49 travel_frequen~        279 research_~                8
#>  3 yes          37 travel_rarely         1373 research_~                2
#>  4 no           33 travel_frequen~       1392 research_~                3
#>  5 no           27 travel_rarely          591 research_~                2
#>  6 no           32 travel_frequen~       1005 research_~                2
#>  7 no           59 travel_rarely         1324 research_~                3
#>  8 no           30 travel_rarely         1358 research_~               24
#>  9 no           38 travel_frequen~        216 research_~               23
#> 10 no           36 travel_rarely         1299 research_~               27
#> # ... with 1,167 more rows, and 29 more variables: education <dbl>,
#> #   education_field <chr>, employee_count <dbl>, employee_number <dbl>,
#> #   environment_satisfaction <dbl>, gender <chr>, hourly_rate <dbl>,
#> #   job_involvement <dbl>, job_level <dbl>, job_role <chr>,
#> #   job_satisfaction <dbl>, marital_status <chr>, monthly_income <dbl>,
#> #   monthly_rate <dbl>, num_companies_worked <dbl>, over_18 <chr>,
#> #   over_time <chr>, percent_salary_hike <dbl>, performance_rating <dbl>,
#> #   relationship_satisfaction <dbl>, standard_hours <dbl>,
#> #   stock_option_level <dbl>, total_working_years <dbl>,
#> #   training_times_last_year <dbl>, work_life_balance <dbl>,
#> #   years_at_company <dbl>, years_in_current_role <dbl>,
#> #   years_since_last_promotion <dbl>, years_with_curr_manager <dbl>
```

```r
# check test dataset
testing(splitted)
```

```
#> # A tibble: 293 x 35
#>    attrition   age business_travel daily_rate department distance_from_h~
#>    <chr>     <dbl> <chr>                <dbl> <chr>                 <dbl>
#>  1 no           29 travel_rarely         1389 research_~               21
#>  2 no           38 travel_rarely          371 research_~                2
#>  3 yes          36 travel_rarely         1218 sales                     9
#>  4 yes          34 travel_rarely          699 research_~                6
#>  5 no           53 travel_rarely         1282 research_~                5
#>  6 yes          24 travel_rarely          813 research_~                1
#>  7 no           36 travel_rarely          852 research_~                5
#>  8 no           27 travel_rarely         1240 research_~                2
#>  9 yes          41 travel_rarely         1360 research_~               12
#> 10 yes          48 travel_rarely          626 research_~                1
#> # ... with 283 more rows, and 29 more variables: education <dbl>,
#> #   education_field <chr>, employee_count <dbl>, employee_number <dbl>,
#> #   environment_satisfaction <dbl>, gender <chr>, hourly_rate <dbl>,
#> #   job_involvement <dbl>, job_level <dbl>, job_role <chr>,
#> #   job_satisfaction <dbl>, marital_status <chr>, monthly_income <dbl>,
#> #   monthly_rate <dbl>, num_companies_worked <dbl>, over_18 <chr>,
#> #   over_time <chr>, percent_salary_hike <dbl>, performance_rating <dbl>,
#> #   relationship_satisfaction <dbl>, standard_hours <dbl>,
#> #   stock_option_level <dbl>, total_working_years <dbl>,
#> #   training_times_last_year <dbl>, work_life_balance <dbl>,
#> #   years_at_company <dbl>, years_in_current_role <dbl>,
#> #   years_since_last_promotion <dbl>, years_with_curr_manager <dbl>
```

---

But sometimes, simple random sampling is not enough:


```r
# target distribution in full dataset
prop.table(table(attrition$attrition))
```

```
#> 
#>        no       yes 
#> 0.8387755 0.1612245
```

```r
# target distribution in train dataset
prop.table(table(training(splitted)$attrition))
```

```
#> 
#>        no       yes 
#> 0.8462192 0.1537808
```

```r
# target distribution in test dataset
prop.table(table(testing(splitted)$attrition))
```

```
#> 
#>        no       yes 
#> 0.8088737 0.1911263
```

This is where we need `strata` argument to use stratified random sampling:


```r
# set seed
set.seed(100)

# create stratified initial split
splitted <- initial_split(attrition, prop = 0.8, strata = "attrition")

# target distribution in full dataset
prop.table(table(attrition$attrition))
```

```
#> 
#>        no       yes 
#> 0.8387755 0.1612245
```

```r
# target distribution in train dataset
prop.table(table(training(splitted)$attrition))
```

```
#> 
#>        no       yes 
#> 0.8385726 0.1614274
```

```r
# target distribution in test dataset
prop.table(table(testing(splitted)$attrition))
```

```
#> 
#>        no       yes 
#> 0.8395904 0.1604096
```


### Cross-sectional resampling


To use k-fold validation splits--and optionally, with repetition--we could use `vfold_cv()`:


```r
# set seed
set.seed(100)

# create k-fold splits with repetition
resampled <- vfold_cv(attrition, v = 3, repeats = 2, strata = "attrition")

# quick check
resampled
```

```
#> #  3-fold cross-validation repeated 2 times using stratification 
#> # A tibble: 6 x 3
#>   splits            id      id2  
#>   <named list>      <chr>   <chr>
#> 1 <split [980/490]> Repeat1 Fold1
#> 2 <split [980/490]> Repeat1 Fold2
#> 3 <split [980/490]> Repeat1 Fold3
#> 4 <split [980/490]> Repeat2 Fold1
#> 5 <split [980/490]> Repeat2 Fold2
#> 6 <split [980/490]> Repeat2 Fold3
```

Each train and test dataset are stored in `splits` column. We could use `analysis()` and `assessment()` to get the train and test dataset, consecutively:


```r
# check train dataset on an example split
analysis(resampled$splits[[1]])
```

```
#> # A tibble: 980 x 35
#>    attrition   age business_travel daily_rate department distance_from_h~
#>    <chr>     <dbl> <chr>                <dbl> <chr>                 <dbl>
#>  1 no           49 travel_frequen~        279 research_~                8
#>  2 yes          37 travel_rarely         1373 research_~                2
#>  3 no           33 travel_frequen~       1392 research_~                3
#>  4 no           27 travel_rarely          591 research_~                2
#>  5 no           59 travel_rarely         1324 research_~                3
#>  6 no           36 travel_rarely         1299 research_~               27
#>  7 no           29 travel_rarely          153 research_~               15
#>  8 no           31 travel_rarely          670 research_~               26
#>  9 no           34 travel_rarely         1346 research_~               19
#> 10 no           22 non_travel            1123 research_~               16
#> # ... with 970 more rows, and 29 more variables: education <dbl>,
#> #   education_field <chr>, employee_count <dbl>, employee_number <dbl>,
#> #   environment_satisfaction <dbl>, gender <chr>, hourly_rate <dbl>,
#> #   job_involvement <dbl>, job_level <dbl>, job_role <chr>,
#> #   job_satisfaction <dbl>, marital_status <chr>, monthly_income <dbl>,
#> #   monthly_rate <dbl>, num_companies_worked <dbl>, over_18 <chr>,
#> #   over_time <chr>, percent_salary_hike <dbl>, performance_rating <dbl>,
#> #   relationship_satisfaction <dbl>, standard_hours <dbl>,
#> #   stock_option_level <dbl>, total_working_years <dbl>,
#> #   training_times_last_year <dbl>, work_life_balance <dbl>,
#> #   years_at_company <dbl>, years_in_current_role <dbl>,
#> #   years_since_last_promotion <dbl>, years_with_curr_manager <dbl>
```

```r
# check test dataset on an example split
assessment(resampled$splits[[1]])
```

```
#> # A tibble: 490 x 35
#>    attrition   age business_travel daily_rate department distance_from_h~
#>    <chr>     <dbl> <chr>                <dbl> <chr>                 <dbl>
#>  1 yes          41 travel_rarely         1102 sales                     1
#>  2 no           32 travel_frequen~       1005 research_~                2
#>  3 no           30 travel_rarely         1358 research_~               24
#>  4 no           38 travel_frequen~        216 research_~               23
#>  5 no           35 travel_rarely          809 research_~               16
#>  6 yes          28 travel_rarely          103 research_~               24
#>  7 no           29 travel_rarely         1389 research_~               21
#>  8 no           32 travel_rarely          334 research_~                5
#>  9 no           38 travel_rarely          371 research_~                2
#> 10 yes          36 travel_rarely         1218 sales                     9
#> # ... with 480 more rows, and 29 more variables: education <dbl>,
#> #   education_field <chr>, employee_count <dbl>, employee_number <dbl>,
#> #   environment_satisfaction <dbl>, gender <chr>, hourly_rate <dbl>,
#> #   job_involvement <dbl>, job_level <dbl>, job_role <chr>,
#> #   job_satisfaction <dbl>, marital_status <chr>, monthly_income <dbl>,
#> #   monthly_rate <dbl>, num_companies_worked <dbl>, over_18 <chr>,
#> #   over_time <chr>, percent_salary_hike <dbl>, performance_rating <dbl>,
#> #   relationship_satisfaction <dbl>, standard_hours <dbl>,
#> #   stock_option_level <dbl>, total_working_years <dbl>,
#> #   training_times_last_year <dbl>, work_life_balance <dbl>,
#> #   years_at_company <dbl>, years_in_current_role <dbl>,
#> #   years_since_last_promotion <dbl>, years_with_curr_manager <dbl>
```

# Data Preprocess using `recipes`

## Rethinking: How do we should treat train and test?

## Reproducible preprocess `recipes`


[`recipes`](https://tidymodels.github.io/recipes/) is part of `tidymodels` that could help us in making a reproducible data preprocess.

There are so many data preprocess approach provided by `recipes`--as you could see in its [full function references page](https://tidymodels.github.io/recipes/reference/index.html) . In this introduction, we will use several preprocess steps related to our example dataset.

There are several steps that we could apply to our dataset--some are very fundamental and sometimes mandatory, but some are also tuneable:

* [`step_rm()`](https://tidymodels.github.io/recipes/reference/step_rm.html) :
    Manually remove unused columns.
* [`step_nzv()`](https://tidymodels.github.io/recipes/reference/step_nzv.html) :
    Automatically filter near-zero varianced columns.
* [`step_string2factor()`](https://tidymodels.github.io/recipes/reference/step_string2factor.html) :
    Manually convert to `factor` columns.
* [`step_downsample()`](https://tidymodels.github.io/recipes/reference/step_downsample.html) :
    Downsampling step to balancing target's class distribution (**tuneable**).
* [`step_center()`](https://tidymodels.github.io/recipes/reference/step_center.html) :
    Normalize the mean of `numeric` column(s) to zero (**tuneable**).
* [`step_scale()`](https://tidymodels.github.io/recipes/reference/step_scale.html) :
    Normalize the standard deviation of `numeric` column(s) to one (**tuneable**).
* [`step_pca()`](https://tidymodels.github.io/recipes/reference/step_pca.html) :
    Shrink `numeric` column(s) to several PCA components (**tuneable**).


### Designing your first preprocess recipes

1. Initiate a recipe using `recipe()`
    + Define your formula in the first argument.
    + Supply a template dataset in `data` argument.
2. Pipe to every needed `step_*()`--always remember to put every step in proper consecutive manner.
3. After finished with every needed `step_*()`, pipe to `prep()` function to train your recipe
    + It will automatically convert all `character` columns to `factor`;
    + If you used `step_string2factor()`, don't forget to specify `strings_as_factors = FALSE`
    + It will train the recipe to the specified dataset in the `recipe()`'s `data` argument;
    + If you want to train to other dataset, you can supply the new dataset to `training` argument, and set the `fresh` argument to `TRUE`

Let's see an example of defining a recipe:


```r
# define preprocess recipe from train dataset
rec <- recipe(attrition ~ ., data = training(splitted)) %>% 
  step_rm(employee_count, employee_number) %>%
  step_nzv(all_predictors()) %>% 
  step_string2factor(all_nominal(), -attrition) %>%
  step_string2factor(attrition, levels = c("yes", "no")) %>%
  step_downsample(attrition, ratio = 1/1, seed = 100) %>%
  step_center(all_numeric()) %>%
  step_scale(all_numeric()) %>%
  step_pca(all_numeric(), threshold = 0.85) %>%
  prep(strings_as_factors = FALSE)

# quick check
rec
```

```
#> Data Recipe
#> 
#> Inputs:
#> 
#>       role #variables
#>    outcome          1
#>  predictor         34
#> 
#> Training data contained 1177 data points and no missing data.
#> 
#> Operations:
#> 
#> Variables removed employee_count, employee_number [trained]
#> Sparse, unbalanced variable filter removed over_18, standard_hours [trained]
#> Factor variables from business_travel, ... [trained]
#> Factor variables from attrition [trained]
#> Down-sampling based on attrition [trained]
#> Centering for age, daily_rate, ... [trained]
#> Scaling for age, daily_rate, ... [trained]
#> PCA extraction with age, daily_rate, ... [trained]
```


There are two ways of obtaining the result from our recipe:

* [`juice()`](https://tidymodels.github.io/recipes/reference/juice.html) :
    Extract preprocessed dataset from `prep()`-ed recipe. Normally, we will use this function to get preprocessed train dataset.

* [`bake()`](https://tidymodels.github.io/recipes/reference/bake.html) :
    Apply a recipe to new dataset. Normally, we use we will use this function to preprocess new dataset, such as test dataset, or prediction dataset.

These are some example on how to get preprocessed train and test dataset:


```r
# get preprocessed train dataset
data_train <- juice(rec)

# quick check
data_train
```

```
#> # A tibble: 380 x 22
#>    business_travel department education_field gender job_role
#>    <fct>           <fct>      <fct>           <fct>  <fct>   
#>  1 travel_rarely   sales      life_sciences   female sales_e~
#>  2 travel_rarely   research_~ other           male   laborat~
#>  3 travel_rarely   research_~ life_sciences   male   laborat~
#>  4 travel_rarely   research_~ medical         male   researc~
#>  5 travel_frequen~ research_~ life_sciences   female researc~
#>  6 travel_rarely   sales      technical_degr~ male   sales_r~
#>  7 travel_rarely   research_~ medical         male   researc~
#>  8 travel_rarely   sales      marketing       male   sales_r~
#>  9 travel_rarely   research_~ technical_degr~ female researc~
#> 10 travel_rarely   research_~ life_sciences   male   laborat~
#> # ... with 370 more rows, and 17 more variables: marital_status <fct>,
#> #   over_time <fct>, attrition <fct>, PC01 <dbl>, PC02 <dbl>, PC03 <dbl>,
#> #   PC04 <dbl>, PC05 <dbl>, PC06 <dbl>, PC07 <dbl>, PC08 <dbl>,
#> #   PC09 <dbl>, PC10 <dbl>, PC11 <dbl>, PC12 <dbl>, PC13 <dbl>, PC14 <dbl>
```

```r
# get preprocessed test dataset
data_test <- bake(rec, testing(splitted))

# quick check
data_test
```

```
#> # A tibble: 293 x 22
#>    attrition business_travel department education_field gender job_role
#>    <fct>     <fct>           <fct>      <fct>           <fct>  <fct>   
#>  1 no        travel_rarely   research_~ life_sciences   female manufac~
#>  2 no        non_travel      research_~ other           female manufac~
#>  3 yes       travel_rarely   sales      life_sciences   male   sales_r~
#>  4 no        travel_rarely   research_~ other           female manager 
#>  5 no        travel_rarely   sales      marketing       male   sales_e~
#>  6 no        travel_rarely   sales      marketing       female sales_r~
#>  7 no        travel_rarely   research_~ other           male   laborat~
#>  8 yes       travel_rarely   research_~ life_sciences   male   laborat~
#>  9 no        travel_frequen~ research_~ medical         female laborat~
#> 10 no        travel_rarely   research_~ life_sciences   male   researc~
#> # ... with 283 more rows, and 16 more variables: marital_status <fct>,
#> #   over_time <fct>, PC01 <dbl>, PC02 <dbl>, PC03 <dbl>, PC04 <dbl>,
#> #   PC05 <dbl>, PC06 <dbl>, PC07 <dbl>, PC08 <dbl>, PC09 <dbl>,
#> #   PC10 <dbl>, PC11 <dbl>, PC12 <dbl>, PC13 <dbl>, PC14 <dbl>
```

# Model Fitting using `parsnip`

## Rethinking: How many machine learning packages you used?

## One vegeta.. I mean package to rule them all: `parsnip`


[`parsnip`](https://tidymodels.github.io/parsnip/)  is part of `tidymodels` that could help us in model fitting and prediction flows.

There are so many models supported by `parsnip`--as you could see in its [full model list](https://tidymodels.github.io/parsnip/articles/articles/Models.html) . In this introduction, we will use random forest as an example model.

There are two part of defining a model that should be noted:

* **Defining model's specification**:
    In this part, we need to define the model's specification, such as `mtry` and `trees` for random forest, through model specific functions. For example, you can use [`rand_forest()`](https://tidymodels.github.io/parsnip/reference/rand_forest.html)  to define a random forest specification. Make sure to check [full model list](https://tidymodels.github.io/parsnip/articles/articles/Models.html)  to see every model and its available arguments.

* **Defining model's engine**:
    In this part, we need to define the model's engine, which determines the package we will use to fit our model. This part could be done using [`set_engine()`](https://tidymodels.github.io/parsnip/reference/set_engine.html)  function. Note that in addition to defining which package we want to use as our engine, we could also passing package specific arguments to this function.

This is an example of defining a random forest model using `randomForest::randomForest()` as our engine:


```r
# set-up model specification
model_spec <- rand_forest(
  mode = "classification",
  mtry = ncol(data_train) - 2,
  trees = 500,
  min_n = 15
)

# set-up model engine
model_engine <- set_engine(
  object = model_spec,
  engine = "randomForest"
)

# quick check
model_engine
```

```
#> Random Forest Model Specification (classification)
#> 
#> Main Arguments:
#>   mtry = ncol(data_train) - 2
#>   trees = 500
#>   min_n = 15
#> 
#> Computational engine: randomForest
```


To fit our model, we have two options:

* Formula interface
* X-Y interface

Note that some packages are behaving differently inside formula and x-y interface. For example, `randomForest::randomForest()` would convert all of our categorical variables into dummy variables in formula interface, but not in x-y interface.


Fit using formula interface using `fit()` function:


```r
# fit the model
model <- fit(
  object = model_engine,
  formula = attrition ~ .,
  data = data_train
)

# quick check
model
```

```
#> parsnip model object
#> 
#> 
#> Call:
#>  randomForest(x = as.data.frame(x), y = y, ntree = ~500, mtry = ~ncol(data_train) -      2, nodesize = ~15) 
#>                Type of random forest: classification
#>                      Number of trees: 500
#> No. of variables tried at each split: 20
#> 
#>         OOB estimate of  error rate: 28.95%
#> Confusion matrix:
#>     yes  no class.error
#> yes 141  49   0.2578947
#> no   61 129   0.3210526
```

Or through x-y interface using `fit_xy()` function:


```r
# fit the model
model <- fit_xy(
  object = model_engine,
  x = select(data_train, -attrition),
  y = select(data_train, attrition)
)

# quick check
model
```

```
#> parsnip model object
#> 
#> 
#> Call:
#>  randomForest(x = as.data.frame(x), y = y, ntree = ~500, mtry = ~ncol(data_train) -      2, nodesize = ~15) 
#>                Type of random forest: classification
#>                      Number of trees: 500
#> No. of variables tried at each split: 20
#> 
#>         OOB estimate of  error rate: 27.37%
#> Confusion matrix:
#>     yes  no class.error
#> yes 138  52   0.2736842
#> no   52 138   0.2736842
```

In this workflow, it should be relatively easy to change the model engine.

Let's try to fit a same model specification, but now using `ranger::ranger()`:


```r
# set-up other model engine
model_engine <- set_engine(
  object = model_spec,
  engine = "ranger",
  seed = 100,
  num.threads = parallel::detectCores() / 2,
  importance = "impurity"
)

# quick check
model_engine
```

```
#> Random Forest Model Specification (classification)
#> 
#> Main Arguments:
#>   mtry = ncol(data_train) - 2
#>   trees = 500
#>   min_n = 15
#> 
#> Engine-Specific Arguments:
#>   seed = 100
#>   num.threads = parallel::detectCores()/2
#>   importance = impurity
#> 
#> Computational engine: ranger
```


Now let's try to fit the model, and see the result:


```r
# fit the model
model <- fit(
  object = model_engine,
  formula = attrition ~ .,
  data = data_train
)

# quick check
model
```

```
#> parsnip model object
#> 
#> Ranger result
#> 
#> Call:
#>  ranger::ranger(formula = formula, data = data, mtry = ~ncol(data_train) -      2, num.trees = ~500, min.node.size = ~15, seed = ~100, num.threads = ~parallel::detectCores()/2,      importance = ~"impurity", verbose = FALSE, probability = TRUE) 
#> 
#> Type:                             Probability estimation 
#> Number of trees:                  500 
#> Sample size:                      380 
#> Number of independent variables:  21 
#> Mtry:                             20 
#> Target node size:                 15 
#> Variable importance mode:         impurity 
#> Splitrule:                        gini 
#> OOB prediction error (Brier s.):  0.1945066
```


Notice that `ranger::ranger()` doesn't behave differently between `fit()` and `fit_xy()`:


```r
# fit the model
model <- fit_xy(
  object = model_engine,
  x = select(data_train, -attrition),
  y = select(data_train, attrition)
)

# quick check
model
```

```
#> parsnip model object
#> 
#> Ranger result
#> 
#> Call:
#>  ranger::ranger(formula = formula, data = data, mtry = ~ncol(data_train) -      2, num.trees = ~500, min.node.size = ~15, seed = ~100, num.threads = ~parallel::detectCores()/2,      importance = ~"impurity", verbose = FALSE, probability = TRUE) 
#> 
#> Type:                             Probability estimation 
#> Number of trees:                  500 
#> Sample size:                      380 
#> Number of independent variables:  21 
#> Mtry:                             20 
#> Target node size:                 15 
#> Variable importance mode:         impurity 
#> Splitrule:                        gini 
#> OOB prediction error (Brier s.):  0.1945066
```


To get the prediction, we could use `predict()` as usual--but note that it would return a tidied `tibble` instead of a `vector`, as in `type = "class"` cases, or a raw `data.frame`, as in `type = "prob"` cases.

In this way, the prediction results would be very convenient for further usage, such as simple recombining with the original dataset:


```r
# get prediction on test
predicted <- data_test %>% 
  bind_cols(predict(model, data_test)) %>% 
  bind_cols(predict(model, data_test, type = "prob"))

# quick check
predicted %>% 
  select(attrition, matches(".pred"))
```

```
#> # A tibble: 293 x 4
#>    attrition .pred_class .pred_yes .pred_no
#>    <fct>     <fct>           <dbl>    <dbl>
#>  1 no        no             0.340     0.660
#>  2 no        no             0.358     0.642
#>  3 yes       yes            0.509     0.491
#>  4 no        no             0.0884    0.912
#>  5 no        no             0.185     0.815
#>  6 no        no             0.448     0.552
#>  7 no        no             0.127     0.873
#>  8 yes       yes            0.748     0.252
#>  9 no        no             0.147     0.853
#> 10 no        yes            0.680     0.320
#> # ... with 283 more rows
```

# Model Evaluation using `yardstick`

## Rethinking: How do we measure the goodness of our model?

## It's always good to bring your own `yardstick`

[`yardstick`](https://tidymodels.github.io/yardstick/)  is part of `tidymodels` that could help us in calculating model performance metrics.

There are so many metrics available by `yardstick`--as you could see in its [full function references page](https://tidymodels.github.io/yardstick/reference/index.html) . In this introduction, we will calculate some model performance metrics for classification task as an example.


There are two ways of calculating model performance metrics, which differ in its input and output:

* `tibble` approach:
    We pass a dataset containing the `truth` and `estimate` to the function, and it will return a `tibble` containing the results, e.g., [`precision()`](https://tidymodels.github.io/yardstick/reference/precision.html)  function.
* `vector` approach:
    We pass a vector as the `truth` and a vector as the `estimate` to the function, and it will return a `vector` which show the results, e.g., [`precision_vec()`](https://tidymodels.github.io/yardstick/reference/precision.html)  function.

Note that some function, like [`conf_mat()`](https://tidymodels.github.io/yardstick/reference/conf_mat.html) , only accept `tibble` approach, since it is not returned a `vector` of length one.


Let's start by reporting the confusion matrix:


```r
# show confusion matrix
predicted %>% 
  conf_mat(truth = attrition, estimate = .pred_class) %>% 
  autoplot(type = "heatmap")
```

<img src="/blog/2019-10-06-tidymodels_files/figure-html/unnamed-chunk-19-1.png" width="672" style="display: block; margin: auto;" />

Now, to calculate the performance metrics, let's try to use the `tibble` approach--which also support `group_by`:


```r
# calculate accuracy
predicted %>% 
  accuracy(attrition, .pred_class)
```

```
#> # A tibble: 1 x 3
#>   .metric  .estimator .estimate
#>   <chr>    <chr>          <dbl>
#> 1 accuracy binary         0.710
```

```r
# calculate accuracy by group
predicted %>% 
  group_by(department) %>% 
  accuracy(attrition, .pred_class) %>% 
  ungroup()
```

```
#> # A tibble: 3 x 4
#>   department           .metric  .estimator .estimate
#>   <fct>                <chr>    <chr>          <dbl>
#> 1 human_resources      accuracy binary         0.636
#> 2 research_development accuracy binary         0.760
#> 3 sales                accuracy binary         0.605
```

Or using `vector` approach, which is more flexible in general:


```r
# metrics summary
predicted %>% 
  summarise(
    accuracy = accuracy_vec(attrition, .pred_class),
    sensitivity = sens_vec(attrition, .pred_class),
    specificity = spec_vec(attrition, .pred_class),
    precision = precision_vec(attrition, .pred_class)
  )
```

```
#> # A tibble: 1 x 4
#>   accuracy sensitivity specificity precision
#>      <dbl>       <dbl>       <dbl>     <dbl>
#> 1    0.710       0.723       0.707     0.321
```

```r
# metrics summary by group
predicted %>% 
  group_by(department) %>% 
  summarise(
    accuracy = accuracy_vec(attrition, .pred_class),
    sensitivity = sens_vec(attrition, .pred_class),
    specificity = spec_vec(attrition, .pred_class),
    precision = precision_vec(attrition, .pred_class)
  ) %>% 
  ungroup()
```

```
#> # A tibble: 3 x 5
#>   department           accuracy sensitivity specificity precision
#>   <fct>                   <dbl>       <dbl>       <dbl>     <dbl>
#> 1 human_resources         0.636       1           0.556     0.333
#> 2 research_development    0.760       0.677       0.776     0.362
#> 3 sales                   0.605       0.786       0.569     0.262
```


Sometimes the model performance metrics could also improving the models final results. For example, through Receiver Operating Curve, we could assess the probability threshold effect to sensitivity and specificity:


```r
predicted %>% 
  roc_curve(attrition, .pred_yes) %>% 
  autoplot()
```

<img src="/blog/2019-10-06-tidymodels_files/figure-html/unnamed-chunk-22-1.png" width="672" style="display: block; margin: auto;" />


And, since it's returning a `tibble`, we could do further data wrangling to help us see it more clearly:


```r
# get roc curve data on test dataset
pred_test_roc <- predicted %>%
  roc_curve(attrition, .pred_yes)

# quick check
pred_test_roc
```

```
#> # A tibble: 295 x 3
#>    .threshold specificity sensitivity
#>         <dbl>       <dbl>       <dbl>
#>  1  -Inf          0                 1
#>  2     0.0422     0                 1
#>  3     0.0487     0.00407           1
#>  4     0.0568     0.00813           1
#>  5     0.0674     0.0122            1
#>  6     0.0685     0.0163            1
#>  7     0.0705     0.0203            1
#>  8     0.0884     0.0244            1
#>  9     0.0922     0.0285            1
#> 10     0.0945     0.0325            1
#> # ... with 285 more rows
```


With some `ggplot2`:


```r
# tidying
pred_test_roc <- pred_test_roc %>% 
  mutate_if(~ is.numeric(.), ~ round(., 4)) %>% 
  gather(metric, value, -.threshold)

# plot sensitivity-specificity trade-off
p <- ggplot(pred_test_roc, aes(x = .threshold, y = value)) +
  geom_line(aes(colour = metric)) +
  labs(x = "Probability Threshold to be Classified as Positive", y = "Value", colour = "Metrics") +
  theme_minimal()
```


and `plotly` magic, it would be perfect:


```r
# convert to plotly
ggplotly(p)
```

<!--html_preserve--><div id="htmlwidget-2b1913c8222382f6d64c" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-2b1913c8222382f6d64c">{"x":{"data":[{"x":[null,0.0422,0.0487,0.0568,0.0674,0.0685,0.0705,0.0884,0.0922,0.0945,0.0975,0.1093,0.1123,0.1171,0.1239,0.1243,0.1248,0.1256,0.1271,0.1401,0.1419,0.1473,0.1514,0.1538,0.1551,0.163,0.1631,0.1635,0.1657,0.1678,0.1712,0.1761,0.1796,0.1807,0.1846,0.1849,0.1853,0.1853,0.1868,0.187,0.1983,0.1988,0.1994,0.1997,0.2012,0.2022,0.2161,0.2171,0.2192,0.222,0.2227,0.229,0.2315,0.2349,0.2354,0.2354,0.237,0.2374,0.2401,0.2402,0.2429,0.2457,0.2475,0.253,0.2577,0.2601,0.2646,0.2671,0.2672,0.2678,0.2684,0.269,0.2692,0.2707,0.2776,0.2778,0.2792,0.2833,0.2863,0.2874,0.2892,0.2892,0.2897,0.2908,0.2927,0.295,0.3023,0.3072,0.3077,0.308,0.3131,0.314,0.3153,0.3159,0.3211,0.3224,0.3238,0.324,0.3241,0.3245,0.3268,0.3275,0.3297,0.3353,0.3355,0.3387,0.34,0.3415,0.3436,0.3447,0.345,0.3465,0.3476,0.3482,0.3524,0.3535,0.3564,0.3568,0.3578,0.3626,0.3646,0.3655,0.3671,0.3673,0.3674,0.3677,0.3682,0.3731,0.3735,0.3735,0.3745,0.376,0.3768,0.3831,0.3858,0.3871,0.3918,0.3928,0.3943,0.3955,0.4018,0.4028,0.4072,0.4078,0.409,0.4129,0.4138,0.4158,0.4167,0.4187,0.4203,0.421,0.4342,0.4363,0.4398,0.4409,0.442,0.4427,0.4449,0.4479,0.4483,0.4483,0.4515,0.4542,0.4576,0.458,0.4608,0.4612,0.4615,0.4618,0.4671,0.4687,0.4688,0.4691,0.4691,0.4751,0.4754,0.4756,0.4758,0.48,0.4824,0.4878,0.4894,0.4906,0.4913,0.4915,0.4934,0.4957,0.5017,0.5044,0.5046,0.5048,0.5063,0.5067,0.5091,0.5105,0.511,0.5116,0.5134,0.5143,0.5149,0.5175,0.5193,0.5211,0.522,0.5223,0.5235,0.5243,0.5258,0.5281,0.529,0.5433,0.5436,0.5467,0.5477,0.5486,0.5497,0.5537,0.5566,0.5573,0.5576,0.5596,0.5679,0.5703,0.5835,0.5845,0.5905,0.5911,0.5921,0.5925,0.5945,0.605,0.6102,0.6104,0.6108,0.6242,0.6344,0.6362,0.6369,0.6388,0.6391,0.6421,0.6439,0.6472,0.6512,0.6549,0.6557,0.6561,0.6561,0.6576,0.6579,0.6591,0.6654,0.6654,0.6655,0.6728,0.6805,0.6881,0.6903,0.7029,0.7076,0.7139,0.7159,0.7186,0.7216,0.7375,0.7387,0.7393,0.7395,0.7459,0.7483,0.7492,0.7571,0.7631,0.7641,0.7645,0.7679,0.7709,0.7737,0.774,0.7755,0.7856,0.7866,0.7913,0.8071,0.8225,0.8261,0.8343,0.843,0.8513,0.858,0.8588,0.8614,0.8912,null],"y":[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9787,0.9574,0.9574,0.9574,0.9574,0.9574,0.9574,0.9362,0.9362,0.9362,0.9362,0.9362,0.9362,0.9362,0.9362,0.9362,0.9362,0.9149,0.9149,0.9149,0.9149,0.9149,0.9149,0.9149,0.9149,0.9149,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8936,0.8723,0.8723,0.8723,0.8723,0.8723,0.8723,0.8723,0.8723,0.8723,0.8723,0.8723,0.8511,0.8511,0.8511,0.8511,0.8511,0.8511,0.8511,0.8511,0.8511,0.8511,0.8511,0.8511,0.8511,0.8298,0.8298,0.8298,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.8085,0.7872,0.7872,0.7872,0.7872,0.766,0.766,0.766,0.7447,0.7447,0.7447,0.7447,0.7447,0.7447,0.7447,0.7447,0.7447,0.7447,0.7447,0.7447,0.7447,0.7234,0.7234,0.7234,0.7234,0.7234,0.7234,0.7234,0.7234,0.7021,0.7021,0.7021,0.7021,0.7021,0.7021,0.7021,0.7021,0.6809,0.6809,0.6809,0.6809,0.6809,0.6809,0.6809,0.6809,0.6809,0.6596,0.6596,0.6596,0.6596,0.6596,0.6596,0.6383,0.6383,0.6383,0.6383,0.617,0.617,0.617,0.617,0.617,0.617,0.5957,0.5957,0.5957,0.5957,0.5957,0.5957,0.5957,0.5745,0.5745,0.5532,0.5532,0.5319,0.5106,0.4894,0.4894,0.4681,0.4681,0.4681,0.4681,0.4681,0.4468,0.4468,0.4468,0.4468,0.4255,0.4255,0.4255,0.4043,0.4043,0.4043,0.4043,0.4043,0.383,0.383,0.383,0.383,0.383,0.383,0.383,0.383,0.3617,0.3404,0.3404,0.3191,0.2979,0.2766,0.2553,0.2553,0.234,0.234,0.234,0.2128,0.2128,0.1915,0.1702,0.1702,0.1489,0.1489,0.1277,0.1064,0.1064,0.1064,0.0851,0.0638,0.0426,0.0213,0],"text":["metric: sensitivity<br />.threshold:   -Inf<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.0422<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.0487<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.0568<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.0674<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.0685<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.0705<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.0884<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.0922<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.0945<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.0975<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1093<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1123<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1171<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1239<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1243<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1248<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1256<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1271<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1401<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1419<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1473<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1514<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1538<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1551<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1630<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1631<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1635<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1657<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1678<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1712<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1761<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1796<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1807<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1846<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1849<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1853<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1853<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1868<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1870<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1983<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1988<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1994<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.1997<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.2012<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.2022<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.2161<br />value: 1.0000","metric: sensitivity<br />.threshold: 0.2171<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2192<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2220<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2227<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2290<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2315<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2349<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2354<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2354<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2370<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2374<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2401<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2402<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2429<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2457<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2475<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2530<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2577<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2601<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2646<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2671<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2672<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2678<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2684<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2690<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2692<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2707<br />value: 0.9787","metric: sensitivity<br />.threshold: 0.2776<br />value: 0.9574","metric: sensitivity<br />.threshold: 0.2778<br />value: 0.9574","metric: sensitivity<br />.threshold: 0.2792<br />value: 0.9574","metric: sensitivity<br />.threshold: 0.2833<br />value: 0.9574","metric: sensitivity<br />.threshold: 0.2863<br />value: 0.9574","metric: sensitivity<br />.threshold: 0.2874<br />value: 0.9574","metric: sensitivity<br />.threshold: 0.2892<br />value: 0.9362","metric: sensitivity<br />.threshold: 0.2892<br />value: 0.9362","metric: sensitivity<br />.threshold: 0.2897<br />value: 0.9362","metric: sensitivity<br />.threshold: 0.2908<br />value: 0.9362","metric: sensitivity<br />.threshold: 0.2927<br />value: 0.9362","metric: sensitivity<br />.threshold: 0.2950<br />value: 0.9362","metric: sensitivity<br />.threshold: 0.3023<br />value: 0.9362","metric: sensitivity<br />.threshold: 0.3072<br />value: 0.9362","metric: sensitivity<br />.threshold: 0.3077<br />value: 0.9362","metric: sensitivity<br />.threshold: 0.3080<br />value: 0.9362","metric: sensitivity<br />.threshold: 0.3131<br />value: 0.9149","metric: sensitivity<br />.threshold: 0.3140<br />value: 0.9149","metric: sensitivity<br />.threshold: 0.3153<br />value: 0.9149","metric: sensitivity<br />.threshold: 0.3159<br />value: 0.9149","metric: sensitivity<br />.threshold: 0.3211<br />value: 0.9149","metric: sensitivity<br />.threshold: 0.3224<br />value: 0.9149","metric: sensitivity<br />.threshold: 0.3238<br />value: 0.9149","metric: sensitivity<br />.threshold: 0.3240<br />value: 0.9149","metric: sensitivity<br />.threshold: 0.3241<br />value: 0.9149","metric: sensitivity<br />.threshold: 0.3245<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3268<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3275<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3297<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3353<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3355<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3387<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3400<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3415<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3436<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3447<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3450<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3465<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3476<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3482<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3524<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3535<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3564<br />value: 0.8936","metric: sensitivity<br />.threshold: 0.3568<br />value: 0.8723","metric: sensitivity<br />.threshold: 0.3578<br />value: 0.8723","metric: sensitivity<br />.threshold: 0.3626<br />value: 0.8723","metric: sensitivity<br />.threshold: 0.3646<br />value: 0.8723","metric: sensitivity<br />.threshold: 0.3655<br />value: 0.8723","metric: sensitivity<br />.threshold: 0.3671<br />value: 0.8723","metric: sensitivity<br />.threshold: 0.3673<br />value: 0.8723","metric: sensitivity<br />.threshold: 0.3674<br />value: 0.8723","metric: sensitivity<br />.threshold: 0.3677<br />value: 0.8723","metric: sensitivity<br />.threshold: 0.3682<br />value: 0.8723","metric: sensitivity<br />.threshold: 0.3731<br />value: 0.8723","metric: sensitivity<br />.threshold: 0.3735<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.3735<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.3745<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.3760<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.3768<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.3831<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.3858<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.3871<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.3918<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.3928<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.3943<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.3955<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.4018<br />value: 0.8511","metric: sensitivity<br />.threshold: 0.4028<br />value: 0.8298","metric: sensitivity<br />.threshold: 0.4072<br />value: 0.8298","metric: sensitivity<br />.threshold: 0.4078<br />value: 0.8298","metric: sensitivity<br />.threshold: 0.4090<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4129<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4138<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4158<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4167<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4187<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4203<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4210<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4342<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4363<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4398<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4409<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4420<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4427<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4449<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4479<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4483<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4483<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4515<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4542<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4576<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4580<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4608<br />value: 0.8085","metric: sensitivity<br />.threshold: 0.4612<br />value: 0.7872","metric: sensitivity<br />.threshold: 0.4615<br />value: 0.7872","metric: sensitivity<br />.threshold: 0.4618<br />value: 0.7872","metric: sensitivity<br />.threshold: 0.4671<br />value: 0.7872","metric: sensitivity<br />.threshold: 0.4687<br />value: 0.7660","metric: sensitivity<br />.threshold: 0.4688<br />value: 0.7660","metric: sensitivity<br />.threshold: 0.4691<br />value: 0.7660","metric: sensitivity<br />.threshold: 0.4691<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4751<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4754<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4756<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4758<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4800<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4824<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4878<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4894<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4906<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4913<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4915<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4934<br />value: 0.7447","metric: sensitivity<br />.threshold: 0.4957<br />value: 0.7234","metric: sensitivity<br />.threshold: 0.5017<br />value: 0.7234","metric: sensitivity<br />.threshold: 0.5044<br />value: 0.7234","metric: sensitivity<br />.threshold: 0.5046<br />value: 0.7234","metric: sensitivity<br />.threshold: 0.5048<br />value: 0.7234","metric: sensitivity<br />.threshold: 0.5063<br />value: 0.7234","metric: sensitivity<br />.threshold: 0.5067<br />value: 0.7234","metric: sensitivity<br />.threshold: 0.5091<br />value: 0.7234","metric: sensitivity<br />.threshold: 0.5105<br />value: 0.7021","metric: sensitivity<br />.threshold: 0.5110<br />value: 0.7021","metric: sensitivity<br />.threshold: 0.5116<br />value: 0.7021","metric: sensitivity<br />.threshold: 0.5134<br />value: 0.7021","metric: sensitivity<br />.threshold: 0.5143<br />value: 0.7021","metric: sensitivity<br />.threshold: 0.5149<br />value: 0.7021","metric: sensitivity<br />.threshold: 0.5175<br />value: 0.7021","metric: sensitivity<br />.threshold: 0.5193<br />value: 0.7021","metric: sensitivity<br />.threshold: 0.5211<br />value: 0.6809","metric: sensitivity<br />.threshold: 0.5220<br />value: 0.6809","metric: sensitivity<br />.threshold: 0.5223<br />value: 0.6809","metric: sensitivity<br />.threshold: 0.5235<br />value: 0.6809","metric: sensitivity<br />.threshold: 0.5243<br />value: 0.6809","metric: sensitivity<br />.threshold: 0.5258<br />value: 0.6809","metric: sensitivity<br />.threshold: 0.5281<br />value: 0.6809","metric: sensitivity<br />.threshold: 0.5290<br />value: 0.6809","metric: sensitivity<br />.threshold: 0.5433<br />value: 0.6809","metric: sensitivity<br />.threshold: 0.5436<br />value: 0.6596","metric: sensitivity<br />.threshold: 0.5467<br />value: 0.6596","metric: sensitivity<br />.threshold: 0.5477<br />value: 0.6596","metric: sensitivity<br />.threshold: 0.5486<br />value: 0.6596","metric: sensitivity<br />.threshold: 0.5497<br />value: 0.6596","metric: sensitivity<br />.threshold: 0.5537<br />value: 0.6596","metric: sensitivity<br />.threshold: 0.5566<br />value: 0.6383","metric: sensitivity<br />.threshold: 0.5573<br />value: 0.6383","metric: sensitivity<br />.threshold: 0.5576<br />value: 0.6383","metric: sensitivity<br />.threshold: 0.5596<br />value: 0.6383","metric: sensitivity<br />.threshold: 0.5679<br />value: 0.6170","metric: sensitivity<br />.threshold: 0.5703<br />value: 0.6170","metric: sensitivity<br />.threshold: 0.5835<br />value: 0.6170","metric: sensitivity<br />.threshold: 0.5845<br />value: 0.6170","metric: sensitivity<br />.threshold: 0.5905<br />value: 0.6170","metric: sensitivity<br />.threshold: 0.5911<br />value: 0.6170","metric: sensitivity<br />.threshold: 0.5921<br />value: 0.5957","metric: sensitivity<br />.threshold: 0.5925<br />value: 0.5957","metric: sensitivity<br />.threshold: 0.5945<br />value: 0.5957","metric: sensitivity<br />.threshold: 0.6050<br />value: 0.5957","metric: sensitivity<br />.threshold: 0.6102<br />value: 0.5957","metric: sensitivity<br />.threshold: 0.6104<br />value: 0.5957","metric: sensitivity<br />.threshold: 0.6108<br />value: 0.5957","metric: sensitivity<br />.threshold: 0.6242<br />value: 0.5745","metric: sensitivity<br />.threshold: 0.6344<br />value: 0.5745","metric: sensitivity<br />.threshold: 0.6362<br />value: 0.5532","metric: sensitivity<br />.threshold: 0.6369<br />value: 0.5532","metric: sensitivity<br />.threshold: 0.6388<br />value: 0.5319","metric: sensitivity<br />.threshold: 0.6391<br />value: 0.5106","metric: sensitivity<br />.threshold: 0.6421<br />value: 0.4894","metric: sensitivity<br />.threshold: 0.6439<br />value: 0.4894","metric: sensitivity<br />.threshold: 0.6472<br />value: 0.4681","metric: sensitivity<br />.threshold: 0.6512<br />value: 0.4681","metric: sensitivity<br />.threshold: 0.6549<br />value: 0.4681","metric: sensitivity<br />.threshold: 0.6557<br />value: 0.4681","metric: sensitivity<br />.threshold: 0.6561<br />value: 0.4681","metric: sensitivity<br />.threshold: 0.6561<br />value: 0.4468","metric: sensitivity<br />.threshold: 0.6576<br />value: 0.4468","metric: sensitivity<br />.threshold: 0.6579<br />value: 0.4468","metric: sensitivity<br />.threshold: 0.6591<br />value: 0.4468","metric: sensitivity<br />.threshold: 0.6654<br />value: 0.4255","metric: sensitivity<br />.threshold: 0.6654<br />value: 0.4255","metric: sensitivity<br />.threshold: 0.6655<br />value: 0.4255","metric: sensitivity<br />.threshold: 0.6728<br />value: 0.4043","metric: sensitivity<br />.threshold: 0.6805<br />value: 0.4043","metric: sensitivity<br />.threshold: 0.6881<br />value: 0.4043","metric: sensitivity<br />.threshold: 0.6903<br />value: 0.4043","metric: sensitivity<br />.threshold: 0.7029<br />value: 0.4043","metric: sensitivity<br />.threshold: 0.7076<br />value: 0.3830","metric: sensitivity<br />.threshold: 0.7139<br />value: 0.3830","metric: sensitivity<br />.threshold: 0.7159<br />value: 0.3830","metric: sensitivity<br />.threshold: 0.7186<br />value: 0.3830","metric: sensitivity<br />.threshold: 0.7216<br />value: 0.3830","metric: sensitivity<br />.threshold: 0.7375<br />value: 0.3830","metric: sensitivity<br />.threshold: 0.7387<br />value: 0.3830","metric: sensitivity<br />.threshold: 0.7393<br />value: 0.3830","metric: sensitivity<br />.threshold: 0.7395<br />value: 0.3617","metric: sensitivity<br />.threshold: 0.7459<br />value: 0.3404","metric: sensitivity<br />.threshold: 0.7483<br />value: 0.3404","metric: sensitivity<br />.threshold: 0.7492<br />value: 0.3191","metric: sensitivity<br />.threshold: 0.7571<br />value: 0.2979","metric: sensitivity<br />.threshold: 0.7631<br />value: 0.2766","metric: sensitivity<br />.threshold: 0.7641<br />value: 0.2553","metric: sensitivity<br />.threshold: 0.7645<br />value: 0.2553","metric: sensitivity<br />.threshold: 0.7679<br />value: 0.2340","metric: sensitivity<br />.threshold: 0.7709<br />value: 0.2340","metric: sensitivity<br />.threshold: 0.7737<br />value: 0.2340","metric: sensitivity<br />.threshold: 0.7740<br />value: 0.2128","metric: sensitivity<br />.threshold: 0.7755<br />value: 0.2128","metric: sensitivity<br />.threshold: 0.7856<br />value: 0.1915","metric: sensitivity<br />.threshold: 0.7866<br />value: 0.1702","metric: sensitivity<br />.threshold: 0.7913<br />value: 0.1702","metric: sensitivity<br />.threshold: 0.8071<br />value: 0.1489","metric: sensitivity<br />.threshold: 0.8225<br />value: 0.1489","metric: sensitivity<br />.threshold: 0.8261<br />value: 0.1277","metric: sensitivity<br />.threshold: 0.8343<br />value: 0.1064","metric: sensitivity<br />.threshold: 0.8430<br />value: 0.1064","metric: sensitivity<br />.threshold: 0.8513<br />value: 0.1064","metric: sensitivity<br />.threshold: 0.8580<br />value: 0.0851","metric: sensitivity<br />.threshold: 0.8588<br />value: 0.0638","metric: sensitivity<br />.threshold: 0.8614<br />value: 0.0426","metric: sensitivity<br />.threshold: 0.8912<br />value: 0.0213","metric: sensitivity<br />.threshold:    Inf<br />value: 0.0000"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(248,118,109,1)","dash":"solid"},"hoveron":"points","name":"sensitivity","legendgroup":"sensitivity","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[null,0.0422,0.0487,0.0568,0.0674,0.0685,0.0705,0.0884,0.0922,0.0945,0.0975,0.1093,0.1123,0.1171,0.1239,0.1243,0.1248,0.1256,0.1271,0.1401,0.1419,0.1473,0.1514,0.1538,0.1551,0.163,0.1631,0.1635,0.1657,0.1678,0.1712,0.1761,0.1796,0.1807,0.1846,0.1849,0.1853,0.1853,0.1868,0.187,0.1983,0.1988,0.1994,0.1997,0.2012,0.2022,0.2161,0.2171,0.2192,0.222,0.2227,0.229,0.2315,0.2349,0.2354,0.2354,0.237,0.2374,0.2401,0.2402,0.2429,0.2457,0.2475,0.253,0.2577,0.2601,0.2646,0.2671,0.2672,0.2678,0.2684,0.269,0.2692,0.2707,0.2776,0.2778,0.2792,0.2833,0.2863,0.2874,0.2892,0.2892,0.2897,0.2908,0.2927,0.295,0.3023,0.3072,0.3077,0.308,0.3131,0.314,0.3153,0.3159,0.3211,0.3224,0.3238,0.324,0.3241,0.3245,0.3268,0.3275,0.3297,0.3353,0.3355,0.3387,0.34,0.3415,0.3436,0.3447,0.345,0.3465,0.3476,0.3482,0.3524,0.3535,0.3564,0.3568,0.3578,0.3626,0.3646,0.3655,0.3671,0.3673,0.3674,0.3677,0.3682,0.3731,0.3735,0.3735,0.3745,0.376,0.3768,0.3831,0.3858,0.3871,0.3918,0.3928,0.3943,0.3955,0.4018,0.4028,0.4072,0.4078,0.409,0.4129,0.4138,0.4158,0.4167,0.4187,0.4203,0.421,0.4342,0.4363,0.4398,0.4409,0.442,0.4427,0.4449,0.4479,0.4483,0.4483,0.4515,0.4542,0.4576,0.458,0.4608,0.4612,0.4615,0.4618,0.4671,0.4687,0.4688,0.4691,0.4691,0.4751,0.4754,0.4756,0.4758,0.48,0.4824,0.4878,0.4894,0.4906,0.4913,0.4915,0.4934,0.4957,0.5017,0.5044,0.5046,0.5048,0.5063,0.5067,0.5091,0.5105,0.511,0.5116,0.5134,0.5143,0.5149,0.5175,0.5193,0.5211,0.522,0.5223,0.5235,0.5243,0.5258,0.5281,0.529,0.5433,0.5436,0.5467,0.5477,0.5486,0.5497,0.5537,0.5566,0.5573,0.5576,0.5596,0.5679,0.5703,0.5835,0.5845,0.5905,0.5911,0.5921,0.5925,0.5945,0.605,0.6102,0.6104,0.6108,0.6242,0.6344,0.6362,0.6369,0.6388,0.6391,0.6421,0.6439,0.6472,0.6512,0.6549,0.6557,0.6561,0.6561,0.6576,0.6579,0.6591,0.6654,0.6654,0.6655,0.6728,0.6805,0.6881,0.6903,0.7029,0.7076,0.7139,0.7159,0.7186,0.7216,0.7375,0.7387,0.7393,0.7395,0.7459,0.7483,0.7492,0.7571,0.7631,0.7641,0.7645,0.7679,0.7709,0.7737,0.774,0.7755,0.7856,0.7866,0.7913,0.8071,0.8225,0.8261,0.8343,0.843,0.8513,0.858,0.8588,0.8614,0.8912,null],"y":[0,0,0.0041,0.0081,0.0122,0.0163,0.0203,0.0244,0.0285,0.0325,0.0366,0.0407,0.0447,0.0488,0.0528,0.0569,0.061,0.065,0.0691,0.0732,0.0772,0.0813,0.0854,0.0894,0.0935,0.0976,0.1016,0.1057,0.1098,0.1138,0.1179,0.122,0.126,0.1301,0.1341,0.1382,0.1423,0.1463,0.1504,0.1545,0.1585,0.1626,0.1667,0.1707,0.1748,0.1789,0.1829,0.1829,0.187,0.1911,0.1951,0.1992,0.2033,0.2073,0.2114,0.2154,0.2195,0.2236,0.2276,0.2317,0.2358,0.2398,0.2439,0.248,0.252,0.2561,0.2602,0.2642,0.2683,0.2724,0.2764,0.2805,0.2846,0.2886,0.2886,0.2927,0.2967,0.3008,0.3049,0.3089,0.3089,0.313,0.3171,0.3211,0.3252,0.3293,0.3333,0.3374,0.3415,0.3455,0.3455,0.3496,0.3537,0.3577,0.3618,0.3659,0.3699,0.374,0.378,0.378,0.3821,0.3862,0.3902,0.3943,0.3984,0.4024,0.4065,0.4106,0.4146,0.4187,0.4228,0.4268,0.4309,0.435,0.439,0.4431,0.4472,0.4472,0.4512,0.4553,0.4593,0.4634,0.4675,0.4715,0.4756,0.4797,0.4837,0.4878,0.4878,0.4919,0.4959,0.5,0.5041,0.5081,0.5122,0.5163,0.5203,0.5244,0.5285,0.5325,0.5366,0.5366,0.5407,0.5447,0.5447,0.5488,0.5528,0.5569,0.561,0.565,0.5691,0.5732,0.5772,0.5813,0.5854,0.5894,0.5935,0.5976,0.6016,0.6057,0.6098,0.6138,0.6179,0.622,0.626,0.6301,0.6341,0.6341,0.6382,0.6423,0.6463,0.6463,0.6504,0.6545,0.6545,0.6585,0.6626,0.6667,0.6707,0.6748,0.6789,0.6829,0.687,0.6911,0.6951,0.6992,0.7033,0.7033,0.7073,0.7114,0.7154,0.7195,0.7236,0.7276,0.7317,0.7317,0.7358,0.7398,0.7439,0.748,0.752,0.7561,0.7602,0.7602,0.7642,0.7683,0.7724,0.7764,0.7805,0.7846,0.7886,0.7927,0.7927,0.7967,0.8008,0.8049,0.8089,0.813,0.813,0.8171,0.8211,0.8252,0.8252,0.8293,0.8333,0.8374,0.8415,0.8455,0.8455,0.8496,0.8537,0.8577,0.8618,0.8659,0.8699,0.8699,0.874,0.874,0.878,0.878,0.878,0.878,0.8821,0.8821,0.8862,0.8902,0.8943,0.8984,0.8984,0.9024,0.9065,0.9106,0.9106,0.9146,0.9187,0.9187,0.9228,0.9268,0.9309,0.935,0.935,0.939,0.9431,0.9472,0.9512,0.9553,0.9593,0.9634,0.9634,0.9634,0.9675,0.9675,0.9675,0.9675,0.9675,0.9715,0.9715,0.9756,0.9797,0.9797,0.9837,0.9837,0.9837,0.9878,0.9878,0.9919,0.9919,0.9919,0.9959,1,1,1,1,1,1],"text":["metric: specificity<br />.threshold:   -Inf<br />value: 0.0000","metric: specificity<br />.threshold: 0.0422<br />value: 0.0000","metric: specificity<br />.threshold: 0.0487<br />value: 0.0041","metric: specificity<br />.threshold: 0.0568<br />value: 0.0081","metric: specificity<br />.threshold: 0.0674<br />value: 0.0122","metric: specificity<br />.threshold: 0.0685<br />value: 0.0163","metric: specificity<br />.threshold: 0.0705<br />value: 0.0203","metric: specificity<br />.threshold: 0.0884<br />value: 0.0244","metric: specificity<br />.threshold: 0.0922<br />value: 0.0285","metric: specificity<br />.threshold: 0.0945<br />value: 0.0325","metric: specificity<br />.threshold: 0.0975<br />value: 0.0366","metric: specificity<br />.threshold: 0.1093<br />value: 0.0407","metric: specificity<br />.threshold: 0.1123<br />value: 0.0447","metric: specificity<br />.threshold: 0.1171<br />value: 0.0488","metric: specificity<br />.threshold: 0.1239<br />value: 0.0528","metric: specificity<br />.threshold: 0.1243<br />value: 0.0569","metric: specificity<br />.threshold: 0.1248<br />value: 0.0610","metric: specificity<br />.threshold: 0.1256<br />value: 0.0650","metric: specificity<br />.threshold: 0.1271<br />value: 0.0691","metric: specificity<br />.threshold: 0.1401<br />value: 0.0732","metric: specificity<br />.threshold: 0.1419<br />value: 0.0772","metric: specificity<br />.threshold: 0.1473<br />value: 0.0813","metric: specificity<br />.threshold: 0.1514<br />value: 0.0854","metric: specificity<br />.threshold: 0.1538<br />value: 0.0894","metric: specificity<br />.threshold: 0.1551<br />value: 0.0935","metric: specificity<br />.threshold: 0.1630<br />value: 0.0976","metric: specificity<br />.threshold: 0.1631<br />value: 0.1016","metric: specificity<br />.threshold: 0.1635<br />value: 0.1057","metric: specificity<br />.threshold: 0.1657<br />value: 0.1098","metric: specificity<br />.threshold: 0.1678<br />value: 0.1138","metric: specificity<br />.threshold: 0.1712<br />value: 0.1179","metric: specificity<br />.threshold: 0.1761<br />value: 0.1220","metric: specificity<br />.threshold: 0.1796<br />value: 0.1260","metric: specificity<br />.threshold: 0.1807<br />value: 0.1301","metric: specificity<br />.threshold: 0.1846<br />value: 0.1341","metric: specificity<br />.threshold: 0.1849<br />value: 0.1382","metric: specificity<br />.threshold: 0.1853<br />value: 0.1423","metric: specificity<br />.threshold: 0.1853<br />value: 0.1463","metric: specificity<br />.threshold: 0.1868<br />value: 0.1504","metric: specificity<br />.threshold: 0.1870<br />value: 0.1545","metric: specificity<br />.threshold: 0.1983<br />value: 0.1585","metric: specificity<br />.threshold: 0.1988<br />value: 0.1626","metric: specificity<br />.threshold: 0.1994<br />value: 0.1667","metric: specificity<br />.threshold: 0.1997<br />value: 0.1707","metric: specificity<br />.threshold: 0.2012<br />value: 0.1748","metric: specificity<br />.threshold: 0.2022<br />value: 0.1789","metric: specificity<br />.threshold: 0.2161<br />value: 0.1829","metric: specificity<br />.threshold: 0.2171<br />value: 0.1829","metric: specificity<br />.threshold: 0.2192<br />value: 0.1870","metric: specificity<br />.threshold: 0.2220<br />value: 0.1911","metric: specificity<br />.threshold: 0.2227<br />value: 0.1951","metric: specificity<br />.threshold: 0.2290<br />value: 0.1992","metric: specificity<br />.threshold: 0.2315<br />value: 0.2033","metric: specificity<br />.threshold: 0.2349<br />value: 0.2073","metric: specificity<br />.threshold: 0.2354<br />value: 0.2114","metric: specificity<br />.threshold: 0.2354<br />value: 0.2154","metric: specificity<br />.threshold: 0.2370<br />value: 0.2195","metric: specificity<br />.threshold: 0.2374<br />value: 0.2236","metric: specificity<br />.threshold: 0.2401<br />value: 0.2276","metric: specificity<br />.threshold: 0.2402<br />value: 0.2317","metric: specificity<br />.threshold: 0.2429<br />value: 0.2358","metric: specificity<br />.threshold: 0.2457<br />value: 0.2398","metric: specificity<br />.threshold: 0.2475<br />value: 0.2439","metric: specificity<br />.threshold: 0.2530<br />value: 0.2480","metric: specificity<br />.threshold: 0.2577<br />value: 0.2520","metric: specificity<br />.threshold: 0.2601<br />value: 0.2561","metric: specificity<br />.threshold: 0.2646<br />value: 0.2602","metric: specificity<br />.threshold: 0.2671<br />value: 0.2642","metric: specificity<br />.threshold: 0.2672<br />value: 0.2683","metric: specificity<br />.threshold: 0.2678<br />value: 0.2724","metric: specificity<br />.threshold: 0.2684<br />value: 0.2764","metric: specificity<br />.threshold: 0.2690<br />value: 0.2805","metric: specificity<br />.threshold: 0.2692<br />value: 0.2846","metric: specificity<br />.threshold: 0.2707<br />value: 0.2886","metric: specificity<br />.threshold: 0.2776<br />value: 0.2886","metric: specificity<br />.threshold: 0.2778<br />value: 0.2927","metric: specificity<br />.threshold: 0.2792<br />value: 0.2967","metric: specificity<br />.threshold: 0.2833<br />value: 0.3008","metric: specificity<br />.threshold: 0.2863<br />value: 0.3049","metric: specificity<br />.threshold: 0.2874<br />value: 0.3089","metric: specificity<br />.threshold: 0.2892<br />value: 0.3089","metric: specificity<br />.threshold: 0.2892<br />value: 0.3130","metric: specificity<br />.threshold: 0.2897<br />value: 0.3171","metric: specificity<br />.threshold: 0.2908<br />value: 0.3211","metric: specificity<br />.threshold: 0.2927<br />value: 0.3252","metric: specificity<br />.threshold: 0.2950<br />value: 0.3293","metric: specificity<br />.threshold: 0.3023<br />value: 0.3333","metric: specificity<br />.threshold: 0.3072<br />value: 0.3374","metric: specificity<br />.threshold: 0.3077<br />value: 0.3415","metric: specificity<br />.threshold: 0.3080<br />value: 0.3455","metric: specificity<br />.threshold: 0.3131<br />value: 0.3455","metric: specificity<br />.threshold: 0.3140<br />value: 0.3496","metric: specificity<br />.threshold: 0.3153<br />value: 0.3537","metric: specificity<br />.threshold: 0.3159<br />value: 0.3577","metric: specificity<br />.threshold: 0.3211<br />value: 0.3618","metric: specificity<br />.threshold: 0.3224<br />value: 0.3659","metric: specificity<br />.threshold: 0.3238<br />value: 0.3699","metric: specificity<br />.threshold: 0.3240<br />value: 0.3740","metric: specificity<br />.threshold: 0.3241<br />value: 0.3780","metric: specificity<br />.threshold: 0.3245<br />value: 0.3780","metric: specificity<br />.threshold: 0.3268<br />value: 0.3821","metric: specificity<br />.threshold: 0.3275<br />value: 0.3862","metric: specificity<br />.threshold: 0.3297<br />value: 0.3902","metric: specificity<br />.threshold: 0.3353<br />value: 0.3943","metric: specificity<br />.threshold: 0.3355<br />value: 0.3984","metric: specificity<br />.threshold: 0.3387<br />value: 0.4024","metric: specificity<br />.threshold: 0.3400<br />value: 0.4065","metric: specificity<br />.threshold: 0.3415<br />value: 0.4106","metric: specificity<br />.threshold: 0.3436<br />value: 0.4146","metric: specificity<br />.threshold: 0.3447<br />value: 0.4187","metric: specificity<br />.threshold: 0.3450<br />value: 0.4228","metric: specificity<br />.threshold: 0.3465<br />value: 0.4268","metric: specificity<br />.threshold: 0.3476<br />value: 0.4309","metric: specificity<br />.threshold: 0.3482<br />value: 0.4350","metric: specificity<br />.threshold: 0.3524<br />value: 0.4390","metric: specificity<br />.threshold: 0.3535<br />value: 0.4431","metric: specificity<br />.threshold: 0.3564<br />value: 0.4472","metric: specificity<br />.threshold: 0.3568<br />value: 0.4472","metric: specificity<br />.threshold: 0.3578<br />value: 0.4512","metric: specificity<br />.threshold: 0.3626<br />value: 0.4553","metric: specificity<br />.threshold: 0.3646<br />value: 0.4593","metric: specificity<br />.threshold: 0.3655<br />value: 0.4634","metric: specificity<br />.threshold: 0.3671<br />value: 0.4675","metric: specificity<br />.threshold: 0.3673<br />value: 0.4715","metric: specificity<br />.threshold: 0.3674<br />value: 0.4756","metric: specificity<br />.threshold: 0.3677<br />value: 0.4797","metric: specificity<br />.threshold: 0.3682<br />value: 0.4837","metric: specificity<br />.threshold: 0.3731<br />value: 0.4878","metric: specificity<br />.threshold: 0.3735<br />value: 0.4878","metric: specificity<br />.threshold: 0.3735<br />value: 0.4919","metric: specificity<br />.threshold: 0.3745<br />value: 0.4959","metric: specificity<br />.threshold: 0.3760<br />value: 0.5000","metric: specificity<br />.threshold: 0.3768<br />value: 0.5041","metric: specificity<br />.threshold: 0.3831<br />value: 0.5081","metric: specificity<br />.threshold: 0.3858<br />value: 0.5122","metric: specificity<br />.threshold: 0.3871<br />value: 0.5163","metric: specificity<br />.threshold: 0.3918<br />value: 0.5203","metric: specificity<br />.threshold: 0.3928<br />value: 0.5244","metric: specificity<br />.threshold: 0.3943<br />value: 0.5285","metric: specificity<br />.threshold: 0.3955<br />value: 0.5325","metric: specificity<br />.threshold: 0.4018<br />value: 0.5366","metric: specificity<br />.threshold: 0.4028<br />value: 0.5366","metric: specificity<br />.threshold: 0.4072<br />value: 0.5407","metric: specificity<br />.threshold: 0.4078<br />value: 0.5447","metric: specificity<br />.threshold: 0.4090<br />value: 0.5447","metric: specificity<br />.threshold: 0.4129<br />value: 0.5488","metric: specificity<br />.threshold: 0.4138<br />value: 0.5528","metric: specificity<br />.threshold: 0.4158<br />value: 0.5569","metric: specificity<br />.threshold: 0.4167<br />value: 0.5610","metric: specificity<br />.threshold: 0.4187<br />value: 0.5650","metric: specificity<br />.threshold: 0.4203<br />value: 0.5691","metric: specificity<br />.threshold: 0.4210<br />value: 0.5732","metric: specificity<br />.threshold: 0.4342<br />value: 0.5772","metric: specificity<br />.threshold: 0.4363<br />value: 0.5813","metric: specificity<br />.threshold: 0.4398<br />value: 0.5854","metric: specificity<br />.threshold: 0.4409<br />value: 0.5894","metric: specificity<br />.threshold: 0.4420<br />value: 0.5935","metric: specificity<br />.threshold: 0.4427<br />value: 0.5976","metric: specificity<br />.threshold: 0.4449<br />value: 0.6016","metric: specificity<br />.threshold: 0.4479<br />value: 0.6057","metric: specificity<br />.threshold: 0.4483<br />value: 0.6098","metric: specificity<br />.threshold: 0.4483<br />value: 0.6138","metric: specificity<br />.threshold: 0.4515<br />value: 0.6179","metric: specificity<br />.threshold: 0.4542<br />value: 0.6220","metric: specificity<br />.threshold: 0.4576<br />value: 0.6260","metric: specificity<br />.threshold: 0.4580<br />value: 0.6301","metric: specificity<br />.threshold: 0.4608<br />value: 0.6341","metric: specificity<br />.threshold: 0.4612<br />value: 0.6341","metric: specificity<br />.threshold: 0.4615<br />value: 0.6382","metric: specificity<br />.threshold: 0.4618<br />value: 0.6423","metric: specificity<br />.threshold: 0.4671<br />value: 0.6463","metric: specificity<br />.threshold: 0.4687<br />value: 0.6463","metric: specificity<br />.threshold: 0.4688<br />value: 0.6504","metric: specificity<br />.threshold: 0.4691<br />value: 0.6545","metric: specificity<br />.threshold: 0.4691<br />value: 0.6545","metric: specificity<br />.threshold: 0.4751<br />value: 0.6585","metric: specificity<br />.threshold: 0.4754<br />value: 0.6626","metric: specificity<br />.threshold: 0.4756<br />value: 0.6667","metric: specificity<br />.threshold: 0.4758<br />value: 0.6707","metric: specificity<br />.threshold: 0.4800<br />value: 0.6748","metric: specificity<br />.threshold: 0.4824<br />value: 0.6789","metric: specificity<br />.threshold: 0.4878<br />value: 0.6829","metric: specificity<br />.threshold: 0.4894<br />value: 0.6870","metric: specificity<br />.threshold: 0.4906<br />value: 0.6911","metric: specificity<br />.threshold: 0.4913<br />value: 0.6951","metric: specificity<br />.threshold: 0.4915<br />value: 0.6992","metric: specificity<br />.threshold: 0.4934<br />value: 0.7033","metric: specificity<br />.threshold: 0.4957<br />value: 0.7033","metric: specificity<br />.threshold: 0.5017<br />value: 0.7073","metric: specificity<br />.threshold: 0.5044<br />value: 0.7114","metric: specificity<br />.threshold: 0.5046<br />value: 0.7154","metric: specificity<br />.threshold: 0.5048<br />value: 0.7195","metric: specificity<br />.threshold: 0.5063<br />value: 0.7236","metric: specificity<br />.threshold: 0.5067<br />value: 0.7276","metric: specificity<br />.threshold: 0.5091<br />value: 0.7317","metric: specificity<br />.threshold: 0.5105<br />value: 0.7317","metric: specificity<br />.threshold: 0.5110<br />value: 0.7358","metric: specificity<br />.threshold: 0.5116<br />value: 0.7398","metric: specificity<br />.threshold: 0.5134<br />value: 0.7439","metric: specificity<br />.threshold: 0.5143<br />value: 0.7480","metric: specificity<br />.threshold: 0.5149<br />value: 0.7520","metric: specificity<br />.threshold: 0.5175<br />value: 0.7561","metric: specificity<br />.threshold: 0.5193<br />value: 0.7602","metric: specificity<br />.threshold: 0.5211<br />value: 0.7602","metric: specificity<br />.threshold: 0.5220<br />value: 0.7642","metric: specificity<br />.threshold: 0.5223<br />value: 0.7683","metric: specificity<br />.threshold: 0.5235<br />value: 0.7724","metric: specificity<br />.threshold: 0.5243<br />value: 0.7764","metric: specificity<br />.threshold: 0.5258<br />value: 0.7805","metric: specificity<br />.threshold: 0.5281<br />value: 0.7846","metric: specificity<br />.threshold: 0.5290<br />value: 0.7886","metric: specificity<br />.threshold: 0.5433<br />value: 0.7927","metric: specificity<br />.threshold: 0.5436<br />value: 0.7927","metric: specificity<br />.threshold: 0.5467<br />value: 0.7967","metric: specificity<br />.threshold: 0.5477<br />value: 0.8008","metric: specificity<br />.threshold: 0.5486<br />value: 0.8049","metric: specificity<br />.threshold: 0.5497<br />value: 0.8089","metric: specificity<br />.threshold: 0.5537<br />value: 0.8130","metric: specificity<br />.threshold: 0.5566<br />value: 0.8130","metric: specificity<br />.threshold: 0.5573<br />value: 0.8171","metric: specificity<br />.threshold: 0.5576<br />value: 0.8211","metric: specificity<br />.threshold: 0.5596<br />value: 0.8252","metric: specificity<br />.threshold: 0.5679<br />value: 0.8252","metric: specificity<br />.threshold: 0.5703<br />value: 0.8293","metric: specificity<br />.threshold: 0.5835<br />value: 0.8333","metric: specificity<br />.threshold: 0.5845<br />value: 0.8374","metric: specificity<br />.threshold: 0.5905<br />value: 0.8415","metric: specificity<br />.threshold: 0.5911<br />value: 0.8455","metric: specificity<br />.threshold: 0.5921<br />value: 0.8455","metric: specificity<br />.threshold: 0.5925<br />value: 0.8496","metric: specificity<br />.threshold: 0.5945<br />value: 0.8537","metric: specificity<br />.threshold: 0.6050<br />value: 0.8577","metric: specificity<br />.threshold: 0.6102<br />value: 0.8618","metric: specificity<br />.threshold: 0.6104<br />value: 0.8659","metric: specificity<br />.threshold: 0.6108<br />value: 0.8699","metric: specificity<br />.threshold: 0.6242<br />value: 0.8699","metric: specificity<br />.threshold: 0.6344<br />value: 0.8740","metric: specificity<br />.threshold: 0.6362<br />value: 0.8740","metric: specificity<br />.threshold: 0.6369<br />value: 0.8780","metric: specificity<br />.threshold: 0.6388<br />value: 0.8780","metric: specificity<br />.threshold: 0.6391<br />value: 0.8780","metric: specificity<br />.threshold: 0.6421<br />value: 0.8780","metric: specificity<br />.threshold: 0.6439<br />value: 0.8821","metric: specificity<br />.threshold: 0.6472<br />value: 0.8821","metric: specificity<br />.threshold: 0.6512<br />value: 0.8862","metric: specificity<br />.threshold: 0.6549<br />value: 0.8902","metric: specificity<br />.threshold: 0.6557<br />value: 0.8943","metric: specificity<br />.threshold: 0.6561<br />value: 0.8984","metric: specificity<br />.threshold: 0.6561<br />value: 0.8984","metric: specificity<br />.threshold: 0.6576<br />value: 0.9024","metric: specificity<br />.threshold: 0.6579<br />value: 0.9065","metric: specificity<br />.threshold: 0.6591<br />value: 0.9106","metric: specificity<br />.threshold: 0.6654<br />value: 0.9106","metric: specificity<br />.threshold: 0.6654<br />value: 0.9146","metric: specificity<br />.threshold: 0.6655<br />value: 0.9187","metric: specificity<br />.threshold: 0.6728<br />value: 0.9187","metric: specificity<br />.threshold: 0.6805<br />value: 0.9228","metric: specificity<br />.threshold: 0.6881<br />value: 0.9268","metric: specificity<br />.threshold: 0.6903<br />value: 0.9309","metric: specificity<br />.threshold: 0.7029<br />value: 0.9350","metric: specificity<br />.threshold: 0.7076<br />value: 0.9350","metric: specificity<br />.threshold: 0.7139<br />value: 0.9390","metric: specificity<br />.threshold: 0.7159<br />value: 0.9431","metric: specificity<br />.threshold: 0.7186<br />value: 0.9472","metric: specificity<br />.threshold: 0.7216<br />value: 0.9512","metric: specificity<br />.threshold: 0.7375<br />value: 0.9553","metric: specificity<br />.threshold: 0.7387<br />value: 0.9593","metric: specificity<br />.threshold: 0.7393<br />value: 0.9634","metric: specificity<br />.threshold: 0.7395<br />value: 0.9634","metric: specificity<br />.threshold: 0.7459<br />value: 0.9634","metric: specificity<br />.threshold: 0.7483<br />value: 0.9675","metric: specificity<br />.threshold: 0.7492<br />value: 0.9675","metric: specificity<br />.threshold: 0.7571<br />value: 0.9675","metric: specificity<br />.threshold: 0.7631<br />value: 0.9675","metric: specificity<br />.threshold: 0.7641<br />value: 0.9675","metric: specificity<br />.threshold: 0.7645<br />value: 0.9715","metric: specificity<br />.threshold: 0.7679<br />value: 0.9715","metric: specificity<br />.threshold: 0.7709<br />value: 0.9756","metric: specificity<br />.threshold: 0.7737<br />value: 0.9797","metric: specificity<br />.threshold: 0.7740<br />value: 0.9797","metric: specificity<br />.threshold: 0.7755<br />value: 0.9837","metric: specificity<br />.threshold: 0.7856<br />value: 0.9837","metric: specificity<br />.threshold: 0.7866<br />value: 0.9837","metric: specificity<br />.threshold: 0.7913<br />value: 0.9878","metric: specificity<br />.threshold: 0.8071<br />value: 0.9878","metric: specificity<br />.threshold: 0.8225<br />value: 0.9919","metric: specificity<br />.threshold: 0.8261<br />value: 0.9919","metric: specificity<br />.threshold: 0.8343<br />value: 0.9919","metric: specificity<br />.threshold: 0.8430<br />value: 0.9959","metric: specificity<br />.threshold: 0.8513<br />value: 1.0000","metric: specificity<br />.threshold: 0.8580<br />value: 1.0000","metric: specificity<br />.threshold: 0.8588<br />value: 1.0000","metric: specificity<br />.threshold: 0.8614<br />value: 1.0000","metric: specificity<br />.threshold: 0.8912<br />value: 1.0000","metric: specificity<br />.threshold:    Inf<br />value: 1.0000"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(0,191,196,1)","dash":"solid"},"hoveron":"points","name":"specificity","legendgroup":"specificity","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":26.2283105022831,"r":7.30593607305936,"b":40.1826484018265,"l":48.9497716894977},"font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-0.00025,0.93365],"tickmode":"array","ticktext":["0.00","0.25","0.50","0.75"],"tickvals":[0,0.25,0.5,0.75],"categoryorder":"array","categoryarray":["0.00","0.25","0.50","0.75"],"nticks":null,"ticks":"","tickcolor":null,"ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":{"text":"Probability Threshold to be Classified as Positive","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-0.05,1.05],"tickmode":"array","ticktext":["0.00","0.25","0.50","0.75","1.00"],"tickvals":[0,0.25,0.5,0.75,1],"categoryorder":"array","categoryarray":["0.00","0.25","0.50","0.75","1.00"],"nticks":null,"ticks":"","tickcolor":null,"ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":{"text":"Value","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":null,"bordercolor":null,"borderwidth":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895},"y":0.96751968503937},"annotations":[{"text":"Metrics","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","showSendToCloud":false},"source":"A","attrs":{"d84c233b198c":{"colour":{},"x":{},"y":{},"type":"scatter"}},"cur_data":"d84c233b198c","visdat":{"d84c233b198c":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
