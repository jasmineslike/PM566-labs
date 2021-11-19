---
title: "Assignment4"
author: "Lili Xu"
date: "11/18/2021"
output: 
    html_document:
      html_preview: false
      keep_md: yes
    github_document:
always_allow_html: true
---



The learning objectives are to conduct data scraping and perform text mining.

# HPC

## Problem 1: Make sure your code is nice

Rewrite the following R functions to make them faster. It is OK (and recommended) to take a look at Stackoverflow and Google


```r
# Total row sums
fun1 <- function(mat) {
  n <- nrow(mat)
  ans <- double(n) 
  for (i in 1:n) {
    ans[i] <- sum(mat[i, ])
  }
  ans
}

fun1alt <- function(mat) {
  # YOUR CODE HERE
  ans <- rowSums(mat)
  ans
}

# Cumulative sum by row
fun2 <- function(mat) {
  n <- nrow(mat)
  k <- ncol(mat)
  ans <- mat
  for (i in 1:n) {
    for (j in 2:k) {
      ans[i,j] <- mat[i, j] + ans[i, j - 1]
    }
  }
  ans
}


fun2alt <- function(mat) {
  # YOUR CODE HERE
  ans <- t(apply(mat, 1, cumsum))
  ans
}


# Use the data with this code
set.seed(2315)
dat <- matrix(rnorm(200 * 100), nrow = 200)


# Test for the first
options(microbenchmark.unit="relative")
microbenchmark::microbenchmark(
  fun1(dat),
  fun1alt(dat), check = "equivalent"
)
```

```
## Unit: relative
##          expr      min       lq     mean   median       uq      max neval cld
##     fun1(dat) 9.112981 9.934128 7.957099 10.46613 10.70485 0.571056   100   b
##  fun1alt(dat) 1.000000 1.000000 1.000000  1.00000  1.00000 1.000000   100  a
```

```r
# Test for the second
options(microbenchmark.unit="relative")
microbenchmark::microbenchmark(
  fun2(dat),
  fun2alt(dat), check = "equivalent"
)
```

```
## Unit: relative
##          expr      min       lq     mean   median       uq       max neval cld
##     fun2(dat) 2.839739 2.460805 1.958179 2.492408 2.566983 0.2240664   100   b
##  fun2alt(dat) 1.000000 1.000000 1.000000 1.000000 1.000000 1.0000000   100  a
```

The last argument, check = “equivalent”, is included to make sure that the functions return the same result.

## Problem 2: Make things run faster with parallel computing
The following function allows simulating PI


```r
sim_pi <- function(n = 1000, i = NULL) {
  p <- matrix(runif(n*2), ncol = 2)
  mean(rowSums(p^2) < 1) * 4
}

# Here is an example of the run
set.seed(156)
sim_pi(1000) # 3.132
```

```
## [1] 3.132
```

In order to get accurate estimates, we can run this function multiple times, with the following code:


```r
# This runs the simulation a 4,000 times, each with 10,000 points
set.seed(1231)
system.time({
  ans <- unlist(lapply(1:4000, sim_pi, n = 10000))
  print(mean(ans))
})
```

```
## [1] 3.14124
```

```
##    user  system elapsed 
##   3.117   0.728   3.852
```

Rewrite the previous code using parLapply() to make it run faster. Make sure you set the seed using clusterSetRNGStream():


```r
# YOUR CODE HERE
library(parallel)
system.time({
  # YOUR CODE HERE
  cl <- makePSOCKcluster(4)
  clusterSetRNGStream(cl, 1231)
  ans <- unlist(parLapply(cl, 1:4000, sim_pi, n=10000))
  print(mean(ans))
  # YOUR CODE HERE
  stopCluster(cl)
})
```

```
## [1] 3.141578
```

```
##    user  system elapsed 
##   0.010   0.004   1.416
```


# SQL
Setup a temporary database by running the following chunk


```r
# install.packages(c("RSQLite", "DBI"))

library(RSQLite)
library(DBI)

# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")

# Download tables
film <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film.csv")
film_category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film_category.csv")
category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/category.csv")

# Copy data.frames to database
dbWriteTable(con, "film", film)
dbWriteTable(con, "film_category", film_category)
dbWriteTable(con, "category", category)
```


When you write a new chunk, remember to replace the r with sql, connection=con. Some of these questions will require you to use an inner join. Read more about them here https://www.w3schools.com/sql/sql_join_inner.asp


## Question 1
How many many movies is there available in each rating category.


```sql
SELECT rating AS Rating_Category, COUNT(*) AS COUNT
FROM film
GROUP BY rating
```


<div class="knitsql-table">


Table: 5 records

|Rating_Category | COUNT|
|:---------------|-----:|
|G               |   180|
|NC-17           |   210|
|PG              |   194|
|PG-13           |   223|
|R               |   195|

</div>


## Question 2
What is the average replacement cost and rental rate for each rating category.


```sql
SELECT rating AS Rating_Category, AVG(replacement_cost) AS Avg_replacement_cost, AVG(rental_rate) AS Avg_rental_rate
FROM film
GROUP BY rating
```


<div class="knitsql-table">


Table: 5 records

|Rating_Category | Avg_replacement_cost| Avg_rental_rate|
|:---------------|--------------------:|---------------:|
|G               |             20.12333|        2.912222|
|NC-17           |             20.13762|        2.970952|
|PG              |             18.95907|        3.051856|
|PG-13           |             20.40256|        3.034843|
|R               |             20.23103|        2.938718|

</div>

## Question 3
Use table film_category together with film to find the how many films there are with each category ID


```sql
SELECT film_category.category_id AS Category_id, COUNT(*) AS COUNT
FROM film 
  INNER JOIN film_category ON film.film_id = film_category.film_id
GROUP BY film_category.category_id
ORDER BY COUNT DESC
```


<div class="knitsql-table">


Table: Displaying records 1 - 10

| Category_id| COUNT|
|-----------:|-----:|
|          15|    74|
|           9|    73|
|           8|    69|
|           6|    68|
|           2|    66|
|           1|    64|
|          13|    63|
|           7|    62|
|          14|    61|
|          10|    61|

</div>

## Question 4
Incorporate table category into the answer to the previous question to find the name of the most popular category.


```sql
SELECT film_category.category_id AS Category_id, category.name AS Category_Name, COUNT(*) AS COUNT
FROM film_category 
  LEFT JOIN film ON film_category.film_id = film.film_id
  LEFT JOIN category ON film_category.category_id = category.category_id
GROUP BY film_category.category_id
ORDER BY COUNT DESC
```


<div class="knitsql-table">


Table: Displaying records 1 - 10

| Category_id|Category_Name | COUNT|
|-----------:|:-------------|-----:|
|          15|Sports        |    74|
|           9|Foreign       |    73|
|           8|Family        |    69|
|           6|Documentary   |    68|
|           2|Animation     |    66|
|           1|Action        |    64|
|          13|New           |    63|
|           7|Drama         |    62|
|          14|Sci-Fi        |    61|
|          10|Games         |    61|

</div>
The most popular film category is **Sports**, which has 74 films.


```r
dbDisconnect(con)
```
