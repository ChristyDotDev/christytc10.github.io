---
title: Using R to scrape and analyse data - Part 1 - Pulling Draft Pick Data
author: Christopher Campbell
date: '2021-03-13'
slug: []
categories:
  - R
  - Blog
  - Learning
tags: []
draft: FALSE
---

R is programming language used for statistical analysis and graphing. I've been using it for
a few months and found it's got a really impressive ecosystem of tools and libraries. My
main interest in using it is to make some sense of the masses of data produced by the NFL to
try and have a less terrible fantasy football team.

One of the first things I wanted to do in R was scrape some useful data from the web to analyse. 
Let's walk through a quick example of how to pull NFL draft pick data from a webpage and parse
it into a clean usable format.


```r
library(rvest)
library(tidyverse)
library(janitor)
library(xml2)

url <- "https://www.pro-football-reference.com/years/2020/draft.htm"
draft_html <- xml2::read_html(url)

draft_table <- draft_html %>%
  rvest::html_nodes("table#drafts") %>%
  rvest::html_table() %>%
  .[[1]] %>%
  janitor::row_to_names(row_number = 1) %>%
  select(Rnd, Pick, Tm, Player, Pos, Age, "College/Univ")
head(draft_table, n=5L)
```

```
##   Rnd Pick  Tm         Player Pos Age College/Univ
## 2   1    1 CIN     Joe Burrow  QB  23          LSU
## 3   1    2 WAS    Chase Young  DE  21     Ohio St.
## 4   1    3 DET    Jeff Okudah  CB  21     Ohio St.
## 5   1    4 NYG  Andrew Thomas   T  21      Georgia
## 6   1    5 MIA Tua Tagovailoa  QB  22      Alabama
```
The steps here are basically:
 - Pull the HTML from a webpage
 - extract a table from the HTML and load it into a dataframe
 - clean the data and make sure the field names are usable
 - pick off the fields which we actually think will be useful
 - shows the first 5

### Grab the HTML
First up, we need to find a webpage with the data we want. One quick point of etiquette I
think it's always worth noting; check a site's robots.txt before scraping any data. This defines which user agents and routes are allowed/disallowed. I won't go into the ins and outs of it here but there's loads of good guides online on how to check a robots.txt file.


```r
url <- "https://www.pro-football-reference.com/years/2020/draft.htm"
draft_html <- xml2::read_html(url)
```

### Extract the HTML table
css selectors

### Get the row names right
janitor package

### Select the right data
select from dplyr
