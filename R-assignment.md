LFSC\_521\_R assignment
================
Yudan Shi
2022/2/6

## Introduction

-   I’ll use data from the **nycflights13** package, which is the
    dataset on flights departing New York City in 2013.

-   Analyze the data using the **tidyverse** package, involving
    *split-apply-combine approach*, and using **ggplot2** to visualize
    the data.

-   Since flight delays usually have a chain reaction, i.e., even if the
    initial problem that caused the delay has been resolved, the already
    delayed preceding flight could cause the following flight to be
    delayed as well.

-   Therefore, I will analyze the relationship between the delay time of
    previous fights and of the subsequent flights.

## Install and Library

``` r
# install.packages("tidyverse")
# install.packages("nycflights13")

library(tidyverse)
library(nycflights13) # Import the dataset
library(ggplot2)
```

## Analysis and Results

Firstly, I’ll calculate the departure delay of the preceding flight from
the same airport using `lag()` function based on the same *origin*.

``` r
(lagged_delays <- flights %>% 
  arrange(origin, month, day, dep_time) %>%
  group_by(origin) %>%
  mutate(dep_delay_lag = lag(dep_delay)) %>%
  filter(!is.na(dep_delay),!is.na(dep_delay_lag))) ## remove rows including NA values
```

    ## # A tibble: 327,649 x 20
    ## # Groups:   origin [3]
    ##     year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
    ##    <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>
    ##  1  2013     1     1      554            558        -4      740            728
    ##  2  2013     1     1      555            600        -5      913            854
    ##  3  2013     1     1      558            600        -2      923            937
    ##  4  2013     1     1      559            600        -1      854            902
    ##  5  2013     1     1      601            600         1      844            850
    ##  6  2013     1     1      606            610        -4      858            910
    ##  7  2013     1     1      607            607         0      858            915
    ##  8  2013     1     1      608            600         8      807            735
    ##  9  2013     1     1      615            615         0      833            842
    ## 10  2013     1     1      622            630        -8     1017           1014
    ## # ... with 327,639 more rows, and 12 more variables: arr_delay <dbl>,
    ## #   carrier <chr>, flight <int>, tailnum <chr>, origin <chr>, dest <chr>,
    ## #   air_time <dbl>, distance <dbl>, hour <dbl>, minute <dbl>, time_hour <dttm>,
    ## #   dep_delay_lag <dbl>

Then, grouped by delay time of the preceding flight, I’ll calculate the
average delay time of the subsequent flights.

And then make a plot to visualize the relationship.

``` r
lagged_delays %>%
  group_by(dep_delay_lag) %>%
  summarise(dep_delay_mean = mean(dep_delay)) %>%
  ggplot(aes(y = dep_delay_mean, x = dep_delay_lag)) +
  geom_point() +
  scale_x_continuous(breaks = seq(0, 1500, by = 120)) +
  labs(y = "Departure Delay (min)", x = "Previous Departure Delay (min)") + 
  theme_classic()
```

![unnamed-chunk-2-1](https://user-images.githubusercontent.com/99147879/152708023-322aab7a-03c8-4347-9942-bd0f6a860b3f.png)

## Discussion

The plot shows the relationship between the mean delay of a flight and
delay of the preceding flight.

For delays less than two hours, the relationship between the delay of
the preceding flight and the current flight is nearly a line.

After that the relationship becomes more variable, as long-delayed
flights are interspersed with flights leaving on-time.

After about 8-hours, a delayed flight is likely to be followed by a
flight leaving on time.
