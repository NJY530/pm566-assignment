Assignment4
================
Jiayi Nie

# HPC

## Problem1: Make sure your code is nice

Rewrite the following R functions to make them faster. It is OK (and
recommended) to take a look at Stackoverflow and Google

``` r
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
  ans <- t(apply(mat, 1, cumsum))
  ans
}


# Use the data with this code
set.seed(2315)
dat <- matrix(rnorm(200 * 100), nrow = 200)

# Test for the first
microbenchmark::microbenchmark(
  fun1(dat),
  fun1alt(dat), unit = "relative", check = "equivalent"
)
```

    ## Unit: relative
    ##          expr     min       lq     mean   median       uq       max neval
    ##     fun1(dat) 9.45097 10.44049 6.903023 10.27866 9.513358 0.4476035   100
    ##  fun1alt(dat) 1.00000  1.00000 1.000000  1.00000 1.000000 1.0000000   100

``` r
# Test for the second
microbenchmark::microbenchmark(
  fun2(dat),
  fun2alt(dat), unit = "relative", check = "equivalent"
)
```

    ## Unit: relative
    ##          expr      min      lq   mean  median      uq       max neval
    ##     fun2(dat) 3.158272 2.41579 1.7627 2.35387 2.30235 0.1988973   100
    ##  fun2alt(dat) 1.000000 1.00000 1.0000 1.00000 1.00000 1.0000000   100

## Problem2: Make things run faster with parallel computing

The following function allows simulating PI

``` r
sim_pi <- function(n = 1000, i = NULL) {
  p <- matrix(runif(n*2), ncol = 2)
  mean(rowSums(p^2) < 1) * 4
}

# Here is an example of the run
set.seed(156)
sim_pi(1000) # 3.132
```

    ## [1] 3.132

In order to get accurate estimates, we can run this function multiple
times, with the following code:

``` r
# This runs the simulation a 4,000 times, each with 10,000 points
set.seed(1231)
system.time({
  ans <- unlist(lapply(1:4000, sim_pi, n = 10000))
  print(mean(ans))
})
```

    ## [1] 3.14124

    ##    user  system elapsed 
    ##   2.822   0.846   3.715

Rewrite the previous code using parLapply() to make it run faster. Make
sure you set the seed using clusterSetRNGStream():

``` r
library(parallel)
system.time({
  
  cl <- parallel::makeCluster(4, setup_strategy = "sequential")
  
  clusterSetRNGStream(cl, 1231)
  
  ans <- unlist(parLapply(cl = cl, 1:4000, sim_pi, n = 10000))
  print(mean(ans))
  
  stopCluster(cl)
  
  ans
})
```

    ## [1] 3.141578

    ##    user  system elapsed 
    ##   0.017   0.010   1.916

# SQL

Setup a temporary database by running the following chunk

``` r
#install.packages(c("RSQLite", "DBI"))

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

## Question1: How many movies is there avaliable in each rating catagory.

``` sql
SELECT rating as 'Rating', COUNT(*) AS 'count of film'
FROM film
GROUP BY rating
```

<div class="knitsql-table">

| Rating | count of film |
| :----- | ------------: |
| G      |           180 |
| NC-17  |           210 |
| PG     |           194 |
| PG-13  |           223 |
| R      |           195 |

5 records

</div>

## Question2: What is the average replacement cost and rental rate for each rating category.

``` sql
SELECT rating,
  AVG(replacement_cost) AS avg_replcement,
  AVG(rental_rate) AS avg_rental
FROM film
GROUP BY rating
```

<div class="knitsql-table">

| rating | avg\_replcement | avg\_rental |
| :----- | --------------: | ----------: |
| G      |        20.12333 |    2.912222 |
| NC-17  |        20.13762 |    2.970952 |
| PG     |        18.95907 |    3.051856 |
| PG-13  |        20.40256 |    3.034843 |
| R      |        20.23103 |    2.938718 |

5 records

</div>

## Question3: Use table film\_category together with film to find the how many films there are witth each category ID

``` sql
SELECT category_id,
  COUNT (*) AS Counts
FROM film AS f
  INNER JOIN film_category AS c
ON f.film_id = c.film_id
GROUP BY category_id
```

<div class="knitsql-table">

| category\_id | Counts |
| :----------- | -----: |
| 1            |     64 |
| 2            |     66 |
| 3            |     60 |
| 4            |     57 |
| 5            |     58 |
| 6            |     68 |
| 7            |     62 |
| 8            |     69 |
| 9            |     73 |
| 10           |     61 |

Displaying records 1 - 10

</div>

## Question4: Incorporate table category into the answer to the previous question to find the name of the most popular category.

``` sql
SELECT film_category.category_id as cate_id, category.name as cate_name,
COUNT(*) as count 
FROM film_category
  LEFT JOIN film ON film_category.film_id = film.film_id
  LEFT JOIN category ON film_category.category_id = category.category_id
GROUP BY cate_id
ORDER BY count DESC
```

<div class="knitsql-table">

| cate\_id | cate\_name  | count |
| -------: | :---------- | ----: |
|       15 | Sports      |    74 |
|        9 | Foreign     |    73 |
|        8 | Family      |    69 |
|        6 | Documentary |    68 |
|        2 | Animation   |    66 |
|        1 | Action      |    64 |
|       13 | New         |    63 |
|        7 | Drama       |    62 |
|       14 | Sci-Fi      |    61 |
|       10 | Games       |    61 |

Displaying records 1 - 10

</div>

The most popular category is Sports.

``` r
dbDisconnect(con)
```
