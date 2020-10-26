---
title: Topic Modelling with Latent Dirichlet Allocation (LDA)
author: Arga Adyatama, Joe Nathan
github: https://github.com/Argaadya/Topic_Modelling
date: '2020-10-26'
slug: topic-modeling-lda
categories:
  - R
tags:
  - Machine Learning
  - Topic Modelling
description: ''
featured: ''
featuredalt: ''
featuredpath: ''
linktitle: ''
type: post
---



# Topic Modelling

## Background

Natural Language Processing (NLP) is a branch of artificial intelligence that is steadily growing both in terms of research and market values[^1]. The ultimate objective of NLP is to read, decipher, understand, and make sense of the human languages in a manner that is valuable[^2]. The are many applications of NLP in various industries, such as:

* SPAM email detection
* Sentiment Analysis
* Text summarization
* Text Generation
* Topic Modelling

On this occation, we will learn about Topic Modelling and it's application in a real case. Before we start the journey, let's consider a simple example.

Suppose that we have the following word cloud, can you guess what these words have in common?

<center> ![](/img/topic_model_lda/politic.png){width="100%"} </center>

The interpretation may differ from one persone to another, but most of you must be agree that the word cloud has a common theme or topic. Perhaps you might say that it is related to economics, or politics, or business. The real theme of the words is unkown, but we as the observer are giving the group of words a meaningful and understandable topic. This activity is what we call as **Topic Modelling**.

> In text mining, we often have collections of documents, such as blog posts or news articles, that we’d like to divide into natural groups so that we can understand them separately. Topic modeling is a method for unsupervised classification of such documents, similar to clustering on numeric data, which finds natural groups of items even when we’re not sure what we’re looking for.
> <footer> Julia Silge and David Robinson</footer>

There are many application of Topic Modelling, even outside of the field of NLP. Some applications of Topic Modelling derived from *Boyd-Graber et al.*[^3] and *Blei et al.*[^4] includes:

* Discover different topic in large corpus of document
* Sentiment Analysis
* Understanding Stance and Polarization in Social Media
* Identify new innovation/discovery in scientific research paper
* Document Classification
* Recommender System

Below is another example of topic modeling from *Blei et al.* where the top words for each topic (arts, budgets, children, and education) are shown. The colored text on the lower part of the figure illustrate that a single document is a collection of words with various topic.

<center> ![](/img/topic_model_lda/topic.PNG){width="100%"} </center>

The popular algorithm for Topic Modeling is **Latent Dirichlet Allocation (LDA)**, which is developed by <a href = "http://www.jmlr.org/papers/volume3/blei03a/blei03a.pdf"> **Blei et al.** </a>. This algorithm can be understood in this two simple properties[^5]:

* **Every document is a mixture of topics**. We imagine that each document may contain words from several topics in particular proportions. For example, in a two-topic model we could say “Document 1 is 90% topic A and 10% topic B, while Document 2 is 30% topic A and 70% topic B.”
* **Every topic is a mixture of words**. For example, we could imagine a two-topic model of American news, with one topic for “politics” and one for “entertainment.” The most common words in the politics topic might be “President”, “Congress”, and “government”, while the entertainment topic may be made up of words such as “movies”, “television”, and “actor”. Importantly, words can be shared between topics; a word like “budget” might appear in both equally.

**The objective** of this article is as follows:

* Understand the advantage and disadvantage of LDA
* Understand how to train and evaluate Topic Modelling with LDA
* Learn how to implement Topic Modeling in a corpus of document
* Learn how to interpret and explore the output of LDA

## Library and Setup

Below is the required package to reproduce the code in this article. 


```r
# Data Wrangling
library(tidyverse)

# Text Processing
library(tm)
library(corpus)
library(tidytext)
library(textclean)
library(lubridate)
library(hunspell)
library(SnowballC)
library(textmineR)
library(scales)

# Visualization
library(ggwordcloud)

# Modeling and Evaluation
library(randomForest)
library(e1071)
library(yardstick)

options(scipen = 999)
```

# General Concept

This section illustrate the first principle and the workflow of Topic Modelling with LDA

## Latent Dirichlet Allocation (LDA)

LDA is a generative probabilistic model of a corpus. Compared to other topic modelling methods such as the unigram model, Latent Semantic Analysis (LSA), and Probabilistic Latent Semantic Analysis (pLSA), the advantage and disadvantage of LDA is as follows:

**Advantages**

- Can find latent topic inside documents
- Supervised learning requires a true label, which may not be available
- LDA is easy to train
- LDA can be paired with word2vec to retain the word representation.
- LDA give interpretable topics

**Disadvantages**

- Only considers document as a bag of words and ignore syntactic information (e.g. word order) and semantic information (e.g. the multiplicity of meanings of a given word)
- Fixed number of topics
- Uncorrelated topics (Dirichlet topic distribution cannot capture correlations)
- Static (no evolution of topics over time)

We will break the concept one step at a time. Back to the last section, the main principle of LDA is these 2 concepts:

* **Every document is a mixture of topics**
* **Every topic is a mixture of words**

We will go to the first concept. LDA does not give a clear answer wether a document belong to a certain topics because a document is considered as a mixture of topics. In a single document, we may find several topics. A news about the financial crisis of 2008 may consists of economics, politics and social topics, blended in a single article that discuss about the market value, the government response and the impact of high unemployment rate. Thus, as an example, a document may consists of 30% economics, 60% politics, and 10% social topics. The percentage of each topic represent the probability of the document to belong a certain topic. Therefore, we can imagine the mixture of topics as a probability distribution such as the following table:


```r
set.seed(123)
data.frame(news = 1:5,
           economics = runif(5, max = 0.5),
           politics = runif(5, max = 0.5)) %>% 
   mutate(social = 1 - (economics + politics))
```

```
#>   news economics   politics    social
#> 1    1 0.1437888 0.02277825 0.8334330
#> 2    2 0.3941526 0.26405274 0.3417947
#> 3    3 0.2044885 0.44620952 0.3493020
#> 4    4 0.4415087 0.27571751 0.2827738
#> 5    5 0.4702336 0.22830737 0.3014590
```

The first news has a 14% chance to be an economics news, 2% of politics news and 83% of social news. The second news has a 39% chance to be an economics news, 26% of politics news and 34% of social news and so on. Higher probability means that the document can be represented by the topic.

The word *Latent* in Latent Dirichlet Allocation refers to the latent or hidden structure inside a document. As we have seen in the previous section, we intuively think that a group of words can have a central theme or topics. Therefore, according to the second principle, a topic is a mixture of words, where a certain words have a strong association with a certain topic. For example, the word *President* have a strong signal that it belong to the topic of politics, or the word *Loan* and *Interest* have a strong signal toward economics topic. Just as a document has a probability distribution for each topic, a topic also has a probability distribution for each words/terms. LDA assume that a document is a bag of words, thus we do not care about the word sequence.

An LDA model is built based on these two principles and has the following graphical structures. 

<center> ![](/img/topic_model_lda/lda_model.jpg){width="100%"} </center>

Notation:

* `\(\alpha\)` = The dirichlet parameter for per-document topic proportion
* `\(\theta_d\)` = Topic distribution of document `\(d\)`
* `\(Z_{d,n}\)` = Topic assignment of the `\(n^{th}\)` word in document `\(d\)`
* `\(W_{d,n}\)` = Observed word of the `\(n^{th}\)` word in document `\(d\)`
* `\(\eta\)` = 
* `\(\beta_k\)` = Words distribution of topic `\(k\)`
* `\(K\)` = Collection of topics
* `\(N\)` = Collection of words within documents
* `\(D\)` = Collection of document in the corpus

LDA belong to a hierarchical bayesian model. A bayesian model has a prior and a posterior. A prior means the probability distribution of certain things before we see the data. For example, we may belief that the probability distribution for a roll of dice is uniformly distributed with every value have the same probability to appear. We have this assumption before we try to roll the dice. Meanwhile, the posterior probability reflects the probability distribution after we see the data, after we roll the dice. For example, after we toss the dice for 10,000 times, the number 5 appears for more than 5,000 tosses. This may change our belief that the value for each side is uniformly distributed, since a single side of the dice has a high chance to appear and we might be suspicious that the dice is not fair. We can calculate the posterior probability using the Bayes Theorem[^6], which we will not discuss on this article. The concept of prior and posterior probability is important to the understanding of the LDA process.

The LDA model consists of 2 different priors distribution: probability distribution of topic to document (topic-document probability) and probability distribution of words to topics (word-topic probability). The probability distribution for the two priors are not uniform or normal distribution, but a Dirichlet distribution with the `\(\alpha\)` and `\(\beta\)` parameters as well as the number of topics as the input. 

The following figure is the illustration of the Dirichlet distribution for each topic to each document. Suppose we have a collection of documents and we are confident that on the corpus we have 3 different topics. Each dot represent a single document and each side of the triangle represent a single topics. The position of each document represent the probability distribution for each topic. We randomly assign the position of each document based on its probability distribution. We can see that document A has a close proximity to the topic of Science. This indicate that document A has a high probability of belong to topic Science and low probability ot belong to Politics or Economy. Meanwhile, document B has a high probability to belong to the topic Economy. Document C is located between Science and Politics, so it has a rather equal probability for those topics and low probability for the topic Economy. The topic D is located at the center of the triangle, which means that it has an equal probability for the three topics. 

<center> ![](/img/topic_model_lda/dirichlet1.png){width="100%"} </center>

The goal of LDA is to make each document as close as possible toward a certain topics, although it may not always be achieved for all document. If the above graphics represent our prior probability for each document to each topics, the following graphic illustrate the posterior probability where all document is having a strong relation with a certain topics.

<center> ![](/img/topic_model_lda/dirichlet2.png){width="100%"} </center>

We may translate the probability distribution into the following table:


```r
data.frame(document = c("A","B","C","D"),
           sciences = c(0.8, 0.1, 0.1, 0.1),
           politics = c(0.1, 0.05, 0.85, 0.8),
           economy = c(0.1, 0.85, 0.05, 0.1))
```

```
#>   document sciences politics economy
#> 1        A      0.8     0.10    0.10
#> 2        B      0.1     0.05    0.85
#> 3        C      0.1     0.85    0.05
#> 4        D      0.1     0.80    0.10
```

The second prior distribution is the probability of a topics belong to certain words: word-topic probability. This prior is concerned with what terms that represent what topic? Is there any terms that have strong connection with certain topics? Let's say we have 4 terms: *President*, *Planet*, *Market*, and *Energy*. Each terms will act as the edge of the triangle. However, since we have more than 3 terms, the shape of the distribution is not a triangle anymore, but a tetrahedron. The following figures illustrate a random assignment of Dirichlet distribution between topics and words.

<center> ![](/img/topic_model_lda/dirichlet3.png){width="100%"} </center>

If we translate the position of each topic into a probabilistic value, we get the following table:


```r
data.frame(topic = c("Politics", "Science", "Economy"),
           president = c(0.8, 0.15, 0.1),
           planet = c(0.05, 0.5, 0.05),
           energy = c(0.1, 0.3, 0.15),
           market = c(0.05, 0.05, 0.7))
```

```
#>      topic president planet energy market
#> 1 Politics      0.80   0.05   0.10   0.05
#> 2  Science      0.15   0.50   0.30   0.05
#> 3  Economy      0.10   0.05   0.15   0.70
```

Based on the table, the terms *President* has strong association with the word politics, *Planet* with Science and *Market* with Economy.

These prior distribution will be used to generate word samples to calculate the posterior probability or the estimate of the true probability.

Let's go back toward the LDA structure.

<center> ![](/img/topic_model_lda/lda_model.jpg){width="100%"} </center>

The generative process is the process of generating a document by randomly selecting words from each topic. The rough description of the process is as follows:

1. Randomly choose a distribution over topics

2. For each word in the document

   a. randomly choose a topic from the distribution over topics
   b. randomly choose a word from the corresponding topic (distribution over the vocabulary)

In order to estimate the posterior probability of the topic-document and word-topic distribution, we need to use Gibbs-sampling or Variational Expectation Maximation (VEM). Both are approximate inference method and has its own characteristic.

## Gibbs Sampling

The Gibbs sampling is one of many method to sample data in Bayesian Statistics. The concept of Gibbs sampling in topic modeling is that we assume that all documents and all terms have a prior topics except for a single word, which we will calculate the posterior probability of belonging to certain topics. The algorithm of Gibbs sampling is as follows:

_____________________________

1. For each iteration `\(i\)`:

1.1 For each document `\(d\)` and word `\(n\)` currently assigned to `\(z_{old}\)` :

1.1.1 Decrement `\(n_{d,z_{old}}\)` and `\(v_{z_{old}, w_{d,n}}\)`

1.1.2 Sample `\(z_{new} = k\)` with probability proportional to `\(\frac{n_{d,k} + \alpha_k\ v_{k,w_{d,n}} + \lambda_{w_{d,n}}}{\Sigma_i=k^K n_{d,i} +\alpha_i\ \Sigma_i v_{k,i} + \lambda_i}\)`

1.1.3 Increment `\(n_{d,z_{new}}\)` and `\(v_{z_{old}, w_{d,n}}\)`

_____________________________

Notation :

`\(z_{old}\)` : prior/early topic assignment

`\(z_{new}\)` : updated topic assignment

`\(n_{d,k}\)` : Number of times document `\(d\)` uses topic `\(k\)`

`\(v_{k, w_{d,n}}\)` : Number of times topic `\(k\)` uses word type `\(w_{d,n}\)`

`\(\alpha_k\)` : Dirichlet parameter for document to topic distribution

`\(\lambda_{w_{d,n}}\)` : Dirichlet parameter for topic to word distribution

We will try with an example. Suppose we have a corpus of documents, with one of them is as follows:

> The government put curfew during pandemic

Suppose that the document has a pre-determined topic assignment:


```r
data.frame(
   topic = c(1, 2, 2, 3, 1, 3),
   term = c("the", "government", "put", "curfew", "during", "pandemic")
) %>% 
   t() %>% 
   as.data.frame()
```

```
#>        V1         V2  V3     V4     V5       V6
#> topic   1          2   2      3      1        3
#> term  the government put curfew during pandemic
```

The following is the count of words for each topics from all documents inside the corpus:


```r
data.frame(
   term = c("the", "government", "put", "curfew", "during", "pandemic"),
   topic_1 = c(200, 10, 30 ,20, 5, 8),
   topic_2 = c(40, 30, 10, 5, 3, 2),
   topic_3 = c(10, 2, 1, 8, 17, 30)
)
```

```
#>         term topic_1 topic_2 topic_3
#> 1        the     200      40      10
#> 2 government      10      30       2
#> 3        put      30      10       1
#> 4     curfew      20       5       8
#> 5     during       5       3      17
#> 6   pandemic       8       2      30
```

For the first iteration and the first term, we would like to update the topic assignment for the word *government*. 


```r
data.frame(
   topic = c(1, "?", 2, 3, 1, 3),
   document = c("The", "government", "put", "curfew", "during", "pandemic")
) %>% 
   t() %>% 
   as.data.frame()
```

```
#>           V1         V2  V3     V4     V5       V6
#> topic      1          ?   2      3      1        3
#> document The government put curfew during pandemic
```

1.1.1 Decrement `\(n_{d,z_{old}}\)` and `\(v_{z_{old}, w_{d,n}}\)`

The word *government* was assigned as topic 2 earlier so we would omit that assignment and reduce the count of word *government* from the topic 2 by 1 (from 30 to 29).


```r
data.frame(
   term = c("the", "government", "put", "curfew", "during", "pandemic"),
   topic_1 = c(200, 10, 30 ,20, 5, 8),
   topic_2 = c(40, 29, 10, 5, 3, 2),
   topic_3 = c(10, 2, 1, 8, 17, 30)
)
```

```
#>         term topic_1 topic_2 topic_3
#> 1        the     200      40      10
#> 2 government      10      29       2
#> 3        put      30      10       1
#> 4     curfew      20       5       8
#> 5     during       5       3      17
#> 6   pandemic       8       2      30
```

1.1.2 Sample `\(z_{new} = k\)` with probability proportional to `\(\frac{n_{d,k} + \alpha_k\ v_{k,w_{d,n}} + \lambda_{w_{d,n}}}{\Sigma_i=k^K n_{d,i} +\alpha_i\ \Sigma_i v_{k,i} + \lambda_i}\)`

Now we update the topic assignment using the earlier formula. We will get the probability for each topic (k) and then sample the topic. Higher probability means that the word will be likely assigned to that topic.

`$$z_{new} = \frac{n_{d,k} + \alpha_k\ v_{k,w_{d,n}} + \lambda_{w_{d,n}}}{\Sigma_{i=k}^K n_{d,i} +\alpha_i\ \Sigma_i v_{k,i} + \lambda_i}$$`

Long story short, after we get the sample, we update the topic assignment for the word. For example, based on the sample, the new topic assignment for the word *government* is topic 1.


```r
data.frame(
   topic = c(1, 1, 2, 3, 1, 3),
   document = c("The", "government", "put", "curfew", "during", "pandemic")
) %>% 
   t() %>% 
   as.data.frame()
```

```
#>           V1         V2  V3     V4     V5       V6
#> topic      1          1   2      3      1        3
#> document The government put curfew during pandemic
```

1.1.3 Increment `\(n_{d,z_{new}}\)` and `\(v_{z_{old}, w_{d,n}}\)`

We increment the counts for each word to each topic based on the newly updated assignment and we continue to update the topic for the next words. The count for word *government* for topic 1 is increased from 10 to 1.


```r
data.frame(
   term = c("the", "government", "put", "curfew", "during", "pandemic"),
   topic_1 = c(200, 11, 30 ,20, 5, 8),
   topic_2 = c(40, 29, 10, 5, 3, 2),
   topic_3 = c(10, 2, 1, 8, 17, 30)
)
```

```
#>         term topic_1 topic_2 topic_3
#> 1        the     200      40      10
#> 2 government      11      29       2
#> 3        put      30      10       1
#> 4     curfew      20       5       8
#> 5     during       5       3      17
#> 6   pandemic       8       2      30
```

# Case Studies

## COVID-19 CBC News

We will try to do a document modelling to find different topics in **CBC News/COVID-19 Articles**. The dataset is acquired from <a href = "https://www.kaggle.com/ryanxjhan/cbc-news-coronavirus-articles-march-26"> Kaggle </a>. The context of the dataset is that has the news media been overreacting or under-reacting during the development of COVID-19? What are the media's main focuses? 

We will try to answer this problem by finding latent information inside corpus using Latent Dirichlet Allocation.

### Import Data

The dataset consists of more than 2,725 articles with 7 columns. To read 'big' data with fast running time, we can use `fread()` function from `data.table` package.


```r
covid_news <- data.table::fread("data_input/covid_news.csv", header = T, encoding = "Latin-1")

glimpse(covid_news)
```

```
#> Rows: 2,725
#> Columns: 7
#> $ V1           <int> 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 14, 15, 16, 17…
#> $ authors      <chr> "['Cbc News']", "['Cbc News']", "['The Associated Press'…
#> $ title        <chr> "Coronavirus a 'wake-up call' for Canada's prescription …
#> $ publish_date <chr> "2020-03-27 08:00:00", "2020-03-27 01:45:00", "2020-03-2…
#> $ description  <chr> "Canadian pharmacies are limiting how much medication ca…
#> $ text         <chr> "Canadian pharmacies are limiting how much medication ca…
#> $ url          <chr> "https://www.cbc.ca/news/health/covid-19-drug-supply-1.5…
```

Data description:

* **V1**: Document index
* **authors**: The author of the news/articles
* **title**: The title of the news/articles
* **publish_date**: The date when the news/article is published
* **description**: The subtitle or headline of the news/article
* **text**: The full text of the news/article
* **url**: The link of the news/article

Below is the sample of the data.


```r
head(covid_news, 10)
```

```
#>     V1
#>  1:  0
#>  2:  1
#>  3:  2
#>  4:  3
#>  5:  4
#>  6:  5
#>  7:  6
#>  8:  7
#>  9:  8
#> 10:  9
#>                                                                                                                                                                                                     authors
#>  1:                                                                                                                                                                                            ['Cbc News']
#>  2:                                                                                                                                                                                            ['Cbc News']
#>  3:                                                                                                                                                                                ['The Associated Press']
#>  4:                                                                                                                                                                                            ['Cbc News']
#>  5:                                                                                                                                                                                            ['Cbc News']
#>  6:                                                                                      ['Mark Gollom Is A Toronto-Based Reporter With Cbc News. He Covers Canadian', 'U.S. Politics', 'Current Affairs.']
#>  7:                                                                                                                                                                                            ['Cbc News']
#>  8:                                                                                                                                                                                ['The Associated Press']
#>  9:                                                                                                                                                                                     ['Thomson Reuters']
#> 10: ['Leah Hendry Is A Tv', 'Radio', 'Online Journalist With Cbc Montreal Investigates. Contact Her Via Our Confidential Tipline', 'Or On Email At Montrealinvestigates Cbc.Ca.', 'Follow Leah On Twitter']
#>                                                                                                             title
#>  1:                                            Coronavirus a 'wake-up call' for Canada's prescription drug supply
#>  2:                                                  Yukon gov't names 2 possible sources of coronavirus exposure
#>  3:                                                             U.S. Senate passes $2T coronavirus relief package
#>  4:                                             Coronavirus: The latest in drug treatment and vaccine development
#>  5:                                                           The latest on the coronavirus outbreak for March 26
#>  6:                                          'Worse' pandemic on horizon unless world deals with wildlife markets
#>  7:                                            What you need to know about COVID-19 in Ottawa on Friday, March 27
#>  8:                                                   Michigan hospitals jammed as coronavirus cases, deaths rise
#>  9:                                                               U.S. coronavirus cases now highest in the world
#> 10: 'Avoid the emergency' pleads Jewish General, as it hurries to find safer ways to evaluate mildly ill patients
#>            publish_date
#>  1: 2020-03-27 08:00:00
#>  2: 2020-03-27 01:45:00
#>  3: 2020-03-26 05:13:00
#>  4: 2020-03-27 00:36:00
#>  5: 2020-03-26 20:57:00
#>  6: 2020-03-27 08:00:00
#>  7: 2020-03-27 08:00:00
#>  8: 2020-03-26 11:02:00
#>  9: 2020-03-26 14:55:00
#> 10: 2020-03-27 08:00:00
#>                                                                                                                                                                                                                                                  description
#>  1: Canadian pharmacies are limiting how much medication can be dispensed to try to prevent shortages, recognizing that most active ingredients for drugs come from India and China and medical supply chains have been disrupted by the spread of COVID-19.
#>  2:                                                                                     The Yukon government has identified two places in Whitehorse — a church and a dental clinic — where people may have been exposed to the coronavirus in recent weeks.
#>  3:                                                                             The Senate has passed an unparalleled $2.2 trillion economic rescue package steering aid to businesses, workers and health care systems engulfed by the coronavirus pandemic
#>  4:                                                                                          Scientists around the world are racing to find novel ways to treat, mitigate or prevent infection from the novel coronavirus. Here are the latest developments.
#>  5:                                                                                                                                                                              The latest on the coronavirus outbreak from CBC News for Thursday, March 26
#>  6:          The continued existence of wildlife markets, which are considered potential breeding grounds for the spread of harmful viruses, means it's just a matter of time before the world is hit with another deadly pandemic, some scientists suggest.
#>  7:                                                                                                                                                                               CBC Ottawa's latest roundup of key points during the coronavirus pandemic.
#>  8:                                                                                                                                                    Michigan hospitals are bracing for a surge of coronavirus cases as infections rise to at least 2,294.
#>  9:     The number of confirmed COVID-19 cases in the U.S. rose to 81,378 on Thursday — more than any other country, overtaking both Italy and China — while the death toll passed the 1,000 mark, according to a running tally by Johns Hopkins University.
#> 10:                                                     Montreal's Jewish General Hospital, one of the designated COVID-19 treatment centres, is setting up trailers outside its main building, where people with minor symptoms can seek medical attention.
#>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               text
#>  1:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  Canadian pharmacies are limiting how much medication can be dispensed to try to prevent shortages, recognizing that most active ingredients for drugs come from India and China and medical supply chains have been disrupted by the spread of COVID-19. Provincial regulatory colleges are complying with the Canadian Pharmacists Association call to limit the amount of medications given to patients to 30-day supplies. The goal is to stop people from refilling prescriptions early and to ensure life-saving drugs don't run short when supply chains are vulnerable. Mina Tadrous is a pharmacist and researcher in Toronto who monitors pharmaceutical supplies. He is worried Canadians will start stockpiling drugs after watching what has been unfolding in the U.S. and other regions as the virus spreads.  He said pharmacists are concerned about drugs such as life-saving inhalers that people might stockpile based on misinformation circulating about potential treatments for COVID-19. """"It's that relationship of how people are reacting rather than the actual supply of medications,"""" he said.  Tadrous said pharmacies get their medications and supplies from wholesalers, who get theirs from distributors who source them from manufacturers. Outside of wholesalers, most of that supply chain is outside of Canada.  Ongoing shortages Dr. Jacalyn Duffin of Queen's University in Kingston, Ont., has long warned about the increasing frequency of drug shortages, including nearly 2,000 ongoing shortages, none of which she attributes to coronavirus. China and India produce 80 per cent of the active ingredients of prescription drugs sold in North America, said Dr. Jacalyn Duffin. (Kas Roussy/CBC) Given that China and India produce 80 per cent of the active ingredients of prescription drugs sold in North America, supply disruptions are expected eventually, Duffin said. """"I think that the coronavirus outbreak is a big wake-up call for us to pay attention to our drug shortages that exist already and to pay attention to where our drugs come from,"""" she said. What you need to know about getting new prescriptions, refillsNew Brunswick offers relief on prescription co-pays during outbreak India, a major supplier of generic drugs to Europe, has already shut down the export of some antibiotics and drugs for hypertension, blood pressure and acetaminophen, Duffin said. As a precaution, the federal government has enacted sweeping legislation that gives it the right to force patented drugmakers to make more medicine if necessary and remove patents as part of the response to the public health emergency. 
#>  2:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             The Yukon government has identified two places in Whitehorse — a church and a dental clinic — where people may have been exposed to the coronavirus in recent weeks. In a news release Thursday evening, the government said anybody who went to either place on certain days should monitor themselves for symptoms that may indicate COVID-19 — fever, cough or difficulty breathing — for up to 14 days, and call 811 if those symptoms develop. The possible sources of exposure are:  Sunday morning services at Bethany Church on the Alaska Highway, on March 8 and March 15, or the church's """"Kids Zone"""" on March 8. Elias Dental clinic, from March 9 to 13, or March 16.  According to the news release, those places were visited on those dates by a person or persons who tested positive for COVID-19. It also says that people who attended those places at the identified times do not need to self-isolate, so long as they remain healthy and do not develop symptoms.  """"The risk to individuals from both of these possible exposures is low,"""" the release says. Bethany Church in Whitehorse. According to the Yukon government, someone who later tested positive for COVID-19 attended Sunday morning services at the church on Mar. 8 and 15. (Paul Tukker/CBC) As of Wednesday, Yukon had three confirmed cases of COVID-19. Health officials announced the first two cases on Sunday, saying it was a couple from Whitehorse. The third person's community has not been disclosed. Yukon bans residential evictions, offers paid sick leave during COVID-19 pandemicCOVID-19 testing centre up and running in Whitehorse The person or people who attended Bethany Church and Elias Dental are recovering at home, according to the government's news release. It also says there is no ongoing risk to the community associated with either the church or the dental clinic.
#>  3:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               The Senate late Wednesday passed an unparalleled $2.2 trillion US economic rescue package steering aid to businesses, workers and health care systems engulfed by the coronavirus pandemic. The unanimous vote came despite misgivings on both sides about whether it goes too far or not far enough and capped days of difficult negotiations as Washington confronted a national challenge unlike it has ever faced. The 880-page measure is the largest economic relief bill in U.S. history. Republican Senate Majority Leader Mitch McConnell appeared sombre and exhausted as he announced the vote — and he released senators from Washington until April 20, though he promised to recall them if needed. """"The legislation now before us now is historic because it is meant to match a historic crisis,"""" said Democratic Senate Minority Leader Chuck Schumer. """"Our health care system is not prepared to care for the sick. Our workers are without work. Our businesses cannot do business. Our factories lie idle. The gears of the American economy have ground to a halt."""" WATCH | U.S. closer to passing aid package as Louisiana becomes potential hotspot: American lawmakers are confident an aid package will be approved this week as officials identify Louisiana as a possible hotspot for COVID-19 cases. 1:59 The package is intended as relief for an economy spiralling into recession or worse and a nation facing a grim toll from an infection that's killed nearly 20,000 people worldwide. Treasury Secretary Steven Mnuchin, asked how long the aid would keep the economy afloat, said: """"We've anticipated three months. Hopefully, we won't need this for three months."""" Underscoring the effort's sheer magnitude, the bill finances a response with a price tag that equals half the size of the entire $4 trillion annual federal budget. Insistently optimistic, President Donald Trump said of the greatest public-health emergency in anyone's lifetime, """"I don't think its going to end up being such a rough patch"""" and anticipated the economy soaring """"like a rocket ship"""" when it's over. Coronavirus impact spreads across U.S., with much of the population under stay-at-home ordersAnyone leaving New York must self-isolate or risk spreading COVID-19, health officials say The drive by leaders to speed the bill through the Senate was slowed as four conservative Republican senators from states who economies are dominated by low-wage jobs demanded changes, saying the legislation as written might give workers like store clerks incentives to stay on unemployment instead of returning return to their jobs since they may earn more money if they're laid off than if they're working. They settled for a failed vote to modify the provision. Other objections floated in from New York Gov. Andrew Cuomo, who has become a prominent Democrat on the national scene as the country battles the pandemic. Cuomo, whose state has seen more deaths from the pandemic than any other, said, """"I'm telling you, these numbers don't work."""" Ardent liberals like Rep. Alexandria Ocasio-Cortez were restless as well, but top Washington Democrats assured them that a additional coronavirus legislation will follow this spring and signalled that delaying the pending measure would be foolish. The sprawling measure is the third coronavirus response bill produced by Congress and by far the largest. It builds on efforts focused on vaccines and emergency response, sick and family medical leave for workers, and food aid. Provides direct payments Democratic House Speaker Nancy Pelosi swung behind the bipartisan agreement, saying it """"takes us a long way down the road in meeting the needs of the American people."""" Senate passage delivered the legislation to the Democratic-controlled House, which will most likely pass it Friday. House members are scattered around the country and the timetable for votes in that chamber was unclear. House Democratic and Republican leaders have hoped to clear the measure for Trump's signature by a voice vote without having to call lawmakers back to Washington. The package would give direct payments to most Americans, expand unemployment benefits and provide a $367 billion program for small businesses to keep making payroll while workers are forced to stay home. It includes a controversial, heavily negotiated $500 billion program for guaranteed, subsidized loans to larger industries, including airlines. Hospitals would get significant help as well. Six days of arduous talks produced the bill, creating tensions among Congress' top leaders, who each took care to tend to party politics as they maneuvered and battled over crafting the legislation. But failure is not an option, nor is starting over, which permitted both sides to include their priorities. """"That Washington drama does not matter any more,"""" McConnell said. """"The Senate is going to stand together, act together, and pass this historic relief package today."""" U.S. Capitol Police officers stand atop of the U.S. Senate stairway, ahead of a vote on the coronavirus relief bill, on Capitol Hill in Washington Wednesday.  (Tom Brenner/Reuters) The bill would provide one-time direct payments to Americans of $1,200 per adult making up to $75,000 a year, and $2,400 to a married couple making up to $150,000, with $500 payments per child. A huge cash infusion for hospitals expecting a flood of COVID-19 patients grew during the talks to an estimated $130 billion. Another $45 billion would fund additional relief through the Federal Emergency Management Agency for local response efforts and community services. Democrats said the package would help replace the salaries of furloughed workers for four months, rather than the three months first proposed. Furloughed workers would get whatever amount a state usually provides for unemployment, plus a $600 per week add-on, with gig workers like Uber drivers covered for the first time. Republicans won inclusion of an """"employee retention"""" tax credit that's estimated to provide $50 billion to companies that retain employees on payroll and cover 50 per cent of workers' paycheque up to $10,000. Companies would also be able to defer payment of the 6.2 per cent Social Security payroll tax.
#>  4:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    Scientists around the world are racing to find novel ways to treat, mitigate or prevent infection from the novel coronavirus. The response has been especially rapid as the virus has spread unchecked across the globe. Still, any treatments are a long way from being proven both effective and safe, and most certainly won't come in time to address the current pandemic. According to some of the top infectious disease experts in the world, even on an accelerated timeline, a vaccine is still likely 12-18 months away — and then the challenge will be producing enough to make it readily available. There is currently no drug therapy or vaccine approved anywhere in the world for COVID-19. Here are the latest developments in the research. Old drugs, new tricks March 26: University of Manitoba researchers begin recruiting participants for a series of trials to test whether the malaria drug hydroxychloroquine can reduce the symptoms and severity of COVID-19 and slow its spread. The trial, in conjunction with researchers at the University of Minnesota, University of Alberta and McGill University Health Centre, will include participants who have tested positive for COVID-19, or who live with someone who has. Long-term use of hydroxychloroquine is known to cause cardiac and eye toxicity, but Dr. Ryan Zarychanski, an associate professor of internal medicine, said the short five-day trial should be """"very safe.""""  Manitoba researchers begin clinical trial with drug they hope can help in COVID-19 fight  """"The idea is to reduce the severity of symptoms,"""" said Zarychanski, a critical care doctor, """"and also reduce the transmissibility of the virus and reduce community spread, which can help keep our population more healthy and reduce the burden that we're expecting on the health care system."""" But experts, including Canada's chief public health officer, warn that hydroxychloroquine can have serious side-effects. They also say there is a danger that hoarding the medication could hurt the many patients who take it for chronic inflammatory diseases like lupus and rheumatoid arthritis.  Canadian doctors urge caution on repurposing malaria medication to fight COVID-19  March 25: New York's Mount Sinai hospital begins treating critically ill COVID-19 patients via plasmapheresis, a century-old therapy that involves removing antibody-rich plasma from blood and returning it to a patient (the same or another) after either treating it or replacing it. In this case, doctors will transfer antibodies from recovered COVID-19 patients into sick ones in the hopes the antibodies will neutralize the disease. .<a href=""""https://twitter.com/MountSinaiNYC?ref_src=twsrc%5Etfw"""">@MountSinaiNYC</a> to begin the transfer of <a href=""""https://twitter.com/hashtag/COVID19?src=hash&amp;ref_src=twsrc%5Etfw"""">#COVID19</a> antibodies into critically ill patients: <a href=""""https://t.co/bpSB2HuSqR"""">https://t.co/bpSB2HuSqR</a> <a href=""""https://t.co/GpEHoHEEM6"""">pic.twitter.com/GpEHoHEEM6</a>&mdash;@IcahnMountSinai March 23: Researchers at the Montreal Heart Institute say they are recruiting up to 6,000 Quebecers over the age of 40 who test positive for the novel coronavirus to study whether colchicine, an anti-inflammatory drug used to treat gout, Behçet's disease and familial Mediterranean fever, is effective in treating respiratory complications of COVID-19, like lung inflammation, before patients need ventilators or die. Quebecers who meet the criteria and wish to take part in the study at the Montreal Heart Institute can find out more from their doctor or by calling 1-877-536-6837. (Montreal Heart Institute) """"It's inexpensive, it's widely available and the reason why we think it might very well work is that it's a powerful anti-inflammatory agent,"""" Dr. Jean-Claude Tardif, the institute's research director, said in an interview with CBC Montreal's Daybreak.   Montreal Heart Institute launches clinical study into colchicine as potential COVID-19 treatment  Immunizing the herd March 23: The Canadian government announces $275 million in funding for a number of Canadian research initiatives into vaccine development and testing. One of them is a Saskatoon lab that for four decades has been working on coronavirus vaccines primarily for animals, including successful vaccines for cattle and pigs.  March 17: China gives the go-ahead for researchers at the country's Academy of Military Medical Sciences to begin human safety tests of an experimental coronavirus vaccine. March 16: Scientists at the Kaiser Permanente Washington Research Institute in Seattle begin a first-stage study of a potential COVID-19 vaccine when they administer the first shot to a volunteer patient. With no clear end in sight, the urgent race for a vaccine is on across the world. This week, hopes for a vaccine grew as some biotech players are seemingly making progress. But, some experts caution that this process cannot be rushed - and that we are still far from seeing a vaccine on the market. 13:25
#>  5:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  Trudeau says rules of Quarantine Act will be enforced to keep people isolated. Canada 'strongly opposed' to U.S. stationing troops near shared border. China prepares to close its borders to foreigners, fearing a 2nd wave of COVID-19. Medical experts warn against claims by Winnipeg acupuncturist advertising 'coronavirus prevention tea.' Read more: Track the coronavirus across Canada and around the world.  A sign in Assiniboine Park in Winnipeg encourages visitors to practice social distancing. (Trevor Brine/CBC) Trudeau says rules of Quarantine Act will be enforced to keep people isolated  A new order requiring travellers returning to Canada from abroad to self-isolate for 14 days is the latest measure from a government trying to deal with both a rise in COVID-19 cases and the growing economic fallout. """"We are implementing the Quarantine Act to keep all Canadians safe,"""" Prime Minister Justin Trudeau said today, calling decisions by some to ignore the call to self-isolate for two weeks """"disappointing"""" and """"dangerous."""" When asked why the quarantine measure wasn't introduced sooner, Trudeau said """"the vast majority"""" of Canadians have been following public health guidelines and taking the precautions necessary to protect themselves and others. But, he said, """"there have been too many people who have not,"""" which required the use of a rule with enforcement measures. There have been questions about whether the government's measures at the border to date have been strict enough, furthering concern that some returning travellers weren't complying with the self-isolation request. Premiers, including Ontario's Doug Ford and Alberta's Jason Kenney, addressed that concern earlier this week, prior to the mandatory measure from Ottawa. Read more Canada 'strongly opposed' to U.S. stationing troops near shared border The Canadian government said it is """"strongly opposed"""" to the idea of sending U.S. troops to the border to intercept illegal migrants as part of that country's response to the coronavirus pandemic. """"This is an entirely unnecessary step, which we would view as damaging to our relationship,"""" Deputy Prime Minister Chrystia Freeland said today following the first public reports. A source with knowledge of discussions by White House officials told CBC News the White House is looking at placing 1,000 troops about 25 kilometres from the 8,891-kilometre-long border and using remote sensors to look out for irregular border-crossers. The source stressed that the U.S. hasn't made a final decision. Canada and the U.S. have a mutual ban in place on non-essential travel across the border, which includes trips for recreational purposes. When that ban was announced, both sides stressed the importance of continuing to allow trade, commerce and cross-border essential workers to move back and forth over the border. Read more  Medical experts warn against claims by Winnipeg acupuncturist advertising 'coronavirus prevention tea' A hidden camera investigation revealed a Winnipeg acupuncturist was selling an herbal tea that he claims can prevent COVID-19. In an email blast to clients last week, Guojian Huang, an acupuncture therapist and specialist in traditional Chinese medicine, said drinking a blend of six herbs in a tea over six days would keep people safe. Health Canada has not approved any product to prevent, treat or cure COVID-19, the illness caused by the novel coronavirus. """"Selling unauthorized health products or making false or misleading claims to prevent, treat or cure COVID-19 is illegal in Canada,"""" a Health Canada spokesperson said in an email to CBC News. """"We take this matter very seriously and we are taking action to stop this activity."""" A Toronto physician who treated patients during the SARS outbreak cautioned people against believing claims about COVID-19 cures. """"My concern is that they're desperate,"""" Dr. Peter Lin said. """"They take this stuff and then now they think they're invincible, and they don't do the precautions that will actually protect them, because now they're trusting in this particular tea."""" Read more  ANALYSIS Chris Hadfield's recommendations for self-isolation Astronaut Chris Hadfield spent months as the commander of the International Space Station, so he knows a thing or two about spending long periods of time in isolation. """"Confined spaces are almost entirely psychological,"""" Hadfield told CBC's Heather Hiscox earlier today. """"I think the real thing to do is to just look at the reality of where you are and the opportunities that you have and the way you can still be productive and interact and be mentally stimulated and just go with that. Stop worrying that it's different, and instead go with the reality of the new set of rules that you're living under."""" Hadfield says the tumult forced on us by COVID-19 is an opportunity to be productive amid a wealth of technology. 16:09 Hadfield said time spent staying at home amid the outbreak presents a unique educational opportunity. """"Right now, maybe for the first time in a long time, you've got some downtime, some opportunity to actually learn things,"""" he said. """"And the internet — the way I'm communicating with you right now — right there at your fingertips is this expertise on unlimited subjects. Anything you've ever been curious about you can now dig into and really get it squared away in your mind."""" THE SCIENCE What are the riskiest surfaces for coronavirus? In general, surfaces are riskier if they are smooth, such as metal or plastic (although copper-containing metals are antimicrobial and can kill viruses more quickly). Viruses generally don't survive as long on porous surfaces like paper or clothing. Surfaces that are touched a lot by a lot of people —such as doorknobs, faucets and phones — are generally riskier as well. Studies have detected coronaviruses on phones, doorknobs, computer mice, toilet handles, latex gloves and sponges in hospital and apartments. The Public Health Agency of Canada, U.S. Centers for Disease Control and Prevention and World Health Organization recommend that you wash your hands often with soap and water for at least 20 seconds (or use hand sanitizer with at least 60 per cent alcohol if soap and water aren't available). You should also avoid touching your eyes, nose and mouth with unwashed hands and clean and disinfect frequently touched surfaces daily. In public places, you should avoid touching surfaces. If you have to touch something, you can use disinfectant wipes to wipe off surfaces that are touched frequently by other people, like grocery cart handles, Ottawa epidemiologist Dr. Rama Nair recommends. AND FINALLY... Free and family-friendly options to watch while staying at home Choosing what to watch while staying home is is an increasingly important decision for families around the world.  (Gareth Copley/Getty Images) On what should have been Opening Day of the Major League Baseball season — leaving a certain Coronavirus Brief writer despondent — choosing what to watch is an increasingly prominent discussion for families amid the ongoing outbreak. With schools shut and many parents now working from home, finding accessible and affordable ways to keep kids entertained can be a challenge, especially with rising subscription fees. Fortunately, there are plenty of free and family-friendly options out there, including:  Tubi, an ad-supported streaming service with a broad catalogue of titles and shows. Kanopy, which partners with public libraries to offer enriching digital content including films, children's series and a wide selection of documentaries. The National Film Board of Canada, which has over 4,000 films, documentaries and short pieces available to stream for free — including a rich library of Indigenous content.  And for those of you missing professional sports, MLB is streaming an array of classic games on its various social media platforms today for free. Read the full article https://www.cbc.ca/news/entertainment/free-family-streaming-covid-19-stay-at-home-1.5509556 Send us your questions Still looking for more information on the outbreak? Read more about COVID-19's impact on life in Canada, or reach out to us at covid@cbc.ca. If you have symptoms of the illness caused by the coronavirus, here's what to do in your part of the country. For full coverage of how your province or territory is responding to COVID-19, visit your local CBC News site. To get this newsletter daily as an email, subscribe here. Click below to watch CBC News Network live CBC News Network showcases the best of CBC journalism, covering breaking stories with speed, and adding context and meaning along the way. CBC News Network is also the destination for original journalism, with added depth from CBC News bureaus across the country and around the world. 0:00
#>  6:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          The continued existence of wildlife markets, which are considered potential breeding grounds for the spread of harmful viruses, means it's just a matter of time before the world is hit with another deadly pandemic, some scientists suggest. """"If we do not deal with this, there is nothing to say that we could not in eighteen months' time have another outbreak, and it could be worse,"""" said Kerry Bowman, an assistant professor and bioethicist at the University of Toronto's Dalla Lana School of Public Health. Scientists believe the novel coronavirus that causes the disease COVID-19 emerged from one of these wildlife markets — also known as """"wet markets"""" — in the Chinese city of Wuhan, possibly through an infected bat. Bats are just one of the animals that are sold at these markets, where customers come to purchase domestic livestock and wildlife, including pigs, chickens, civet cats, bamboo rats, porcupines and pangolins. But Bowman said shutting down such markets may prove extremely challenging, as these cultural practices date back thousands of years and have become part of a multi-billion-dollar global industry. Experts call for global ban on live animal markets, wildlife trade amidst coronavirus outbreakWhy tracing the animal source of coronavirus matters Spillover event Bowman said the main concern with these markets is a spillover event, when viruses transfer from one species to another and then cross over to humans. On very rare occasions, humans will transmit that virus from one person to another, which is what occurred with Severe Acute Respiratory Syndrome (SARS) and, by all indications, is happening with COVID-19. WATCH | An explainer on where the coronavirus came from It started with animal to human transmission. But the novel coronavirus is now spreading between people. 0:25 """"Mathematically, this is not a common occurrence. But if you actually wanted to create a laboratory-like experiment to design the conditions for a spillover event, you would create the kind of wildlife market that you have in China,"""" Bowman said. In a recent video message about COVID-19, famed primatologist Jane Goodall warned that the close relationship between people and wild animals in these markets """"has unleashed the terror and misery of new viruses."""" SARS may have spread to humans through wild mammals. The Ebola epidemics in west and central Africa are thought to have originated from bats, while Middle East Respiratory Syndrome (MERS) is suspected to have come from camels. All of these infected animals were in wildlife markets, where every day millions of people around the world still get their food. Not just in China Bowman emphasized that China is not the only culprit, nor is Asia """"the only continent in the world that has this challenge."""" """"But it's particularly tenacious in the Far East, and extends into Vietnam, a lot of Southeast Asia, Indonesia,"""" he said. Civet cats wait for food at a farm in Liuan, China.  (China Photos/Getty Images) In February, China announced a ban on the trade and consumption of wild animals. But it also imposed tough restrictions following the SARS outbreak in 2003, only to see the industry slowly re-emerge. Many environmentalists say such bans contain loopholes. The selling of wild animals accounts for """"a significant portion of the economy. And it has created an economic opportunity for a lot of people,"""" Bowman said. There are different estimates, but Bowman said the size of the global wildlife trade is pegged at somewhere between $7 billion to $23 billion US a year. Animals in unsanitary conditions Bowman, who conducts research on this issue, said when he last visited a market in Wuhan, he counted 57 species of animals, about two-thirds of them wild, mostly of Asian origin. Cages were stacked on top of each other, in unsanitary conditions. He said operators had """"high-powered hoses that are blasting around urine, feces, blood from one cage to another to another to another."""" In terms of the general concepts of infectious diseases, wildlife markets are """"a perfect opportunity for the mixing of bacteria and viruses as well as transmission to other groups,"""" said Jason Stull, assistant professor at the University of Prince Edward Island's Atlantic Veterinary College. Not only that, but stress and malnutrition reduce the immune system of animals and potentially exacerbate this problem, Stull said. For example, an animal under duress may be more likely to shed higher amounts of virus. """"All of these things likely can contribute to movement back and forth of diseases,"""" Stull said. Many infectious diseases linked to wildlife According to the EcoHealth Alliance, a New York-based organization that conducts scientific research into emerging infectious diseases, about three-fourths of all such diseases are somehow linked to wildlife. William Karesh, executive vice president for health and policy at the EcoHealth Alliance, said the current coronavirus outbreak was likely spread in two possible ways. It could have been a wild animal being sold in the market that contaminated the market. It's also possible that a vendor in the market was infected somewhere else and then infected their customers. The animals that end up in the market are coming from two places — hunted in the wild or bred on farms. WATCH | How bats likely spread the novel coronavirus  Bowman said there are deep cultural roots with this industry — thousands of years of tradition of eating wild animals. As well, the animals are used for traditional Chinese medicine, luxury goods and the pet trade. """"What's really changed is that this has gone from occasional domestic use with emerging populations in combination with the burgeoning wealth to a massive commercial enterprise,"""" Bowman said. Focus should be on education Karesh said ending these practices will take time, likely generations, and can only be done through education and helping countries improve their food systems. He suggested that instead of banning all wildlife trade, countries should focus on those animals that are more likely to have viruses that can be transmitted to humans — like rodents, bats and non-human primates. He said the international community must come to grips with the growing and unsustainable use of wildlife, or we will """"continue to see pandemics."""" """"There are three to five emerging diseases every year, and only by luck and the grace of God ... they don't turn into pandemics each time.""""
#>  7: Recent developments:  Two new cases in western Quebec; Ontario has not released locations of new cases. Federal government covering 75 per cent of staff wages for small businesses. Brockville now has COVID-19 testing at the Memorial Centre. NCC closes vehicle access to Greenbelt.  What you should know Ottawa hospitals are preparing for a surge in COVID-19 patients as the number of people infected with the coronavirus rises each day. The Ottawa Hospital is doubling its number of intensive care beds and seeking donations of masks and other personal protective equipment at coviddonations@toh.ca.  Doctors, nurses and cleaning staff in Ottawa are already starting to ration disposable masks to conserve the current supply.  The Montfort and Queensway Carleton hospitals are preparing to open up urgent care centres for COVID-19 patients.  LISTEN: Critical care clinics 'ready to be opened' if needed   Dr. Andrew Willmore, medical director of emergency management at the Ottawa Hospital, says there are several care centres standing by in case of a rise in COVID-19 cases. 0:49 Public health officials say there's still time to slow the spread of COVID-19 if residents keep up physical distancing or self-isolation orders and recommendations, if required.  Physical distancing means avoiding non-essential trips out, working from home and cancelling gatherings, even with friends or extended family. Ottawa Public Health advises residents to only be with members of their own household and stay at least two metres away from everyone else. LISTEN: 'Too soon to say' how social distancing is affecting COVID-19 spread In a news conference conducted by telephone, Vera Etches, Ottawa’s chief medical officer of health, says a delay in getting test results and the continuing return of travellers from abroad make it too early to say how social distancing measures are affecting the spread of COVID-19. 0:42 Public health officials are also urging anyone who's had close contact with someone who has travelled outside the country to self-solate for 14 days. That means staying home for two weeks and asking relatives, friends or neighbours to deliver groceries, medication and other supplies. All deliveries should be left at the door to maintain a two-metre distance. People who feel sick should also self-isolate for 14 days or until 24 hours after their symptoms are gone, whichever is longer. Travellers who return to Canada must now enter a mandatory 14-day period of self-isolation or face a fine of up to $750,000, or as much as six months in jail, unless they're an essential worker. Many municipalities have declared states of emergency.  Ottawa mayor declares state of emergency Eastern Ontario leaders declare states of emergency  In Ottawa this allows the city to buy needed equipment and supplies without the usual procurement process, including personal protective equipment, food for the vulnerable and hotel rooms for emergency workers. Ontario and Quebec have ordered all non-essential businesses to close. The Champlain Bridge between Ottawa and Gatineau on March 24, 2020. Mayors of both cities have asked people not to cross the border unless it's necessary. (Francis Ferland/CBC) Ontario Provincial Police said Friday officers will fine individuals or businesses that break the physical distancing rules. Police in Quebec are also enforcing a ban on gatherings of more than two people.  WATCH: Here's how some provinces are discouraging social gatherings The provinces have opened up non-compliance hotlines or websites for the public to report people who are not following social distancing or isolation rules. 1:56 Sports venues such as fields and courts are closed to discourage gatherings. Park spaces remain open but playgrounds are closed.  Quebec schools are closed until at least May, while Ontario has launched an e-learning program while its schools remain closed, likely past the initial date of April 6. WATCH: CBC Ottawa's Local Daily for Thursday, March 26 Lucy van Oldenbarneveld, Adrian Harewood and Omar Dabaghi-Pacheco bring you the latest on the COVID-19 pandemic in Ottawa. 13:08 Public transit authorities are scaling back service because ridership has dropped substantially.  Ottawa residents needing information can still call 311, and all essential services such as garbage and recycling collection, as well as some bylaw services, will continue.  Service Canada is closing its centres to in-person visits as of Friday, March 27, focusing on telephone and online work. Spread of COVID-19 in Ottawa Ontario's Ministry of Health says 32 people have now tested positive for COVID-19 in Ottawa. Their Friday morning update did not include a local breakdown.  Health Canada worker tests positive for COVID-19 Senators announce 2nd player dealing with coronavirus  Ottawa Public Health says it is investigating 75 confirmed cases in the city, a total which includes cases that have tested positive once but have yet to be validated by a second test.  The city saw its first COVID-19-related death on March 25, a man in his 90s with no travel history.  WATCH: Doctors answer your questions about the coronavirus Doctors answer your questions about the coronavirus in Canada, including whether it’s necessary to change clothes after work. 5:05 Ottawa's medical officer of health has said last weekend computer models suggest hundreds and hundreds of people in Ottawa have COVID-19, many without knowing it. Vera Etches said the virus could infect 4,000 people a day at its peak if physical distancing and self-isolation recommendations aren't respected. Ontario has 993 confirmed cases of COVID-19. Quebec has 2,021 presumptive and confirmed cases. Fifty-three deaths in Canada have been linked to COVID-19, including 18 each in Ontario and Quebec.   What are the symptoms of COVID-19? Symptoms of COVID-19 range from a very mild, cold-like illness to a severe lung infection. The most common symptoms include fever, fatigue and a dry cough. They may take up to 14 days to appear, which is why that's the period of self-isolation. Older people, those with compromised immune systems and those with underlying medical problems such as high blood pressure, heart problems or diabetes are more likely to develop serious illness. WATCH: Here's how Canadians are helping each other through the pandemic How Canadians across the country are helping each other through the COVID-19 pandemic. 3:04 The coronavirus primarily spreads through droplets when an infected person coughs or sneezes. The virus can also spread through close, prolonged contact, such as touching or handshaking, and via surfaces such as door handles, mobile phones, tables and light switches if they touch their eyes, nose or mouth before washing their hands. When to get tested Anyone in Ottawa who has a new or worsening cough or fever and has travelled outside Canada, or has been in contact with a confirmed case, should go to the COVID-19 screening centre at the Brewer Arena. The centre is open from 9 a.m. to 8 p.m. daily at 151 Brewer Way, off Bronson Avenue near Carleton University. You don't have to call ahead.  If you meet some of the criteria but don't have symptoms, you won't be tested and should self-isolate for 14 days. If you have severe symptoms, call 911. WATCH: Ottawa homeless shelters trying to prevent the spread of COVID-19 Wendy Muckle, executive director of Ottawa Inner City Health, says shelters are taking as many precautions as possible to limit the spread of COVID-19, but with so many shelters near or at capacity, social distancing is still a challenge. 0:57 In western Quebec: Gatineau's downtown assessment location is at 135 blvd. Saint-Raymond. Outaouais resident should call the regional help line at 819-644-4545 if they have a cough or fever, whether they've travelled or not. If your symptoms require a trip to the emergency room, call ahead if your condition allows to let them know your travel history.  Kingston, Ont. The assessment centre in Kingston is now at the Kingston Memorial Centre at 303 York St. It is open 10 a.m. to 8 p.m. If you develop mild to moderate symptoms after travelling, either contact your health-care provider or go to the test site. Kingston's public health unit says to check its website for information, and call Telehealth at 1-866-797-0000 with any remaining questions. Other communities The public health unit in the Belleville, Ont., area is asking people only call it at 613-966-5500 if they've checked the website and still have questions. The same advice goes for Leeds, Grenville and Lanark's unit at 1-800-660-5853 extension 2499. It opened a testing site by referral only at the Brockville Memorial Centre at 100 Magedoma Blvd. that's open 11 a.m. to 6 p.m. daily. Referrals can come from a family doctor or the public health unit and will only be given to the sick and people who have left the country or been in close contact with a suspected or confirmed case. Hawkesbury, Ont., has an assessment centre at 750 Laurier St. open from 9 a.m. to 5 p.m. Monday to Friday.  Like Ottawa, only go there if you have new or worsening symptoms and have travelled or been in contact with a confirmed case. Go to CHEO if you're looking after an infant younger than six months old that fits this description. Self-isolate if you have mild symptoms, go to the hospital if your symptoms are severe. Only people older than age 70, who have chronic health problems or compromised immune systems can call 613-933-1375 from 8 a.m. to 8 p.m. to ask about a home visit from paramedics. WATCH: Violinist performs for grandmother quarantined in retirement home Viera Zmiyiwsky planned a surprise violin performance for her grandma who is quarantined in her retirement home, but she ended up with a bigger audience than expected. 1:56 Renfrew County is providing home testing under some circumstances. Its public health unit says people who have symptoms or have been in close contact with a confirmed case should use the province's self-assessment tool. Call Telehealth<U+200B><U+200B><U+200B>, their health care provider or it at 613-735-8654 if they still have more questions. Anyone who doesn't have or can't reach a family doctor can call its new primary care centre at 1-844-727-6404 if they have questions about their health. The province of Ontario generally advises people experiencing symptoms to call Telehealth or their health care provider. There has been a lag of four days or more to get Ontario's test results, with more being done per day than can be processed.  The province says it's doubling its testing capacity by the end of the week and nearly quadrupling that by mid-April. In the Outaouais, the local health agency is calling anyone whose tests take more than a week to get back to them. It said its goal is a result in 24 to 48 hours. First Nations communities The Mohawk communities of Akwesasne and Mohawks of the Bay of Quinte (MBQ) have declared a state of emergency to prepare for possible cases. Anyone in MBQ who has symptoms can call 613-967-3603 to talk to a nurse. A home test may be possible after that. In Akwesasne, community members are asked to carry their status cards when crossing the Canada-U.S. border for essential trips. The Algonquin communities of Kitigan Zibi and Pikwakanagan have scaled back non-essential services and are asking residents to follow public health advice. Pikwakanagan's election on Saturday, March 28 is going ahead, with members strongly encouraged to vote remotely. For more information, visit:  Ottawa Public Health, your local eastern Ontario health unit, the Ontario Ministry of Health (in several languages), including their self-assessment tool.  the Public Health Agency of Canada. the Centre intégré de santé et de services sociaux de l'Outaouais (in French). 
#>  8:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       Michigan hospitals are bracing for a surge of coronavirus cases as infections rise to at least 2,294. Deaths jumped to 43 from 24. Eighty-five per cent of all cases were reported in Wayne, Oakland and Macomb counties, but even the Upper Peninsula has a few. Beaumont Health and Henry Ford Health System in southeastern Michigan said they were caring for more than 1,000 patients at their 13 hospitals. Altus, a company near Grand Rapids, making hundreds of carts for hospital ventilators, which help desperate COVID-19 victims breathe. Altus is hiring more people and adding shifts. More from CBC News Windsor: Detroit-area hospital system 'almost at capacity,' says Michigan governorFront-line ER doctor says health-care workers are preparing for 'hordes of people' with COVID-19PhotosWindsor Islamic Association creates helpline to ensure seniors are fed
#>  9:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  The number of confirmed COVID-19 cases in the U.S. rose to 83,836 on Thursday — more than any other country, overtaking both Italy and China — while the death toll passed the 1,000 mark, according to a running tally by Johns Hopkins University. The university said Thursday at least 1,209 people in the U.S. have died from the disease caused by the coronavirus. Meanwhile, hospitals and government authorities in New York, New Orleans and other hot spots grappled with a dire shortage of supplies, staff and sick beds. Medical facilities were running short of ventilators and protective masks, and were being hampered by limited testing capacity. The U.S. Centers for Disease Control and Prevention (CDC) reported an increase of 13,987 cases and 257 deaths in its most recent measuring period, between Tuesday and Wednesday at 4 p.m. ET. The increases in the previous 24-hour period were 144 deaths and about 11,000 new cases. To help businesses deal with the outbreak, the Trump administration has decided it will be easing back on environmental enforcement and compliance requirements, according to an Environmental Protection Agency memo. The decision follows requests by the energy industry and other sectors. AnalysisPoliticians who consider sacrificing the old for the sake of the economy face a backlash: Don PittisNo, the new coronavirus wasn't created in a lab, scientists say """"In general, the EPA does not expect to seek penalties for violations of routine compliance monitoring, integrity testing, sampling, laboratory analysis, training, and reporting or certification obligations in situations where the EPA agrees that COVID-19 was the cause,"""" the EPA memo read. On the same day — seemingly responding to an unprecedented stimulus awaiting approval by the U.S. House of Representatives — Wall Street rallied for a third straight session. The S&P 500 gained 154.51 points, or 6.24 per cent, and the Dow Jones rose 1,351.62 points, or 6.38 per cent. Those gains capped off the largest three-day percentage gains since 1931 and 1933 respectively, while the Dow's rise of more than 20 per cent since March 23 meets some analysts' definition of a bull market. New York as the U.S. epicentre More than one-third of COVID-19 fatalities in the United States have been in New York state, where Gov. Andrew Cuomo has warned hospitals could soon run out of beds and ventilators. In his daily news conference Thursday, Cuomo said that 385 have died in the state from the virus, an increase from the previous day of 100. He said the goal was to get to a hospital-bed capacity of 140,000, up from the current 53,000 available Authorities were scouting new sites to build temporary facilities, he said. Patients wear masks as they wait in line for a COVID-19 test at Elmhurst Hospital Center on Wednesday in New York City.   (John Minchillo/The Associated Press) """"The number of ventilators we need is so astronomical. It's not like they have them sitting in the warehouse … there is no stockpile available,"""" Cuomo said. The federal government was sending """"everything we can"""" to help New York, including ventilators, gloves and protective masks for use in hospitals, White House trade adviser Peter Navarro said. """"But we've also got an emerging problem in New Orleans. We got Detroit, we've got Chicago, Seattle, California, my home state. Like I said, there's planes in the air everywhere now,"""" Navarro added. THE LATESTCoronavirus: What's happening in Canada and around the world on March 26Canada 'strongly opposed' to U.S. stationing troops near shared border Dr. Anthony Fauci, head of National Institute of Allergy and Infectious Diseases, called the acceleration of cases in New York """"quite disturbing."""" He also warned that the virus could return for the next northern winter. """"We hope we get a respite as we get into April, May and June. It is likely to come around next season because it's a very vigorous virus,"""" Fauci told WNYC public radio during an interview in New York on Thursday. New York's neighbour is also dealing with a surge in cases. New Jersey Gov. Phil Murphy said cases there had increased by nearly 2,500 from the previous day, with the death toll spiking from 19 to 81. Rural Georgia, Louisiana dealing with spikes The rampant infections in New York City have dominated much of the national conversation about the disease. But far from the coasts, smaller communities are also preparing for things to get worse, as they have in Albany, Ga. The largest hospital in Georgia's mostly rural, southwest corner is rapidly running out of space amid the highest rate of coronavirus infection in the entire state. It was believed to have started when a person with the virus came into the region to attend a funeral. A view of Bourbon Street in New Orleans is shown on Wednesday amid the COVID-19 outbreak. (Jonathan Bachman/Reuters) """"I can't tell you how it's going to be Friday, much less in the week after,"""" said Scott Steiner, CEO of the four-hospital health system that includes Phoebe Putney Memorial Hospital in Albany. At Phoebe Putney, intensive care beds are filled with COVID-19 patients and employees are hand-sewing masks to help stretch dwindling supplies. Roughly 90,000 people live in Albany and surrounding Dougherty County. The virus's impact has been outsized compared with the community's population. Infections countywide have surpassed 100, including seven deaths, making it Georgia's hardest-hit county outside metro Atlanta. In New Orleans, doctors and hospital officials are seeing an increased number of patients needing intensive care and are working to avoid the possible overwhelming of their systems. Louisiana officials are considering housing patients in hotels and a convention centre in the city. """"We are seeing an escalation in cases across our system,"""" said Warner Thomas, chief executive at Ochsner Medical Center, Louisiana's largest hospital system. Underscoring the threat to health-care workers striving to cope with the pandemic, Thomas said 300 Ochsner employees were under quarantine, including 60 diagnosed with COVID-19. Fed chair is listening to pandemic experts Hospitals, laid-off workers and struggling companies will receive badly needed economic aid under the record-setting relief bill approved by the U.S. Senate late on Wednesday in a 96-0 vote. Democratic House Speaker Nancy Pelosi said she expects that her chamber will approve the bill in a strong bipartisan vote on Friday. President Donald Trump has said he would sign the bill into law. Americans should receive direct deposits for financial aid within three weeks as soon as the bill is signed into law, the U.S. Treasury Secretary Steven Mnuchin said on Thursday.   U.S. Senate passes $2T coronavirus relief packageU.S. jobless claims soared to record 3.2M last week as COVID-19 crisis took hold Jobless claims soared to a record 3.3 million on Thursday, nearly five times the previous weekly record of 695,000 from the recession of 1982. The report may understate the problem as the official statistics typically have not included the self-employed or independent contractors. Federal Reserve Chair Jerome Powell said the United States already """"may well be in recession."""" Powell warned that reactivating the economy would have to wait until the virus was under control, despite Trump's stated desire to resume economic activity by Easter, April 12. """"The first order of business will be to get the spread of the virus under control and then resume economic activity,"""" Powell told NBC's Today Show. Danni Askini had to fight tooth and nail just to get tested for the coronavirus. Now she's fighting to cover the cost of her treatment. 7:32 Powell said he expects economic activity """"to resume and move back up in the second half of the year,"""" but that the board was paying attention to the warnings of experts like Fauci.  Most confirmed U.S. military cases are domestic The U.S. military, meanwhile, has decided it will stop providing some of the more granular data about coronavirus infections within its ranks. U.S. Defence Secretary Mark Esper said he wanted some of the more mission-specific information to be withheld to prevent compromising operational security. """"I'm not going to get into a habit where we start providing numbers across all the commands and we come to a point six, seven weeks from now where we have some concerns in some locations and reveal information that could put people at risk,"""" said Esper. Officials told Reuters there are more domestic cases than among forces overseas. About 85 per cent of confirmed coronavirus cases in the air force were of personnel in the U.S., while the navy said roughly 90 per cent of its cases were located domestically. Esper noted that commanders overseas have greater ability to impose restrictions on the movement of troops and their families.
#> 10:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         The Jewish General Hospital plans to set up trailers outside its main building, where people with minor COVID-19 symptoms can seek medical attention. Unlike the drive-thru testing clinic going up in the parking lot of the Cavendish Mall in Côte Saint-Luc this weekend, the trailers will be more medically oriented, said Francine Dupuis, the associate CEO of the CIUSSS Central West.West-Central Montreal, the health agency that oversees the hospital. After evaluating the patient, doctors will determine if that person is well enough to be sent home to wait until the symptoms disappear, or whether they should be sent to a clinic or to the emergency ward. """"We really want people to avoid the emergency,"""" said Dupuis. """"It's the last place to go."""" """"Those who go there should be sent there because a doctor has determined that their symptoms were acute enough."""" On the front lines: What life is like for a nurse at Montreal's pop-up COVID-19 testing site As soon as the trailers are set up, within the next week or so, an infection control specialist will figure out how to make them safe to ensure patients don't contaminate one another, she said. """"We are looking for doctors in the community to staff these trailers,"""" she said. Dupuis said anyone who has mild symptoms could be evaluated there. Spreading out COVID-19 patients Senior administrator Francine Dupuis says the aim of setting up trailers outside the Jewish General Hospital is to keep COVID-positive patients out of the emergency ward. 'Those who go there should be sent there because a doctor has determined that their symptoms were acute enough,' Dupuis said. (CIUSSS West-Central Montreal) At the beginning of the crisis, the Jewish General Hospital was designated as the main COVID-19 treatment centre in Montreal for acutely ill adult patients. But as the number of positive cases rises, the province has designated more hospitals in the Montreal region to spread out the load. """"It's better to share with other hospitals, because if the other hospital has a fair number of ICU beds, it allows us to continue giving services to other sick people,"""" said Dupuis. """"It's a balance."""" As of Thursday night, Dupuis said the Jewish General had 37 COVID-19 positive patients, with 14 of those in the intensive care unit. """"But it changes every hour,"""" she warned.""""So it's very dangerous to give the numbers because the very next hour may be different."""" Nurses treating COVID-19 patients say they urgently need protective gear For acutely ill, recovery is slow Recovery is a slow process, but Dupuis confirmed some patients in ICU have improved enough to be moved to a unit for less acutely ill patients or have gone home. But with so much still unknown about the virus, Dupuis said, those patients are being closely monitored. """"It's not clear how long we have to monitor them before they are completely out of [danger],"""" said Dupuis. She said health care teams here have looked at the Hong Kong experience, where in some cases, patients started doing better, and hospitals """"stopped being very conservative in terms of monitoring, and the contagion started going up again."""" Dupuis said she believes a careful, cautious approach — including the social isolation provisions Quebec now has in place — will keep the number of deaths low in Quebec compared to what some European countries are witnessing. Staffing levels remain good at the hospital, Dupuis said. Nurses who are acute care specialists are where they should be, leaving other nurses to cover less critically ill patients. """"We've also had a large number of nurses, more recently retired, who decided to come back and help us,"""" said Dupuis. The cancellation of elective surgeries has also freed up some doctors who can help cover off other areas of the hospital. Dupuis said the Jewish General still has room to treat COVID-19 patients and considers the situation under control and manageable. """"Our staff is very well-trained, and it's quite under control,"""" said Dupuis.
#>                                                                                                                      url
#>  1:                                                        https://www.cbc.ca/news/health/covid-19-drug-supply-1.5511659
#>  2:                             https://www.cbc.ca/news/canada/north/yukon-coronavirus-exposure-bethany-church-1.5511906
#>  3:                                               https://www.cbc.ca/news/world/senate-coronavirus-relief-bill-1.5510475
#>  4:                               https://www.cbc.ca/news/health/coronavirus-treatment-and-vaccine-development-1.5511346
#>  5:                                https://www.cbc.ca/news/the-latest-on-the-coronavirus-outbreak-for-march-26-1.5511398
#>  6:                                  https://www.cbc.ca/news/world/wildlife-markets-china-coronavirus-pandemic-1.5510045
#>  7:             https://www.cbc.ca/news/canada/ottawa/covid19-coronavirus-ottawa-symptoms-information-march-27-1.5511735
#>  8:                                      https://www.cbc.ca/news/canada/windsor/michigan-hospitals-coronavirus-1.5510518
#>  9:                                                      https://www.cbc.ca/news/world/us-coronavirus-thursday-1.5510581
#> 10: https://www.cbc.ca/news/canada/montreal/jewish-general-hospital-to-set-up-trailers-for-covid-19-evaluation-1.5511901
```

Based on the title of the sample news, some article reported about the virus spread and transmission while some others talk about what the government does in response to the COVID-19. We will explore it further using the topic modelling.

### Text Cleansing

We will clean the text first before proceed further. The text cleansing process includes:

* lowercase all text
* remove html tag 
* lengthen shortened words (don't, I'll)
* replace all `-` with white space
* remove the common word of `coronavirus`, `covid 19`, and `covid`
* remove all punctuation
* replace all numeric character into words
* remove double white space
* remove white space at the start and the end of text


```r
covid_clean <- covid_news %>% 
   mutate(text_clean = text %>% 
             replace_non_ascii() %>% 
             replace_html(symbol = F) %>% # remove html tag
             str_replace_all("[0-9]", " ") %>% 
             str_replace_all("[-|]", " ") %>% # replace "-" with space
             tolower() %>% #lowercase
             str_remove_all("coronavirus|covid 19|covid|canadian|canadians") %>%  # remove common words
             replace_symbol() %>%
             replace_contraction() %>% 
             replace_word_elongation() %>%  # lengthen shortened word
             str_replace_all("[[:punct:]]", " ") %>% # remove punctuation
             str_replace_all(" dr ", " doctor ") %>% 
             make_plural() %>%
             str_replace_all(" s ", " ") %>%  
             str_squish() %>% # remove double whitespace
             str_trim() # remove whitespace at the start and end of the text
          )
```

Let's see the remaining number of words on each document in our corpus.


```r
document_length <- sapply(strsplit(covid_clean$text, " "), length)

document_length %>% 
   summary()
```

```
#>    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
#>    14.0   431.0   632.0   713.1   896.0  5100.0
```

The shortest document has 14 words while the longest has 5100 words. LDA will works better if the text input has a lot of words inside the sentence. We will filter document that has at least consists of 100 words.


```r
covid_clean <- covid_clean %>% 
   slice(which(document_length > 100))

dim(covid_clean)
```

```
#> [1] 2710    8
```

We have 2,710 number of documents remanining.

### Document-Term Matrix

The next step is to tokenize the text and create a Document-Term Matrix (DTM) from our text data. We will also remove any stop words such as `the` or `is` since they are irrelevant for this problem. We will also do stem words into their basic form, such as from `walking` into `walk`. To get better stemming, we also change all `positive` terms into `positives`. The stemming function is created manually by with `hunspell`[^7] stemming at its core. `hunspell` stemming give better result compared to the basic stemming algorithm such as Porter stemming algorithm[^8].


```r
stem_hunspell <- function(term) {
    # look up the term in the dictionary
    stems <- hunspell_stem(term)[[1]]
    
    if (length(stems) == 0) { # if there are no stems, use the original term
        stem <- term
    } else { # if there are multiple stems, use the last one
        stem <- stems[[length(stems)]]
    }
    return(stem)
}

news_term <- covid_clean %>% 
   unnest_tokens(output = "word", input = text_clean) %>% 
   anti_join(stop_words)  %>% 
   mutate(word = ifelse(word == "positive", "positives", word),
          word = text_tokens(word, stemmer = stem_hunspell) %>% as.character() ) %>% 
   drop_na(word) %>% 
   count(V1, word)
```

Next, we will transform the data into document-term matrix (DTM). The value inside the matrix represent the `term frequency` or the number of terms appear inside each document.


```r
dtm_news <- news_term %>% 
   cast_dtm(document = V1, term = word, value = n)

inspect(dtm_news)
```

```
#> <<DocumentTermMatrix (documents: 2710, terms: 25191)>>
#> Non-/sparse entries: 511876/67755734
#> Sparsity           : 99%
#> Maximal term length: 29
#> Weighting          : term frequency (tf)
#> Sample             :
#>      Terms
#> Docs  canada day govern heal home people province public test virus
#>   106     27  22     13   26   10     33       26      5   20    13
#>   141     18  12      9   28    4     26       21     16   19    16
#>   172     23  14     10   25   14     20       18     11   25    16
#>   18       8   7      2   79   16     28       42     24   41    14
#>   197     15  11      7   23   16     24       15     17   16    15
#>   212     16  13     10   23   13     36       21      8   26    12
#>   305     25  17     15   23   14     20       13     16    2     7
#>   35      12  19     12   25   12     35       15      8   11     8
#>   49      18  11      9   27   20     34       14     11   15     5
#>   56      19  21     10   27   19     22       17     10   22    10
```

We have 2,710 documents with total terms of more than 28,000 terms. We will remove rare word that occur only in less than 5 documents and also the common words that appear in more than 90% of all documents. This is intended to give us a collections of terms that is common enough and shared by several documents to indicate a shared topics/latent information but also unique enough that it is not shared by all documents.


```r
word_freq <- findFreqTerms(dtm_news, 
                           lowfreq = 5, 
                           highfreq = nrow(dtm_news)*0.9
                           )

dtm_news <- dtm_news[ , word_freq]
dtm_news
```

```
#> <<DocumentTermMatrix (documents: 2710, terms: 9138)>>
#> Non-/sparse entries: 442433/24321547
#> Sparsity           : 98%
#> Maximal term length: 19
#> Weighting          : term frequency (tf)
```

The number of terms drastically drop from 29,000 to around 9,000 terms. We will use this data to train the LDA model.

### Topic Modelling with LDA

We will create an LDA model with `k = 3` topics. The choice of number of topics is arbitrary, but we will show you how to find the optimal number of topics later. We will use Gibbs-sampling to estimate the parameter using 5000 iterations of sampling and 4000 burn-in iterations. The burn-in iteration means that we only collecting samples starting from iteration of 4000, since the earlier iteration is still unstable and may not reflect the actual distribution of the data.

Since the computation is quite long (around 30-60 minutes), I have prepared the pre-trained model in the next chunk.


```r
dtm_lda <- Matrix::Matrix(as.matrix(dtm_news), sparse = T)

set.seed(123)
lda_news <- FitLdaModel(dtm = dtm_lda, 
                        k = 3, 
                        iterations = 5000,
                        burnin = 4000, 
                        calc_coherence = T
                        )
```

The details about the parameter of `FitLdaModel()` function is as follows:

* dtm = input data, must be in the form of document-term matrix (dtm)
* k = number of topics
* iterations = maximum number of iterations


```
#> List of 9
#>  $ phi           : num [1:3, 1:9138] 0.000000166 0.000000256 0.000020914 0.001365947 0.001274729 ...
#>   ..- attr(*, "dimnames")=List of 2
#>   .. ..$ : chr [1:3] "t_1" "t_2" "t_3"
#>   .. ..$ : chr [1:9138] "acetaminophen" "act" "actual" "america" ...
#>  $ theta         : num [1:2710, 1:3] 0.547362 0.997833 0.000236 0.043015 0.456297 ...
#>   ..- attr(*, "dimnames")=List of 2
#>   .. ..$ : chr [1:2710] "0" "1" "2" "3" ...
#>   .. ..$ : chr [1:3] "t_1" "t_2" "t_3"
#>  $ gamma         : num [1:3, 1:9138] 0.012 0.0121 0.9759 0.5079 0.3081 ...
#>   ..- attr(*, "dimnames")=List of 2
#>   .. ..$ : chr [1:3] "t_1" "t_2" "t_3"
#>   .. ..$ : chr [1:9138] "acetaminophen" "act" "actual" "america" ...
#>  $ data          :Formal class 'dgCMatrix' [package "Matrix"] with 6 slots
#>   .. ..@ i       : int [1:442433] 0 17 1311 0 2 4 12 17 18 19 ...
#>   .. ..@ p       : int [1:9139] 0 3 531 570 725 929 943 1337 1479 1502 ...
#>   .. ..@ Dim     : int [1:2] 2710 9138
#>   .. ..@ Dimnames:List of 2
#>   .. ..@ x       : num [1:442433] 1 1 3 4 1 3 12 7 1 3 ...
#>   .. ..@ factors : list()
#>  $ alpha         : Named num [1:3] 0.1 0.1 0.1
#>   ..- attr(*, "names")= chr [1:3] "t_1" "t_2" "t_3"
#>  $ beta          : Named num [1:9138] 0.05 0.05 0.05 0.05 0.05 0.05 0.05 0.05 0.05 0.05 ...
#>   ..- attr(*, "names")= chr [1:9138] "acetaminophen" "act" "actual" "america" ...
#>  $ log_likelihood:'data.frame':	500 obs. of  2 variables:
#>   ..$ iteration     : num [1:500] 0 10 20 30 40 50 60 70 80 90 ...
#>   ..$ log_likelihood: num [1:500] -5815798 -5770673 -5741072 -5732850 -5729403 ...
#>  $ coherence     : Named num [1:3] 0.108 0.11 0.144
#>   ..- attr(*, "names")= chr [1:3] "t_1" "t_2" "t_3"
#>  $ labels        : chr [1:3, 1:2] "school" "dollar" "quarantine" "social" ...
#>   ..- attr(*, "dimnames")=List of 2
#>   .. ..$ : chr [1:3] "t_1" "t_2" "t_3"
#>   .. ..$ : chr [1:2] "label_1" "label_2"
#>  - attr(*, "class")= chr "lda_topic_model"
```

Below are some important attribute acquired from the LDA Model:

* **phi** : Posterior per-topic-per-word probabilities
* **theta** : Posterior per-document-per-topic probabilities
* **alpha** : Prior per-document-per-topic probabilities
* **beta** : Prior per-document-per-topic probabilities
* **coherence** : The probabilistic coherence of each topic

If a term has a high value of theta, it has a high probability of that term being generated from that topic. This also indicates that the term has a high association toward a certain topic.


```r
lda_news$theta %>% 
   head() %>% 
   as.data.frame() %>% 
   set_names(paste("Topic", 1:3)) %>% 
   rownames_to_column("document")  
```

```
#>   document      Topic 1     Topic 2     Topic 3
#> 1        0 0.5473617292 0.134138589 0.318499682
#> 2        1 0.9978331528 0.001083424 0.001083424
#> 3        2 0.0002356823 0.942964883 0.056799434
#> 4        3 0.0430145752 0.107003199 0.849982225
#> 5        4 0.4562965655 0.249136834 0.294566600
#> 6        5 0.0441518203 0.315259489 0.640588691
```

Remember that LDA assumes that a topic is a mixture of words. The posterior probability for per-topic-per-word assignment is represented by the *phi* value. The sum of all phi for a topic is 1.


```r
lda_news$phi %>% 
   rowSums()
```

```
#> t_1 t_2 t_3 
#>   1   1   1
```

To get the top terms for each topic, we can use the `GetTopTerms` function.


```r
GetTopTerms(lda_news$phi, 10) %>% 
   as.data.frame()
```

```
#>         t_1      t_2        t_3
#> 1    school   dollar       port
#> 2    social     cent quarantine
#> 3  distance minister     flight
#> 4     staff  company       ease
#> 5    office   market      wuhan
#> 6   student  support       ship
#> 7      stay  federal  passenger
#> 8      hand    price     cruise
#> 9    centre     busy    patient
#> 10  medical business      world
```

### Exploration

As we have stated earlier, LDA merely give us the hidden/latent structure inside the corpus of our documents. It is our job as the user to interpret the latent information and assign labels for each generated topic.

#### Word-Topic Probabilities

Below is the top words for each topic. LDA doesn't specifically inform us about what each topic is about. By looking at the representative words of each topic, we as the human will give meaning to each topic. The top terms in the first topic seems to tells us about the impact of the virus outbreak, as indicated by the word `school`, `social distance`, `store`, and `student`. The second topic talks about the politics and economy regarding the COVID-19, since we see the presence of words `cent`, `federal`, `minister`, and `company`. The third topic tell us about the spread and transmission of COVID-19, indicated by the word `patient`, `dr` (doctor), `quarantine`, and `flight`.

As we can see, by using LDA, even though we don't have the true labels or class, the model can generate association between words and topics by assigning probabilities. Using k = 3, we have 3 topics that can easily interpreted since they are quite different from each other.


```r
news_word_topic <- GetTopTerms(lda_news$phi, 30) %>% 
   as.data.frame() %>% 
   set_names(paste("Topic", 1:3))

news_word_topic
```

```
#>       Topic 1   Topic 2       Topic 3
#> 1      school    dollar          port
#> 2      social      cent    quarantine
#> 3    distance  minister        flight
#> 4       staff   company          ease
#> 5      office    market         wuhan
#> 6     student   support          ship
#> 7        stay   federal     passenger
#> 8        hand     price        cruise
#> 9      centre      busy       patient
#> 10    medical  business         world
#> 11     family  economic       chinese
#> 12   announce   economy       medical
#> 13   continue    impact     infection
#> 14      store  announce          turn
#> 15  essential   million           new
#> 16        new    crisis        infect
#> 17    measure      lead       toronto
#> 18    provide     world          mask
#> 19       risk    budget           ill
#> 20      chief      bank      positive
#> 21    patient   trudeau         italy
#> 22    ontario      plan          risk
#> 23     supply       oil      minister
#> 24    protect       pay        deaths
#> 25     monday  industry          told
#> 26 provincial president       ontario
#> 27   resident   billion international
#> 28     online     party       airport
#> 29   employee     trump        family
#> 30       food      fund        global
```

We can also present top words in each topic using visualization. Here, we will visualize the top 50 terms in each topics using word cloud.


```r
news_word_topic %>% 
   rownames_to_column("id") %>%
   mutate(id = as.numeric(id)) %>% 
   pivot_longer(-id, names_to = "topic", values_to = "term") %>% 
   ggplot(aes(label = term, size = rev(id), color = topic, alpha = rev(id))) +
   geom_text_wordcloud(seed = 123) +
   facet_wrap(~topic, scales = "free") +
   scale_alpha_continuous(range = c(0.4, 1)) +
   scale_color_manual(values = c( "dodgerblue4", "firebrick4", "darkgreen")) +
   theme_minimal() +
   theme(strip.background = element_rect(fill = "firebrick"),
         strip.text.x = element_text(colour = "white"))
```

<img src="/blog/topic-model-lda_files/figure-html/unnamed-chunk-26-1.png" width="100%" style="display: block; margin: auto;" />

#### Document-Topic Probabilities {.tabset}

We can also acquire the probability of a document belong to certain topics. We will use this metric to check whether our guest about the interpretation of each topic is make sense and if each topic is different enough subjectively. 

##### Social Issues

The following table shows the top 10 news title that has the highest probability to the topic of social issues of coronavirus.


```r
news_doc_topic <- lda_news$theta %>% 
   as.data.frame() %>% 
   rownames_to_column("id") 

news_doc_topic %>% 
   arrange(desc(t_1)) %>% 
   left_join(covid_clean %>% 
                mutate(V1 = as.character(V1)) %>% 
                select(V1, title), 
             by = c("id" = "V1")) %>% 
   column_to_rownames("id") %>% 
   select(title, everything()) %>% 
   head(10)
```

```
#>                                                                                             title
#> 4126                           No school for Manitoba K-12 students for 3 weeks starting March 23
#> 234  Online screening tool for coronavirus up and running in Manitoba, chief nursing officer says
#> 2693         'Really upsetting': YK1 school board votes to close for remainder of the school year
#> 2698                On-reserve COVID-19 testing set up in Algonquin First Nation after complaints
#> 2716                      Universities in Waterloo, Guelph suspend classes amid COVID-19 concerns
#> 3060               COVID-19 concerns prompt West Vancouver school to close early for spring break
#> 1768             School is out! Here's some advice for kids — from kids — on how to pass the time
#> 3374                                    UPEI ending in-person classes for semester after March 20
#> 3004                  All Edmonton recreation centres to close indefinitely amid COVID-19 worries
#> 2744          4 COVID-19 cases confirmed in Manitoba, chief provincial public health officer says
#>            t_1          t_2          t_3
#> 4126 0.9995219 0.0002390629 0.0002390629
#> 234  0.9993318 0.0003341129 0.0003341129
#> 2693 0.9992518 0.0003741115 0.0003741115
#> 2698 0.9992041 0.0003979308 0.0003979308
#> 2716 0.9991978 0.0004011231 0.0004011231
#> 3060 0.9991978 0.0004011231 0.0004011231
#> 1768 0.9991913 0.0004043672 0.0004043672
#> 3374 0.9991847 0.0004076641 0.0004076641
#> 3004 0.9991536 0.0004231909 0.0004231909
#> 2744 0.9991201 0.0004399472 0.0004399472
```

##### Politics and Economy

The following table shows the top 10 news title that has the highest probability to belong the topic of politics and economy.


```r
news_doc_topic %>% 
   arrange(desc(t_2)) %>% 
   left_join(covid_clean %>% 
                mutate(V1 = as.character(V1)) %>% 
                select(V1, title), 
             by = c("id" = "V1")) %>% 
   column_to_rownames("id") %>% 
   select(title, everything()) %>% 
   head(10)
```

```
#>                                                                                                   title
#> 4479       Alberta premier says layoffs loom as oilpatch warns of 'catastrophic' impact from low prices
#> 301                          TSX loses another 8% as Canadian oil price falls to lowest level on record
#> 3375                                       'Batten down the hatches': Oilpatch braces as prices plummet
#> 531                                 Oil plunge, coronavirus fears prompt panic selling on stock markets
#> 2406                            'Agonizing': The prospect of a historic oil glut weighs on crude prices
#> 183                                         Loonie dips below 70 cents US as coronavirus impact lingers
#> 3566 'They squandered the good times': Poilievre says Canada lacks 'cushion' to weather economic crises
#> 4435           NDP says Premier Scott Moe is 'playing political games' on possibility of early election
#> 3443                     Singh calls on Liberals to start processing applications for financial aid now
#> 4285                         Gas prices set to fall 7 cents a litre across Canada on Wednesday: analyst
#>               t_1       t_2          t_3
#> 4479 0.0002479544 0.9995041 0.0002479544
#> 301  0.0003297066 0.9993406 0.0003297066
#> 3375 0.0003307972 0.9993384 0.0003307972
#> 531  0.0003352330 0.9993295 0.0003352330
#> 2406 0.0003432887 0.9993134 0.0003432887
#> 183  0.0003672420 0.9992655 0.0003672420
#> 3566 0.0004060089 0.9991880 0.0004060089
#> 4435 0.0004076641 0.9991847 0.0004076641
#> 3443 0.0004161465 0.9991677 0.0004161465
#> 4285 0.0004967710 0.9990065 0.0004967710
```

##### Virus Transmission

The following table shows the top 10 news title that has the highest probability to belong the topic of virus transmission.


```r
news_doc_topic %>% 
   arrange(desc(t_3)) %>% 
   left_join(covid_clean %>% 
                mutate(V1 = as.character(V1)) %>% 
                select(V1, title), 
             by = c("id" = "V1")) %>% 
   column_to_rownames("id") %>% 
   select(title, everything()) %>% 
   head(10)
```

```
#>                                                                                                title
#> 3997                                    Death toll from new coronavirus outbreak in China reaches 17
#> 996                                               Timeline: How the new coronavirus arose and spread
#> 1058               Canadian doctor, WHO team heading to China, where coronavirus death toll tops 900
#> 1092                                   Canada ready to help China contain coronavirus if it asks: PM
#> 1429                                       Patient in Japan infected with new coronavirus from China
#> 1407           France confirms 3 cases as coronavirus outbreak spreads to Europe, U.S. and elsewhere
#> 1076                                Quebec couple, both 75, diagnosed with coronavirus aboard cruise
#> 4571                                        South Korea reports 1st possible case of viral pneumonia
#> 1223 1st person with coronavirus disease in B.C. has fully recovered, provincial health officer says
#> 658                          Ontario's 7th case of coronavirus reported in man who travelled to Iran
#>               t_1          t_2       t_3
#> 3997 0.0002255809 0.0002255809 0.9995488
#> 996  0.0003083565 0.0003083565 0.9993833
#> 1058 0.0004144219 0.0004144219 0.9991712
#> 1092 0.0004144219 0.0004144219 0.9991712
#> 1429 0.0004231909 0.0004231909 0.9991536
#> 1407 0.0004361099 0.0004361099 0.9991278
#> 1076 0.0004418913 0.0004418913 0.9991162
#> 4571 0.0004478280 0.0004478280 0.9991043
#> 1223 0.0004710316 0.0004710316 0.9990579
#> 658  0.0006049607 0.0006049607 0.9987901
```

#### Topic Proportion over Time

We will illustrate a distant view on the topics in the data over time. Let's see the range of date when each article is published.


```r
range(covid_clean$publish_date)
```

```
#> [1] "2019-12-22 18:36:00" "2020-03-27 08:30:00"
```

The first article start at the end of December 2019 and the latest article is on March 2020. We will group the data into weekly interval and see the proportion of each topic across the weeks.


```r
news_doc_topic %>% 
   left_join(covid_clean %>% 
                mutate(V1 = as.character(V1)) %>% 
                select(V1, title, publish_date), 
             by = c("id" = "V1")) %>% 
   select(-id) %>% 
   select(title, everything()) %>% 
   pivot_longer(c(t_1, t_2, t_3), names_to = "topic", values_to = "theta") %>% 
   mutate(topic = case_when( topic == "t_1" ~ "Social Issues",
                             topic == "t_2" ~ "Virus Transmission",
                             TRUE ~ "Politics and Economy") %>% 
             factor(levels = c("Virus Transmission", "Social Issues", "Politics and Economy")),
          publish_date = ymd_hms(publish_date),
          time = floor_date(publish_date, unit = "week") %>% as.Date()
          ) %>% 
   group_by(time, topic) %>% 
   summarise(theta = mean(theta)) %>% 
   ggplot(aes(time, theta, fill = topic, color = topic)) +
   geom_line() +
   geom_point(show.legend = F) +
   theme_minimal() +
   theme(legend.position = "top") +
   scale_x_date(date_breaks = "1 weeks", 
                labels = date_format(format = "%d\n%b")) +
   scale_y_continuous() +
   scale_fill_manual(values = c("firebrick", "orange", "dodgerblue3")) +
   labs(x = NULL, y = expression(theta), color = NULL, 
        title = "Topic Proportions Over Time on Weekly Interval")
```

<img src="/blog/topic-model-lda_files/figure-html/unnamed-chunk-31-1.png" width="672" style="display: block; margin: auto;" />

As we can see, for COVID-19 case (late 2019-2020), as the time goes, more articles are reporting more about the social issues of the coronavirus. There is a one week time gap between in the late December to early January where no news are collected. News regarding the source of the virus or new cases is less reported on March since almost all country have been contracted by the virus and thus people and the government are more concerned about their survival and well-being.

However, LDA can't accurately track the change inside the topic over time since LDA assume that the order of the document does not matter. This assumption may be unrealistic when analyzing long-running collections that span years or centuries, since a topic may change from time to time. For example, the topic Virus Transmission may containt mostly about the transmission and new cases in early 2020 and more about the virus source and characteristics in the later period. These kind of change inside the topic is not detected by LDA. A more advanced and improved version LDA that can accomodate this is the Dynamic Topic Model[^9], a model that respects the ordering of the documents and gives a richer posterior topical structure than LDA.

### Evaluating LDA

Although LDA is an unsupervised learning, we can still measure some of its performance. Traditionally, and still for many practical applications, to evaluate if “the correct thing” has been learned about the corpus, an implicit knowledge and “eyeballing” approaches are used. Ideally, we’d like to capture this information in a single metric that can be maximized, and compared.

#### Eye-Balling Test

The evaluation of a topic model can be done by looking at the content directly, such as the top-n words like what we previously did. We can decide whether the collection of words inside each topic make sense or contain certain similarity.


```r
news_word_topic
```

```
#>       Topic 1   Topic 2       Topic 3
#> 1      school    dollar          port
#> 2      social      cent    quarantine
#> 3    distance  minister        flight
#> 4       staff   company          ease
#> 5      office    market         wuhan
#> 6     student   support          ship
#> 7        stay   federal     passenger
#> 8        hand     price        cruise
#> 9      centre      busy       patient
#> 10    medical  business         world
#> 11     family  economic       chinese
#> 12   announce   economy       medical
#> 13   continue    impact     infection
#> 14      store  announce          turn
#> 15  essential   million           new
#> 16        new    crisis        infect
#> 17    measure      lead       toronto
#> 18    provide     world          mask
#> 19       risk    budget           ill
#> 20      chief      bank      positive
#> 21    patient   trudeau         italy
#> 22    ontario      plan          risk
#> 23     supply       oil      minister
#> 24    protect       pay        deaths
#> 25     monday  industry          told
#> 26 provincial president       ontario
#> 27   resident   billion international
#> 28     online     party       airport
#> 29   employee     trump        family
#> 30       food      fund        global
```

#### Intrinsic Measures

One of the most popular metric to evaluate a topic model is by looking at the topic coherence. Topic Coherence measures the degree of semantic similarity between the top words in a single topic. The `textmineR` implements a new topic coherence measure based on probability theory. Probabilistic coherence measures how associated words are in a topic, controlling for statistical independence.

> "Suppose you have a corpus of articles from the sports section of a newspaper. A topic with the words {sport, sports, ball, fan, athlete} would look great if you look at correlation, without correcting for independence. But we actually know that it’s a terrible topic because the words are so frequent in this corpus as to be meaningless. In other words, they are highly correlated with each other but they are statistically-independent of each other."
> <footer> Thomas W. Jones</footer>

The intuition of the probabilistic coherence is that it measure how probable a pair of words will come from the same documents than from a random document in the corpus. For example, if we the top 3 words of a topic is `apple`, `banana`, and `cheese`, we can calculate the topic coherence by averaging the following numbers:

1. `\(P(apple|banana) - P(banana)\)`

2. `\(P(apple|cheese) - P(cheese)\)`

3. `\(P(banana|cheese) - P(cheese)\)`

Description:

- `\(P(apple|banana)\)` : The probability that words apple and banana appear together in the same documents
- `\(P(banana)\)` : The probability that the word banana appear in the corpus as a whole

You can get the coherence for each topic by calling the `coherence` object from the LDA models. By default, the topic coherence only look for the top 5 words of each topic.


```r
lda_news$coherence
```

```
#>       t_1       t_2       t_3 
#> 0.1082933 0.1103960 0.1437167
```

We will try to find the optimal number of topics by finding the average probabilistic coherence for several number of topics, ranging from k = 10 to k = 100 with interval of 10. To speed up the computation, we will only use 200 sampling iterations with burnin iteration of 180 for the sake of illustration since higher number of iterations can run for hours or even days.


```r
dtm_lda <- Matrix::Matrix(as.matrix(dtm_news), sparse = T)
k_list <- seq(10, 100, by = 10)

model_list <- TmParallelApply(X = k_list, FUN = function(k){

  m <- FitLdaModel(dtm = dtm_lda, 
                   k = k, 
                   iterations = 500, 
                   burnin = 200,
                   calc_coherence = TRUE)
  
  m <- mean(m$coherence)
  
  return(m)
}, 
cpus = 4
)
```



```r
k_list <- seq(10, 100, by = 10)

model_list <- read_rds("data_input/coherence result.Rds")

iter_k <- data.frame(
   k = k_list,
   coherence = model_list %>% unlist()
)

iter_k %>%
   mutate(max_k = which(coherence == max(coherence)) * 10) %>% 
   ggplot(aes(k, coherence)) +
   geom_vline(aes(xintercept = max_k), alpha = 0.5, lty = "dashed") +
   geom_line(color = "skyblue4") +
   geom_point() +
   scale_x_continuous(breaks = seq(0, 200, 20)) +
   labs(x = "Number of Topics", y = "Coherence", title = "Coherence Score over Number of Topics") +
   theme_minimal() +
   theme(panel.grid.minor = element_blank())
```

<img src="/blog/topic-model-lda_files/figure-html/unnamed-chunk-35-1.png" width="672" style="display: block; margin: auto;" />

The optimal number of topics can be chosen by picking the number of topics that give the highest average coherence.

There are also other methods to evaluate the topic model. It will be too much to discuss them on this article. You can visit Julia Silge blogpost[^10] to see some of the evaluation metrics.

#### Extrinsic Measures

Model performance toward a specific task, such as text classification. If the topics is regarded as a feature for classification model, we can use accuracy or any other classification metrics to check if the topic model is good enough to do the job.


## Document Classification

LDA can also be treated as a model for dimensionality reduction. Each document can be reduce into its associated gamma values. Dimensionality reduction is crucial in text mining since using each word as a feature will result in large dataset and longer computation. In this section, we will compare the performance of text classification using different treatment of dimensionality reduction.

### Import Data

We will import data about workplace review. The data consists of 60,672 observations with the review text and the sentiment.




```r
df <- data.table::fread("data_input/review.csv")
glimpse(df)
```

```
#> Rows: 60,672
#> Columns: 2
#> $ Sentiment <chr> "positives", "negatives", "positives", "negatives", "positi…
#> $ Review    <chr> "People are smart and friendly", "Bureaucracy is slowing th…
```

### Text Cleansing

We will do the usual data cleansing:

- Lowecase all text
- Replace all html symbol
- Replace word elongation
- Remove all puncuations and numbers
- Remove extra white space



```r
df_clean <- df %>% 
   mutate(text_clean = Review %>% 
             tolower() %>% 
             replace_html() %>% 
             replace_word_elongation() %>% 
             str_replace_all("-", " ") %>% 
             str_remove_all("[[:punct:]]") %>% 
             str_remove_all("[[0-9]]") %>% 
             str_squish() %>% 
             str_trim())

df_clean %>% head()
```

```
#>    Sentiment
#> 1: positives
#> 2: negatives
#> 3: positives
#> 4: negatives
#> 5: positives
#> 6: negatives
#>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  Review
#> 1:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        People are smart and friendly
#> 2:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   Bureaucracy is slowing things down
#> 3:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           1) Food, food, food. 15+ cafes on main campus (MTV) alone. Mini-kitchens, snacks, drinks, free breakfast/lunch/dinner, all day, errr'day.  2) Benefits/perks. Free 24:7 gym access (on MTV campus). Free (self service) laundry (washer/dryer) available. Bowling alley. Volley ball pit. Custom-built and exclusive employee use only outdoor sport park (MTV). Free health/fitness assessments. Dog-friendly. Etc. etc. etc.  3) Compensation. In ~2010 or 2011, Google updated its compensation packages so that they were more competitive.  4) For the size of the organization (30K+), it has remained relatively innovative, nimble, and fast-paced and open with communication but, that is definitely changing (for the worse).  5) With so many departments, focus areas, and products, *in theory*, you should have plenty of opportunity to grow your career (horizontally or vertically). In practice, not true.  6) You get to work with some of the brightest, most innovative and hard-working/diligent minds in the industry. There's a ""con"" to that, too (see below).
#> 4:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        1) Work/life balance. What balance? All those perks and benefits are an illusion. They keep you at work and they help you to be more productive. I've never met anybody at Google who actually time off on weekends or on vacations. You may not hear management say, ""You have to work on weekends/vacations"" but, they set the culture by doing so - and it inevitably trickles down. I don't know if Google inadvertently hires the work-a-holics or if they create work-a-holics in us. Regardless, I have seen way too many of the following: marriages fall apart, colleagues choosing work and projects over family, colleagues getting physically sick and ill because of stress, colleagues crying while at work because of the stress, colleagues shooting out emails at midnight, 1am, 2am, 3am. It is absolutely ridiculous and something needs to change.  2) Poor management. I think the issue is that, a majority of people love Google because they get to work on interesting technical problems - and these are the people that see little value in learning how to develop emotional intelligence. Perhaps they enjoy technical problems because people are too ""difficult."" People are promoted into management positions - not because they actually know how to lead/manage, but because they happen to be smart or because there is no other path to grow into. So there is a layer of intelligent individuals who are horrible managers and leaders. Yet, there is no value system to actually do anything about that because ""emotional intelligence"" or ""adaptive leadership"" are not taken seriously.  3) Jerks. Sure, there are a lot of brilliant people - but, sadly, there are also a lot of jerks (and, many times, they are one and the same). Years ago, that wasn't the case. I don't know if the pool of candidates is getting smaller, or maybe all the folks with great personalities cashed out and left, or maybe people are getting burned out and it's wearing on their personality and patience. I've heard stories of managers straight-up cussing out their employees and intimidating/scaring their employees into compliance.  4) It's a giant company now and, inevitably, it has become slower moving and is now layered with process and bureaucracy. So many political battles, empire building, territory grabbing. Google says, ""Don't be evil."" But, that practice doesn't seem to be put into place when it comes to internal practices. :(
#> 5: * If you're a software engineer, you're among the kings of the hill at Google. It's an engineer-driven company without a doubt (that *is* changing, but it's still very engineer-focused). * The perks are amazing. Yes, free breakfast, lunch, an dinner every weekday. Aaaaaamazing holiday parties (at Waldorf Astoria, NY Public Library, MoMA, etc.), overnight ski trips to Vermont, overnight nature trips to the Poconos in the summer, summer picnics at Chelsea piers, and on and on and on. I don't see this going away unless the company starts hurting financially. * Speaking of which, the company is doing quite well, which reflects in bonuses and equity grants. * There a huge diversity of work ranging from defending independent journalism worldwide (Google Project Shield) to crisis response during disasters (see Maps during Hurricane Sandy or Tsunamis), to the best machine learning experts and projects in the world, to more mundane revenue-driving projects in advertising, there's really something for everybody. * It's easy to move around within the company as long as you're in good standing (the vast majority of engineers are). * The company is amazingly open: every week Larry Page and Sergey Brin host what's called TGIF where food, beer, wine, etc. is served, a new project is presented, and afterward there's an open forum to ask the executives anything you want. It's truly fair game to ask anything, no matter how controversial, and frequently the executives will be responsive. * No, nobody cares if you use an iPhone, Facebook, shop with Amazon, stream using Spotify, or refuse to use Google+. The company is amazingly open and flexible.  Neither pro nor con, but general information on work-life balance, promotions, and advancement. * Work life balance can be what you want it to be on most teams. (Some teams are in more competitive sectors and require more crazy hours all the time - but very few of them). If you do what's expected, you'll be fine at least for a handful of years. Working a roughly 40 hour work week is possible, and many people do it. There are also people who are hyper-motived and work like crazy just because they love it, or because they're competitive, or they want to get a promotion. If you work 40 hour weeks without putting in anything extra, you'll fall behind them as they advance and you stand still - and maybe that doesn't matter, so it works out for everybody. But at least know where you would realistically stand. * If you excel and work your butt off, you'll be compensated and promoted. If you let yourself be a code monkey, and just sit coding with your head down all day, you'll be fine but won't advance. A big complaint from some Googlers is about not being able to advance ""even at Google"" with pure coding. Sure, if you're the uber genius who created MapReduce and Bigtable, you're going to advance like a rocket without having to do anything but coding, but if you're like most engineers at Google -- smarter than average, but just average compared to other Googlers -- you're just a good coder and not revolutionary. Code monkeys are important to actually get stuff done, and to be sure you absolutely need to be a good coder as a software engineer (it's the minimum requirement), but code monkeys won't advance because they're not leaders and they're easy to replace. To get promoted you need to lead and do more than just code. There are plenty of ways to lead other than being an official tech lead, so this isn't actually _that_ hard, so the real point is just that you can't just sit there coding what other people tell you to code all day and expect to advance.
#> 6:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             * It *is* becoming larger, and with it comes growing pains: bureaucracy, slow to respond to market threats, bloated teams, cross-divisional tension (though nothing remotely approaching that of Microsoft's internal tension). * The quality of the engineers is possibly dropping, but possibly not. It's hard to get real metrics, because as the absolute number of people grows, naturally the number of bad apples grows, as a percentage it's supposedly the same as it ever was, but with larger numbers of poorer quality engineers it just _feels_ like things might be changing for the worse. * Also with growth means more internal-confidential data leaks (again, because of the raw numbers of people) -- product announcements being ruined, etc. That means the company has to be tighter-lipped internally to avoid leaks, which makes things less open. It's still an amazingly open place, but less so than it was even a couple years ago. The good thing is they recognize it and actively look to improve things because they know how important it is to keep the good culture.
#>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   text_clean
#> 1:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             people are smart and friendly
#> 2:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        bureaucracy is slowing things down
#> 3:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   food food food + cafes on main campus mtv alone mini kitchens snacks drinks free breakfastlunchdinner all day erday benefitsperks free gym access on mtv campus free self service laundry washerdryer available bowling alley volley ball pit custom built and exclusive employee use only outdoor sport park mtv free healthfitness assessments dog friendly etc etc etc compensation in ~ or google updated its compensation packages so that they were more competitive for the size of the organization k+ it has remained relatively innovative nimble and fast paced and open with communication but that is definitely changing for the worse with so many departments focus areas and products in theory you should have plenty of opportunity to grow your career horizontally or vertically in practice not true you get to work with some of the brightest most innovative and hard workingdiligent minds in the industry theres a con to that too see below
#> 4:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 worklife balance what balance all those perks and benefits are an illusion they keep you at work and they help you to be more productive ive never met anybody at google who actually time off on weekends or on vacations you may not hear management say you have to work on weekendsvacations but they set the culture by doing so and it inevitably trickles down i dont know if google inadvertently hires the work a holics or if they create work a holics in us regardless i have seen way too many of the following marriages fall apart colleagues choosing work and projects over family colleagues getting physically sick and ill because of stress colleagues crying while at work because of the stress colleagues shooting out emails at midnight am am am it is absolutely ridiculous and something needs to change poor management i think the issue is that a majority of people love google because they get to work on interesting technical problems and these are the people that see little value in learning how to develop emotional intelligence perhaps they enjoy technical problems because people are too difficult people are promoted into management positions not because they actually know how to leadmanage but because they happen to be smart or because there is no other path to grow into so there is a layer of intelligent individuals who are horrible managers and leaders yet there is no value system to actually do anything about that because emotional intelligence or adaptive leadership are not taken seriously jerks sure there are a lot of brilliant people but sadly there are also a lot of jerks and many times they are one and the same years ago that wasnt the case i dont know if the pool of candidates is getting smaller or maybe all the folks with great personalities cashed out and left or maybe people are getting burned out and its wearing on their personality and patience ive heard stories of managers straight up cussing out their employees and intimidatingscaring their employees into compliance its a giant company now and inevitably it has become slower moving and is now layered with process and bureaucracy so many political battles empire building territory grabbing google says dont be evil but that practice doesnt seem to be put into place when it comes to internal practices
#> 5: if youre a software engineer youre among the kings of the hill at google its an engineer driven company without a doubt that is changing but its still very engineer focused the perks are amazing yes free breakfast lunch an dinner every weekday amazing holiday parties at waldorf astoria ny public library moma etc overnight ski trips to vermont overnight nature trips to the poconos in the summer summer picnics at chelsea piers and on and on and on i dont see this going away unless the company starts hurting financially speaking of which the company is doing quite well which reflects in bonuses and equity grants there a huge diversity of work ranging from defending independent journalism worldwide google project shield to crisis response during disasters see maps during hurricane sandy or tsunamis to the best machine learning experts and projects in the world to more mundane revenue driving projects in advertising theres really something for everybody its easy to move around within the company as long as youre in good standing the vast majority of engineers are the company is amazingly open every week larry page and sergey brin host whats called tgif where food beer wine etc is served a new project is presented and afterward theres an open forum to ask the executives anything you want its truly fair game to ask anything no matter how controversial and frequently the executives will be responsive no nobody cares if you use an iphone facebook shop with amazon stream using spotify or refuse to use google+ the company is amazingly open and flexible neither pro nor con but general information on work life balance promotions and advancement work life balance can be what you want it to be on most teams some teams are in more competitive sectors and require more crazy hours all the time but very few of them if you do whats expected youll be fine at least for a handful of years working a roughly hour work week is possible and many people do it there are also people who are hyper motived and work like crazy just because they love it or because theyre competitive or they want to get a promotion if you work hour weeks without putting in anything extra youll fall behind them as they advance and you stand still and maybe that doesnt matter so it works out for everybody but at least know where you would realistically stand if you excel and work your butt off youll be compensated and promoted if you let yourself be a code monkey and just sit coding with your head down all day youll be fine but wont advance a big complaint from some googlers is about not being able to advance even at google with pure coding sure if youre the uber genius who created mapreduce and bigtable youre going to advance like a rocket without having to do anything but coding but if youre like most engineers at google smarter than average but just average compared to other googlers youre just a good coder and not revolutionary code monkeys are important to actually get stuff done and to be sure you absolutely need to be a good coder as a software engineer its the minimum requirement but code monkeys wont advance because theyre not leaders and theyre easy to replace to get promoted you need to lead and do more than just code there are plenty of ways to lead other than being an official tech lead so this isnt actually that hard so the real point is just that you cant just sit there coding what other people tell you to code all day and expect to advance
#> 6:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            it is becoming larger and with it comes growing pains bureaucracy slow to respond to market threats bloated teams cross divisional tension though nothing remotely approaching that of microsofts internal tension the quality of the engineers is possibly dropping but possibly not its hard to get real metrics because as the absolute number of people grows naturally the number of bad apples grows as a percentage its supposedly the same as it ever was but with larger numbers of poorer quality engineers it just feels like things might be changing for the worse also with growth means more internal confidential data leaks again because of the raw numbers of people product announcements being ruined etc that means the company has to be tighter lipped internally to avoid leaks which makes things less open its still an amazingly open place but less so than it was even a couple years ago the good thing is they recognize it and actively look to improve things because they know how important it is to keep the good culture
```

Next, we inspect the summary of the length of each document. The maximum number of words in a document is 2161 terms.


```r
document_length <- sapply(strsplit(df_clean$text_clean, " "), length)

document_length %>% 
   summary()
```

```
#>    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
#>     0.0     7.0    14.0    25.7    27.0  2161.0
```

We will only take documents with more than 50 terms/words.


```r
df_clean <- df_clean %>% 
   slice(which(document_length > 50))

dim(df_clean)
```

```
#> [1] 6271    3
```

### Cross-Validation

We split the data into the training set (80%) and the testing set (20%). We will also check the class proportion of the target variable in the training set.


```r
set.seed(123)
index <- sample(nrow(df_clean), nrow(df_clean)*0.8)

data_train <- df_clean[index, ]
data_test <- df_clean[-index, ]

table(df_clean$Sentiment) %>% prop.table()
```

```
#> 
#> negatives positives 
#> 0.6451922 0.3548078
```

As we can see, there is a class imbalance between the negative and postive sentiment, so we will upsample the minority class first.


```r
library(caret)
set.seed(123)
data_train <- upSample(x = data_train %>% select(-Sentiment), 
                       y = as.factor(data_train$Sentiment), yname = "sentiment") 

glimpse(data_train)
```

```
#> Rows: 6,474
#> Columns: 3
#> $ Review     <chr> "As it has grown big, Google has become miserably stagnant…
#> $ text_clean <chr> "as it has grown big google has become miserably stagnant …
#> $ sentiment  <fct> negatives, negatives, negatives, negatives, negatives, neg…
```

### Document-Term Matrix

Next, we create the document-term matrix (DTM) for each document. The term will be a combination of unigram (1-gram) and bigram (2-gram) for each documents.


```r
stem_hunspell <- function(term) {
    # look up the term in the dictionary
    stems <- hunspell_stem(term)[[1]]
    
    if (length(stems) == 0) { # if there are no stems, use the original term
        stem <- term
    } else { # if there are multiple stems, use the last one
        stem <- stems[[length(stems)]]
    }
    return(stem)
}

train_term <- data_train %>% 
   rownames_to_column("id") %>% 
   unnest_tokens(output = "word", input = text_clean) %>% 
   anti_join(stop_words)  %>% 
   mutate(word = text_tokens(word, stemmer = stem_hunspell) %>% as.character()) %>% 
   drop_na(word) %>% 
   count(id, word)

train_bigram <- data_train %>% 
   rownames_to_column("id") %>% 
   unnest_tokens(output = "word", input = text_clean, token = "ngrams", n = 2) %>% 
   drop_na(word) %>% 
   count(id, word)

test_term <- data_test %>% 
   rownames_to_column("id") %>% 
   unnest_tokens(output = "word", input = text_clean) %>% 
   anti_join(stop_words)  %>% 
   mutate(word = text_tokens(word, stemmer = stem_hunspell) %>% as.character()) %>% 
   drop_na(word) %>% 
   count(id, word) 

test_bigram <- data_test %>% 
   rownames_to_column("id") %>% 
   unnest_tokens(output = "word", input = text_clean, token = "ngrams", n = 2) %>% 
   drop_na(word) %>% 
   count(id, word)
```

Here is the resulting DTM from the corpus of text data. 


```r
dtm_train_review <- train_term %>% 
   bind_rows(train_bigram) %>% 
   cast_dtm(document = id, term = word, value = n)

dtm_test <- test_term %>% 
   bind_rows(test_bigram) %>% 
   cast_dtm(document = id, term = word, value = n)

inspect(dtm_train_review)
```

```
#> <<DocumentTermMatrix (documents: 6474, terms: 204309)>>
#> Non-/sparse entries: 885025/1321811441
#> Sparsity           : 100%
#> Maximal term length: 48
#> Weighting          : term frequency (tf)
#> Sample             :
#>       Terms
#> Docs   amazon company employee if you in the manage of the people team time
#>   1016      8       8        3      3      2     11      4      6    3    2
#>   1117     20       7        8      5      2      3      3      4    0    7
#>   1611      0       0        0      6      3      2      2      9    0    1
#>   2298     26       7        6      0      8     29      8      6   32    6
#>   2536      6       0        1      3      6      3      5      5    5    4
#>   3347     15       5        0      1      2      3      6      9   12   10
#>   3355      8       0        2      8      3      3      2      0    0   13
#>   4261      2      10        4      4      1      1      1     10    4    5
#>   4510      8       0        2      8      3      3      2      0    0   13
#>   694       0       1        3      0      1      1      4      5    0    4
```

### Final Preprocessing

We will continue to reduce the number of terms used by only choose words that appear in at least 5 documents and maximum appear in 80% of all documents. We get the final number of terms about 260,000 terms in 6500 documents.


```r
word_freq <- findFreqTerms(dtm_train_review, lowfreq =  5, highfreq = nrow(dtm_train_review)*0.8)

dtm_train <- dtm_train_review[ , word_freq ]

dtm_train
```

```
#> <<DocumentTermMatrix (documents: 6474, terms: 25908)>>
#> Non-/sparse entries: 621283/167107109
#> Sparsity           : 100%
#> Maximal term length: 30
#> Weighting          : term frequency (tf)
```

**LDA**

We will build the LDA topic model for the document-term matrix. We will use number of topic (k) = 50, with 5000 iterations and 4000 burn-in. Since the process is relatively long, we also have saved the previously trained topic model in the next chunk. 

The topic distribution for each document ($\theta$) will be used as the features for the machine learning model. Using only 50 topics, we expect a 99.8% dimensionality reduction.

`$$Dimensionality\ reduction = 1 - \frac{50}{25908} = 0.998 = 99.8\%$$`


```r
dtm_lda <- Matrix::Matrix(as.matrix(dtm_train), sparse = T)

set.seed(123)
lda_review <- FitLdaModel(dtm = dtm_lda, 
                        k = 50, 
                        iterations = 5000,
                        burnin = 4000
                        )
```


```r
lda_review <- read_rds("data_input/lda review.Rds")
```

Finally, we prepare the features and the target variable of the training set for model fitting.


```r
train_y <- data_train$sentiment[ rownames(lda_review$theta) %>% as.numeric() ]

train_x <- lda_review$theta
```

**Bernoullie Convertion**

For the conventional naive bayes, we will convert the numerical value (the frequency of each term in each document) into a categorical whether the term is presence in the document or not.


```r
bernoulli_conv <- function(x){
        x <- as.factor(as.numeric(x > 0))
}

train_bayes <- Matrix::Matrix(as.matrix(dtm_train), sparse = T)

train_bn <- apply(dtm_train, 2, bernoulli_conv)
test_bn <- apply(dtm_test, 2, bernoulli_conv)
```

### Model Fitting and Evaluation

We will use Random Forest and Naive Bayes to fit the data from LDA. We will also compare the performance of the model with a baseline model of Naive Bayes without the dimensionality reduction from LDA.

#### LDA with Random Forest

The random forest model will be trained using 500 trees and `mtry` parameter of 2. The error rate from the Out of Bag (OOB) observation is around 6.3% or similar to 93% of accuracy.


```r
library(randomForest)

set.seed(123)
rf_lda <- randomForest(x = train_x, 
                       y = train_y, 
                       ntree = 500, 
                       mtry = 2)

rf_lda
```

```
#> 
#> Call:
#>  randomForest(x = train_x, y = train_y, ntree = 500, mtry = 2) 
#>                Type of random forest: classification
#>                      Number of trees: 500
#> No. of variables tried at each split: 2
#> 
#>         OOB estimate of  error rate: 6.27%
#> Confusion matrix:
#>           negatives positives class.error
#> negatives      3135       102  0.03151066
#> positives       304      2933  0.09391412
```

Next, we will prepare the testing dataset. To get the features of probability distribution of each topic for each document, we ran the topic model on the DTM of the testing set using only 100 iterations and burn-in of 80.


```r
dtm_lda_test <- Matrix::Matrix(as.matrix(dtm_test), sparse = T)

# Get the topic probabilities for each document
set.seed(123)
test_x <- predict(lda_review,
                  newdata = dtm_lda_test,
                  iterations = 100,
                  burnin = 80
                  )
```

Next, we predict the testing set using the trained model and see the performance via confusion matrix.


```r
set.seed(123)
pred_test <- predict(rf_lda, test_x)
pred_prob <-  predict(rf_lda, test_x, type = "prob")

test_y <- data_test$Sentiment[ rownames(dtm_test) %>% as.numeric() ]

pred_lda <- data.frame(predicted = factor(pred_test, levels = c("positives", "negatives")),
                       actual = factor(test_y, levels = c("positives", "negatives"))
                       )

conf_mat(pred_lda, 
         truth = actual, 
         estimate = predicted)
```

```
#>            Truth
#> Prediction  positives negatives
#>   positives       271        16
#>   negatives       175       793
```

We then translate the confusion matrix into several evaluation matrix, such as accuracy, recall/sensitivity, precision and F1 measure. We also calculate the area under curve (AUC) to check the model sensitivity toward change of classification threshold.


```r
result_lda_rf <- data.frame(
   accuracy = accuracy_vec( truth = pred_lda$actual, 
                            estimate = pred_lda$predicted),
   
   recall = sens_vec( truth = pred_lda$actual, 
                      estimate = pred_lda$predicted),
   
   precision = precision_vec( truth = pred_lda$actual, 
                              estimate = pred_lda$predicted),
   
   F1 = f_meas_vec(truth = pred_lda$actual,
                   estimate = pred_lda$predicted),
   
   AUC = roc_auc_vec(truth = pred_lda$actual, 
                     estimate = pred_prob[, 2])
) %>% 
   mutate_all(scales::percent, accuracy = 0.01)

result_lda_rf
```

```
#>   accuracy recall precision     F1    AUC
#> 1   84.78% 60.76%    94.43% 73.94% 94.23%
```

#### LDA with Naive Bayes

We will feed the same LDA dataset using the Naive Bayes as comparison.


```r
naive_lda <- naiveBayes(x = train_x,
                        y = train_y)

pred_test <- predict(naive_lda, test_x)
pred_prob <-  predict(naive_lda, test_x, type = "raw")

pred_lda_bayes <- data.frame(predicted = factor(pred_test, levels = c("positives", "negatives")),
                             actual = factor(test_y, levels = c("positives", "negatives"))
                             )

conf_mat(pred_lda_bayes, 
         truth = actual, 
         estimate = predicted)
```

```
#>            Truth
#> Prediction  positives negatives
#>   positives       316        24
#>   negatives       130       785
```
Here are the evaluation metrics for the Naive Bayes model on LDA dataset.


```r
result_lda_bayes <- data.frame(
   accuracy = accuracy_vec( truth = pred_lda_bayes$actual, 
                            estimate = pred_lda_bayes$predicted),
   
   recall = sens_vec( truth = pred_lda_bayes$actual, 
                      estimate = pred_lda_bayes$predicted),
   
   precision = precision_vec( truth = pred_lda_bayes$actual, 
                              estimate = pred_lda_bayes$predicted),
   
   F1 = f_meas_vec(truth = pred_lda_bayes$actual,
                   estimate = pred_lda_bayes$predicted),
   
   AUC = roc_auc_vec(truth = pred_lda_bayes$actual, 
                     estimate = pred_prob[, 2])
) %>% 
   mutate_all(scales::percent, accuracy = 0.01)

result_lda_bayes
```

```
#>   accuracy recall precision     F1    AUC
#> 1   87.73% 70.85%    92.94% 80.41% 94.91%
```

#### N-gram with Naive Bayes

Lastly, we will train a Naive Bayes model on the original document-term matrix dataset that consist of 25,000+ terms as a baseline or benchmark model. Since the prediction process of naive bayes is taking too much time, we've prepared the prediction result in `Rds` format.


```r
naive_gram <- naiveBayes(x = train_bn, 
                         y = train_y)

pred_test_gram <- predict(naive_gram, test_bn)
pred_prob_gram <-  predict(naive_gram, test_bn, type = "raw")
```


```r
pred_test_gram <- read_rds("data_input/pred_test_gram.Rds")
pred_prob_gram <- read_rds("data_input/pred_prob_gram.Rds")
pred_gram_bayes <- data.frame(predicted = factor(pred_test_gram, levels = c("positives", "negatives")),
                             actual = factor(test_y, levels = c("positives", "negatives"))
                             )

conf_mat(pred_gram_bayes, 
         truth = actual, 
         estimate = predicted)
```

```
#>            Truth
#> Prediction  positives negatives
#>   positives       420        88
#>   negatives        26       721
```

Here are the evaluation metrics for the Naive Bayes model on the original dataset.


```r
result_gram_bayes <- data.frame(
   accuracy = accuracy_vec( truth = pred_gram_bayes$actual, 
                            estimate = pred_lda_bayes$predicted),
   
   recall = sens_vec( truth = pred_gram_bayes$actual, 
                      estimate = pred_gram_bayes$predicted),
   
   precision = precision_vec( truth = pred_gram_bayes$actual, 
                              estimate = pred_gram_bayes$predicted),
   
   F1 = f_meas_vec(truth = pred_gram_bayes$actual,
                   estimate = pred_gram_bayes$predicted),
   
   AUC = roc_auc_vec(truth = pred_gram_bayes$actual, 
                     estimate = pred_prob_gram[, 2])
) %>% 
   mutate_all(scales::percent, accuracy = 0.01)

result_gram_bayes
```

```
#>   accuracy recall precision     F1    AUC
#> 1   87.73% 94.17%    82.68% 88.05% 97.29%
```

This is the recap of performances of all trained models. Using only 50 features (with 99.8% of dimensionality reduction) extracted from the original DTM using topic model, there are some interesting finding. The recall/sensitivity of the LDA models, both Random Forest and Naive Bayes, is far lower than the Naive Bayes using the original DTM. However, the LDA models has better precision with more than 93% precision on the testing dataset. The trade-off between dimensionality reduction and the model performance for this case is a worthy one, since the dimension is heavily reduced (again, 99.8% reduction) while the model still have an acceptable performance. The dimensionality reduction also result in faster computation. This is especially useful because the prediction process of Naive Bayes with many features take too much time.


```r
result_lda_rf %>% 
   bind_rows(result_lda_bayes, result_gram_bayes) %>% 
   mutate(
      model = c("Random Forest", "Naive Bayes", "Naive Bayes"),
      method = c("LDA", "LDA", "n-Gram"),
      `n features` = c( 50, 50, ncol(dtm_train) )
   ) %>% 
   select(method, model, everything()) %>% 
   rename_all(str_to_title) 
```

```
#>   Method         Model Accuracy Recall Precision     F1    Auc N Features
#> 1    LDA Random Forest   84.78% 60.76%    94.43% 73.94% 94.23%         50
#> 2    LDA   Naive Bayes   87.73% 70.85%    92.94% 80.41% 94.91%         50
#> 3 n-Gram   Naive Bayes   87.73% 94.17%    82.68% 88.05% 97.29%      25908
```


# Reference

[^1]: [Natural Language Processing Is a Key Engine of AI Market Growth, Enabling 44 Discrete Use Cases Across 17 Industries](https://tractica.omdia.com/newsroom/press-releases/natural-language-processing-is-a-key-engine-of-ai-market-growth-enabling-44-discrete-use-cases-across-17-industries/)
[^2]: [A Simple Introduction to Natural Language Processing](https://becominghuman.ai/a-simple-introduction-to-natural-language-processing-ea66a1747b32)
[^3]: [Applications of Topic Models](https://www.nowpublishers.com/article/DownloadSummary/INR-030)
[^4]: [Latent Dirichlet Allocation](http://www.jmlr.org/papers/volume3/blei03a/blei03a.pdf)
[^5]: [Text Mining with R](https://www.tidytextmining.com/topicmodeling.html)
[^6]: [Bayesian Statistics](https://statswithr.github.io/book/the-basics-of-bayesian-statistics.html)
[^7]: [Hunspell Stemmer](https://github.com/ropensci/hunspell)
[^8]: [Porter Stemming Algorithm](http://snowball.tartarus.org/algorithms/porter/stemmer.html)
[^9]: [Dynamic Topic Models](http://www.cs.columbia.edu/~blei/papers/BleiLafferty2006a.pdf)
[^10]: [Training, evaluating, and interpreting topic models](https://juliasilge.com/blog/evaluating-stm/)
