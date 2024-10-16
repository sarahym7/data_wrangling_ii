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

# other example but doesnt work 

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

# other but doesnt run
drug_use_html %>% 
  html_nodes(css= "table") %>% 
  first() %>% 
  html_table() %>% 
  slice(-1) %>% 
  as_tibble()
```

Now lets shift and try to get a non-table collection of data from the
website

Suppose we’d like to scrape the data about the Star Wars Movies from the
IMDB page. The first step is the same as before – we need to get the
HTML.

## Star Wars Movie Info

I want the data from [here](https://www.imdb.com/list/ls070150896/)

``` r
swm_html = 
  read_html("https://www.imdb.com/list/ls070150896/")

#can also do this 

url = "https://www.imdb.com/list/ls070150896/"

swm_htmls = read_html(url)
```

The information isn’t stored in a handy table, so we’re going to isolate
the CSS selector for elements we care about. A bit of clicking around
gets me something like below.

Grab elements that we want: - before said give me the tables but we dont
know the css tag imdb for movie titles. We willfocus on titles and make
a vector. Get the star wars html and extract

``` r
title_vectors_ex_one = 
  swm_html %>% 
  html_nodes(css=".lister-item-header a") %>% 
  html_text()#because its not a table with info we dont know which css function, so use selector gadget page

gross_rev_vec = 
  swm_html %>% 
  html_nodes(css=".text-small:nth-child(7) span:nth-child(5)") %>% 
  html_text()

runtime_vec= 
  swm_html %>% 
  html_nodes(css=".runtime") %>% 
  html_text()

#putting it all together 

swm_df_video= 
  tibble(
    tibble= title_vec,
    gross_rev_vec = gross_rev_vec,
    runtime_vec = runtime_vec
  )
```

``` r
title_vec = 
  swm_html %>% 
  html_elements(".ipc-title-link-wrapper .ipc-title__text") %>% 
  html_text()

metascore_vec = 
  swm_html %>% 
  html_elements(".metacritic-score-box") %>% 
  html_text()

runtime_vec = 
  swm_html %>% 
  html_elements(".dli-title-metadata-item:nth-child(2)") %>% #knowing how long the movies are, find elements associated                                                                #with runtime pull it out and 
  html_text()   #convert to text

swm_df = 
  tibble(
    title = title_vec,
    score = metascore_vec,
    runtime = runtime_vec)
```

We need to extract some bit of css for the movie title on page, click
selector gadget and scroll through will highlight boxes of content and
paths. Click on smallest box that includes what we care about. unclick
things that show up unrelated that you want. Then selector will tell you
the package to use.video

## Using API

request ap and export as CSV

The following is from the API GET in package want you tell them the API
you care about

``` r
nyc_water = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.csv") %>%  #we got a csv but to work with we need an xtra step
  content("parsed")   #same way we got HTML and needed to parse to go from table to tibble need content step to parse.
                        #we can be more specific on kind of way we want content
```

## BRFSS data

go to the cdc dataset look at data.gov a lot info from api, we can
export an entire dataset but they get updated so use API acess via soda
api and straight to CSV query, path etc are functions that APIs use run
?GET and provides several other examples

``` r
brfss_2010 = 
  GET("https://chronicdata.cdc.gov/resource/acme-vg9e.csv",  #bad news we have 1000 rows and in cdc says 140k
      query = list("$limit" = 5000)) |> #use the limit for socrata API default is 1000 , $offset default is 0.we know we have to use list, so set up query as a list where $limit =5000 and hopefully it makes the request of 5000 rows and its unlimited so we can go up to 150k
  content("parsed")
```

## Some data aren’t so nice

``` r
pokemon_data = 
  GET("http://pokeapi.co/api/v2/pokemon/1") %>%  
  content()
#if we just take this and try to extra content will give a long structured collection of the output. can also get structured nested in structure. accessing data first part of problem 

#easy things to get 

pokemon_data$name 
pokemon_data$height
pokemon_data$abilities
```

``` r
nyc_water = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.json") |> #json structure is v different, use json instead of csv if we want second variable as a collection of things
  content("text") |>  #cant parse, instead use the text from json which looks worse 
  jsonlite::fromJSON() |>  #we can parse if we use json lite and makes look like a df now 
  as_tibble()  #convert to a tibble 

brfss_smart2010 = 
  GET("https://chronicdata.cdc.gov/resource/acme-vg9e.csv",
      query = list("$limit" = 5000)) |> 
  content("parsed")

poke = 
  GET("http://pokeapi.co/api/v2/pokemon/1") |>
  content()

poke[["name"]]

poke[["height"]]

poke[["abilities"]]
```
