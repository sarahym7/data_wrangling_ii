Reading Data From The Web
================
Sarahy Martinez
2024-10-09

``` r
library(tidyverse)
library(rvest)   #new packages, tidyverse adjacent 
library(httr)

#format for your plots and graphs

knitr:: opts_chunk$set(
  fig.width = 6,
  fig.asp = .6,
  out.width = "90%"
)

theme_set(theme_minimal() + theme(legend.position = "bottom"))

options(
  ggplot2.continuous.colour = "viridis",
  ggplot2.continuous.fill = "viridis"
)

scale_colour_discrete = scale_colour_viridis_d
scale_fill_discrete = scale_fill_viridis_d
```

## Scrape a table

First, let’s make sure we can load the data from the web.

I want the first page from \[this page\]
<http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm>

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"
drug_use_html = read_html(url)
# name of dataframe = read_html 

drug_use_html  #when we print its a bit of a mess but dont care, just happy that we have an html doc
```

Doesn’t look like much, but we’re there. Rather than trying to grab
something using a CSS selector, let’s try our luck extracting the tables
from the HTML.

Extract the tables that you want, you can do CSS selector but css tag
for html is common enough that he is just going to guess that what
you’re looking for is a table.

``` r
# option 1 
drug_use_html %>% 
  html_nodes(css = "table")    #extracts html nodes with a particular css tag, have table but not good bc dont have tibble

#option 2
drug_use_html %>% 
  html_table()
```

This has extracted all of the tables on the original page; that’s why we
have a list with 15 elements. (We haven’t really talked about lists yet,
but for now you can think of them as a general collection of objects in
R. As we proceed, syntax for extracting individual elements from a list
will become clear, and we’ll talk lots about lists in list columns.)

We’re only focused on the first table for now, so let’s get the contents
from the first list element.

``` r
#extracting the tables and focusing on one 

table_marj = 
  drug_use_html %>% 
  html_table()  %>%
  first() 

print(table_marj)

# other example 

drug_use_html %>% 
  html_nodes(css = "table") %>% 
  first() %>%   #still formatted html and we want to format as text so we will parce html to strip html and  
  html_table  #get wanted pieces by using the html_table function 
```

In table here if you look at it you’ll notice a problem: the “note” at
the bottom of the table appears in every column in the first row. We
need to remove that…

``` r
table_marj = 
  drug_use_html  %>%
  html_table()  %>%
  first() %>% 
  slice(-1) #removes the first 

table_marj
#notice that data is stored as characters also some superscripts, subscripts, data not necessarily tidy, very least we pulled data from the internet

# other 
drug_use_html %>% 
  html_nodes(css= "table") %>% 
  first() %>% 
  html_table() %>% 
  slice(-1) %>% 
  as_tibble()
```
