---
title: Bayesian Statistics and A/B Testing
author: Arga Adyatama
github: https://github.com/Argaadya
date: '2021-03-08'
slug: bayesian-statistics-and-a-b-testing
categories:
  - R
tags:
  - Statistics
  - Bayesian
description: ''
featured: 'bs.png'
featuredalt: ''
featuredpath: 'date'
linktitle: ''
type: post
---

<style>
body {
text-align: justify}
</style>



# Introduction

Statistics is one of the most essential tools in doing research. Statistics deals with uncertainty, both in our everyday life or in business operation. However, people sometimes discouraged from learning statistics because there are so much statistics test to remember. Sometimes people abuse this test by ignoring the underlying assumption. The following graph illustrate different kind of test for different situation[^1].

<center> ![](/img/ab_test/test.png) </center> 

We need a better framework for our statistical thinking that is more transparent and flexible to use. Need no worry, there is a growing body of new kind of statistics that matched with our daily framework of thinking. This field is known as `Bayesian statistics`. In this article we will find out what Bayesian statistics is and how can we apply it in A/B testing for marketing purposes.

# Library and Setup

The following are the required libraries that will be used throughout the post.


```r
# Data Wrangling
library(tidyverse)
library(lubridate)

# Bayes AB Test
library(bayesAB)

# Visualization
library(waffle)
library(ggthemes)
library(scales)

# Plot Background Set
theme_set(theme_void() + theme(legend.position = "top"))

options(scipen = 999)
```

# Probability

The common definition of probability is that it help us measure uncertainty or how likely something is to happen. You may be familiar with the problem of flipping coin and try to figure out what is the probability of getting a head. Another example would how likely it is for me to come late at work if I wake up at 8 am, what is the probability that tomorrow will be rain, etc. We use probability to state that we don't have certain answer for some problems so we have to measure our uncertainty.

In the field of statistics, there are two school of thought regarding probability: the Frequentist and the Bayesian. We will explore what's their difference.

## Frequentist Probability

The frequentist approach to probability is the one you have studied in the college. It state probability as the frequency of some event compared to the other possible events. Let's imagine the typical coin flipping problem and we state that there are only 2 possible event: head or tail. 

<center> ![](/img/ab_test/coin_flip.gif) </center> 


$$
\Omega = \begin{Bmatrix} head, tail \end{Bmatrix}
$$

You can easily calculate the probability of getting head as $\frac{1}{2}$ because there are only 2 event and getting a head happened once.

$$
P(head) = \frac{1}{2}
$$

The same rule apply when we throw a dice and guess which number will come out.

<center> ![](/img/ab_test/dice.gif) </center> 


$$
\Omega = \begin{Bmatrix} 1, 2, ..., 6 \end{Bmatrix}
$$

The probability of getting a six is as follows:

$$
P(six) = \frac{1}{6}
$$

This frequentist probability assume that if you repeated flipping coin on infinite number of time you will approximately get those number. We can simulate what happen when we do coin flipping with different number of trials.

First, we will toss a single coin. The `seed` will make sure that we can reproduce the random sampling. 


```r
# Toss a single coin
set.seed(123)
sample(c("Head", "Tail"), 1)
```

```
#> [1] "Head"
```

Now we try to toss the coin 10 times and see the result. We can measure the probability by calculating how many head that we got in 10 trials. For this trials, we get the probability of head to be **0.6**. That's quite far from the theoretical probability that is 0.5. This is because the number of trials is to small. Let's try with the bigger one.















































































































