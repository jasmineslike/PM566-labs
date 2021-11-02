---
title: "Lab9"
author: "Lili Xu"
date: "10/29/2021"
output: 
    html_document:
      html_preview: false
      keep_md: yes
    github_document:
always_allow_html: true
---



# Problem 1: Think
Give yourself a few minutes to think about what you just learned. List three examples of problems that you believe may be solved using parallel computing, and check for packages on the HPC CRAN task view that may be related to it.

# Problem 2: Before you
The following functions can be written to be more efficient without using parallel:

    1.This function generates a n x k dataset with all its entries distributed poission with mean lambda.


```r
fun1 <- function(n = 100, k = 4, lambda = 4) {
  x <- NULL
  for (i in 1:n)
    x <- rbind(x, rpois(k, lambda))
  
  # return(x)
  x
}

fun1(5,10)
```

```
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
## [1,]    6    1    2    2    5    6    3    3    4     7
## [2,]    7    5    3    2    3    4    5    5    1     3
## [3,]    0    5    4    2    3    7    3    4    4     3
## [4,]    1    4    2    4    3    4    3    5    7     6
## [5,]    5    4    2    5    2    2    1    1    2     8
```

```r
fun1alt <- function(n = 100, k = 4, lambda = 4) {
  matrix(rpois(n * k,lambda),nrow = n, ncol = k)
}

# Benchmarking
microbenchmark::microbenchmark(
  fun1(n = 1000),
  fun1alt(n = 1000)
)
```

```
## Unit: microseconds
##               expr      min        lq       mean    median         uq       max
##     fun1(n = 1000) 5841.843 6556.7775 10341.5991 8833.9350 11432.8840 37852.043
##  fun1alt(n = 1000)  162.556  178.4595   216.1852  191.6475   203.6885  2394.984
##  neval cld
##    100   b
##    100  a
```

    2.Find the column max (hint: Checkout the function max.col()).
    

```r
# Data Generating Process (10 x 10,000 matrix)
set.seed(1234)
x <- matrix(rnorm(1e4), nrow=10)

# Find each column's max value
fun2 <- function(x) {
  apply(x, 2, max)
}

fun2alt <- function(x) {
  idx <- max.col(t(x))  
  x[cbind(idx, 1:ncol(x))]
}

# Do we get the same?
all(fun2(x) == fun2alt(x))
```

```
## [1] TRUE
```

```r
# Benchmarking
microbenchmark::microbenchmark(
  fun2(x),
  fun2alt(x)
)
```

```
## Unit: microseconds
##        expr      min       lq      mean    median       uq      max neval cld
##     fun2(x) 1123.076 1239.088 1489.5995 1305.6905 1390.470 5332.331   100   b
##  fun2alt(x)  133.408  165.657  224.3799  176.1355  190.742 3346.852   100  a
```

# Problem 3: Parallelize everyhing
We will now turn our attention to non-parametric bootstrapping. Among its many uses, non-parametric bootstrapping allow us to obtain confidence intervals for parameter estimates without relying on parametric assumptions.

The main assumption is that we can approximate many experiments by resampling observations from our original dataset, which reflects the population.

This function implements the non-parametric bootstrap:

```r
my_boot <- function(dat, stat, R, ncpus = 1L) {
  
  # Getting the random indices
  n <- nrow(dat)
  idx <- matrix(sample.int(n, n*R, TRUE), nrow=n, ncol=R)
 
  # Making the cluster using `ncpus`
  # STEP 1: GOES HERE
  cl <- makePSOCKcluster(ncpus)
   # STEP 2: GOES HERE
  clusterSetRNGStream(cl, 123)  # Equivalent to `set.seed(123)`
  clusterExport(cl, varlist = c("idx", "dat", "stat"), envir = environment())
  
    # STEP 3: THIS FUNCTION NEEDS TO BE REPLACES WITH parLapply
  ans <- parLapply(cl = cl, seq_len(R), function(i) {
    stat(dat[idx[,i], , drop=FALSE])
  })
  
  # Coercing the list into a matrix
  ans <- do.call(rbind, ans)
  
  # STEP 4: GOES HERE
  stopCluster(cl)
  
  ans
  
}
```

    1.Use the previous pseudocode, and make it work with parallel. Here is just an example for you to try:

```r
# Bootstrap of an OLS
my_stat <- function(d) coef(lm(y ~ x, data=d))

# DATA SIM
set.seed(1)
n <- 500; R <- 5e4

x <- cbind(rnorm(n)); y <- x*5 + rnorm(n)

# Checking if we get something similar as lm
ans0 <- confint(lm(y~x))
ans1 <- my_boot(dat = data.frame(x, y), my_stat, R = R, ncpus = 2L)

# You should get something like this
t(apply(ans1, 2, quantile, c(.025,.975)))
```

```
##                   2.5%      97.5%
## (Intercept) -0.1381408 0.04697934
## x            4.8693718 5.04465851
```

```r
##                   2.5%      97.5%
## (Intercept) -0.1372435 0.05074397
## x            4.8680977 5.04539763
ans0
```

```
##                  2.5 %     97.5 %
## (Intercept) -0.1379033 0.04797344
## x            4.8650100 5.04883353
```

```r
##                  2.5 %     97.5 %
## (Intercept) -0.1379033 0.04797344
## x            4.8650100 5.04883353
```
    
    2.Check whether your version actually goes faster than the non-parallel version:

```r
system.time(my_boot(dat = data.frame(x, y), my_stat, R = 4000, ncpus = 1L))
```

```
##    user  system elapsed 
##   0.087   0.015   4.488
```

```r
system.time(my_boot(dat = data.frame(x, y), my_stat, R = 4000, ncpus = 2L))
```

```
##    user  system elapsed 
##   0.108   0.018   2.506
```


# Problem 4: Compile this markdown document using Rscript
Once you have saved this Rmd file, try running the following command in your terminal:

Rscript --vanilla -e 'rmarkdown::render("[full-path-to-your-Rmd-file.Rmd]")' &
Where [full-path-to-your-Rmd-file.Rmd] should be replace with the full path to your Rmd file… :).
