---
title: "Rplicate Series: Gone Baby Gone"
author: Arga Adyatama
github: https://github.com/Argaadya
date: '2020-11-25'
slug: rplicate-series-gone-baby-gone
categories:
  - R
tags:
  - Data Visualization
  - Rplicate
  - ggplot2
description: ''
featured: 'gone-baby-gone.png'
featuredalt: ''
featuredpath: 'date'
linktitle: ''
type: post
---



Welcome again to the Rplicate Series! In this 4th article of the series, we will replicate The Economist plot titled _"Gone Baby Gone"_. In the process, we will explore ways to use **transformed value as our axes**, adding **horizontal/vertical line**, and making an **elbow line** (and generally more flexible) annotation for repelled texts.

<center> ![](/img/rplicate4/original.png){width="60%"} </center>

Let's dive in below!

# Load Packages

These are the packages and some set up that we will use.


```r
library(tidyverse) # for data wrangling
library(scales) # customize axes in plot
library(ggrepel) # add & customize repelled text
library(ggthemes) # provide previously made themes
library(grid) # enhance the layouting of plot
options(scipen = 100) # to prevent R displaying scientific notation
```

# Dataset

The plot we are going to make tells us about the decrease of South Korean women fertility rates which falls to a record low, as reported in the [original article](https://www.economist.com/graphic-detail/2019/08/30/south-koreas-fertility-rate-falls-to-a-record-low).

## Data Collection

The data was acquired from World Bank and downloaded bofore use:

* Fertility Rate data: <https://data.worldbank.org/indicator/SP.DYN.TFRT.IN>
* Population data: <https://data.worldbank.org/indicator/SP.POP.TOTL?end=2018&start=1996>
* GDP per Capita data: <https://data.worldbank.org/indicator/NY.GDP.PCAP.CD>


```r
data_tfr <- read_csv(file = "data_input/world_bank_fertility.csv",
                     skip = 4)
data_pop <- read_csv(file = "data_input/world_bank_pop.csv",
                     skip = 4)
data_gdp <- read_csv(file = "data_input/world_bank_gdp.csv",
                     skip = 4)
```

## Data Preprocessing

Since the article only use the population data from 2017, we will clean the data first.


```r
# selecting data from 2017

## fertility rates
data_tfr <- data_tfr %>% 
  select(`Country Name`,'2017') %>% 
  rename(tfr = "2017")

## population
data_pop <- data_pop %>% 
  select(`Country Name`,'2017') %>% 
  rename(pop = "2017")

## gdp
data_gdp <- data_gdp %>% 
  select(`Country Name`,'2017') %>% 
  rename(gdp = "2017")

head(data_tfr,10)
```

```
#> # A tibble: 10 x 2
#>    `Country Name`         tfr
#>    <chr>                <dbl>
#>  1 Aruba                 1.80
#>  2 Afghanistan           4.48
#>  3 Angola                5.62
#>  4 Albania               1.71
#>  5 Andorra              NA   
#>  6 Arab World            3.27
#>  7 United Arab Emirates  1.73
#>  8 Argentina             2.28
#>  9 Armenia               1.60
#> 10 American Samoa       NA
```

Next, we will combine the data while also removing NAs and arrange the data descendingly based on population:


```r
df <- data_tfr %>% 
  left_join(data_pop) %>% 
  left_join(data_gdp) %>%  
  na.omit() %>%                 
  rename(country = `Country Name`) %>%  
  arrange(desc(pop)) 

# quick check
head(df, 40)
```

```
#> # A tibble: 40 x 4
#>    country                      tfr        pop    gdp
#>    <chr>                      <dbl>      <dbl>  <dbl>
#>  1 World                       2.43 7510990456 10769.
#>  2 IDA & IBRD total            2.55 6335039629  4859.
#>  3 Low & middle income         2.56 6306560891  4743.
#>  4 Middle income               2.33 5619111361  5229.
#>  5 IBRD only                   2.07 4731120193  6052.
#>  6 Early-demographic dividend  2.51 3207188541  3545.
#>  7 Lower middle income         2.76 2981420591  2187.
#>  8 Upper middle income         1.84 2637690770  8663.
#>  9 East Asia & Pacific         1.80 2314202003 10333.
#> 10 Late-demographic dividend   1.71 2276319334  8943.
#> # ... with 30 more rows
```

Look at the table above! Looks like we have some previously made clusters of countries in the data. To prevent redundancy, we need to remove them first.


```r
df <- df %>% slice(-c(1:15,18:43))
head(df,10)
```

```
#> # A tibble: 10 x 4
#>    country              tfr        pop    gdp
#>    <chr>              <dbl>      <dbl>  <dbl>
#>  1 China               1.63 1386395000  8759.
#>  2 India               2.30 1338658835  1981.
#>  3 United States       1.77  325147121 59928.
#>  4 Indonesia           2.34  264645886  3837.
#>  5 Pakistan            3.41  207896686  1467.
#>  6 Brazil              1.71  207833831  9881.
#>  7 Nigeria             5.46  190873311  1969.
#>  8 Bangladesh          2.08  159670593  1564.
#>  9 Russian Federation  1.76  144496740 10751.
#> 10 Japan               1.43  126785797 38332.
```

Now we are ready to make the plot.

# Plotting

## Scatter Plot

First, we plot the data into scatter plot which the size of the dot indicating the population size.


```r
p <- df %>% 
  ggplot() +
  geom_point(aes(x = gdp, y = tfr, size = pop),
             color = "#62c9d8",
             alpha = 0.5,
             show.legend = FALSE)
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-6-1.png" width="672" style="display: block; margin: auto;" />

## Set Axis & Aesthetic

As you can see above, the mapping between x and y values are quite extreme (forming a curve pointing at a position near zero at both axis). We can smooth the visualization by **using transformed value for our axes**. We will use the _log10 scale_ for our x-axis since the values are high in range and possible for the transformation.


```r
p <- p +
  scale_x_continuous(trans = "log10", # there are many more transformation function that you can explore!
                     expand = c(0.05,0), # adjusting spaces on the min-max
                     labels = number_format(big.mark = ","), # add comma to axis labels
                     limits = c(100,110000))
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-7-1.png" width="672" style="display: block; margin: auto;" />

Next, we can also set the y-axis:


```r
p <- p +
  scale_y_continuous(breaks = seq(from = 0, to = 8, by = 2),
                     limits = c(0,8.3),
                     position = "right",
                     expand = c(0,0))

p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-8-1.png" width="672" style="display: block; margin: auto;" />

Following the transformation, we can also scale the aesthetic element `size` using `scale_size_continues()`.


```r
p <- p +
  scale_size_continuous(range = c(3,16))

p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-9-1.png" width="672" style="display: block; margin: auto;" />

## Line & Annotation

Next, we will add a horizontal line representing "Replacement Fertility Level" using `geom_hline()`. As a side note, you can use `geom_vline()` and set the _xintercept_ aesthetic to add a vertical line.


```r
p <- p +
  geom_hline(aes(yintercept= 2.1),
             color = "#edb0ad",
             linetype = "dashed",
             size=1)

p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-10-1.png" width="672" style="display: block; margin: auto;" />

We will also add text annotation for the line and the text below it:


```r
p <- p +
  geom_text(aes(x = 100,y = 2.4, label = "Replacement fertility level"),
            color = "#e07b78",
            hjust = "left",
            size = 4.5)
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-11-1.png" width="672" style="display: block; margin: auto;" />


```r
p <- p +
  geom_text(aes(x = 100, y = 0.5, label = "Circle size = Population, 2017"),
            color = "#8e9093",
            hjust = "left",
            size = 4)

p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-12-1.png" width="672" style="display: block; margin: auto;" />

## Customize Repelled Text

Now we will highlights and add annotation for selected countries. Since the annotation line is an elbow shape, we need to be creative. To my best knowledge, there is no packages that can directly make an elbow annotation. I'll show you how to make one using a combination of `geom_text_repel()` and `geom_segment()`.

We will manually add the text one by one based on how it positioned on the plot. First, let's add the Niger text which is quite simple compared to the others. Notice that we need to filter the data first so that we can custom the repelled text individually for each selected country.


```r
# filter the data to be used
df_niger <- df %>% filter(country == "Niger")

p <- p +
  geom_text_repel(aes(x = gdp, y = tfr, label = country),
                  data = df_niger,
                  nudge_x = -.15, # position from the x aesthetic
                  direction = "x") # direction of the label/repelled text (x/y)

p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-13-1.png" width="672" style="display: block; margin: auto;" />

Then we higlight the point of Niger by layering the plot using new `geom_point()`:


```r
p <- p +
  geom_point(aes(x = gdp, y = tfr, size = pop),
             data = df_niger,
             shape = 21,
             fill = "#2fc1d3",
             color ="black", 
             show.legend = F)
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-14-1.png" width="672" style="display: block; margin: auto;" />

The second one is for India and China. This one is more complicated for we have to create an elbow line. We can do that by using a combination of `geom_segment()` and `geom_text_repel()`. We first have to create a small vertical line as the starting point of the elbow line. We can use `geom_segment()` as seen below.


```r
# filter the data to be used
df_inch <- df %>% filter(country %in% c("India","China"))

p <- p +
  geom_segment(aes(x = gdp, xend = gdp, y = tfr, yend = tfr-.7),
               data = df_inch)
  
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-15-1.png" width="672" style="display: block; margin: auto;" />

and then we can use `geom_text_repel()` to make the repelled text, following from the previously made position of `geom_segment()`:


```r
p <- p +
  geom_text_repel(aes(x = gdp, y = tfr-.7, label = country),
                  data = df_inch,
                  nudge_x = -.3,
                  direction = "x")
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-16-1.png" width="672" style="display: block; margin: auto;" />

Do not forget to highlight the point: 


```r
p <- p +
  geom_point(aes(gdp, tfr, size = pop),
             data = df_inch,
             shape = 21,
             fill = "#2fc1d3",
             color = "black", 
             show.legend = F)

p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-17-1.png" width="672" style="display: block; margin: auto;" />

We can follow the previous step for the other countries as it follows a similar pattern:

1. Create `geom_segment()` (if we need to)
2. Create `geom_text_repel()` and set its direction
3. Highlight the point using new `geom_point()`

Below is the code for the other countries:

* Japan & United States


```r
# filter the data to be used
df_japus <- df %>% filter(country %in% c("Japan","United States"))

p <- p +
  geom_text_repel(aes(gdp, tfr, label = country),
                  data = df_japus,
                  nudge_y = 1.5,
                  direction = "y")
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-18-1.png" width="672" style="display: block; margin: auto;" />


```r
p <- p +
  geom_point(aes(gdp, tfr, size = pop),
             data = df_japus,
             shape = 21,
             fill = "#2fc1d3",
             color = "black",
             show.legend = F)
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-19-1.png" width="672" style="display: block; margin: auto;" />

* Hongkong


```r
# filter the data to be used
df_hongkong <- df %>% filter(country == "Hong Kong SAR, China")
df_hongkong <- df_hongkong %>% mutate(country = "Hong Kong")

p <- p +
  geom_text_repel(aes(gdp, tfr, label=country),
                  data = df_hongkong,
                  nudge_y = -.7,
                  direction = "y")
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-20-1.png" width="672" style="display: block; margin: auto;" />


```r
p <- p +
  geom_point(aes(gdp, tfr, size = pop),
             data = df_hongkong,
             shape = 21,
             fill = "#2fc1d3",
             color = "black",
             show.legend = F)
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-21-1.png" width="672" style="display: block; margin: auto;" />

* Singapore 


```r
# filter the data to be used
df_sing <- df %>% filter(country == "Singapore")

p <- p +
  geom_segment(aes(x = gdp, xend = gdp, y = tfr, yend = tfr-.5),
               data = df_sing,
               color = "black")
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-22-1.png" width="672" style="display: block; margin: auto;" />


```r
p <- p +
  geom_text_repel(aes(gdp, tfr-.5, label = country),
                  data = df_sing,
                  color = "black",
                  nudge_x = 0.3,
                  direction = "x")

p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-23-1.png" width="672" style="display: block; margin: auto;" />


```r
p <- p +
  geom_point(aes(gdp, tfr, size = pop),
             data = df_sing,
             shape = 21,
             fill = "#2fc1d3",
             color = "black",
             show.legend = F)

p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-24-1.png" width="672" style="display: block; margin: auto;" />

* South Korea


```r
# filter the data to be used
df_kor <- df %>% filter(country == "Korea, Rep.")
df_kor <- df_kor %>% mutate(country = "South Korea")

p <- p +
  geom_segment(aes(x = gdp, xend = gdp, y = tfr, yend = tfr-.5),
               data = df_kor)
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-25-1.png" width="672" style="display: block; margin: auto;" />


```r
p <- p +
  geom_text_repel(aes(gdp, tfr-.5, label = country),
                  data = df_kor,
                  nudge_x = -.3,
                  direction = "x",
                  fontface = "bold") # bold font, spesifically set for Korea

p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-26-1.png" width="672" style="display: block; margin: auto;" />


```r
p <- p +
  geom_point(aes(gdp, tfr, size = pop),
             data = df_kor,
             shape = 21,
             fill = "#2fc1d3",
             color = "black", 
             show.legend = F)

p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-27-1.png" width="672" style="display: block; margin: auto;" />

## Title & Label

Next we can add and customize title and label for the plot.


```r
p <- p +
   labs(title = "Gone baby gone",
       subtitle = "GDP and fertility, 2017                                                                                                          Fertility Rate",
       x = "\n GDP per capita, $, log scale", #\n for adding space between axis and x-axis
       y = NULL,
        caption = "Source: World Bank")

p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-28-1.png" width="672" style="display: block; margin: auto;" />

## Plot Theme

And as a finishing touch, we can customize theme for our plot.


```r
p <- p +
  theme(plot.background = element_blank(),
        panel.background = element_blank(),
        panel.grid.major.y = element_line(colour = "gray80"))
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-29-1.png" width="672" style="display: block; margin: auto;" />


```r
p <- p +
  theme(axis.ticks.length.x = unit(2,"mm"),
        axis.ticks.y = element_blank(),
        axis.line.x = element_line(color = "black"),
        axis.text.y = element_text(vjust = 0),
        axis.title.x = element_text(colour = "black", size = 12))
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-30-1.png" width="672" style="display: block; margin: auto;" />


```r
p <- p +
  theme(plot.title = element_text(face = "bold"),
        plot.caption = element_text(color = "gray30", hjust = 0))
```


## Final Result

Now let's display our final plot!


```r
p
```

<img src="/blog/2020-11-25-rplicate-series-gone-baby-gone_files/figure-html/unnamed-chunk-32-1.png" width="672" style="display: block; margin: auto;" />

Thank you for reading and we hope this article can help you to create a more interesting visualization. We hope you can find the delight in exploring data visualization in R. Happy learning and coding!

