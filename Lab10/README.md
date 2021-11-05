---
title: "Lab 10"
author: "Lili Xu"
date: "11/5/2021"
output: 
    html_document:
      html_preview: false
      keep_md: yes
    github_document:
always_allow_html: true
---



## Setup


```r
# install.packages(c("RSQLite", "DBI"))

library(RSQLite)
library(DBI)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")

# Download tables
actor <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/actor.csv")
rental <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/rental.csv")
customer <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/customer.csv")
payment <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/payment_p2007_01.csv")

# Copy data.frames to database
dbWriteTable(con, "actor", actor)
dbWriteTable(con, "rental", rental)
dbWriteTable(con, "customer", customer)
dbWriteTable(con, "payment", payment)
```


Are the tables there?


```r
dbListTables(con)
```

```
## [1] "actor"    "customer" "payment"  "rental"
```

You can also use knitr + SQL!

TIP: Use can use the following QUERY to see the structure of a table

```sql
PRAGMA table_info(actor)
```


This is equivalent to use `dbGetQuery`

#### Actor

```r
dbGetQuery(con, "PRAGMA table_info(actor)") %>%
  knitr::kable()
```



| cid|name        |type    | notnull|dflt_value | pk|
|---:|:-----------|:-------|-------:|:----------|--:|
|   0|actor_id    |INTEGER |       0|NA         |  0|
|   1|first_name  |TEXT    |       0|NA         |  0|
|   2|last_name   |TEXT    |       0|NA         |  0|
|   3|last_update |TEXT    |       0|NA         |  0|

#### Customer

```r
dbGetQuery(con, "PRAGMA table_info(customer)")　%>%
  knitr::kable()
```



| cid|name        |type    | notnull|dflt_value | pk|
|---:|:-----------|:-------|-------:|:----------|--:|
|   0|customer_id |INTEGER |       0|NA         |  0|
|   1|store_id    |INTEGER |       0|NA         |  0|
|   2|first_name  |TEXT    |       0|NA         |  0|
|   3|last_name   |TEXT    |       0|NA         |  0|
|   4|email       |TEXT    |       0|NA         |  0|
|   5|address_id  |INTEGER |       0|NA         |  0|
|   6|activebool  |TEXT    |       0|NA         |  0|
|   7|create_date |TEXT    |       0|NA         |  0|
|   8|last_update |TEXT    |       0|NA         |  0|
|   9|active      |INTEGER |       0|NA         |  0|

#### Rental

```r
dbGetQuery(con, "PRAGMA table_info(rental)") %>%
  knitr::kable()
```



| cid|name         |type    | notnull|dflt_value | pk|
|---:|:------------|:-------|-------:|:----------|--:|
|   0|rental_id    |INTEGER |       0|NA         |  0|
|   1|rental_date  |TEXT    |       0|NA         |  0|
|   2|inventory_id |INTEGER |       0|NA         |  0|
|   3|customer_id  |INTEGER |       0|NA         |  0|
|   4|return_date  |TEXT    |       0|NA         |  0|
|   5|staff_id     |INTEGER |       0|NA         |  0|
|   6|last_update  |TEXT    |       0|NA         |  0|

#### Payment

```r
dbGetQuery(con, "PRAGMA table_info(payment)") %>%
  knitr::kable()
```



| cid|name         |type    | notnull|dflt_value | pk|
|---:|:------------|:-------|-------:|:----------|--:|
|   0|payment_id   |INTEGER |       0|NA         |  0|
|   1|customer_id  |INTEGER |       0|NA         |  0|
|   2|staff_id     |INTEGER |       0|NA         |  0|
|   3|rental_id    |INTEGER |       0|NA         |  0|
|   4|amount       |REAL    |       0|NA         |  0|
|   5|payment_date |TEXT    |       0|NA         |  0|

## Exercise 1
Retrive the actor ID, first name and last name for all actors using the actor table. Sort by last name and then by first name.

And using the LIMIT clause (`head()` in R ) to just show the first 5


```r
dbGetQuery(con, "
/* This is COMMENT! */
SELECT actor_id, first_name, last_name
FROM actor /* YOU CAN ADD COMMENTS USING
MULTIPLE LINES! */
ORDER by last_name, first_name 
LIMIT 5") %>%
  knitr::kable()
```



| actor_id|first_name |last_name |
|--------:|:----------|:---------|
|       58|CHRISTIAN  |AKROYD    |
|      182|DEBBIE     |AKROYD    |
|       92|KIRSTEN    |AKROYD    |
|      118|CUBA       |ALLEN     |
|      145|KIM        |ALLEN     |

## Exercise 2
Retrive the actor ID, first name, and last name for actors whose last name equals ‘WILLIAMS’ or ‘DAVIS’.


```r
dbGetQuery(con, "
SELECT actor_id, first_name, last_name
FROM actor
WHERE last_name IN ('WILLIAMS', 'DAVIS')
LIMIT 5") %>%
  knitr::kable()
```



| actor_id|first_name |last_name |
|--------:|:----------|:---------|
|        4|JENNIFER   |DAVIS     |
|       72|SEAN       |WILLIAMS  |
|      101|SUSAN      |DAVIS     |
|      110|SUSAN      |DAVIS     |
|      137|MORGAN     |WILLIAMS  |


## Exercise 3
Write a query against the rental table that returns the IDs of the customers who rented a film on July 5, 2005 (use the rental.rental_date column, and you can use the date() function to ignore the time component). Include a single row for each distinct customer ID.



```r
dbGetQuery(con, "
SELECT DISTINCT customer_id
FROM rental
WHERE date(rental_date) = '2005-07-05'
LIMIT 5") %>%
  knitr::kable()
```



| customer_id|
|-----------:|
|         565|
|         242|
|          37|
|          60|
|         594|


## Exercise 4


### Exercise 4.1
Construct a query that retrives all rows from the payment table where the amount is either 1.99, 7.99, 9.99.

```r
q <- dbSendQuery(con, "
SELECT *
FROM payment
WHERE amount IN (1.99, 7.99, 9.99)")

dbFetch(q , n = 10) %>%
  knitr::kable()
```



| payment_id| customer_id| staff_id| rental_id| amount|payment_date               |
|----------:|-----------:|--------:|---------:|------:|:--------------------------|
|      16050|         269|        2|         7|   1.99|2007-01-24 21:40:19.996577 |
|      16056|         270|        1|       193|   1.99|2007-01-26 05:10:14.996577 |
|      16081|         282|        2|        48|   1.99|2007-01-25 04:49:12.996577 |
|      16103|         294|        1|       595|   1.99|2007-01-28 12:28:20.996577 |
|      16133|         307|        1|       614|   1.99|2007-01-28 14:01:54.996577 |
|      16158|         316|        1|      1065|   1.99|2007-01-31 07:23:22.996577 |
|      16160|         318|        1|       224|   9.99|2007-01-26 08:46:53.996577 |
|      16161|         319|        1|        15|   9.99|2007-01-24 23:07:48.996577 |
|      16180|         330|        2|       967|   7.99|2007-01-30 17:40:32.996577 |
|      16206|         351|        1|      1137|   1.99|2007-01-31 17:48:40.996577 |

```r
dbClearResult(q) 
```


### Exercise 4.2
Construct a query that retrives all rows from the `payment` table where the amount is greater then 5


```r
dbGetQuery(con, "
SELECT *
FROM payment
WHERE amount > 5
LIMIT 5") %>%
  knitr::kable()
```



| payment_id| customer_id| staff_id| rental_id| amount|payment_date               |
|----------:|-----------:|--------:|---------:|------:|:--------------------------|
|      16052|         269|        2|       678|   6.99|2007-01-28 21:44:14.996577 |
|      16058|         271|        1|      1096|   8.99|2007-01-31 11:59:15.996577 |
|      16060|         272|        1|       405|   6.99|2007-01-27 12:01:05.996577 |
|      16061|         272|        1|      1041|   6.99|2007-01-31 04:14:49.996577 |
|      16068|         274|        1|       394|   5.99|2007-01-27 09:54:37.996577 |

#### Bouns: Count how many are


```r
dbGetQuery(con, "
SELECT COUNT(*)
FROM payment
WHERE amount > 5") %>%
  knitr::kable()
```



| COUNT(*)|
|--------:|
|      266|

Counting per `staff_id`


```r
dbGetQuery(con, "
SELECT staff_id, COUNT(*) AS N
FROM payment
WHERE amount > 5
GROUP BY staff_id") %>%
  knitr::kable()
```



| staff_id|   N|
|--------:|---:|
|        1| 151|
|        2| 115|

### Exercise 4.3
Construct a query that retrives all rows from the payment table where the amount is greater then 5 and less then 8


```r
dbGetQuery(con, "
SELECT *
FROM payment
WHERE amount > 5 AND amount < 8
LIMIT 5") %>%
  knitr::kable()
```



| payment_id| customer_id| staff_id| rental_id| amount|payment_date               |
|----------:|-----------:|--------:|---------:|------:|:--------------------------|
|      16052|         269|        2|       678|   6.99|2007-01-28 21:44:14.996577 |
|      16060|         272|        1|       405|   6.99|2007-01-27 12:01:05.996577 |
|      16061|         272|        1|      1041|   6.99|2007-01-31 04:14:49.996577 |
|      16068|         274|        1|       394|   5.99|2007-01-27 09:54:37.996577 |
|      16074|         277|        2|       308|   6.99|2007-01-26 20:30:05.996577 |


## Exercise 5
Retrive all the payment IDs and their amount from the customers whose last name is ‘DAVIS’.



```r
dbGetQuery(con, "
SELECT payment.payment_id, payment.amount
FROM payment
  INNER JOIN customer ON payment.customer_id = customer.customer_id
WHERE customer.last_name = 'DAVIS'") %>%
  knitr::kable()
```



| payment_id| amount|
|----------:|------:|
|      16685|   4.99|
|      16686|   2.99|
|      16687|   0.99|



## Exercise 6


### Exercise 6.1
Use COUNT(*) to count the number of rows in rental


```r
dbGetQuery(con, "
SELECT COUNT(*) as Total_Rentals
FROM rental") %>%
  knitr::kable()
```



| Total_Rentals|
|-------------:|
|         16044|


### Exercise 6.2
Use COUNT(*) and GROUP BY to count the number of rentals for each customer_id


```r
dbGetQuery(con, "
SELECT customer_id, COUNT(*) AS 'N Rentals'
FROM rental
GROUP BY customer_id
LIMIT 10") %>%
  knitr::kable()
```



| customer_id| N Rentals|
|-----------:|---------:|
|           1|        32|
|           2|        27|
|           3|        26|
|           4|        22|
|           5|        38|
|           6|        28|
|           7|        33|
|           8|        24|
|           9|        23|
|          10|        25|


### Exercise 6.3
Repeat the previous query and sort by the count in descending order


```r
dbGetQuery(con, "
SELECT customer_id, COUNT(*) AS 'N Rentals'
FROM rental
GROUP BY customer_id
ORDER BY `N Rentals` DESC
LIMIT 10") %>%
  knitr::kable()
```



| customer_id| N Rentals|
|-----------:|---------:|
|         148|        46|
|         526|        45|
|         236|        42|
|         144|        42|
|          75|        41|
|         469|        40|
|         197|        40|
|         468|        39|
|         178|        39|
|         137|        39|



### Exercise 6.4
Repeat the previous query but use HAVING to only keep the groups with 40 or more.


```r
dbGetQuery(con, "
SELECT customer_id, COUNT(*) AS 'N Rentals'
FROM rental
GROUP BY customer_id
HAVING `N Rentals` >= 40
ORDER BY `N Rentals` DESC") %>%
  knitr::kable()
```



| customer_id| N Rentals|
|-----------:|---------:|
|         148|        46|
|         526|        45|
|         236|        42|
|         144|        42|
|          75|        41|
|         469|        40|
|         197|        40|


## Exercise 7

The following query calculates a number of summary statistics for the payment table using MAX, MIN, AVG and SUM


```r
dbGetQuery(con, "
SELECT 
  MAX(amount) AS Max, 
  MIN(amount) AS Min, 
  AVG(amount) AS Avg, 
  SUM(amount) AS Sum
FROM payment") %>%
  knitr::kable()
```



|   Max|  Min|      Avg|     Sum|
|-----:|----:|--------:|-------:|
| 11.99| 0.99| 4.169775| 4824.43|


### Exercise 7.1
Modify the above query to do those calculations for each customer_id


```r
dbGetQuery(con, "
SELECT 
  customer_id, 
  MAX(amount) AS Max, 
  MIN(amount) AS Min,
  AVG(amount) AS Avg, 
  SUM(amount) AS Sum
FROM payment
GROUP BY customer_id
LIMIT 10") %>%
  knitr::kable()
```



| customer_id|  Max|  Min|      Avg|   Sum|
|-----------:|----:|----:|--------:|-----:|
|           1| 2.99| 0.99| 1.990000|  3.98|
|           2| 4.99| 4.99| 4.990000|  4.99|
|           3| 2.99| 1.99| 2.490000|  4.98|
|           5| 6.99| 0.99| 3.323333|  9.97|
|           6| 4.99| 0.99| 2.990000|  8.97|
|           7| 5.99| 0.99| 4.190000| 20.95|
|           8| 6.99| 6.99| 6.990000|  6.99|
|           9| 4.99| 0.99| 3.656667| 10.97|
|          10| 4.99| 4.99| 4.990000|  4.99|
|          11| 6.99| 6.99| 6.990000|  6.99|



### Exercise 7.2
Modify the above query to only keep the customer_ids that have more then 5 payments


```r
dbGetQuery(con, "
SELECT 
  customer_id, 
  MAX(amount) AS Max, 
  MIN(amount) AS Min,
  AVG(amount) AS Avg, 
  SUM(amount) AS Sum
FROM payment
GROUP BY customer_id
HAVING COUNT(customer_id) > 5") %>%
  knitr::kable()
```



| customer_id|  Max|  Min|      Avg|   Sum|
|-----------:|----:|----:|--------:|-----:|
|          19| 9.99| 0.99| 4.490000| 26.94|
|          53| 9.99| 0.99| 4.490000| 26.94|
|         109| 7.99| 0.99| 3.990000| 27.93|
|         161| 5.99| 0.99| 2.990000| 17.94|
|         197| 3.99| 0.99| 2.615000| 20.92|
|         207| 6.99| 0.99| 2.990000| 17.94|
|         239| 7.99| 2.99| 5.656667| 33.94|
|         245| 8.99| 0.99| 4.823333| 28.94|
|         251| 4.99| 1.99| 3.323333| 19.94|
|         269| 6.99| 0.99| 3.156667| 18.94|
|         274| 5.99| 2.99| 4.156667| 24.94|
|         371| 6.99| 0.99| 4.323333| 25.94|
|         506| 8.99| 0.99| 4.132857| 28.93|
|         596| 6.99| 0.99| 3.823333| 22.94|



## Clean up
Run the following chunk to disconnect from the connection.

```r
# clean up
dbDisconnect(con)
```

