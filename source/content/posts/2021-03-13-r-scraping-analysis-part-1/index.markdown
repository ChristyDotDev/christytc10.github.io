---
title: Using R to Scrape and Analyse Data - Part 1 - Pulling Draft Pick Data
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
it into a clean usable format. Here's some code which pulls the 2020 NFL Draft data from Pro Football Reference.


```r
library(rvest)
library(tidyverse)
library(janitor)
library(xml2)

url <- "https://www.pro-football-reference.com/years/2020/draft.htm"
draft_html <- xml2::read_html(url)

final_table <- draft_html %>%
  rvest::html_nodes("table#drafts") %>%
  rvest::html_table() %>%
  .[[1]] %>%
  janitor::row_to_names(row_number = 1) %>%
  select(Rnd, Pick, Tm, Player, Pos, Age, "College/Univ") %>%
  dplyr::mutate(Rnd = as.integer(Rnd),
                    Pick = as.integer(Pick),
                    Age = as.integer(Age))
head(final_table, 5)
```

```
##   Rnd Pick  Tm         Player Pos Age College/Univ
## 2   1    1 CIN     Joe Burrow  QB  23          LSU
## 3   1    2 WAS    Chase Young  DE  21     Ohio St.
## 4   1    3 DET    Jeff Okudah  CB  21     Ohio St.
## 5   1    4 NYG  Andrew Thomas   T  21      Georgia
## 6   1    5 MIA Tua Tagovailoa  QB  22      Alabama
```

Don't worry if the code here looks a bit daunting, it basically breaks down to a few simple steps
chained together. 

The steps here are basically:
 - Pull the HTML from a webpage
 - extract a table from the HTML and load it into a dataframe
 - clean the data and make sure the field names are usable
 - pick off the fields which we actually think will be useful
 - print the first 5 (just to show it worked)

### Grab the HTML
First up, we need to find a webpage with the data we want. One quick point of etiquette I
think it's always worth noting; check a site's robots.txt before scraping any data. This defines which user agents and routes are allowed/disallowed. I won't go into the ins and outs of it here but there's loads of good guides online on how to check a robots.txt file.


```r
url <- "https://www.pro-football-reference.com/years/2020/draft.htm"
draft_html <- xml2::read_html(url)
print(draft_html)
```

```
## {html_document}
## <html data-version="klecko-" data-root="/home/pfr/build" itemscope="" itemtype="https://schema.org/WebSite" lang="en" class="no-js">
## [1] <head>\n<meta http-equiv="Content-Type" content="text/html; charset=UTF-8 ...
## [2] <body class="pfr">\n<div id="wrap">\n  \n  <div id="header" role="banner" ...
```

### Extract the HTML table

It's not immediately obvious from what was printed out there but we've basically pulled the html from the webpage
into a variable (`draft_html`) that we can manipulate and use in the code after.


```r
draft_table <- draft_html %>%
  rvest::html_nodes("table#drafts") %>%
  rvest::html_table() %>%
  .[[1]]
head(draft_table)
```

```
##                                            Misc Misc    Approx Val Approx Val
## 1 Rnd Pick  Tm         Player Pos Age   To  AP1   PB St      CarAV       DrAV
## 2   1    1 CIN     Joe Burrow  QB  23 2020    0    0  0          7          7
## 3   1    2 WAS    Chase Young  DE  21 2020    0    1  1         14         14
## 4   1    3 DET    Jeff Okudah  CB  21 2020    0    0  0          2          2
## 5   1    4 NYG  Andrew Thomas   T  21 2020    0    0  1          8          8
## 6   1    5 MIA Tua Tagovailoa  QB  22 2020    0    0  0          5          5
##      Passing Passing Passing Passing Passing Rushing Rushing Rushing Receiving
## 1  G     Cmp     Att     Yds      TD     Int     Att     Yds      TD       Rec
## 2 10     264     404    2688      13       5      37     142       3         0
## 3 15       0       0       0       0       0       0       0       0         0
## 4  9       0       0       0       0       0       0       0       0         0
## 5 16       0       0       0       0       0       0       0       0         0
## 6 10     186     290    1814      11       5      36     109       3         0
##   Receiving Receiving                                        
## 1       Yds        TD Solo Int  Sk College/Univ              
## 2         0         0                       LSU College Stats
## 3         0         0   32     7.5     Ohio St. College Stats
## 4         0         0   41   1         Ohio St. College Stats
## 5         0         0                   Georgia College Stats
## 6         0         0                   Alabama College Stats
```

This is bit that rvest does really well; picking off the right HTML node and parsing it into something usable.
The html_nodes method lets you pick nodes from the HTML using CSS selectors just like you would in Javascript for a
webpage. In this case `table#drafts` means we are wanting a "table" element with the ID "drafts". Once we have the right nodes, html_table() takes the nodes and turns them into a tibble (basically a simple dataframe).

`[[1]]` above is basically because these methods handle lists of nodes and lists of tibbles and we want to pick off the first one to use. If you look at the output above, it's starting to look like draft data. One tripping hazard though is that if you look closely, the first row which looks like headers is actually being treated as rows of data.

### Get the row names right

[Janitor](https://cran.r-project.org/web/packages/janitor/vignettes/janitor.html#elevate-column-names-stored-in-a-data.frame-row) is a package we can use to clean up data


```r
draft_table <- draft_html %>%
  rvest::html_nodes("table#drafts") %>%
  rvest::html_table() %>%
  .[[1]] %>%
  janitor::row_to_names(row_number = 1)
head(draft_table)
```

```
##   Rnd Pick  Tm         Player Pos Age   To AP1 PB St CarAV DrAV  G Cmp Att  Yds
## 2   1    1 CIN     Joe Burrow  QB  23 2020   0  0  0     7    7 10 264 404 2688
## 3   1    2 WAS    Chase Young  DE  21 2020   0  1  1    14   14 15   0   0    0
## 4   1    3 DET    Jeff Okudah  CB  21 2020   0  0  0     2    2  9   0   0    0
## 5   1    4 NYG  Andrew Thomas   T  21 2020   0  0  1     8    8 16   0   0    0
## 6   1    5 MIA Tua Tagovailoa  QB  22 2020   0  0  0     5    5 10 186 290 1814
## 7   1    6 LAC Justin Herbert  QB  22 2020   0  0  1    13   13 15 396 595 4336
##   TD Int Att Yds TD Rec Yds TD Solo Int  Sk College/Univ              
## 2 13   5  37 142  3   0   0  0                       LSU College Stats
## 3  0   0   0   0  0   0   0  0   32     7.5     Ohio St. College Stats
## 4  0   0   0   0  0   0   0  0   41   1         Ohio St. College Stats
## 5  0   0   0   0  0   0   0  0                   Georgia College Stats
## 6 11   5  36 109  3   0   0  0                   Alabama College Stats
## 7 31  10  55 234  5   0   0  0                    Oregon College Stats
```

### Select the right data

There's quite a lot of data there for each player that for this case we probably don't really need so let's pick off the most relevant stuff so we have a nice valuable subset of data. I just really want to know who was drafted at each pick and by what team. 

A nice way to do this is the select method from dplyr. In true dplyr fashion, it pretty much does what it says on the tin; allowing us to select which columns we want to keep.


```r
draft_table <- draft_html %>%
  rvest::html_nodes("table#drafts") %>%
  rvest::html_table() %>%
  .[[1]] %>%
  janitor::row_to_names(row_number = 1) %>%
  select(Rnd, Pick, Tm, Player, Pos, Age, "College/Univ")

head(draft_table)
```

```
##   Rnd Pick  Tm         Player Pos Age College/Univ
## 2   1    1 CIN     Joe Burrow  QB  23          LSU
## 3   1    2 WAS    Chase Young  DE  21     Ohio St.
## 4   1    3 DET    Jeff Okudah  CB  21     Ohio St.
## 5   1    4 NYG  Andrew Thomas   T  21      Georgia
## 6   1    5 MIA Tua Tagovailoa  QB  22      Alabama
## 7   1    6 LAC Justin Herbert  QB  22       Oregon
```

### Tweaking data types

You might not see from the printed tables but some of the columns above which are supposed to represent numeric values are of type "chr". This basically means that the field is treated as a text string rather than a number. It looks fine now but when you try and run mathematical functions against them it will trip you up. For example "10" is less than "2" when comparing strings. So, if we want to be able to sort by age or pick the players taken in the first 3 rounds, we'll need to convert those fields to a numeric type. R supports integers or numeric (basically a double or integer) but for our cases here I think integer will do fine.

The [mutate](https://dplyr.tidyverse.org/reference/mutate.html) method can be used for this. There are some libraries which support less verbose ways of doing this but I quite like how clear using the `mutate` method is. 


```r
draft_table <- draft_table %>% 
      dplyr::mutate(Rnd = as.integer(Rnd),
                    Pick = as.integer(Pick),
                    Age = as.integer(Age))
# Now we can use the integer fields in mathematical comparisons. For example:
early_round_rbs <- draft_table %>%
  dplyr::filter(Rnd <= 3) %>%
  dplyr::filter(Pos == 'RB')
print(early_round_rbs)
```

```
##    Rnd Pick  Tm                Player Pos Age    College/Univ
## 1    1   32 KAN Clyde Edwards-Helaire  RB  21             LSU
## 2    2   35 DET         D'Andre Swift  RB  21         Georgia
## 3    2   41 IND       Jonathan Taylor  RB  21       Wisconsin
## 4    2   52 LAR             Cam Akers  RB  21     Florida St.
## 5    2   55 BAL          J.K. Dobbins  RB  21        Ohio St.
## 6    2   62 GNB             AJ Dillon  RB  22     Boston Col.
## 7    3   66 WAS        Antonio Gibson  RB  22         Memphis
## 8    3   76 TAM       Ke'Shawn Vaughn  RB  23        Illinois
## 9    3   86 BUF             Zack Moss  RB  22            Utah
## 10   3   93 TEN       Darrynton Evans  RB  22 Appalachian St.
```
Now we can use the integer fields in mathematical comparisons. For example:

```r
early_round_rbs <- draft_table %>%
  dplyr::filter(Rnd <= 3) %>%
  dplyr::filter(Pos == 'RB')
print(early_round_rbs)
```

```
##    Rnd Pick  Tm                Player Pos Age    College/Univ
## 1    1   32 KAN Clyde Edwards-Helaire  RB  21             LSU
## 2    2   35 DET         D'Andre Swift  RB  21         Georgia
## 3    2   41 IND       Jonathan Taylor  RB  21       Wisconsin
## 4    2   52 LAR             Cam Akers  RB  21     Florida St.
## 5    2   55 BAL          J.K. Dobbins  RB  21        Ohio St.
## 6    2   62 GNB             AJ Dillon  RB  22     Boston Col.
## 7    3   66 WAS        Antonio Gibson  RB  22         Memphis
## 8    3   76 TAM       Ke'Shawn Vaughn  RB  23        Illinois
## 9    3   86 BUF             Zack Moss  RB  22            Utah
## 10   3   93 TEN       Darrynton Evans  RB  22 Appalachian St.
```

`dplyr::filter(Rnd <= 3)` does pretty much what you'd expect, filtering the dataset based on some conditions you pass in. If the row has a value of `Rnd` which is less than or equal to 3, the row is included and passed to the next step, otherwise it is discarded. 

### Next

Next post, I'll try and do something useful with the data. Maybe we can pull the last 10 years' draft picks and find trends in the data. 
