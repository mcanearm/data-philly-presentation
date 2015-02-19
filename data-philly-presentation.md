Web-Scraping and Data Munging in R: Hadley Style
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

![hadley](hadley-rice.png)


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
Check out the repository at [https://github.com/mcanear/craig-scrape](https://github.com/mcanearm/scraig-scrape) for source code.

```r
library(CraigScrape)

fields <- c('time', '.hdrlnk', '.housing', 'small', '.price')
reno.apartments <- lapply(urls, function(url) {
  reno.data <- ScrapeCraigslist(url, '.row', fields)
})

reno <- do.call('rbind', reno.apartments)
```

Wait - how did you scrape that?
===


```r
ScrapeCraigslist <- function(url, row.selector, fields) {
  raw.html <- rvest::html(url)
  
  # Return data by css selector on the page
  overall.data <- pblapply(fields, function(field){
    field.data <- raw.html %>%
      html_nodes(row.selector) %>%
      html_node(field) %>%
      html_text()
  })
  
  overall.data <- do.call('cbind.data.frame', overall.data)
  names(overall.data) <- fields
  
  return(overall.data)
}
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
- **`l`: list**
- **`d`: dataframe**
- `m`: multiple inputs
- `r`: repeat multiple times
- `_`: nothing
- First letter: input
- Second letter: output


plyr Example (ddply)
===
**dd**ply - dataframe in, dataframe out!

```r
# Get a dataframe with just the values you want
library(plyr)
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
- Lots of simple data cleaning.
- Complex data cleaning tasks are best handled outside plyr (in my opinion).

*Ex.*


```r
reno.df$housing[1:5]
```

```
[1] "/ 3br - "           "/ 2br - 875ft2 - "  "/ 1br - 470ft2 - " 
[4] "/ 3br - 1181ft2 - " "/ 2br - 1016ft2 - "
```



dplyr
===
- Data selection and querying workhorse
- simple syntax
- `magrittr` and `%>%`
- Emphasis on readability and ease of use
- Some C++ optimizations


dplyr Examples
===
- Grab all the apartments under $1000.

```r
sub_1k <- filter(reno.df, price < 1000)
head(sub_1k[, c('small', 'price')])
```

```
              small price
1  950 Nutmeg Place   699
2              Reno   700
3              <NA>   950
4              <NA>   850
5              <NA>   748
6              Reno   639
```

Chaining Operations
===
- Using the `%>%`, one can pass objects from function to function
- Allows for very readable code
- Ex: get all apartments between $800 and $1200 in Reno, arranged in decreasing
order by square footage per bedroom.


```r
mid_range <- reno.df %>% filter(price >= 800 & price <= 1200, grepl('Reno', small)==TRUE) %>%
  mutate(br_sqft = round(footage2/beds, 2)) %>% 
  arrange(desc(br_sqft)) %>% 
  select(br_sqft, price)
```

Output
===

```r
mid_range[1:5, ]
```

```
  br_sqft price
1     892   905
2     828  1100
3     800  1000
4     796   899
5     770  1195
```
It works just like any other data frame.
===
<img src="data-philly-presentation-figure/unnamed-chunk-14-1.png" title="plot of chunk unnamed-chunk-14" alt="plot of chunk unnamed-chunk-14" width="700px" height="700px" style="display: block; margin: auto;" />

Thanks!
===
- E-mail: <mcanearm@gmail.com>
- Github: <https://github.com/mcanearm>
- Special Thanks to Hadley Wickham
- Questions?
