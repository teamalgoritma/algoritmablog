---
title: 'Rplicate Series: Cherry Bomb'
author: Nabiilah Ardini Fauziyyah
github: https://github.com/NabiilahArdini
date: '2020-10-02'
slug: rplicate-series-cherry-bomb
categories:
  - R
tags:
  - Data Visualization
  - ggplot2
  - Rplicate
description: ''
featured: ''
featuredalt: ''
featuredpath: ''
linktitle: ''
type: post
---



In this second article on **Rplicate Series**, We will share to you one way to replicate the plot titled **Cherry Bomb** from the article ["Japan’s cherry blossoms are emerging increasingly early, 7th April 2017"](https://www.economist.com/graphic-detail/2017/04/07/japans-cherry-blossoms-are-emerging-increasingly-early). The raw dataset used for this graph was obtained from a [phenological dataset](http://atmenv.envi.osakafu-u.ac.jp/aono/kyophenotemp4/) that was previously collected by Dr. Yasuyuki Aono from Osaka Prefecture University[^1][^2].

<center> ![](/img/rplicate2/cherrybomb.png){width="60%"} </center>

While replicating the plot, we will also learn how to:

* various data pre-processing tricks such as separate a column into several columns, create date data from several columns, and reverse levels of a categorical column
* create a scatterplot with a custom shape/image as points 
* using `geom_smooth()` for visualizing trend in the data
* explore facets and various theme editing on ggplot2

Let's dive in below!

# Load Packages

These are the basic packages that we will use. Along the way, there will be some packages that we will load just before its use to better understand and remember its functionality.


```r
library(dplyr) # to simplify data transformation
library(tidyr) # to tidying data
library(lubridate) # to deal with date
library(ggplot2) # to create plot
```

# Import Dataset

The first thing we have to do is to load the dataset into R.


```r
sakura <- read.csv("data_input/sakura.csv")
tail(sakura)
```

```
#>        AD full.flowering.doy full.flowering.date source data.type
#> 1213 2013                 93                 403      8         0
#> 1214 2014                 94                 404      8         0
#> 1215 2015                 93                 403      8         0
#> 1216 2016                 NA                 404      8         0
#> 1217 2017                 NA                 409      8         0
#> 1218 2018                 NA                 330      8         0
#>                   reference
#> 1213 NEWS-PAPER(ARASHIYAMA)
#> 1214 NEWS-PAPER(ARASHIYAMA)
#> 1215 NEWS-PAPER(ARASHIYAMA)
#> 1216 NEWS-PAPER(ARASHIYAMA)
#> 1217 NEWS-PAPER(ARASHIYAMA)
#> 1218 NEWS-PAPER(ARASHIYAMA)
```

These are the descriptions of each column:

* **AD**: year in A.D.
* **full.flowering.doy**: full flowering day of year
* **full.flowering.date**: full flowering date (eg.: 401 = April 1st)
* **source**: source code
* **data.type**: data source code
* **reference**: name of old document/references

# Data Wrangling

The next step is *data wrangling* or the process of tidying and transforming data from a raw dataset into another format that is easier for plotting.


```r
# selecting column for plotting
sakura <- sakura %>% 
  select(AD, full.flowering.date)

sample_n(sakura,6) # sampling rows
```

```
#>     AD full.flowering.date
#> 1 1367                 407
#> 2 1344                 411
#> 3  886                  NA
#> 4 1528                 415
#> 5  978                  NA
#> 6 1045                  NA
```
After some exploration, we find that most of the data came from early AD contain missing records of its full flowering date. Therefore, the next step is to obtain the data with no missing values on its `full.flowering.date`, and for better visualization, we can also rename the "AD" column to "Year".


```r
sakura.used <- sakura %>% 
  filter(full.flowering.date != "NA") %>% 
  rename(Year=AD)

head(sakura.used)
```

```
#>   Year full.flowering.date
#> 1  812                 401
#> 2  815                 415
#> 3  831                 406
#> 4  851                 418
#> 5  853                 414
#> 6  864                 409
```

If you see the data above, the *month* and *day of month* information is still stored in one column `full.flowering.date`. Meanwhile, we need both information separately to display it on the y-axis. Moreover, it is still stored in an integer format with no separator between both information. We will need to separate both information into different columns. To do that, we can use the `separate()` function, as seen below: 


```r
sakura.plot <- sakura.used %>% 
  separate(full.flowering.date, 
           into = c("month", "day"), # name of the new columns
           sep = -2) # separate from the last 2 digits

head(sakura.plot)
```

```
#>   Year month day
#> 1  812     4  01
#> 2  815     4  15
#> 3  831     4  06
#> 4  851     4  18
#> 5  853     4  14
#> 6  864     4  09
```
Now, we only need to remove the leading zero of some digits in the `day` column. We can use the function `f_num()` from **numform** package. The numform package contains various functions to format numbers for publication.


```r
library(numform)

sakura.plot <- sakura.plot %>% 
  mutate(day = f_num(day, 
                     digits = 0)) # to set for 0 number behind decimal point 

head(sakura.plot)
```

```
#>   Year month day
#> 1  812     4   1
#> 2  815     4  15
#> 3  831     4   6
#> 4  851     4  18
#> 5  853     4  14
#> 6  864     4   9
```
The next step we are going to do is to combine year, month, and day information into a full date format. Let's store it into a column named `date`. After that, let's replace the month value into its respective month label. We can also change the data type of column `day` into integers, or just replace it with the day information obtained from the column `date`.


```r
sakura.plot <- sakura.plot %>% 
  mutate(date = as.Date(paste(Year, month, day, sep = "-")),
         month = month(date, label = T, abbr = F),
         day = day(date))
  

head(sakura.plot)
```

```
#>   Year month day       date
#> 1  812 April   1 0812-04-01
#> 2  815 April  15 0815-04-15
#> 3  831 April   6 0831-04-06
#> 4  851 April  18 0851-04-18
#> 5  853 April  14 0853-04-14
#> 6  864 April   9 0864-04-09
```


```r
unique(sakura.plot$month)
```

```
#> [1] April Maret Mei  
#> 12 Levels: Januari < Februari < Maret < April < Mei < Juni < ... < Desember
```

If we look at the order of our `month` column, it is still in the reverse format. We can reverse the order of the levels by using the function `fct_rev()` from **forcats** package. Forcats package itself contains many functions form working with categorical variables/factors.


```r
sakura.plot <- sakura.plot %>% 
  mutate(month = forcats::fct_rev(month))

unique(sakura.plot$month)
```

```
#> [1] April Maret Mei  
#> 12 Levels: Desember < November < Oktober < September < Agustus < ... < Januari
```

We have now obtained the data ready for plotting.

# Create Plot

In this section, we will display the code for plotting followed by its result below. Try tracking on what has been added to the code and how it gives changes to the plot!

## The Canvas


```r
# blank canvas
p <- ggplot(sakura.plot, aes(x = Year, y = day))

p
```

<img src="/blog/2020-10-02-rplicate-series-cherry-bomb_files/figure-html/unnamed-chunk-10-1.png" width="672" style="display: block; margin: auto;" />

## Add Facets


```r
# add facets_grid by month, horizontally
p + facet_grid(month~.)
```

<img src="/blog/2020-10-02-rplicate-series-cherry-bomb_files/figure-html/unnamed-chunk-11-1.png" width="672" style="display: block; margin: auto;" />

```r
# customize facets
p2 <- p + facet_grid(month~., 
             scales = "free", # free scales on both x & y axis 
             space = "free", # panel's height & width will vary based on data
             switch = "y") # facet label to be displayed on th right


p2
```

<img src="/blog/2020-10-02-rplicate-series-cherry-bomb_files/figure-html/unnamed-chunk-12-1.png" width="672" style="display: block; margin: auto;" />

## Add `geom_point`

The basic idea is to create a scatterplot from our data.


```r
# basic idea
p2 + geom_point()
```

<img src="/blog/2020-10-02-rplicate-series-cherry-bomb_files/figure-html/unnamed-chunk-13-1.png" width="672" style="display: block; margin: auto;" />

But let's make it more interesting by changing the point shape into a custom cherry blossom image. There are several ways to add custom shape as point (scatterplot) in ggplot:

**1. Use extended ASCII symbol in the argument `shape`**

The standard shape in geom_point only has options from shape 1-25. The extended number is actually options for various ASCII characters. We can use the ASCII character number 42 to obtain a flower-like shape as seen below.


```r
p3 <- p2 + geom_point(shape = 42, size = 5, color = "#f64b77")

p3
```

<img src="/blog/2020-10-02-rplicate-series-cherry-bomb_files/figure-html/unnamed-chunk-14-1.png" width="672" style="display: block; margin: auto;" />

**2. Add image as point using `geom_image()`**

We can use `geom_image()` from the package **ggimage** to add an image as a point in our plot. For that to work, we need to store the custom image into a column in our data. In this demo, we will use an icon made by [Freepik](https://www.flaticon.com/authors/freepik) from [Flaticon](https://www.flaticon.com). This is our icon image that has been saved locally: 

<center> ![](/img/rplicate2/cherry.png){width="60%"} </center>


```r
# adding new column to store image
sakura.plot2 <- sakura.plot %>% 
  mutate(image = "img/cherry.png")
```


```r
# create plot
library(ggimage)

ggplot(sakura.plot2, aes(x = Year, y = day)) + 
  facet_grid(month~., scales = "free", space = "free_y", switch = "y") +
  geom_image(aes(image = image))
```

<img src="/blog/2020-10-02-rplicate-series-cherry-bomb_files/figure-html/unnamed-chunk-16-1.png" width="672" style="display: block; margin: auto;" />

The latter option seems to be more flexible, but unfortunately, the image cannot be displayed properly due to the small panel size (result from setting `space = "free"` in facets). For that reason, we will use the first option instead.

## Add `geom_smooth`

Below we will add a smooth line using `geom_smooth()`. Because we also need legend for our smooth line, we can specify the color and fill label within `aes()`.


```r
p4 <- p3 +

  # make trend line
  geom_smooth(aes(fill = "Trend"), span = 0.1, se = FALSE, color = "#644128") +
  
  # make confidence interval with no trend line
  geom_smooth(aes(color = "Confidence Interval"), span = 0.1, fill = "#a56c56", linetype = 0)

p4
```

<img src="/blog/2020-10-02-rplicate-series-cherry-bomb_files/figure-html/unnamed-chunk-17-1.png" width="672" style="display: block; margin: auto;" />

## Costumize Axis

We can specify our axis manually for our plot that (unfortunately) did not follow a specific sequence.


```r
p5 <- p4 + 
scale_x_continuous(
  limits = c(812,2020),
  breaks = c(800, 900, 1000, 1100, 1200, 1300, 1400, 1500, 1600, 1700, 1800, 1900, 2000, 2016),
  labels = c("800", "", "1000", "", "1200", "", "1400", "", "1600", "", "1800", "", "", "2016")) +
scale_y_continuous(
  breaks = c(1, 10, 20),
  labels = c("1st", "10th", "20th"))

p5
```

<img src="/blog/2020-10-02-rplicate-series-cherry-bomb_files/figure-html/unnamed-chunk-18-1.png" width="672" style="display: block; margin: auto;" />

## Add Labels

We can add labels and make some of them italics using the function `expression()`:


```r
p6 <- p5 +
labs(title = "Cherry Bomb", 
     subtitle = "Date of cherry-blossom peak-bloom in Kyoto, Japan, 800AD - 2016",
     x = expression(italic("Year")),
     y = expression(italic("Date of cherry-blossom peak-bloom")),
     caption = "Source: Yasuyuki Aono, Osaka Prefecture University")

p6
```

<img src="/blog/2020-10-02-rplicate-series-cherry-bomb_files/figure-html/unnamed-chunk-19-1.png" width="672" style="display: block; margin: auto;" />

## Costumize Theme

Last but not least is to custom our theme. Below is an extensive list of options we can use to custom our plot theme, and there are still many more. You can try to modify this code and see how it affects our plot to better understand its use. We can also use a custom font which we have discussed in the previous article ["Rplicate Series: Happiness of The Third Age"](https://algotech.netlify.app/blog/rplicate-happiness-of-the-third-age/).


```r
library(extrafont)

# importing fonts to R
# font_import() 

# load all fonts so it can be used
loadfonts(device = "win") 
```


```r
cherrybomb <- p6 + theme(
  # general
  text = element_text(family = "Segoe UI"),
  
  # panel
  panel.background = element_rect(fill = "white"),
  panel.grid.major.y = element_line(color = "#a56c56", 
                                    linetype = "solid"),
  panel.grid.major.x = element_blank(),
  panel.grid.minor = element_blank(),
  panel.spacing = unit(0.1, "cm"),
  
  # axis
  axis.line.x = element_line(colour = "black"),
  axis.line.y = element_blank(),
  axis.text = element_text(size = 8),
  axis.title = element_text(size = 8),
  
  # legends
  legend.position = c(0.81, 1),
  legend.key = element_blank(),
  legend.box = "horizontal",
  legend.box.spacing = unit(1, "mm"),
  legend.title = element_blank(),
  legend.text = element_text(size = 8),
  legend.background = element_blank(),
  
  # labs
  plot.title = element_text(hjust = 0, 
                            face = "bold", 
                            size = 12),
  plot.subtitle = element_text(hjust = 0, 
                               face = "plain", 
                               size = 10),
  plot.caption = element_text(size = 8, 
                              colour = "#B3B1B1", 
                              hjust = 0),
  
  # facets title
  strip.placement = "outside",
  strip.background = element_rect(fill = "#e8dbd6"),
  strip.switch.pad.grid = unit(0.2, "cm"),
  strip.text = element_text(size = 7))
```

And here is our final Cherry Bomb Plot!


```r
cherrybomb
```

<img src="/blog/2020-10-02-rplicate-series-cherry-bomb_files/figure-html/unnamed-chunk-22-1.png" width="672" style="display: block; margin: auto;" />

# Additional Notes

This graph was both fun and challenging to make. Unfortunately, we have not managed to replicate one minor detail. If you take a more detailed look at the plot, we still have an **uncomplete label for the tickmarks in the y-axis**. I find it difficult to create different y-axis tickmarks and their labels for each facet in the plot. It is very unfortunate that the facet_grid function is not yet provided with the ability to have/set different scales/limits and breaks for each facet. And yet, I also haven't found a way to assign different breaks and limits for each facet. Thankfully the communities are developing it right now. For more info, you can [click here](https://github.com/zeehio/facetscales).

Thank you for reading and we hope this article can help you to find the delight in exploring data visualization in R. Happy learning and coding!


[^1]: Contents of Sakura or Cherry-blossom tree phenological data on Excel format was partly added, replaced re-edited, and reconstructed by Aono and Kazui (2008); Aono and Saito (2010); Aono, 2012.

[^2]: Aono (2012; Chikyu Kankyo (Global Environment), 17, 21-29) 
