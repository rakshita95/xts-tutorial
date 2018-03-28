xts
================

### Introduction

Xts stands for extensible time series and is an extension of zoo.
Primarily, it makes working with time series data more consistent by
incorporating various R objects used to represent time. By converting
data to an xts object, we no longer need to worry about the class of
object used to represent time (POSIXct, Date etc.). This avoids
confusion as xts objects work consistently across a range of functions
as we will see in the tutorial. It also provides many useful functions
for manipulating and analyzing time series data and handling missing
values.

### Structure of an XTS object

XTS objects can be viewed as simple R matrices with an index of
corresponding date and time. It can also contain additional time based
attributes like date created etc. (hence, extensible)

To understand this better, let us create an xts object `work` that
stores the number of hours Bob worked along with some attributes about
him like birth date. We can look at the structure of an xts object using
the `str()` function.

``` r
library(xts)

# Create the object data using 5 random numbers
hours <- rnorm(20, mean = 8)

# Create dates as a Date class object starting from 1922-01-01
dates <- seq(as.Date("1922-01-01"), length = 20, by = "days")

# Create birthday a POSIXct date class object
dob <- as.POSIXct("1900-01-08")

# Create xts object work
work <- xts(x = hours, order.by = dates, born = dob)

str(work)
```

    ## An 'xts' object on 1922-01-01/1922-01-20 containing:
    ##   Data: num [1:20, 1] 8.25 8.11 8.24 8.04 9.44 ...
    ##   Indexed by objects of class: [Date] TZ: UTC
    ##   xts Attributes:  
    ## List of 1
    ##  $ born: POSIXct[1:1], format: "1900-01-08"

Similarly, any xts has object has 2 main components: \* The core matrix
part which contains the data \* The index part which contains the date
and time information.

Many a time, we might have to retrieve these components from the xts
object. This can be done by passing the xts object through `coredata`
and `index` functions as shown below:

``` r
# Extract the core data 
hours = coredata(work)
 
# View the class of core data
class(hours)
```

    ## [1] "matrix"

``` r
# Extract the index 
index = index(work)
 
# View the class of index
class(index)
```

    ## [1] "Date"

An important advantage of xts is its ability to incorporate date and
time information from any of the R classes used to represent time like
`Date`, `POSIXct` or some other class. This means that the order.by
argument takes data sequence in any of the common R time objects and
converts in into an internal representation.

### Subsetting

It is often required when working with time series data to filter
observations over a certain time range like a week, day or month. Rows
can be located in multiple ways - using ISO-8601 strings, other date
objects, logicals, or integers.

ISO-8601 strings are especially helpful for this task as they allow
complex ranges and intervals to be specified efficiently. ISO-8601 is an
internationally recognized and accepted way to not only represent time
and date but also ranges and repeating intervals. Let us look at some
examples on how to subset an xts object `x` using `ISO-8601`
strings:

| Function                          | ISSO String                          |
| --------------------------------- | ------------------------------------ |
| Year 2016                         | x\[“2016”\]                          |
| January 1, 2016 to March 22, 2016 | x\[“20160101/20160322”\]             |
| Up to and including January 2016  | x\[“/201601”\]                       |
| Jan 13th 2010                     | x\[“2010-01-13”\] or x\[“20130113”\] |

Recurring intraday intervals can also be specified using ISSO strings
followed by `T`. For example, `rainfall["T08:00/T10:00"]` gives rainfall
recorded between 8am and 10 am on all days.

We are also often interested in questions like profits over the last few
weeks/ months. The `last` and `first` functions are invaluable in
answering these questions.

``` r
last(work, n = '2 weeks')
first(work, n = '2 weeks')
```

n can be a numeric or a character string. Numeric indicates that the
first/ last n observations are selected. And negative number indicates
that all but first/last n observations are selected.

n can also be a character string of the format “n period.type”. secs,
seconds, mins, hours, days, weeks, months, quarters, and years are all
valid period.types.

These are especially useful for complex queries of the type: Extract the
first three days of the Bob’s second week at work.

``` r
first(last(first(work, "2 weeks"), "1 weeks"), "3 days")
```

    ##                [,1]
    ## 1922-01-02 8.109287
    ## 1922-01-03 8.242150
    ## 1922-01-04 8.041559

### Working with multiple time series

A lot of caution needs to exercised when performing operations on
multiple time series as each row in time series denotes observation at a
point in time. One of the time series can be missing data for a certain
time steps or frquency of the two time series can be different.

XTS overcomes this issue by first aligning the time series by
*intersection* of the indexes before performing any observation.

``` r
a <- work[1:5]
a
```

    ##                [,1]
    ## 1922-01-01 8.248757
    ## 1922-01-02 8.109287
    ## 1922-01-03 8.242150
    ## 1922-01-04 8.041559
    ## 1922-01-05 9.439858

``` r
b<- work[2]
b
```

    ##                [,1]
    ## 1922-01-02 8.109287

``` r
a+b
```

    ##                  e1
    ## 1922-01-02 16.21857

However, we might sometimes want to merge different time series while
retaining data points in both datasets rather than just the
intersection.

merge() allows arbitrary number of objects to be combined by specifying
1. The type of join that needs to be performed 2. The strategy to be
followed to fill in missing values.

To illustrate this, let us again add series a and b, but this time we
assume the missing observations in b to be 0.

``` r
df <- merge(a, b, join = "left", fill = 0)
df$a +df$b
```

    ##                    a
    ## 1922-01-01  8.248757
    ## 1922-01-02 16.218574
    ## 1922-01-03  8.242150
    ## 1922-01-04  8.041559
    ## 1922-01-05  9.439858

### Handling missing value

One of the common ways to impute missing values, especially in high
frequency time series data, is to replace the missing value with the
last observed value. This can be done using na.locf(). This is called
the *l*ast *o*bservation *c*arried *f*orward *a*pproach. We can aslo set
the missing values to the next most recent observed value by setting the
fromLast argument to TRUE.

``` r
work[3] <- NA
work[1:5]
```

    ##                [,1]
    ## 1922-01-01 8.248757
    ## 1922-01-02 8.109287
    ## 1922-01-03       NA
    ## 1922-01-04 8.041559
    ## 1922-01-05 9.439858

``` r
na.locf(work[1:5]) 
```

    ##                [,1]
    ## 1922-01-01 8.248757
    ## 1922-01-02 8.109287
    ## 1922-01-03 8.109287
    ## 1922-01-04 8.041559
    ## 1922-01-05 9.439858

``` r
na.locf(work[1:5], fromLast=TRUE) 
```

    ##                [,1]
    ## 1922-01-01 8.248757
    ## 1922-01-02 8.109287
    ## 1922-01-03 8.041559
    ## 1922-01-04 8.041559
    ## 1922-01-05 9.439858

Another staretgy that can be useful depending on the problem, is impute
the missing values based on simple linear (*in time\!*) interpolation
betwen points.This can be done using the `na.approx()` function.

### Other common operations

#### Lag

Another common before modelling or analyzing the properties of time
series is to look at lags of time series. This can be done using the
lag() function. A positive value of k will shift values k steps ahead in
time and negative value will shift it k steps back in time.

``` r
x <- work[5:10] 
cbind(lag = lag(x, k = 1) ,original=x,lead=lag(x, k = -1))  
```

    ##                 lag original     lead
    ## 1922-01-05       NA 9.439858 9.531269
    ## 1922-01-06 9.439858 9.531269 7.870201
    ## 1922-01-07 9.531269 7.870201 7.139898
    ## 1922-01-08 7.870201 7.139898 7.362017
    ## 1922-01-09 7.139898 7.362017 8.503289
    ## 1922-01-10 7.362017 8.503289       NA

Note that zoo follows opposite notation for k. Lag functions of zoo and
xts also differ in how they handle NAs introduced by lag. xts keeps them
whereas zoo drops them.

#### Differencing

Most timeseries in real world are non-stationary and need to be
differenced and made stationary before modelling.

For example, a second order 2 day differenced time series can be
obtained by:

``` r
diff(work, lag = 2, differences = 1)
```

### Conclusion

Through this tutorial, I introduced the key concepts and structure of
xts package. We have also looked at subsetting the data efficiently,
working with multiple time series without errors and common missing data
imputation strategies for time series. However, there are wide range of
useful tasks xts supports that we have not covered. Some of them are as
follows:

1.  Finding periodicity
2.  Finding end points
3.  Calculating rolling statistics
4.  Changing frequency of the time series

I have attached resources in the references section for you to dwell
deeper into any of these
functions.

### References

<https://cran.r-project.org/web/packages/xts/vignettes/xts.pdf>

<https://cran.r-project.org/web/packages/xts/xts.pdf>

<https://www.datacamp.com/courses/manipulating-time-series-data-in-r-with-xts-zoo>

<https://www.rstudio.com/wp-content/uploads/2015/02/rmarkdown-cheatsheet.pdf>
