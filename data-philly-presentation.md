Web-Scraping and Data Munging in R: An Ode to Hadley Wickham
========================================================
author: Matthew McAnear
date: Feb. 19, 2015

Intro
===

- Matthew McAnear
- Data Scientist with Seer Interactive
- Bucknell '13 / UPenn '14
- Volunteer Work
- I live in South Philly.

***

![me](baby.png)

Surprise
===
I'm moving to Reno tomorrow.
![reno](philly_reno.png)

Packages in this talk
===
- `rvest`
- `plyr`
- `dplyr`
- `magrittr`

Web-Scraping
===
First, you need to create a list of URLs to scrape.


```r
base_url <- 'http://reno.craigslist.org/search/apa?s='
page_index <- seq(0, 500, by=100)
urls <- paste0(base_url, page_index)
```

This Gives You:
===

```r
urls
```

```
[1] "http://reno.craigslist.org/search/apa?s=0"  
[2] "http://reno.craigslist.org/search/apa?s=100"
[3] "http://reno.craigslist.org/search/apa?s=200"
[4] "http://reno.craigslist.org/search/apa?s=300"
[5] "http://reno.craigslist.org/search/apa?s=400"
[6] "http://reno.craigslist.org/search/apa?s=500"
```

Using Rvest
===
Check out the repository at [https://github.com/mcanear/shoppeR](https://github.com/mcanearm/shoppeR) for source code.

```r
library(shoppeR)

fields <- c('time', '.hdrlnk', '.housing', 'small', '.price')
reno.apartments <- lapply(urls, function(url) {
  reno.data <- ScrapeCraigslist(url, '.row', fields)
})

reno <- do.call('rbind', reno.apartments)
```


What does this data look like?
===

```
'data.frame':	600 obs. of  5 variables:
 $ time    : Factor w/ 2 levels "Feb 18","Feb 17": 1 1 1 1 1 1 1 1 1 1 ...
 $ .hdrlnk : Factor w/ 563 levels "$299 FIRST MONTH RENT!! DONT MISS OUT!",..: 41 8 9 35 43 66 16 47 11 30 ...
 $ .housing: Factor w/ 274 levels "/ 1322ft2 - ",..: 33 31 4 36 22 18 6 15 31 29 ...
 $ small   : Factor w/ 222 levels " (1071 Hospital Rd., Schurz, NV)",..: 16 6 17 NA NA NA 17 6 6 29 ...
 $ .price  : Factor w/ 189 levels "$1005","$1013",..: 6 33 34 50 44 37 31 30 33 44 ...
```

How do we clean all this data?
============================================================================
- **plyr**
- **dplyr**
- **magrittr**

What are plyr and dplyr?
========================================================

- R Packages for data cleaning
- Hadley Wickham (`ggplot2`, `rvest`, `stringr`...)
- Focus is on easy syntax and readability

plyr Naming Conventions
===
- `a`: array
- `l`: list
- `d`: dataframe
- `m`: multiple inputs
- `r`: repeat multiple times
- `_`: nothing
- First letter: input
- Second letter: output

Loading Some Example Data
========================================================

Same/similar data, but scraped yesterday.

```r
library(plyr)
library(shoppeR)

data(reno)
```


plyr Example (ddply)
===
**dd**ply - dataframe in, dataframe out!

```r
# Get a dataframe with just the values you want
reno.df <- ddply(reno, .(), mutate, 
                 price = as.integer(gsub('\\$', '', as.character(.price))),
                 small = gsub('\\(|\\)', '', as.character(small)),
                 housing = as.character(.housing),
                 time = as.character(time),
                 description = as.character(.hdrlnk))
```

ddply Output
========================================================
*Some Columns ommitted from the output.

```
    time             small price
1 Feb 18      Reno / Tahoe  1142
2 Feb 18  950 Nutmeg Place   699
3 Feb 18              Reno   700
4 Feb 18              <NA>   950
5 Feb 18              <NA>   850
6 Feb 18              <NA>   748
```

Another plyr Example (dlply)
===

**dl**ply - dataframe in, list out!

```r
mean.rents <- dlply(reno.df, ~time, summarize,
                    mean.rent = round(mean(price, na.rm=TRUE), 0))
mean.rents[1:2]
```

```
$`Feb 17`
  mean.rent
1      1016

$`Feb 18`
  mean.rent
1       979
```

What is plyr best for?
===
- Lots of data simple data cleaning.
- Complex data cleaning tasks are best handled outside plyr (in my opinion).

*Ex.*


```r
reno.df$housing[1:5]
```

```
[1] "/ 3br - "           "/ 2br - 875ft2 - "  "/ 1br - 470ft2 - " 
[4] "/ 3br - 1181ft2 - " "/ 2br - 1016ft2 - "
```




```
Error in `$<-.data.frame`(`*tmp*`, "housing", value = c(" 3br ", " 2br ",  : 
  replacement has 530 rows, data has 600
```
