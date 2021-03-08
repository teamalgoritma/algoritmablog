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
featured: ''
featuredalt: ''
featuredpath: ''
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

You can easily calculate the probability of getting head as `\(\frac{1}{2}\)` because there are only 2 event and getting a head happened once.

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


```r
# Sampling
set.seed(123)
flip_coin <- sample(x = c("Head", "Tail"), size = 10, replace = T)

# The probability
prop_coin <- table(flip_coin)[1]/length(flip_coin)

# Visualization
data.frame(outcome = flip_coin) %>% 
  count(outcome) %>% 
  ggplot(aes(fill = outcome, values = n)) +
  geom_waffle(n_rows = 2, color = "#1D2024") +
  scale_fill_manual(values = c("firebrick", "dodgerblue4")) +
  labs(title = "Random 10 Coins Tosses",
       subtitle = paste0("Probability of Getting Head: ", prop_coin)
       )
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-4-1.png" width="672" style="display: block; margin: auto;" />

Here we try with 100 number of trials. The probability is getting closer to 0.5, although still quite far.


```r
# Sampling
set.seed(123)
flip_coin <- sample(x = c("Head", "Tail"), size = 100, replace = T)

# The probability
prop_coin <- table(flip_coin)[1]/length(flip_coin)

# Visualization
data.frame(outcome = flip_coin) %>% 
  count(outcome) %>% 
  ggplot(aes(fill = outcome, values = n)) +
  geom_waffle(n_rows = 4, color = "#1D2024") +
  scale_fill_manual(values = c("firebrick", "dodgerblue4")) +
  labs(title = "Random 100 Coins Tosses",
       subtitle = paste0("Probability of Getting Head: ", prop_coin)
       )
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-5-1.png" width="672" style="display: block; margin: auto;" />

Now we will try with even bigger number of trials. 


```r
# Sampling
set.seed(123)
flip_coin <- sample(x = c("Head", "Tail"), size = 1e4, replace = T)

# The probability
prop.table(table("10,000 Coin Flipping" = flip_coin))
```

```
#> 10,000 Coin Flipping
#>   Head   Tail 
#> 0.5017 0.4983
```

With 10,000 random coin flipping, the probability is getting very close to the theoretical one but still not close enough. We can try to simulate more coin flipping with 1,000,000.


```r
# Sampling
set.seed(123)
flip_coin <- sample(x = c("Head", "Tail"), size = 1e6, replace = T)

# The probability
prop.table(table("1,000,000 Coin Flipping" = flip_coin))
```

```
#> 1,000,000 Coin Flipping
#>     Head     Tail 
#> 0.500304 0.499696
```

To get very close to the theoretical probability, one must repeat the same experiment in infinite number of trials. The problem is, not all event or experiment can be done in such amount of trials. Some event may only happened once or it would be very hard to do experiment with plenty amount of trials due to limited resources. How do we measure the probability of a candidate would win the presidential election? We can't repeat the same election twice. What is the probability it will rain tomorrow? Or what if you come home and found out that your laptop is missing with your house back door opened. What is the probability of you getting robbed? Also, the probability 0.5 derived from getting 2 head from 4 trials and the probability from getting 500 head from 10 trials will give you different impression. You will be more likely to trust the later one. 

<center> ![](/img/ab_test/coin_trial.png) </center> 


To answer such case, we need another method to measure uncertainty. But don't worry, we will not entirely throw away the previous concept of probability. Instead, we will improve upon it. Enter the Bayesian statistics.


```r
# Plot Background Set
theme_set(theme_pander() + theme(legend.position = "top"))
```

## Bayesian Probability

Instead of considering probability as a mere reflection of the frequency of a particular event, we can consider probability as a way to state our belief about something. Those belief can always change and be updated with new information. 

For example, if you were invited to go on a vacation to some beautiful beach on the next weekend. In general, you would love to take that invitation and have some fun. You have imagined what you will do in there, be it playing sand, playing volleyball or just stare silently at the ocean. I may state that your probability to take that invitation is very high.

$$
P(go\ to\ beach) = high
$$

<center> ![](/img/ab_test/sunny beach.jpg) </center> 

But wait a minute, when you check the weather forecast for the next weekend, you see that it would be rainy and possibly a thunderstorm. Well, it would not be so much fun if we catch a cold and has limited things to play. You start to reconsider your decision to accept that invitation. So, now I am pretty confident that the probability for you to take that invitation is change from high to low. 

On the following equation, we add `\(forecast = rain\)` to state that the probability has been calculated after we add new information of the weather forecast.

$$
P(go\ to\ beach| forecast =\ rain) = low
$$

<center> ![](/img/ab_test/rainy beach.jpeg) </center> 

That's the general concept of how we can change probability in the light of new data. I have my prior believe about your probability to take that invitation because I believe almost everyone would love to go the beach. However, my belief change after we know a new information from the weather forecast. We have done that kind of thing unconsciously almost every day. Bayesian probability is just a method to formalizing our way of thinking into proper probability and math notation.

Consider another example from Kruschke[^2]. Suppose you are the famous Sherlock Holmes and currently in the middle of a murder case. There are four suspects. Since you just arrive at the scene, you don't know about anything and thus all suspects are equally likely to be the culprit, illustrated by the upper left part of the following figure. The credibility is just another way of saying the probaiblity. The `Prior` shows our believe before seeing new data while the `Posterior` is our *updated* belief after seeing new data. You started to collect new data by doing observation in the murder scene. The first evidence you found indicate that the suspect `A` has a strong alibi, making him impossible to be the murder. We then adjust our believe, now we only consider `A` to be impossible and the rest of the suspects has higher chance to be the murder, indicated by the lower left and the top middle part of the figure. Now we start collecting the second evidence. Turns out this evidence eliminate suspect `B` from our list of murder. Once again, we update our belief. Now remain only 2 suspects left. To put the final nail in the coffin, the third evidence we found also put a strong alibi for the suspect `C`. Once again we adjust our belief by making it impossible for suspect `C` to be the murder. All that remain is the suspect `D`. Since there is no other suspect, suspect `D` has a certain chance to be the murder. This is the famous quote by Sherlock regarding this issue.

> How often have I said to you that when you have eliminated the impossible, whatever remains, however improbable, must be the truth?

This reallocation of credibility/probability is not only intuitive, it is also what the exact mathematics of Bayesian inference prescribe.

<center> ![](/img/ab_test/inference.png) </center> 

### Conditional Probability and Bayes Theorem

When a probability change based on a certain condition, then we can call it as the conditional probability. This include our previous example of your probability to take the invitation. 

$$
P(go\ to\ beach| forecast =\ rain) = low
$$

To illustrate another form of conditional probability, we will use the example of color blindness, a deficiency where the person unable or less able to distinguish some colors. In the general population, about 4.25 percent of people are color blind. This means that the probability of someone acquired with color blindness is 0.0425.

$$
P(color\ blind) = 0.0425
$$

Color blindness is caused by a defective gene in the X chromosome. Since males only have a single X chromosome while females have two X chromosomes, men are about 16 times more likely to suffer adverse effects of a defective X chromosome and therefore to be color blind. So while the rate of color blindness for the entire population is 4.25 percent, it is only 0.5 percent in females but 8 percent in males.

$$
P(color\ blind| males) = 0.08\\
P(color\ blind| females) = 0.005\\
$$

If we want to get the probability that someone is male and having colorblind, we can use the following formula:

$$
P(A,B) = P(A)\times P(B|A)
$$

The `\(P(A,B)\)` means that we want to get the probability that someone is male and also having a color blindness. The `\(P(B|A)\)` shows the probability of having a color blindness in male population. We can use the `product rule of probability`, which is that if we want to get the probability of two events to be happen at the same time is the product of probability of each event. Let's say that in the population we get 50:50 ratio between males and female.

$$
P(male, color\ blindness) = P(male)\times P(color\ blind| male)\\
P(male, color\ blindness) = 0.5\times 0.08 = 0.04
$$

### Bayes Theorem

Alternately, we can also try to find the probability that someone is a male given that the person is having color blindness ($P(A|B)$). We just need to rewrite the previous formula into something called `Bayes Theorem`.

$$
P(A|B) = \frac{P(A)\times P(B|A)}{P(B)}
$$

Consider the following case.

> We are currently researching a new testing method to detect the SARS-COV2 virus that cause the global pandemic. We test 10,000 people with 500 have the COVID-19 disease while the rest are healthy. The test was able to correctly detect 450 people with COVID-19 disease and 8,500 healthy people. So, what is the probability that you have the COVID-19 disease if the test result show positive?

<center> ![](/img/ab_test/drug-testing-index-lab.jpg) </center> 

You may want to jump to conclusion that the test can detect Covid-19 by simply divide the number of people tested positive for COVID-19 (450 people) with the total number of people with COVID-19 (500 people) which will give you the probability of 90% or 90% accuracy. 

$$
P(Test\ Positive| Covid) = \frac{450}{500} = 0.9
$$

However, this only answer half the problem because the test is not only correctly predict people with COVID-19 but also incorrectly predict healthy people as positive of COVID-19. We call it as false positive because the positive prediction is wrong.

Bayes theorem will give us more complete picture for this problem. With the following equation, we consider not only people with actual COVID-19 disease but also the healthy one. 

$$
P(Covid|Test\ Positive) = \frac{P(Covid)\times P(Test\ Positive|Covid)}{P(Test\ Positive)} \\
$$

We will run down the number starting from the frequency for each event.

- Number of people tested as positive: 450 + 1500 = 1950
- Number of people with Covid-19: 500
- Number of people with Covid-19 and tested positive: 450

Now we can calculate each element of the Bayes theorem

$$
P(Covid) = \frac{500}{10000} = 0.05\\
P(Test\ Positive) = \frac{1950}{10000} = 0.195\\
P(Test\ Positive| Covid) = \frac{450}{500} = 0.9
$$

Finally, we can get the probability that someone is actually getting a COVID-19 while tested positive


```r
p_covid <- 500/10000
p_positive <- 1950/10000
p_positive_covid <- 450/500

p_covid * p_positive_covid / p_positive
```

```
#> [1] 0.2307692
```

$$
P(Covid|Test\ Positive) = \frac{0.05\times 0.9}{0.195} = 0.2307\\
$$

Why such low number? That is because we have a lot of false positive where healthy people are incorrectly predicted as having COVID-19.

> Whenever the condition of interest is very rare, having a test that finds all the true cases is still no guarantee that a positive result carries much information at all. The reason is that most positive results are false positives, even when all the true positives are detected correctly.
<footer> McElreath, 2019</footer>

Bayes Theorem illustrate in how we should think in Bayesian framework. If we dissect each part of it, we will see that it has three main part:

**Prior**

The prior is the probability of something before we see or observe the data. On the previouse case, the prior is the probability that someone has COVID-19 before we see the result of the test.

**Likelihood**

The likelihood is the probability of observing the data.

**Posterior**

The posterior is the probability as the product of our prior believe (before observing data) and the likelihood (the observed data). The product between likelihood and prior must be normalized by dividing the product with some value to get a valid probability (in range of between 0-1).

<center> ![](/img/ab_test/data-science-bayes-theorem-2.jpg) </center> 

Often, we are not really sure the exact probability of something happened but instead we describe it as a range of possibility or a distribution of a probability. We will go back to Bayes theorem later later after we learn how to determine probability distribution.

## Probability Distribution

An event always have a counterpart. For example, the probability of landing a head in coin flip has the counterpart of landing a tail. In throwing dice, there are six possible values of the dice that can happened. Each event will have its own probability to happen. Sometimes they are equally likely to happen, sometimes they don't. The spread of this probability for each event or some values is called the `probability distribution`. There are many kind of probability distribution, but for this post we will only consider on 3 kind of it: uniform distribution, binomial distribution, and beta distribution.

### Uniform Distribution

A uniform distribution means that all possible outcomes are equally likely to occur. Example of this is when we flip a coin or throwing a dice. To illustrate this, imagine you are throwing a dice for 1,000,000 times and count the number of each side of the dice that occur. All side of dice will eventually have the same amount of chance to happen.


```r
# Sampling
set.seed(123)
flip_coin <- sample(x = 1:6, size = 1e6, replace = T)

data.frame(flip_coin) %>% 
  ggplot(aes(flip_coin)) +
  geom_bar(fill = "firebrick4") +
  scale_x_continuous(breaks = 1:6) +
  labs(x = "Side of the Dice",
       title = "Frequency of Each Side of Dice after 1,000,000 Trials"
       )
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-10-1.png" width="672" style="display: block; margin: auto;" />

We can directly get and visualize the probability for each event using `dunif()`.


```r
fun_unif <- function(x) dunif(x, min = 0, max = 6)

data.frame(x = 0:6) %>% 
  mutate(y = fun_unif(x)) %>% 
  ggplot(aes(x, y))  +
  geom_col(fill = "dodgerblue4") +
  scale_x_continuous(breaks = 0:10) +
  labs(x = "Side of Die",
       y = "Probability",
       title = "Uniform Distribution of Throwing a Dice"
       )
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-11-1.png" width="672" style="display: block; margin: auto;" />

### Binomial Distribution

Different probability distribution can be used for different context. In coin flipping case or other binary outcome trials, there is the famous `binomial distribution`. Let's imagine you are flipping ten coins consecutively. From the ten flips, what is the probability that you will get 5 heads and 5 tails? What is the probability of landing 7 heads and 3 tails? Such question can be answered by the binomial distribution.

Let's do some simulation. In this part, we will create a simulation of 10 coin flips and calculate how many heads that we get. We assume that the coin is fair with both head and tail are equally likely to come out. After we calculate how many heads we get, we do another 10 coin flips experiment. We repeat this process for 10,000 times. After the experiment end, how many of the trials resulted in 5 heads, how many resulted in 3 heads, etc. The following figure show the result of the simulations.


```r
# Simulating 10 coin flips trials for 10,000 times
set.seed(123)
flip_coin <- map(1:1e4, 
                 function(x) sample(x = c("Head", "Tail"), size = 10, replace = T)
                 ) %>% 
  map(table) %>% # Get the number of head and tail in each experiment
  map_dbl(function(x) x[1]) # Get number of head in each experiment

data.frame(flip_coin) %>% 
  ggplot(aes(flip_coin)) +
  geom_bar(fill = "firebrick4") +
  scale_x_continuous(breaks = 1:10) +
  labs(x = "Number of Head in 10 Flips",
       title = "How Many Head You Can Get from 10 Coin Flips?",
       subtitle = "\nCollected after 10,000 experiments. Each experiment consist of 10 coin flips."
       )
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-12-1.png" width="672" style="display: block; margin: auto;" />

AS we can see, with our assumption that we have fair coin with 50% probability of landing a head, we will more likely to get 5 heads out of 10 flips in most of the trials and we are less likely to get only 1 or even 9 heads out of 10.

The binomial distribution can be represented with the following formula:

$$
Binomial(k, n, p) = \begin{pmatrix} n \\ k \end{pmatrix}\times p^k\times (1-p)^{n-k}
$$

Description

- `k`: Number of success (for this example, landing a head)
- `n`: Number of sample
- `p`: Probability of a single success

We can generate the probability distribution directly using `dbinom()`. For example, if we want to get the probability of landing 5 head from 10 trials if probability of getting a head is 0.5 is as follows.


```r
dbinom(5, 10, 0.5)
```

```
#> [1] 0.2460938
```

We can visualize the probability for each event.


```r
fun_binom <- function(x) dbinom(x, size = 10, prob = 0.5)

data.frame(x = 0:10) %>% 
  mutate(y = fun_binom(x)) %>% 
  ggplot(aes(x, y))  +
  geom_col(fill = "dodgerblue4") +
  scale_x_continuous(breaks = 0:10) +
  labs(x = "Number of Head in 10 Flips",
       y = "Probability",
       title = "Binomial Distribution of 10 Coin Flips"
       )
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-14-1.png" width="672" style="display: block; margin: auto;" />

### Beta Distribution

Often, we are more interested in finding the probability or the value of certain parameters instead of simulating the outcome. If we know the probability of a single coin flip, we can find the probability of landing 5 heads in 10 trials. But what if all we have is the data? Can we determine the probability of a coin flip if all we know is that it give us 8 heads in 10 trials? That's the question statistics want to solve, finding the value of a parameter given the data. 

Consider the following case. Your friend said that he is super smart and able to answer any questions that you throw at him. To back up his claim, he ask you to ask him anything from your favorite meal to the MVP of the NBA in 1971. You give him 10 different questions and he is able to answer 8 of them. After you know the result, do you really  believe that your friend is crazy smart or just being lucky? In this case, you know the outcome of the trial but you don't know what is the probability of him being correct. We can try to answer that questions by try different probabilities using grid search. We try to test 100 different parameter p (probability of being superhuman) from 0 to 1.


```r
prob_value <- seq(0, 1, 0.01)

fun_binom <- function(x) dbinom(8, size = 10, prob = x)

data.frame(x = prob_value) %>% 
  mutate(y = map_dbl(x, fun_binom)) %>% 
  ggplot(aes(x,y)) +
  geom_point()
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-15-1.png" width="672" style="display: block; margin: auto;" />

The highest probability for the parameter p seems to be somewhere around 0.8. For this case, we still able to figure out which value of parameter p that will give us the highest possible result. However, we still have many possible value of parameter p that has not been tested yet. We don't have a finite number of parameter p, rather, we have an infinite number of p ranging from 0 to 1 that need to be tested. Thus, using a discrete probability distribution like the binomial distribution can only help a little. Another problem is that since not all possible values are calculated, the total value from the 100 different parameters that we have calculated is not sum up to 1, which means that we violated the rule of probability. We need a proper probability distribution to handle such problem. 

Let me introduce the beta distribution. Unlike the binomial distribution, which breaks up the values into discrete range, the beta distribution represents a continuous range of values that will allows us to represent our infinite number of possible hypotheses[^3]. Beta distribution is useful in many different scenarios. The formula for the beta distribution is as follows:

$$
Beta(p, \alpha, \beta) = \frac{p^{\alpha -1}\times (1-p)^{\beta-1}}{beta(\alpha, \beta)}
$$

Description in context of binary outcome:

- `p`: Probability of an event. This corresponds to our different hypotheses for the possible probabilities for the coin flip. 
- `\(\alpha\)`: How many times we observe an event we care about, such as getting 5 heads from 10 trials.
- `\(\beta\)`: How many times the event we care about did not happen. 

The total number of trials is `\(\alpha\)` + `\(\beta\)`. This is different than the binomial distribution, where we have `k` observations we are interested in and a finite number of `n` total trials. 

We will once again draw the previous superhuman guessing case with beta distribution. With beta distribution, we get the smooth curve that display the density for each parameter p of your friend being superhuman, which peaked at around 0.85. This means that the `binomial(8, 10, 0.85)` has the best chance at explaining your friend amazing guess.


```r
fun_beta <- function(x) dbeta(x, 8, 2)

data.frame(x = seq(0, 1, length.out = 100)) %>% 
  ggplot() +
  geom_function(fun = fun_beta) +
  scale_x_continuous(breaks = seq(0,1,0.1)) +
  labs(title = "Beta(8,2)",
       x = "Probability of Your Friend Being Superhuman",
       y = "Density")
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-16-1.png" width="672" style="display: block; margin: auto;" />

# Element of Bayesian Statistics

Now let's go back to the element of the bayesian statistics. To help us understand this part, we will use an example of the classic coin flipping trials. After we understand how probability distribution workds, espescially the beta distribution, it will be easier to understand the concept.

## Prior

The prior represent the strength of our belief before we see the data. You can have a strong belief that the probability should be somewhere around certain values. You can also have a weak belief about any values so you consider that all values of the probability are equally possible. A prior is said to be `weak` prior when the probability is widely distributed while a `strong` prior tend to concentrated in certain areas.

Suppose we are interested in doing 10 coin flip and see if the coin is fair (have 50:50 chance to land a head) and not being manipulated. Before we do the experiment, we might have some belief about the possible values of the probability of landing a head. Then we convert our prior belief into a probability distribution. The most conservative prior is when we consider all probabilities between 0 and 1 are equally possible. This kind of prior is considered as a `weak` prior and can be illustrated by the `Beta(1,1)` distribution as follows.


```r
fun_beta <- function(x) dbeta(x, 1, 1)

data.frame(x = seq(0, 1, length.out = 100)) %>% 
  ggplot() +
  geom_function(fun = fun_beta) +
  labs( y = "Density", x = "Probability of Landing a Head")
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-17-1.png" width="672" style="display: block; margin: auto;" />

As you can see, this is a perfectly straight line, so that all outcomes are then equally likely. This prior is called a `noninformative prior` because it is used to state that we have no clues at all about the probability of landing a head and we consider that it is possible to land a head with 1%, 50%, or 99% chance. The idea of using a noninformative prior is that we can add a prior to help smooth out our estimate, but that prior is not biased toward any particular outcome. Even though the prior seems to be a good prior and a fair prior because there is no subjective belief in it, most statisticians recommend not to use this prior. This is because unless we are completely clueless about our experiment or state of the world, we rarely find things that are truly random and all of its outcome are equally likely. For this coin flipping example, you must have a strong believe that a coin should have a fair output, with the probability of landing a head is 50:50 with the outcome of the tail. This is because we have years of experience in playing with the coin and study them during our college time in statistics class. It would be absurd to believe that there is almost 0% probability of landing a head before we see the data.

> The best priors are backed by data, and there is never really a true “fair” prior when you have a total lack of data. Everyone brings to a problem their own experiences and perspective on the world. The value of Bayesian reasoning, even when you are subjectively assigning priors, is that you are quantifying your subjective belief.
<footer> Kurt, 2019</footer>

Now that we understand that a `noninformative prior` is potentially dangerous for us, we can try to create another weak prior but with some of our personal belief. We can use `Beta(3,3)` for example to state that we believe the chance of landing a head and landing a tail is equal so the probability of landing a head would be somewhere around 0.5 but we still allow other possibility to happen. If we are strongly believe that a coin flip should result in 50:50 chance of landing a head, we can use stronger prior by using `Beta(50,50)`. Increasing the shape parameter of the beta distribution of the prior will give us a stronger prior. A strong prior will require more data to be dismissed.

We can visualize all of the mentioned priors. As you can see, the stronger prior will give us a more concentrated probability distribution compared to the weaker one.


```r
fun_beta2 <- function(x) dbeta(x, 3, 3)
fun_beta3 <- function(x) dbeta(x, 50, 50)

data.frame(x = seq(0, 1, length.out = 1e4)) %>% 
  mutate("Beta(1,1)" = map_dbl(x, fun_beta),
         "Beta(3,3)" = map_dbl(x, fun_beta2),
         "Beta(50,50)" = map_dbl(x, fun_beta3)
         ) %>% 
  pivot_longer(-x, names_to = "variant") %>% 
  ggplot(aes(x = x, y = value)) +
  geom_line(aes(color = variant)) +
  labs(x = "Probability of Landing a Head", y = "Density",
       title = "Different Kind of Prior",
       color = NULL) 
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-18-1.png" width="672" style="display: block; margin: auto;" />

## Likelihood

The likelihood tells us how likely the data is given our belief. Suppose we do 10 coin flipping experiment and get the following result. From 10 flipping coins we acquire 7 `head` and 3 `tail`.

$$
Tail\ Head\ Head\ Tail\ Head\ Head\ Tail\ Head\ Head\ Head
$$


```r
result <- c("tail", "head", "head", "tail", "head", "head", "tail", "head", "head", "head")

table(result)
```

```
#> result
#> head tail 
#>    7    3
```

Let's visualize the likelihood of us seeing 10 trials with 7 head and 3 tails given our belief of the probability of getting a head. Given our parameter of `p` (probability of landing a head), how likely it is to get  head and 3 tails?


```r
fun_obs <- function(x) dbeta(x, 7, 3)

data.frame(x = seq(0, 1, length.out = 100)) %>% 
  ggplot() +
  geom_function(fun = fun_obs) +
  labs( y = "Density", x = "Probability of Landing a Head (p)",
        title = "Likelihood of Having 7 Heads and 3 Tails Given p"
        )
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-20-1.png" width="672" style="display: block; margin: auto;" />

Higher density shows that it is more likely that we will see 7 head and 3 tails in 10 trials if the probability of landing a head is around 0.75. We will be less likely to see such even of 7 heads and 3 tails if the probability of landing a head is 0.3 and it is even impossible to see such event if the probability of landing a head is close to 1.

## Posterior and Parameter Estimation

We have understand what a prior and what a likelihood is. A prior is our belief before see the data, a likelihood reflect how likely will we get the data given our belief. Let's put the prior and the likelihood in the same plot as follows. So how do we calculate the posterior distribution from this information?


```r
data.frame(x = seq(0, 1, length.out = 1e4)) %>% 
  mutate("Prior Beta(3,3)" = map_dbl(x, fun_beta2),
         Likelihood = map_dbl(x, fun_obs)
         ) %>% 
  pivot_longer(-x, names_to = "variant") %>% 
  ggplot(aes(x = x, y = value, fill = variant)) +
  geom_ribbon(aes(xmin = x, xmax = x, ymin = 0, ymax = value),
              alpha = 0.75) +
  labs(x = "Probability of Landing a Head", y = "Density",
       fill = NULL) 
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-21-1.png" width="672" style="display: block; margin: auto;" />

Recall the Bayes formula. On the right side, the numerator shows the prior and the likelihood. So what is the `\(P(B)\)` on the denominator?

$$
P(A|B) = \frac{P(A)\times P(B|A)}{P(B)}
$$

In the previous use case of Covid testing, the `\(P(B)\)` represent the probability of getting a positive result from the test. This denominator is used to normalize the product of the prior and likelihood so the range of all values will be a valid probability with a range of 0-1. If we have all of these pieces of information, we can calculate exactly how strongly we should believe in our hypothesis given the data we have observed. However, we rarely able to get this information and it is pretty hard to compute. But, there is a way. given that we have only a likelihood and a prior, we can use the proportional form of Bayes theorem. The posterior is proportional to the product of the prior and the likelihood. Using this proportional form of Bayes theorem means that our posterior distribution does not necessarily sum to 1. However, this form is enough if all we care is comparing different hypothesis, which is another topic for another day.

$$
Posterior \propto likelihood\times prior
$$

Lucky for us, there is an easy way to combine beta distributions that will give us a normalized posterior when all we have is the likelihood and the prior. You can simply just add the `\(\alpha\)` or the first shape and the `\(\beta\)` or the second shape of the beta distribution of both priors and likelihood. Because this is so simple, working with the beta distribution is very convenient for Bayesian statistics.

$$
Beta(\alpha_{posterior}, \beta_{posterior}) = Beta(\alpha_{prior} + \alpha_{likelihood}, \beta_{prior} + \beta_{likelihood})
$$

For our coin flipping test, with the prior of `Beta(3,3)` we willl get the following posterior:

$$
Beta(\alpha_{posterior}, \beta_{posterior}) = Beta(3 + 7, 3 + 3) = Beta(10, 6)
$$

We can visualize the posterior probability distribution to see which value of parameter p (probability of landing a head) is mot likely to be.


```r
fun_posterior <- function(x) dbeta(x, 10, 6)

data.frame(x = seq(0, 1, length.out = 100)) %>% 
  ggplot() +
  geom_function(fun = fun_posterior) +
  labs( y = "Density", x = "Probability of Landing a Head (p)",
        title = "Posterior Probability"
        )
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-22-1.png" width="672" style="display: block; margin: auto;" />

We will use Monte-Carlo simulation to simulate the probability of landing a head in 1,000,000 trials given the beta distribution of our posterior. By doing simulation, the probability values with higher chance to appear will occur more frequently. Sampling from the posterior is a good way to get the estimated parameter.

> A Monte Carlo simulation is any technique that makes use of random sampling to solve a problem. Each sample is chosen based on its probability in the distribution so that samples in a high-probability region will appear more frequently.


```r
n_trial <- 1e6

set.seed(123)
sample_post <- rbeta(n = n_trial, 3+7, 3+3)
```

Bayesian loves to work with distribution instead of a single point value. To summarize our posterior distribution, we will use something called `Credibility Interval`. A credibility interval is a lower and upper bound of values describing a range of high probability, usually 95%, 99%, or 90%. The choice of the percentage is arbitrary and there is no set of rule for it. Most of the time it is just a consensus. In the following example, 95% credibility interval is just our way of saying that there is 95% chance that probability of landing a head is between 0.383 and 0.836.


```r
cat("95% Credibility Interval of the Posterior Probability\n")
```

```
#> 95% Credibility Interval of the Posterior Probability
```

```r
quantile(sample_post, probs = c(0.025, 0.5, 0.975))
```

```
#>      2.5%       50%     97.5% 
#> 0.3834289 0.6304564 0.8365127
```

`Credibility Intervals` is different from the frequentist approach of `Confidence Intervals`. 

Let's visualize the posterior distribution with the 95% credibility interval from the noninformative prior `Beta(1,1)`.


```r
data.frame(x = seq(0, 1, length.out = 1e4)) %>% 
  mutate(post =  dbeta(x, 3+7, 3+3)) %>% 
  ggplot(aes(x, post)) +
  geom_ribbon(aes(xmin = x, xmax = x, ymin = 0, ymax = post),
              fill = "dodgerblue4", alpha = 0.75) +
  geom_vline(xintercept = quantile(sample_post, probs = c(0.025, 0.975)),
             lty = "dashed") +
  labs(title = "Posterior Probability Distribution",
       subtitle = "With Prior Beta(3,3)",
       fill = NULL,
       y = "Density", x = "Probability") +
  scale_x_continuous(breaks = seq(0, 1, 0.1))
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-25-1.png" width="672" style="display: block; margin: auto;" />

You can also see that the posterior probability is just the product between the prior and the likelihood.


```r
data.frame(x = seq(0, 1, length.out = 1e4)) %>% 
  mutate(Prior = map_dbl(x, fun_beta2),
         Likelihood = map_dbl(x, fun_obs),
         Posterior = map_dbl(x, function(x) dbeta(x, 3+7, 3+3) )
         ) %>% 
  pivot_longer(-x, names_to = "variant") %>% 
  ggplot(aes(x = x, y = value, fill = variant)) +
  geom_ribbon(aes(xmin = x, xmax = x, ymin = 0, ymax = value),
              alpha = 0.75) +
  labs(x = "Probability of Landing a Head", y = "Density",
       title = "Posterior is the Product of Prior and Likelihood",
       subtitle = "With Prior Beta(3,3)",
       fill = NULL) 
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-26-1.png" width="672" style="display: block; margin: auto;" />


```r
data.frame(x = seq(0, 1, length.out = 1e4)) %>% 
  mutate(Prior = map_dbl(x, fun_beta3),
         Likelihood = map_dbl(x, fun_obs),
         Posterior = map_dbl(x, function(x) dbeta(x, 50+7, 50+3) )
         ) %>% 
  pivot_longer(-x, names_to = "variant") %>% 
  ggplot(aes(x = x, y = value, fill = variant)) +
  geom_ribbon(aes(xmin = x, xmax = x, ymin = 0, ymax = value),
              alpha = 0.75) +
  labs(x = "Probability of Landing a Head", y = "Density",
       title = "Posterior is the Product of Prior and Likelihood",
       subtitle = "With Prior Beta(50,50)",
       fill = NULL) 
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-27-1.png" width="672" style="display: block; margin: auto;" />

We can visualize the posterior probability for different priors. The posterior distribution will consider both priors and the observed data. As you can see, if you have a weak prior or even uninformative prior with `Beta(1,1)`, the result will be more influenced by the data, with the 95% credibility interval located between 0.389 to 0.891 with the median is located at 0.676, close to 0.7 or 70%. With stronger prior, the data become less relevant because with the small sample we have is not enough to outweigh the priors, as we can see at the prior `Beta(50,50)` that only slightly tilted to the right side but still pretty much concentrated between 0.384 to 0.782. Strong priors should only be used if you have a strong argument to justify it.


```r
fun_beta <- function(x) dbeta(x, 1+7, 1+3)
fun_beta2 <- function(x) dbeta(x, 3+7, 3+3)
fun_beta3 <- function(x) dbeta(x, 50+7, 50+3)

data.frame(x = seq(0, 1, length.out = 1e4)) %>% 
  mutate("Beta(8,4)" = map_dbl(x, fun_beta),
         "Beta(10,6)" = map_dbl(x, fun_beta2),
         "Beta(57,53)" = map_dbl(x, fun_beta3)
         ) %>% 
  pivot_longer(-x, names_to = "variant") %>% 
  ggplot(aes(x, value, fill = variant)) +
  geom_ribbon(aes(xmin = x, xmax = x, ymin = 0, ymax = value),
              alpha = 0.75) +
  scale_fill_brewer(palette = "Dark2") +
  labs(title = "Posterior Probability Distribution",
       fill = NULL,
       y = "Density", x = "Probability")
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-28-1.png" width="672" style="display: block; margin: auto;" />

Here we can compile the 95% credibility interval from each posterior distribution.


```r
set.seed(123)
sample_b1 <- rbeta(n = n_trial, 1+7, 1+3)
sample_b2 <- rbeta(n = n_trial, 3+7, 3+3)
sample_b3 <- rbeta(n = n_trial, 50+7, 50+3)

quantile(sample_b1, probs = c(0.025, 0.5, 0.975)) %>% 
  as.data.frame() %>% 
  bind_cols(
    quantile(sample_b2, probs = c(0.025, 0.5, 0.975)) %>% 
      as.data.frame(),
    quantile(sample_b2, probs = c(0.025, 0.5, 0.975)) %>% 
      as.data.frame()
  ) %>% 
  setNames(c("Beta(8,4)", "Beta(10, 6)", "Beta(57, 53)"))
```

```
#>       Beta(8,4) Beta(10, 6) Beta(57, 53)
#> 2.5%  0.3897559   0.3836042    0.3836042
#> 50%   0.6763476   0.6304491    0.6304491
#> 97.5% 0.8908559   0.8369329    0.8369329
```

We have observed with a small sample of data, the prior still have some effect toward the posterior. However, as you have more and more data, the prior become less relevant. The following figure shows the posterior distribution with the same prior but now we have a result of 1,000 trials with 700 head and 300 tails. Observe how the posterior distribution become very close to each other and favor more toward the observed data. That's how we intuitively think in real life. With small data, we might have some doubt since the data contradicts our previous belief. However, as more data comes in, our belief start to change and now we believe more toward the new information from the data. Stronger prior believe will require more data to convince.


```r
fun_beta <- function(x) dbeta(x, 1+700, 1+300)
fun_beta2 <- function(x) dbeta(x, 3+700, 3+300)
fun_beta3 <- function(x) dbeta(x, 50+700, 50+300)

data.frame(x = seq(0, 1, length.out = 1e4)) %>% 
  mutate("Beta(701, 301)" = map_dbl(x, fun_beta),
         "Beta(703, 303)" = map_dbl(x, fun_beta2),
         "Beta(750, 350)" = map_dbl(x, fun_beta3)
         ) %>% 
  pivot_longer(-x, names_to = "variant") %>% 
  ggplot(aes(x, value, fill = variant)) +
  geom_ribbon(aes(xmin = x, xmax = x, ymin = 0, ymax = value),
              alpha = 0.75) +
  scale_fill_brewer(palette = "Dark2") +
  facet_wrap(~variant, nrow = 3) +
  labs(title = "Posterior Probability Distribution",
       fill = NULL,
       y = "Density", x = "Probability") 
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-30-1.png" width="672" style="display: block; margin: auto;" />

# A/B Testing

A/B testing or AB testing is a way for us to compare which would give better result between two different treatment (treatment A and treatment B). AB testing is very common in the field of information technology, especially the one that related to web development, user experience and digital marketing. For example, Facebook has changed its landing page multiple times in the past. In the following figure, we have two types of Facebook landing pages. As the product manager of Facebook, you want to get as many people to join and create new accounts for Facebook. You can use AB testing to see which landing page that will give you more conversion rate.

<center> ![](/img/ab_test/landing.png) </center> 

By designing propr A/B testing workflow, we can achieve better result and can please the customer better. The A/B testing start by defining what business goal you want to achieve. For example, as an employee of youtube we want people to spend more time and increase their watch time in youtube. We want to test the new variant of the youtube mobile app that has new recommendation algorithm. Then, the visitor of youtube will be randomly directed into the old app while the rest will be directed into the new variant. This randomization is done in order to avoid bias in visitor selection. After several times we will monitor the watch time of each variant, let's say for a week. Then, we will evaluate by using hypothesis testing which variant has better performance and how good is it. Is it worth the risk? Is it only slightly better or even outcompete the old variant? All of that question will be answered using the Bayesian statistics.

<center> ![](/img/ab_test/ab_test.png) </center> 

There are several benefit of using Bayesian statistics instead of the traditional hypothesis testing (called Null Hypothesis Significance Testing or NHST)[^4]:

- Easier to interpret and communicate
- Can stop the test early if there is a clear winner or run it for longer if you need more samples
- The use of prior information outside the observed experiment data

## Case Study: Udacity Landing Page

The study case is from Udacity's Course and the data is acquired from [erdiolmezogullari](https://github.com/erdiolmezogullari/ml-ab-testing). The data contain A/B testing result of creating two different landing page layout. We will check if the new variant of the landing page can give us better conversion rate.


```r
data_ab <- read.csv("data_input/ab_data.csv")

glimpse(data_ab)
```

```
#> Rows: 294,478
#> Columns: 5
#> $ user_id      <int> 851104, 804228, 661590, 853541, 864975, 936923, 679687, 7…
#> $ timestamp    <chr> "2017-01-21 22:11:48.556739", "2017-01-12 08:01:45.159739…
#> $ group        <chr> "control", "control", "treatment", "treatment", "control"…
#> $ landing_page <chr> "old_page", "old_page", "new_page", "new_page", "old_page…
#> $ converted    <int> 0, 0, 0, 0, 1, 0, 1, 0, 1, 1, 0, 0, 0, 0, 0, 1, 0, 1, 0, …
```

Data Description:

- `user_id`: Unique ID of the user
- `timestamp`: The recorded time when the user visit the landing page
- `group`: Whether the user belong to control group or the treatment group
- `landing_page`: Which landing page the user see
- `converted`: Whether the user converted or subscribed to the course.

### Data Preprocessing

For the start, we will do some data preprocessing before analyzing the data.


```r
data_ab <- data_ab %>% 
  mutate(timestamp = ymd_hms(timestamp),
         hour = hour(timestamp),
         day = wday(timestamp, label = T),
         landing_page = str_replace(landing_page, "_", " ") %>% 
           str_to_title()
         ) %>% 
  mutate_if(is.character, as.factor)

summary(data_ab)
```

```
#>     user_id         timestamp                         group       
#>  Min.   :630000   Min.   :2017-01-02 13:42:05   control  :147202  
#>  1st Qu.:709032   1st Qu.:2017-01-08 02:06:48   treatment:147276  
#>  Median :787934   Median :2017-01-13 13:21:07                     
#>  Mean   :787974   Mean   :2017-01-13 13:40:10                     
#>  3rd Qu.:866912   3rd Qu.:2017-01-19 01:43:51                     
#>  Max.   :945999   Max.   :2017-01-24 13:41:54                     
#>                                                                   
#>    landing_page      converted           hour        day       
#>  New Page:147239   Min.   :0.0000   Min.   : 0.00   Sun:40436  
#>  Old Page:147239   1st Qu.:0.0000   1st Qu.: 6.00   Mon:46060  
#>                    Median :0.0000   Median :12.00   Tue:47777  
#>                    Mean   :0.1197   Mean   :11.51   Wed:40122  
#>                    3rd Qu.:0.0000   3rd Qu.:17.00   Thu:39739  
#>                    Max.   :1.0000   Max.   :23.00   Fri:40159  
#>                                                     Sat:40185
```

Let's check the frequency of each control group and the landing page presented. The user who belong to `control` group should get the `old page` landing page while the `treatment` group should get the `new page`.


```r
table("Landing Page" = data_ab$landing_page, "Group" = data_ab$group)
```

```
#>             Group
#> Landing Page control treatment
#>     New Page    1928    145311
#>     Old Page  145274      1965
```

Looks like some users are directed in the wrong landing page, so we will filter the data that get the correct landing page only.


```r
data_ab <- data_ab %>% 
  filter((group == "control" & landing_page == "Old Page") |
           (group == "treatment" & landing_page == "New Page")
           )
```

Let's continue by checking if there are any missing values in our data.


```r
data_ab %>% 
  is.na() %>% 
  colSums()
```

```
#>      user_id    timestamp        group landing_page    converted         hour 
#>            0            0            0            0            0            0 
#>          day 
#>            0
```

### Exploratory Data Analysis

Check how many number of users in each group to see if the group is balanced.


```r
table("Group" = data_ab$group)
```

```
#> Group
#>   control treatment 
#>    145274    145311
```

Let's check the conversion rate by hour and weekdays to see if there are any specific time where the conversion rate increase. 


```r
data_ab %>% 
  group_by(day, hour, landing_page) %>% 
  summarise(converted = mean(converted)) %>% 
  ungroup() %>% 
  ggplot(aes(hour, day, fill = converted)) +
  geom_tile(color = "white") +
  scale_fill_binned(low = "firebrick4", high = "lightyellow", labels = percent_format(accuracy = 1)) +
  scale_x_continuous(breaks = seq(0,22,2)) +
  facet_wrap(~landing_page, nrow = 2) +
  labs(y = NULL, title = "Convertion Rate", fill = NULL) +
  theme(legend.key.width = unit(15, "mm"))
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-37-1.png" width="672" style="display: block; margin: auto;" />

It looks like there is no apparent change between the old page variant and the new page variant based on the heatmap. However, we will test it using the Bayesian hypothesis testing.

### Hypothesis Testing

First we collect all the conversion result from each variant.


```r
variant_a <- data_ab %>% 
  filter(landing_page == "Old Page") %>% 
  pull(converted)

variant_b <- data_ab %>% 
  filter(landing_page == "New Page") %>% 
  pull(converted)
```

#### Prior

Next, we need to define our prior for the conversion rate. This prior must not come from the observed data from the experiment but instead should come from our believe, which can be based on some historical data or other external information source, such as from online survey. Since we don't have any data before do the AB testing, let's say that we believe that our conversion rate should be around 10%, for every 100 people that land on our website, 10 people will subscribe for the content.  We then translate this into a prior probability distribution. We can use a weak prior since we are not really sure that our landing page could achieve those number and perhaps there are other factors that will cause a better or worse conversion. We will use `Beta(2, 8)` for our prior probability distribution. Remember that when we use `beta` distribution for a binary trial, the first shape represent the number of success or converted while the second is the number of not success.


```r
fun_beta <- function(x) dbeta(x, 2, 8)

data.frame() %>% 
  ggplot() +
  geom_function(fun = fun_beta) +
  labs(x = "Conversion Rate", y = "Density")
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-39-1.png" width="672" style="display: block; margin: auto;" />

#### Observed Data

Next, we can check the conversion rate based on the observed data.


```r
data_agg <- data_ab %>% 
  group_by(group, landing_page) %>% 
  summarise(total_visit = n(),
            conversion = sum(converted),
            not_converted = total_visit - sum(conversion),
            conversion_rate = conversion / total_visit
            )

data_agg
```

```
#> # A tibble: 2 x 6
#> # Groups:   group [2]
#>   group     landing_page total_visit conversion not_converted conversion_rate
#>   <fct>     <fct>              <int>      <int>         <int>           <dbl>
#> 1 control   Old Page          145274      17489        127785           0.120
#> 2 treatment New Page          145311      17264        128047           0.119
```

#### Posterior

##### Parameter Estimation

Now we can calculate the posterior probability for each variant by incorporating the information from the prior and the observed data using the beta distribution.

$$
Beta(\alpha_{posterior}, \beta_{posterior}) = Beta(\alpha_{prior} + \alpha_{likelihood}, \beta_{prior} + \beta_{likelihood})
$$

We will visualize the density curve for the posterior probability of each variant. The distribution represent all possible values of conversion rate for each variant. The higher the density for a single parameter of conversion rate, the more likely that those conversion rate are the conversion rate of the variant. For example, in the following figure we can see that the conversion rate of the Old Page variant is more likely to be somewhere around 12% to 12.25% because the density is higher in those region.


```r
prior_alpha <- 2
prior_beta <- 8

fun_beta_old <- function(x) dbeta(x, prior_alpha + data_agg$conversion[1], prior_beta + data_agg$not_converted[1])
fun_beta_new <- function(x) dbeta(x, prior_alpha + data_agg$conversion[2], prior_beta + data_agg$not_converted[2])

data.frame(x = seq(0, 1, length.out = 1e4)) %>% 
  mutate(old = map_dbl(x, fun_beta_old),
         new = map_dbl(x, fun_beta_new)
         ) %>% 
  pivot_longer(-x, names_to = "variant") %>% 
  mutate(variant = ifelse(variant == "old", "Old Page", "New Page")) %>% 
  ggplot(aes(x = x, y = value)) +
  geom_ribbon(aes(fill = variant, xmin = x, xmax = x, ymin = 0, ymax = value),
              alpha = 0.75) +
  labs(x = "Conversion Rate", y = "Density",
       title = "Posterior Probability of Conversion Rate",
       fill = NULL) +
  scale_x_continuous(limits = c(0.115, 0.125), 
                     labels = percent_format())
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-41-1.png" width="672" style="display: block; margin: auto;" />

Clearly, our data suggests that variant A (the old landing page) is superior, in that it garners a higher conversion rate. However, we know that the true conversion rate is one of a range of possible values. We can also see here that there is an overlap between the possible true conversion rates for the old variant and the new variant. What if we were just unlucky in our variant B (new landing page) responses, and variant B's true conversion rate is in fact much higher? What if we were also just lucky with variant A, and its conversion rate is in fact much lower? It is easy to see a possible world in which variant A is actually the better variant, even though it did worse on our test. So the real question is: how sure can we be that variant B is the better variant? 

To answer such question, we will simulate the posterior using the Monte-Carlo simulation with 1,000,000 trials. We are going to randomly sample from the two distributions, where each sample is chosen based on its probability in the distribution so that samples in a high-probability region will appear more frequently.

We can see the distribution of the posterior in each variant. The values represent the estimated parameter of conversion rate in each variant. If you want to take a single value for the estimated parameter, you can choose the median or the 50% percentile of the conversion rate. For variant A, that would be around 12.04% while for variant B is around 11.88%.


```r
n_trial <- 1e6

set.seed(123)
sample_old <- rbeta(n = n_trial, prior_alpha + data_agg$conversion[1], prior_beta + data_agg$not_converted[1])
sample_new <- rbeta(n = n_trial, prior_alpha + data_agg$conversion[2], prior_beta + data_agg$not_converted[2])

cat("Posterior Probability Distribution of Variant A (Old Page)\n")
```

```
#> Posterior Probability Distribution of Variant A (Old Page)
```

```r
quantile(sample_old)
```

```
#>        0%       25%       50%       75%      100% 
#> 0.1162741 0.1198146 0.1203893 0.1209656 0.1246456
```

```r
cat("\nPosterior Probability Distribution of Variant B (New Page)\n")
```

```
#> 
#> Posterior Probability Distribution of Variant B (New Page)
```

```r
quantile(sample_new)
```

```
#>        0%       25%       50%       75%      100% 
#> 0.1147905 0.1182400 0.1188111 0.1193842 0.1228733
```

It is better to present the parameter as a probability distribution rather than a single value because by then, you can determine the credible interval or the most likely range of value of the conversion rate. The 90% credible interval for variant A can be interpreted as *90% of all possible values of conversion rate are located between 11.89% and 12.18%* while for variant B it is somewhere between 11.74% and 12.02%.


```r
cat("90% Credible Interval of Variant A (Old Page)\n")
```

```
#> 90% Credible Interval of Variant A (Old Page)
```

```r
quantile(sample_old, probs = c(0.05, 0.95))
```

```
#>        5%       95% 
#> 0.1189921 0.1217993
```

```r
cat("\n90% Credible Interval of Variant B (New Page)\n")
```

```
#> 
#> 90% Credible Interval of Variant B (New Page)
```

```r
quantile(sample_new, probs = c(0.05, 0.95))
```

```
#>        5%       95% 
#> 0.1174202 0.1202123
```

##### Improvement

To see how many times variant A produce better conversion rate than B, we can use the ratio between the total number of variant A beat variant B divided by the number of trials. This value is equivalent with the *p-value* that you may have encounter when using the NHST method. However, this value is easier to interpret and pretty intuitive compared to p-value, which mostly has been misunderstood. 


```r
p_a_superior <- sum(sample_old > sample_new) / n_trial
p_a_superior
```

```
#> [1] 0.905237
```

What we see here is that in 90 percent of the 1,000,000 trials, variant A was superior. We can imagine this as looking at 1,000,000 possible worlds. Based on the distribution of possible conversion rates for each variant, in 90 percent of the worlds variant A was the better of the two. This result shows that we have a pretty strong belief based on our prior belief and the observed data that the old variant or the old landing page is the better variant. It turns out that variant B is no better than our old variant A.

We can calculate how much better variant A compared to the variant B in terms of the percent of improvement. Is variant A 20% better? Or is it 10% better? To answer such question, we can use the following formula:

$$
Improvement\ A\ over\ B = \frac{A-B}{B}
$$


```r
# Calculate the improvement
sample_diff <- (sample_old - sample_new) / sample_new

# Get the density
dens_diff <- density(sample_diff) 

# Visualization
df_diff <- data.frame(x = dens_diff$x,
                      y = dens_diff$y) 

df_diff %>% 
  mutate(type = "full") %>% 
  bind_rows(
    df_diff %>% 
      filter(x < 0) %>% 
      mutate(type = "part")
    ) %>% 
  ggplot(aes(x,y, fill = type)) +
  geom_ribbon(aes(xmin = x, xmax =x , ymin = 0, ymax =  y),
              alpha = 0.75, show.legend = F)  +
  geom_text(data = data.frame(),
            aes(x = 0.015, y = 15, label = percent(p_a_superior, accuracy = 0.01)),
            inherit.aes = F, size = 6
            ) +
  scale_fill_manual(values = c( "firebrick4", "skyblue2")) +
  scale_x_continuous(labels = percent_format(accuracy = 0.01)) +
  labs(subtitle = "Red Area Shows Percent of Improvement from Variant A (Old Page) over Variant B (New Page)",
       x = "(A-B)/B", y = "Density"
       )
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-45-1.png" width="672" style="display: block; margin: auto;" />

The result is the same with our previous step. Here we still can see that the variant A is better than variant B in 90.52% of all trials. The x-axis shows the percentage of how much A is better than B in term of the conversion rates. Most of the times variant A is better but sometimes variant B is also better. 

We can create the credible interval of the percentage. Based on the 90% credibility interval, improvement of variant A over variant B is ranging from -0.33% to 3.02%.


```r
# Credible Interval

cat("90% Credible Interval of (A - B)/B\n")
```

```
#> 90% Credible Interval of (A - B)/B
```

```r
quantile(sample_diff, probs = c(0.05, 0.95))
```

```
#>           5%          95% 
#> -0.003346881  0.030199500
```

We can also use the cumulative probability for more analysis. For example, using the cumulative probability curve we can expect that there is 25% chance that variant A is 2% better than variant B. 


```r
plot( ecdf(sample_diff), 
      xlab = "Improvement", ylab = "Cumulative Probability",
      main = "Cumulative Probability of variant A over variant B")
abline(h = 0.5)
abline(h = 0.75, lty = "dashed", col = "red")
abline(h = 0.25, lty = "dashed")
abline(v = 0.02, lty = "dashed", col = "red")
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-47-1.png" width="672" style="display: block; margin: auto;" />

Based on our analysis so far, we have see no evidence that variant B or the new design of the landing page can give us better conversion rate, in fact there is a great chance that it would perform worse than the old design of the landing page. It would be better if we go back to the old landing page and try another new variant to test. Using the bayesian statics, we are not only capable of estimating the conversion rates and see if variant A is better than B. We can go further by inspecting how much variant A is better than B and is there a high chance that variant A is really better than B? Are we willing to risk our money based on our finding?

##### Using `bayesAB` package

We have shown so far that it is really easy to build a Bayesian hypothesis testing by our own. Alternatively, instead of writing the code from scratch to do the hypothesis testing, we can use the `bayesTest()` function from the `bayesAB` package. The function will give us the comprehensive result of what we have done previously.


```r
# Fit bernoulli test
set.seed(123)
AB1 <- bayesTest(variant_a, # variant A
                 variant_b, # variant B
                 priors = c('alpha' = 2, 'beta' = 8), # prior distribution with beta(2,8)
                 distribution = 'bernoulli',  # bernoulli distribution for binary problem
                 n_samples = 1e6 # number of data to sample from posteriors
                 )

summary(AB1)
```

```
#> Quantiles of posteriors for A and B:
#> 
#> $Probability
#> $Probability$A
#>        0%       25%       50%       75%      100% 
#> 0.1162741 0.1198146 0.1203893 0.1209656 0.1246456 
#> 
#> $Probability$B
#>        0%       25%       50%       75%      100% 
#> 0.1147905 0.1182400 0.1188111 0.1193842 0.1228733 
#> 
#> 
#> --------------------------------------------
#> 
#> P(A > B) by (0)%: 
#> 
#> $Probability
#> [1] 0.905237
#> 
#> --------------------------------------------
#> 
#> Credible Interval on (A - B) / B for interval length(s) (0.9) : 
#> 
#> $Probability
#>           5%          95% 
#> -0.003346881  0.030199500 
#> 
#> --------------------------------------------
#> 
#> Posterior Expected Loss for choosing B over A:
#> 
#> $Probability
#> [1] 0.0004478022
```

The expected loss is a combination of how probable it is that B has a lower conversion rate than A (the risk that you are wrong to switch), and, if B is worse, how much worse it is on average (the potential downside).

The plot will directly give you 3 information:

- The curve of the prior distribution
- The posterior distribution of each variant
- Improvement of variant A over variant B


```r
plot(AB1)
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-49-1.png" width="672" style="display: block; margin: auto;" /><img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-49-2.png" width="672" style="display: block; margin: auto;" /><img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-49-3.png" width="672" style="display: block; margin: auto;" />

## Case Study: Hotel Booking Website

We will do another case study. This time is the dataset of experimentation about a new hotel booking system of an anoymous hotel in UK.


```r
data_hotel <- read.csv("data_input/Website Results.csv")

glimpse(data_hotel)
```

```
#> Rows: 1,451
#> Columns: 4
#> $ variant        <chr> "A", "A", "A", "A", "A", "A", "A", "A", "A", "A", "A", …
#> $ converted      <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE,…
#> $ length_of_stay <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 2, 0, 0, 0…
#> $ revenue        <dbl> 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0…
```

Data Description:

- `variant`: (A) Control Group, (B) Treatment Group
- `converted`: Whether the customer successfully makes a booking
- `length_of_stay`: Number of days of stay
- `revenue`: Revenue generated from the successful booking

### Data Preprocessing

Let's check if there are any missing values in the data.


```r
data_hotel %>% 
  is.na() %>% 
  colSums()
```

```
#>        variant      converted length_of_stay        revenue 
#>              0              0              0              0
```

To help us easier to understand what each variant is, we will convert them into `control` and `treatment` group.


```r
data_hotel <- data_hotel %>% 
  mutate(converted = as.numeric(converted),
         variant = ifelse(variant == "A", "control", "treatment")
         )
```

### Hypothesis Testing

#### Prior

Since we don't have the historical data of the conversion rate, we will use external resource. Here, we will use conversion rate information from the UK average[^5].

> What is the average conversion for a hotel website? 2.2%. That's according to a recent Fastbooking study. The UK firm 80 Days, in an extensive report titled "The great unanswered digital questions of hoteliers" came to a similar conclusion: 2% percent. Both studies were conducted on hundreds of hotels, different in size, star rating, and location.

Since we are pessimistic people, we will use the 2% conversion rate as our prior here. Now we need to convert them into the proper beta distribution. Our priors are pretty general and has a weak arguments. A different hotel in a different location my have very different conversion rate. Therefore, we will put a weak prior so that we can rely more on the data. We will use `Beta(2, 98)` for the prior.


```r
fun_beta <- function(x) dbeta(x, 2, 98)

data.frame() %>% 
  ggplot() +
  geom_function(fun = fun_beta) +
  labs(x = "Conversion Rate", y = "Density")
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-53-1.png" width="672" style="display: block; margin: auto;" />

#### Observed Data

Let's check the conversion rate based on the observed data.


```r
data_hotel %>% 
  count(converted, variant) %>% 
  pivot_wider(names_from = converted, values_from = n) %>% 
  setNames(c("variant", "not_converted", "converted")) %>% 
  mutate(conversion_rate = converted / (converted + not_converted))
```

```
#> # A tibble: 2 x 4
#>   variant   not_converted converted conversion_rate
#>   <chr>             <int>     <int>           <dbl>
#> 1 control             701        20          0.0277
#> 2 treatment           693        37          0.0507
```

#### Posterior

Let's do the hyphotesis testing. We will put the `treatment` group as variant A while the `control` group as the variant B. 


```r
variant_a <- data_hotel %>% 
  filter(variant == "treatment") %>% 
  pull(converted)

variant_b <- data_hotel %>% 
  filter(variant == "control") %>% 
  pull(converted)

AB2 <- bayesTest(variant_a,
                 variant_b,
                 priors = c('alpha' = 2, 'beta' = 98), 
                 distribution = 'bernoulli',  
                 n_samples = 1e6
                 )

summary(AB2)
```

```
#> Quantiles of posteriors for A and B:
#> 
#> $Probability
#> $Probability$A
#>         0%        25%        50%        75%       100% 
#> 0.02056207 0.04185121 0.04662847 0.05172414 0.08912482 
#> 
#> $Probability$B
#>          0%         25%         50%         75%        100% 
#> 0.008013844 0.022816701 0.026415310 0.030363989 0.060617120 
#> 
#> 
#> --------------------------------------------
#> 
#> P(A > B) by (0)%: 
#> 
#> $Probability
#> [1] 0.98608
#> 
#> --------------------------------------------
#> 
#> Credible Interval on (A - B) / B for interval length(s) (0.9) : 
#> 
#> $Probability
#>       5%      95% 
#> 0.152282 1.753274 
#> 
#> --------------------------------------------
#> 
#> Posterior Expected Loss for choosing B over A:
#> 
#> $Probability
#> [1] 0.001330352
```

Let's put the visualization here.


```r
plot(AB2)
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-56-1.png" width="672" style="display: block; margin: auto;" /><img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-56-2.png" width="672" style="display: block; margin: auto;" /><img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-56-3.png" width="672" style="display: block; margin: auto;" />


```r
posterior_a <- AB2$posteriors$Probability$A
posterior_b <- AB2$posteriors$Probability$B

cat("90% Credible Interval of Treatment Variant\n")
```

```
#> 90% Credible Interval of Treatment Variant
```

```r
quantile(posterior_a, probs = c(0.05, 0.5,0.95))
```

```
#>         5%        50%        95% 
#> 0.03555704 0.04662847 0.05965968
```

```r
cat("\n90% Credible Interval of Control Variant\n")
```

```
#> 
#> 90% Credible Interval of Control Variant
```

```r
quantile(posterior_b, probs = c(0.05, 0.5, 0.95))
```

```
#>         5%        50%        95% 
#> 0.01824963 0.02641531 0.03664246
```

The 90% credible interval shows that the conversion rate for the `treatment` group is somewhere around 3.55% to 5.96% while the `control` group is between 1.82% and 3.66%. This clearly shows that the new treatment variant is better. Based on the posterior simulation, out of 1,000,000 trials we can expect that the treatment group is 98.6% better than the control group. You can also expect that there is around 32% chance that the new variant (treatment variant) is 100% or better than the control variant based on the cumulative probability. There is almost 0% chance that the treatment is worst than the control group.


```r
sample_diff <- (posterior_a - posterior_b) / posterior_b

plot(ecdf(sample_diff), 
     xlim = c(-0.5, 2.5), 
     xlab = "Conversion Rate",
     main = "Improvement of Treatment over Control Group")
abline(v = 1, lty = "dashed")
abline(h = 0.68, lty = "dashed")
```

<img src="/blog/2021-03-08-bayesian-statistics-and-a-b-testing_files/figure-html/unnamed-chunk-58-1.png" width="672" style="display: block; margin: auto;" />


# Reference

[^1]: [McElreath, Richard. 2019. Statistical Rethinking:A Bayesian Course with Examples in R and Stan. Florida: CRC Press](https://www.amazon.com/Statistical-Rethinking-Bayesian-Examples-Chapman/dp/036713991X)
[^2]: [Kruschke, John K. Doing Bayesian Data Analysis A Tutorial with R, JAGS, and Stan. Londong: Elsevier](https://www.amazon.com/Doing-Bayesian-Data-Analysis-Tutorial/dp/0123814855)
[^3]: [Kurt, Will. 2019. Bayesian Statistics The Fun Way. San Fransisco: No Starch Press, Inc.](https://www.amazon.com/Bayesian-Statistics-Fun-Will-Kurt/dp/1593279566)
[^4]: [Easy Evaluation of Decision Rules in Bayesian A/B testing](https://www.chrisstucchio.com/blog/2014/bayesian_ab_decision_rule.html)
[^5]: [2.2% - a closer look into hotel conversion rates](https://www.phocuswire.com/2-2-a-closer-look-into-hotel-conversion-rates)
