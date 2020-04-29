---
title: Creating Choropleth with Mapshaper and R
author: Ardhito Utomo
github: https://github.com/ardhitoutomo
date: '2019-08-18'
slug: creating-choropleth-with-mapshaper-and-r
categories:
  - R
tags:
  - Geocoding
  - Mapshaper
  - Leaflet
  - Map
  - Data Visualization
description: ''
featured: 'banner_choropleth.jpg'
featuredalt: ''
featuredpath: 'date'
linktitle: ''
type: post
---




*Geospatial* is one of the important things in data processing. With *geospatial*, we can provide information only with the help of maps so that information can be conveyed properly.

There are many types of *geospatial* that we can do, such as *Dot Map*, *Connection Map*, *Choropleth*, *Hexbin Map*, and *Bubble Map*[^note_1]. Each type of geospatial has its own function, and each of these types also requires different types of information/data.

This time, we will learn about one form of geocoding which is **choropleth**. Our pursuit of this material:

* Retrieving Data
  + Using **Shapefile**
    - Types of data used
    - Introduction to **mapshaper**
    - Choose the data you want to focus on
    - Reading `.json` data
  + Using `.rds`
* Processing Data
  + Hydrant Data
  + Data Merge
* Presenting Data with `leaflet`

## Retrieving Data


```r
library(prettydoc)
library(tidyverse)
library(ggplot2)
library(leaflet)
library(geojsonio)
library(htmlwidgets)
library(htmltools)
```

First of all, we would like to download the data from [GADM] (https://gadm.org/download_country_v3.html). We will focus on **Indonesia**. In data retrieval, there are two ways we can do it: process the file as a Shapefile data type first, or directly process it using R with `.rds` data. We could also read the entire data using the `readOGR` function from the `rgdal` package, but we won't discuss that this time.

### Shapefile Data

Shapefile data is a collection of data used in many *geospatial* applications. The advantage of Shapefile data is that besides being accessible in many map processing applications, Shapefile data can also be viewed and processed in **mapshaper** (we will discuss it after this) to ensure the information in it before extracting it as the file we need.

When we look at Shapefile data (`.shp`) that we have downloaded from GADM, we can see that each section is divided into several files at once [^note_2]:

+ `.shp`: main data containing geometry information (main)
+ `.shx`: index file of each geometry (main)
+ `.dbf`: data that contains attributes in each area in the data (main)
+ `.prj`: file that stores coordinate system information (optional, used in ArcGIS)
+ `.cpg`: file used to specify the *code page* to identify the character set used (optional)

Because the information that we use is spread to several files at once with different extensions, we have to change it first to one file so that it is more easily accessed/processed in R. For this reason, we will process it first in [mapshaper](https://mapshaper.org/). After some processing and cleaning, we can extract the data in the form of `.json`.

<center> <img src="/img/mapshaper/0.png" width="90%"> </center>

Mapshaper will receive some data and collect it at once. From all 5 extensions we from shapefiles, we can put everything except `.cpg`. We set everything up, we click on the *detect intersection path* section to confirm whether our data have any *intersection*/*contact point* or not, then we *import*.

<center> <img src="/img/mapshaper/1.png" width="90%"> </center>

From the results we have imported, we can see that the `gadm36_IDN_3` data is the data that cuts across the district. In this data, it also happens that the map we get is a 'good' map, because there is no *meeting point*/*intersection*[^note_3].

<center> <img src="/img/mapshaper/2.png" width="90%"> </center>

<center> <img src="/img/mapshaper/2_intersection.png" width="90%"> </center>

<center> <img src="/img/mapshaper/2_intersection_zoom.png" width="90%"> </center>

After that, we would want to see all information about each area that we have from the data we have imported, by pressing the `i` button on the top right. We can see that there is information about the country (in the variable `NAME_0`), provinces (` NAME_1`), districts/cities (`NAME_2`), sub-districts (` NAME_3`), also villages (`NAME_4`) of each place. In addition, there are IDs for each section, but we will not use them here.

We can directly export it in the form of TopoJSON (similar if not lighter than GeoJSON, and we can use this because our data is clean enough [^note_4]). But for now, we will try to select/filter the areas we want to focus on.

In the mapshaper, there is a *console* that we can use to process map data. We can see commands that we can use with `help` and see some examples of using console with `tips`.

We will try to select only in the sub-district (`gadm36_IDN_3`) and we filter only on the DKI Jakarta area:
```
filter 'NAME_1 == "Jakarta Raya"'
```

<center> <img src="/img/mapshaper/3_jakarta_kecamatan.png" width="90%"> </center>

After that, we will export the data as TopoJSON by clicking 'Export' at the top right.

Then, to read the `.json` data in R, we will use the `geojsonio` package. In the `geojson_read` function, we specify `what = "sp"` to make sure the data we read is a spatial class, not *list*. Due to the data we have from different sources and differences in the "Pulau Seribu" section (vs. "Kep. Seribu"), we will omit that section.

```r
# Read the data
jakarta_json <- geojson_read("data_input/clean/gadm36_IDN_3_jkt.json", what = "sp")
```

If you want to check the data first, you can use `@` to see the contents of the table 

```r
head(jakarta_json@data)
```

```
#>     id GID_0    NAME_0   GID_1       NAME_1 NL_NAME_1     GID_2        NAME_2
#> 1 <NA>   IDN Indonesia IDN.7_1 Jakarta Raya           IDN.7.1_1 Jakarta Barat
#> 2 <NA>   IDN Indonesia IDN.7_1 Jakarta Raya           IDN.7.1_1 Jakarta Barat
#> 3 <NA>   IDN Indonesia IDN.7_1 Jakarta Raya           IDN.7.1_1 Jakarta Barat
#> 4 <NA>   IDN Indonesia IDN.7_1 Jakarta Raya           IDN.7.1_1 Jakarta Barat
#> 5 <NA>   IDN Indonesia IDN.7_1 Jakarta Raya           IDN.7.1_1 Jakarta Barat
#> 6 <NA>   IDN Indonesia IDN.7_1 Jakarta Raya           IDN.7.1_1 Jakarta Barat
#>   NL_NAME_2       GID_3           NAME_3 VARNAME_3 NL_NAME_3    TYPE_3
#> 1           IDN.7.1.1_1       Cengkareng                     Kecamatan
#> 2           IDN.7.1.2_1 Grogolpetamburan                     Kecamatan
#> 3           IDN.7.1.3_1        Kalideres                     Kecamatan
#> 4           IDN.7.1.4_1       Kebonjeruk                     Kecamatan
#> 5           IDN.7.1.5_1        Kembangan                     Kecamatan
#> 6           IDN.7.1.6_1         Palmerah                     Kecamatan
#>      ENGTYPE_3    CC_3 HASC_3
#> 1 Sub-district 3174070       
#> 2 Sub-district 3174040       
#> 3 Sub-district 3174080       
#> 4 Sub-district 3174020       
#> 5 Sub-district 3174010       
#> 6 Sub-district 3174030
```


```r
# Change the data type to `sf`
jakarta_json_mod <- sf::st_as_sf(jakarta_json)

# Removing `Kepulauan Seribu` and some columns we won't use 
jakarta_json_mod <- jakarta_json_mod %>% 
  # Change the name as template
  mutate(NAME_3 = str_replace_all(NAME_3, fixed(" "), "") %>% str_to_title()) %>%       
  # Removing `Kepulauan Seribu`
  filter(NAME_2 != "Kepulauan Seribu")  %>%                        
  # Removing some columns
  dplyr::select(-c(id, NL_NAME_1, NL_NAME_2, NL_NAME_3, VARNAME_3, HASC_3, TYPE_3, ENGTYPE_3)) 

glimpse(jakarta_json_mod)
```

```
#> Observations: 45
#> Variables: 10
#> $ GID_0    <chr> "IDN", "IDN", "IDN", "IDN", "IDN", "IDN", "IDN", "IDN", "I...
#> $ NAME_0   <chr> "Indonesia", "Indonesia", "Indonesia", "Indonesia", "Indon...
#> $ GID_1    <chr> "IDN.7_1", "IDN.7_1", "IDN.7_1", "IDN.7_1", "IDN.7_1", "ID...
#> $ NAME_1   <chr> "Jakarta Raya", "Jakarta Raya", "Jakarta Raya", "Jakarta R...
#> $ GID_2    <chr> "IDN.7.1_1", "IDN.7.1_1", "IDN.7.1_1", "IDN.7.1_1", "IDN.7...
#> $ NAME_2   <chr> "Jakarta Barat", "Jakarta Barat", "Jakarta Barat", "Jakart...
#> $ GID_3    <chr> "IDN.7.1.1_1", "IDN.7.1.2_1", "IDN.7.1.3_1", "IDN.7.1.4_1"...
#> $ NAME_3   <chr> "Cengkareng", "Grogolpetamburan", "Kalideres", "Kebonjeruk...
#> $ CC_3     <chr> "3174070", "3174040", "3174080", "3174020", "3174010", "31...
#> $ geometry <MULTIPOLYGON> MULTIPOLYGON (((106.7004 -6..., MULTIPOLYGON (((1...
```

### Data `.rds`
For `.rds` data, we can directly select `R(sf)` in GADM[^note_5]. After that, we can immediately read it.

```r
indonesia_rds <- read_rds("data_input/clean/gadm36_IDN_3_sf.rds")
```

Then because we want to focus on DKI Jakarta, we will process the data first

```r
jakarta_rds <- indonesia_rds %>% 
  mutate(NAME_3 = str_replace_all(NAME_3, fixed(" "), "") %>% str_to_title()) %>%
  filter(NAME_1 == "Jakarta Raya", NAME_2 != "Kepulauan Seribu") %>% 
  dplyr::select(-c(NL_NAME_1, NL_NAME_2, NL_NAME_3, VARNAME_3, HASC_3, TYPE_3, ENGTYPE_3))

glimpse(jakarta_rds)
```

```
#> Observations: 45
#> Variables: 10
#> $ GID_0    <chr> "IDN", "IDN", "IDN", "IDN", "IDN", "IDN", "IDN", "IDN", "I...
#> $ NAME_0   <chr> "Indonesia", "Indonesia", "Indonesia", "Indonesia", "Indon...
#> $ GID_1    <chr> "IDN.7_1", "IDN.7_1", "IDN.7_1", "IDN.7_1", "IDN.7_1", "ID...
#> $ NAME_1   <chr> "Jakarta Raya", "Jakarta Raya", "Jakarta Raya", "Jakarta R...
#> $ GID_2    <chr> "IDN.7.1_1", "IDN.7.1_1", "IDN.7.1_1", "IDN.7.1_1", "IDN.7...
#> $ NAME_2   <chr> "Jakarta Barat", "Jakarta Barat", "Jakarta Barat", "Jakart...
#> $ GID_3    <chr> "IDN.7.1.1_1", "IDN.7.1.2_1", "IDN.7.1.3_1", "IDN.7.1.4_1"...
#> $ NAME_3   <chr> "Cengkareng", "Grogolpetamburan", "Kalideres", "Kebonjeruk...
#> $ CC_3     <chr> "3174070", "3174040", "3174080", "3174020", "3174010", "31...
#> $ geometry <MULTIPOLYGON [°]> MULTIPOLYGON (((106.7004 -6..., MULTIPOLYGON ...
```

We can see that both data already contain the same information and with the same class. Please note that the next time we use `leaflet`, one of the data type they can read is spatial data frames from package `lf` (see `?leaflet` for further details).


```r
class(jakarta_rds)
```

```
#> [1] "sf"         "data.frame"
```

```r
class(jakarta_json_mod)
```

```
#> [1] "sf"         "data.frame"
```

We can continue to use one of the data we have read previously. This time, we will use the data that had been read earlier through mapshaper (`jakarta_json_mod`).


```r
rm(indonesia_rds, jakarta_rds)
```

## Processing The Data
In the data processing stage, we will try to match the contents of the hydrant data which we will later display the information on the map, with the geometry data that have previously read (and processed). This is done to avoid clashes due to different regional names, and it also more reasonable to do so.

### Hydrant Data
We will read the hydrant data which contains information about the hydrants scattered in the Greater Jakarta area along with the conditions (Good/Damaged/Missing) and the location name of the area (**not** the location of the hydrant).

```r
hidran <- read.csv("data_input/clean/hidran.csv")

hidran_mod <- hidran %>% 
  mutate(lokasi = str_replace_all(lokasi, fixed(" "), "") %>% str_to_title()) %>% 
  filter(wilayah != "Kepulauan Seribu") 

head(hidran_mod)
```

```
#>   no       wilayah       lokasi kondisi jumlah  latitude longitude
#> 1  1 Jakarta Pusat       Gambir    Baik     20 -6.169115  106.8165
#> 2  2 Jakarta Pusat   Tanahabang    Baik     36 -6.206441  106.8122
#> 3  3 Jakarta Pusat      Menteng    Baik     30 -6.195425  106.8346
#> 4  4 Jakarta Pusat    Joharbaru    Baik     15 -6.185153  106.8575
#> 5  5 Jakarta Pusat Cempakaputih    Baik     21 -6.180330  106.8687
#> 6  6 Jakarta Pusat    Kemayoran    Baik     34 -6.160978  106.8533
```

There are several other modifications that will be made:

* The condition "Lost"/"Hilang" will be replaced by "Damaged"/"Rusak" due to the "Hilang" factor gives very little or no contribution to the data
* Conditions/"kondisi" will be added by region and location again so that the level from the original "Rusak" and "Rusak" from "Hilang" can be combined into one factor only


```r
hidran_mod <- hidran_mod %>% 
  mutate(kondisi = ifelse(kondisi == "Hilang", "Rusak", paste(kondisi))) %>% 
  group_by(wilayah, lokasi, kondisi) %>% 
  summarise(jumlah = sum(jumlah)) %>% 
  ungroup()

glimpse(hidran_mod)
```

```
#> Observations: 82
#> Variables: 4
#> $ wilayah <fct> Jakarta Barat, Jakarta Barat, Jakarta Barat, Jakarta Barat,...
#> $ lokasi  <chr> "Cengkareng", "Cengkareng", "Grogolpetamburan", "Grogolpeta...
#> $ kondisi <chr> "Baik", "Rusak", "Baik", "Rusak", "Baik", "Rusak", "Baik", ...
#> $ jumlah  <int> 13, 1, 21, 4, 3, 3, 5, 2, 36, 7, 7, 7, 7, 7, 21, 10, 20, 9,...
```

### Combining Data

In merging data, there are several things that we'll do:

* Combining the `json` data we have with the `hydrant` data, where `NAME_2` in the `json` data will be paired with `territory`/`wilayah` in the `hydrant` data, and `NAME_3` with `location`/`lokasi`
* Performs the `complete` function to fill `kondisi`(`Baik`/`Rusak`) in areas not previously listed in the `hydrant` data
* Sort the data to then fill in the `NA` values
* Use the `fill` function from `tidyr` to fill missing values **except** the `amount`/`jumlah` and `kondisi` column, in a downward direction. We do this because there are conditions when we have geometric data but do not have a `jumlah` for each `kondisi`, and vice versa. By doing this, we will get complete information on each column except `jumlah`, `geometry`, and `kondisi` only
* Fill in the value in the `geometry` column using looping function. Here the `lengths` function is used which returns whether a list has contents or not. If it is empty, it will be filled with a list above it (using the `list.take` function from the `rlist` library and taking the list to `[i-1]` or one row above)
* Replacing the value of `NA` in `jumlah` with `0`
* Removes lines with `kondisi == NA`


```r
jakarta_hidran <- jakarta_json_mod %>% 
  left_join(hidran_mod, by = c("NAME_2" = "wilayah", "NAME_3" = "lokasi")) %>% 
  complete(kondisi, NAME_3) %>% 
  arrange(NAME_3, NAME_0) %>% 
  tidyr::fill(-c(jumlah, kondisi), .direction = "down") 

# Create a looping function to fill empty row in `geometri` column
for (i in 1:nrow(jakarta_hidran)) {
  jakarta_hidran$geometry[i] <- ifelse(!lengths(jakarta_hidran$geometry[i]), 
                                    rlist::list.take(jakarta_hidran$geometry[i-1], n = 1), 
                                    jakarta_hidran$geometry[i])
}

jakarta_hidran <- jakarta_hidran %>% 
  mutate(jumlah = replace_na(jumlah, replace = 0)) %>% 
  na.omit()

glimpse(jakarta_hidran)
```

```
#> Observations: 90
#> Variables: 12
#> $ kondisi  <chr> "Baik", "Rusak", "Baik", "Rusak", "Baik", "Rusak", "Baik",...
#> $ NAME_3   <chr> "Cakung", "Cakung", "Cempakaputih", "Cempakaputih", "Cengk...
#> $ GID_0    <chr> "IDN", "IDN", "IDN", "IDN", "IDN", "IDN", "IDN", "IDN", "I...
#> $ NAME_0   <chr> "Indonesia", "Indonesia", "Indonesia", "Indonesia", "Indon...
#> $ GID_1    <chr> "IDN.7_1", "IDN.7_1", "IDN.7_1", "IDN.7_1", "IDN.7_1", "ID...
#> $ NAME_1   <chr> "Jakarta Raya", "Jakarta Raya", "Jakarta Raya", "Jakarta R...
#> $ GID_2    <chr> "IDN.7.4_1", "IDN.7.4_1", "IDN.7.2_1", "IDN.7.2_1", "IDN.7...
#> $ NAME_2   <chr> "Jakarta Timur", "Jakarta Timur", "Jakarta Pusat", "Jakart...
#> $ GID_3    <chr> "IDN.7.4.1_1", "IDN.7.4.1_1", "IDN.7.2.1_1", "IDN.7.2.1_1"...
#> $ CC_3     <chr> "3172080", "3172080", "3173050", "3173050", "3174070", "31...
#> $ jumlah   <dbl> 51, 12, 21, 10, 13, 1, 10, 16, 57, 9, 0, 0, 16, 1, 0, 0, 9...
#> $ geometry <MULTIPOLYGON> MULTIPOLYGON (((106.9508 -6..., MULTIPOLYGON (((1...
```

When we do a simple `leaflet` mapping using `jakarta_hidran` data, we get an error like this:

```r
leaflet(jakarta_hidran) %>% 
  addTiles() %>% 
  addPolygons()
```

```
#> Error in polygonData.default(data): Don't know how to get path data from object of class tbl_df
```

This is because when we perform the `complete` function, our data will be converted into` tbl_df` data type, and this type is not accepted by `leaflet`.

```r
class(jakarta_hidran)
```

```
#> [1] "tbl_df"     "tbl"        "data.frame"
```

For that, we will change it back to `sf`.

```r
jakarta_hidran_sf <- jakarta_hidran %>% sf::st_as_sf()

class(jakarta_hidran_sf)
```

```
#> [1] "sf"         "tbl_df"     "tbl"        "data.frame"
```

## Showing the Choropleth using `leaflet`

In presenting data, we will first put our data into leaflet. Then, the appropriate coloring for *choropleth* will be formed. Palette is determined from red to blue (please see *cheatsheet* that has been provided). In addition, a popup is also made for each place.

```r
# leaflet
m <- leaflet(jakarta_hidran_sf)

# Coloring
temp_baik <- jakarta_hidran_sf %>% 
  filter(kondisi == "Baik") %>% 
  dplyr::select(GID_3, NAME_3, jumlah)
temp_rusak <- jakarta_hidran_sf %>% 
  filter(kondisi == "Rusak") %>% 
  dplyr::select(GID_3, NAME_3, jumlah) 

# -1 <= x <= 1
col_formula <- (temp_baik$jumlah - temp_rusak$jumlah)/(temp_baik$jumlah + temp_rusak$jumlah) 
col <- col_formula %>% replace(is.nan(.), 0)

bins <- c(-1, fivenum(col))

pal <- colorBin("RdYlBu", domain = col, bins = bins)

# Popup
popup.cont  <- paste("<h2><b>", jakarta_hidran_sf$NAME_3, "</b></h2>",
   "<h4><b> Baik: ", temp_baik$jumlah, "</h4></b>",
   "<h4><b> Rusak: ", temp_rusak$jumlah, "</h4></b>"
)
```

And finally, we will form a map with *tiles* provided by `CartoDB` using the `addProviderTiles` function. Then we set polygons and legends on the map. We also change the name of each element in the legend to make it more sensible.

```r
map1 <- m %>% 
  addProviderTiles(providers$CartoDB.DarkMatter) %>% 
  addPolygons(fillColor = ~pal(col),
              weight = 1,
              opacity = 1,
              color = "black",
              dashArray = "3",
              fillOpacity = 0.5,
              label = paste0("Kecamatan: ", jakarta_hidran_sf$NAME_3),
              popup = popup.cont) %>%
  addLegend("bottomright", 
            pal = pal,
            values = ~col,
            title = "Hydrant Condition",
            labFormat = labelFormat(digits = 2),
            opacity = 1)

map1
```



<iframe seamless src="/img/mapshaper/map1.html" width="100%" height="500"></iframe>


For the record, we can also arrange each part of the *legend* by adjusting the colors one by one (taken by using `inspect element`) and we can label it from there. If it is not done like this, then the text in the legend cannot be changed and is still shown in numeric form even though it has been given the `labels` parameter. Also, be aware that in order for the `colors` parameter to work, the `pal` parameter in `addLegend` must be removed.


```r
map2 <- m %>% 
  addProviderTiles(providers$CartoDB.DarkMatter) %>% 
  addPolygons(fillColor = ~pal(col),
              weight = 1,
              opacity = 1,
              color = "black",
              dashArray = "3",
              fillOpacity = 0.5,
              label = paste0("Kecamatan: ", jakarta_hidran_sf$NAME_3),
              popup = popup.cont) %>%
  addLegend("bottomright", 
            values = ~col,
            colors =c("#D7191C", "#FDAE61", "#FFFFBF", "#ABD9E9", "#2C7BB6"), # Taken from `inspect element` with condition parameter `pal` exists
            labels= c("Terrible", "Bad", "Decent/No Hydrant", "Good", "Great"),
            title = "Hydrant Condition",
            labFormat = labelFormat(digits = 2),
            opacity = 1)
map2
```



<iframe seamless src="/img/mapshaper/map2.html" width="100%" height="500"></iframe>


## Additional Information

[^note_1]: https://www.data-to-viz.com/ - examples of using maps for data
[^note_2]: http://webhelp.esri.com/arcgisdesktop/9.3/index.cfm?TopicName=Shapefile_file_extensions - Information about extensions from shapefiles
[^note_3]: If the data we read has a meeting point, we can simplify it before we extract them to avoid things that are not desirable. This can be done by `simplify` at the top right. To see information for each point, please check the `?` Section.
[^note_4]: https://stackoverflow.com/questions/14740705/difference-between-geojson-and-topojson - Differences between GeoJSON and TopoJSON
[^note_5]: http://strimas.com/r/tidy-sf/ - `sf` replaces `sp` to handle spatial objects, and is formed to complement `tidyverse`
