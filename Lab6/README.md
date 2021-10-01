---
title: "Lab6"
author: "Lili Xu"
date: "10/1/2021"
output: 
    html_document:
      toc: yes 
      toc_float: yes 
      highlight: haddock
      theme: cosmo
      keep_md: yes
    github_document:
      keep_html: false
      html_preview: false
always_allow_html: true
---



## Download the Dataset


```r
library(tidyverse)
```

```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.0 ──
```

```
## ✓ ggplot2 3.3.3     ✓ purrr   0.3.4
## ✓ tibble  3.1.0     ✓ dplyr   1.0.7
## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
## ✓ readr   1.4.0     ✓ forcats 0.5.1
```

```
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(tibble)
library(dplyr)
library(ggplot2)
library(tidytext)
```



```r
if (!file.exists("mtsamples.csv"))
  download.file(
    url = "https://raw.githubusercontent.com/USCbiostats/data-science-data/master/00_mtsamples/mtsamples.csv",
    destfile = "mtsamples.csv",
    method = "libcurl",
    timeout = 60
  )
mtsamples <- read.csv("mtsamples.csv")
mtsamples <- as_tibble(mtsamples)
mtsamples
```

```
## # A tibble: 4,999 x 6
##        X description    medical_special… sample_name  transcription   keywords  
##    <int> <chr>          <chr>            <chr>        <chr>           <chr>     
##  1     0 " A 23-year-o… " Allergy / Imm… " Allergic … "SUBJECTIVE:, … "allergy …
##  2     1 " Consult for… " Bariatrics"    " Laparosco… "PAST MEDICAL … "bariatri…
##  3     2 " Consult for… " Bariatrics"    " Laparosco… "HISTORY OF PR… "bariatri…
##  4     3 " 2-D M-Mode.… " Cardiovascula… " 2-D Echoc… "2-D M-MODE: ,… "cardiova…
##  5     4 " 2-D Echocar… " Cardiovascula… " 2-D Echoc… "1.  The left … "cardiova…
##  6     5 " Morbid obes… " Bariatrics"    " Laparosco… "PREOPERATIVE … "bariatri…
##  7     6 " Liposuction… " Bariatrics"    " Liposucti… "PREOPERATIVE … "bariatri…
##  8     7 " 2-D Echocar… " Cardiovascula… " 2-D Echoc… "2-D ECHOCARDI… "cardiova…
##  9     8 " Suction-ass… " Bariatrics"    " Lipectomy… "PREOPERATIVE … "bariatri…
## 10     9 " Echocardiog… " Cardiovascula… " 2-D Echoc… "DESCRIPTION:,… "cardiova…
## # … with 4,989 more rows
```




## Question 1: What specialties do we have?

We can use count() from dplyr to figure out how many different catagories do we have? Are these catagories related? overlapping? evenly distributed?


```r
specialties <- mtsamples %>%
  count(medical_specialty)

specialties %>%
  arrange(desc(n))%>%
  top_n(15) %>%
  knitr::kable()
```

```
## Selecting by n
```



|medical_specialty             |    n|
|:-----------------------------|----:|
|Surgery                       | 1103|
|Consult - History and Phy.    |  516|
|Cardiovascular / Pulmonary    |  372|
|Orthopedic                    |  355|
|Radiology                     |  273|
|General Medicine              |  259|
|Gastroenterology              |  230|
|Neurology                     |  223|
|SOAP / Chart / Progress Notes |  166|
|Obstetrics / Gynecology       |  160|
|Urology                       |  158|
|Discharge Summary             |  108|
|ENT - Otolaryngology          |   98|
|Neurosurgery                  |   94|
|Hematology - Oncology         |   90|

There are `r nrow(speclialties)' specialties. Let's take a look at the distrubution.

```r
ggplot(mtsamples, aes(x = medical_specialty)) +
  geom_histogram(stat = "count") +
  coord_flip()
```

```
## Warning: Ignoring unknown parameters: binwidth, bins, pad
```

![](README_files/figure-html/dist-1.png)<!-- -->

```r
# method 2
ggplot(specialties, aes(x = n, y = fct_reorder(medical_specialty, n))) +
  geom_col()
```

![](README_files/figure-html/dist2-1.png)<!-- -->

There are not evenly ( uniformly) distribution.

## Question 2


```r
mtsamples %>% 
  unnest_tokens(output = word, input = transcription) %>%
  count(word, sort = TRUE) %>%
  top_n(20) %>%
  ggplot(aes(x=n, y = fct_reorder(word,n))) +
    geom_col()
```

```
## Selecting by n
```

![](README_files/figure-html/token-transcript-1.png)<!-- -->

The word "patient" seems to be important, but we observe a lot of stopwords

## Question 3


## Question 4


## Question 5


## Question 6


## Question 7

