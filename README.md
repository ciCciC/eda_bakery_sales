Exploratory Data Analysis: Bakery sales
================
Koray Poyraz
10/5/2021

loading libraries

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
    ## ✓ tibble  3.1.5     ✓ dplyr   1.0.7
    ## ✓ tidyr   1.1.4     ✓ stringr 1.4.0
    ## ✓ readr   2.0.2     ✓ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(lubridate)
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following objects are masked from 'package:base':
    ## 
    ##     date, intersect, setdiff, union

## Applying Peng’s checklist

load Bakery data as csv

``` r
df_bakery <- read_csv('data/Bakery.csv')
```

    ## Rows: 20507 Columns: 5

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (3): Items, Daypart, DayType
    ## dbl  (1): TransactionNo
    ## dttm (1): DateTime

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

view variable names

``` r
names(df_bakery)
```

    ## [1] "TransactionNo" "Items"         "DateTime"      "Daypart"      
    ## [5] "DayType"

check packaging

``` r
str(df_bakery)
```

    ## spec_tbl_df [20,507 × 5] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
    ##  $ TransactionNo: num [1:20507] 1 2 2 3 3 3 4 5 5 5 ...
    ##  $ Items        : chr [1:20507] "Bread" "Scandinavian" "Scandinavian" "Hot chocolate" ...
    ##  $ DateTime     : POSIXct[1:20507], format: "2016-10-30 09:58:11" "2016-10-30 10:05:34" ...
    ##  $ Daypart      : chr [1:20507] "Morning" "Morning" "Morning" "Morning" ...
    ##  $ DayType      : chr [1:20507] "Weekend" "Weekend" "Weekend" "Weekend" ...
    ##  - attr(*, "spec")=
    ##   .. cols(
    ##   ..   TransactionNo = col_double(),
    ##   ..   Items = col_character(),
    ##   ..   DateTime = col_datetime(format = ""),
    ##   ..   Daypart = col_character(),
    ##   ..   DayType = col_character()
    ##   .. )
    ##  - attr(*, "problems")=<externalptr>

top and bottom

``` r
head(df_bakery)
```

    ## # A tibble: 6 × 5
    ##   TransactionNo Items         DateTime            Daypart DayType
    ##           <dbl> <chr>         <dttm>              <chr>   <chr>  
    ## 1             1 Bread         2016-10-30 09:58:11 Morning Weekend
    ## 2             2 Scandinavian  2016-10-30 10:05:34 Morning Weekend
    ## 3             2 Scandinavian  2016-10-30 10:05:34 Morning Weekend
    ## 4             3 Hot chocolate 2016-10-30 10:07:57 Morning Weekend
    ## 5             3 Jam           2016-10-30 10:07:57 Morning Weekend
    ## 6             3 Cookies       2016-10-30 10:07:57 Morning Weekend

``` r
tail(df_bakery)
```

    ## # A tibble: 6 × 5
    ##   TransactionNo Items        DateTime            Daypart   DayType
    ##           <dbl> <chr>        <dttm>              <chr>     <chr>  
    ## 1          9682 Tacos/Fajita 2017-09-04 14:32:58 Afternoon Weekend
    ## 2          9682 Coffee       2017-09-04 14:32:58 Afternoon Weekend
    ## 3          9682 Tea          2017-09-04 14:32:58 Afternoon Weekend
    ## 4          9683 Coffee       2017-09-04 14:57:06 Afternoon Weekend
    ## 5          9683 Pastry       2017-09-04 14:57:06 Afternoon Weekend
    ## 6          9684 Smoothies    2017-09-04 15:04:24 Afternoon Weekend

check your n’s

``` r
# These N checks are done to get an understanding of the attributes count

# check N by Items
df_bakery %>% count(Items)
```

    ## # A tibble: 94 × 2
    ##    Items                        n
    ##    <chr>                    <int>
    ##  1 Adjustment                   1
    ##  2 Afternoon with the baker    44
    ##  3 Alfajores                  369
    ##  4 Argentina Night              7
    ##  5 Art Tray                    38
    ##  6 Bacon                        1
    ##  7 Baguette                   152
    ##  8 Bakewell                    48
    ##  9 Bare Popcorn                 5
    ## 10 Basket                       6
    ## # … with 84 more rows

``` r
# check N by Daypart, amount sales per Daypart
df_bakery %>% count(Daypart)
```

    ## # A tibble: 4 × 2
    ##   Daypart       n
    ##   <chr>     <int>
    ## 1 Afternoon 11569
    ## 2 Evening     520
    ## 3 Morning    8404
    ## 4 Night        14

``` r
# check N by DayType, amount sales per DayType
df_bakery %>% count(DayType)
```

    ## # A tibble: 2 × 2
    ##   DayType     n
    ##   <chr>   <int>
    ## 1 Weekday 12807
    ## 2 Weekend  7700

``` r
# check N by Date, amount sales per date
df_bakery %>% 
  mutate(Date=as.Date(DateTime)) %>% 
  count(Date)
```

    ## # A tibble: 159 × 2
    ##    Date           n
    ##    <date>     <int>
    ##  1 2016-01-11   150
    ##  2 2016-01-12    83
    ##  3 2016-02-11   164
    ##  4 2016-02-12   104
    ##  5 2016-03-11   189
    ##  6 2016-03-12   191
    ##  7 2016-04-11   187
    ##  8 2016-04-12   121
    ##  9 2016-05-11   275
    ## 10 2016-05-12   117
    ## # … with 149 more rows

check missing values

``` r
# check if any column has any missing value
amount_nas <- df_bakery %>% 
  select_if(function(x) any(is.na(x)))

str_c('There are: ', length(amount_nas), ' missing values')
```

    ## [1] "There are: 0 missing values"

simple plot to get a idea of the data

``` r
df_bakery %>% 
  mutate(Date=as.Date(DateTime)) %>% 
  ggplot(aes(Date)) +
  geom_freqpoly(aes(color=Daypart), bins=30) +
  theme_minimal() +
  scale_y_log10() +
  labs(
    title='Sales over time', 
    subtitle = 'levels in day part', 
    y='sales log10', 
    caption='Based on data from "The Bread Basket" \n 2016-2017')
```

![](assignment_3_files/figure-gfm/simple%20plot%20to%20get%20a%20idea%20of%20the%20data-1.png)<!-- -->

preparation for visualization

``` r
# The idea here is to get a insight in the sales during weekdays and hours over time

# a new tibble with 2 new variables, wday (week day) and hour
df_bakery_wday <- df_bakery %>% 
  arrange(DateTime) %>% 
  mutate(wday = wday(DateTime, label = TRUE, week_start=1, abbr=FALSE, locale='en_GB'), 
         hour = hour(DateTime))

# display tibble
df_bakery_wday
```

    ## # A tibble: 20,507 × 7
    ##    TransactionNo Items  DateTime            Daypart DayType wday    hour
    ##            <dbl> <chr>  <dttm>              <chr>   <chr>   <ord>  <int>
    ##  1           178 Coffee 2016-01-11 07:51:20 Morning Weekday Monday     7
    ##  2           178 Pastry 2016-01-11 07:51:20 Morning Weekday Monday     7
    ##  3           179 Coffee 2016-01-11 08:20:50 Morning Weekday Monday     8
    ##  4           179 Pastry 2016-01-11 08:20:50 Morning Weekday Monday     8
    ##  5           180 Jam    2016-01-11 08:22:28 Morning Weekday Monday     8
    ##  6           180 Tea    2016-01-11 08:22:28 Morning Weekday Monday     8
    ##  7           181 Tea    2016-01-11 08:45:20 Morning Weekday Monday     8
    ##  8           181 Tea    2016-01-11 08:45:20 Morning Weekday Monday     8
    ##  9           182 Pastry 2016-01-11 09:07:47 Morning Weekday Monday     9
    ## 10           182 Pastry 2016-01-11 09:07:47 Morning Weekday Monday     9
    ## # … with 20,497 more rows

analyzing the max and min in hour

``` r
# display the max and min tuples of hour to get a view for filtering
max_hour <- max(df_bakery_wday$hour)
min_hour <- min(df_bakery_wday$hour)

rbind(
  df_bakery_wday[which.max(df_bakery_wday$hour),], 
  df_bakery_wday[which.min(df_bakery_wday$hour),]
  )
```

    ## # A tibble: 2 × 7
    ##   TransactionNo Items            DateTime            Daypart DayType wday   hour
    ##           <dbl> <chr>            <dttm>              <chr>   <chr>   <ord> <int>
    ## 1          6588 Valentine's card 2017-02-14 23:29:03 Night   Weekday Tues…    23
    ## 2          4090 Bread            2017-01-01 01:21:05 Morning Weekend Sund…     1

``` r
# as the company switched to online shopping, people make also orders in the evening
df_bakery_wday %>% 
  filter(between(hour, 20, max_hour)) %>% 
  arrange(desc(hour))
```

    ## # A tibble: 36 × 7
    ##    TransactionNo Items         DateTime            Daypart DayType wday     hour
    ##            <dbl> <chr>         <dttm>              <chr>   <chr>   <ord>   <int>
    ##  1          6588 Valentine's … 2017-02-14 23:29:03 Night   Weekday Tuesday    23
    ##  2          6589 Valentine's … 2017-02-14 23:38:41 Night   Weekday Tuesday    23
    ##  3          9648 Vegan Feast   2017-08-04 23:20:54 Night   Weekend Friday     23
    ##  4          9448 Juice         2017-05-04 22:15:55 Night   Weekday Thursd…    22
    ##  5          9448 Mineral water 2017-05-04 22:15:55 Night   Weekday Thursd…    22
    ##  6          9638 Vegan Feast   2017-08-04 22:41:09 Night   Weekend Friday     22
    ##  7          9640 Vegan Feast   2017-08-04 22:43:06 Night   Weekend Friday     22
    ##  8          9641 Vegan Feast   2017-08-04 22:45:51 Night   Weekend Friday     22
    ##  9          9642 Scandinavian  2017-08-04 22:47:27 Night   Weekend Friday     22
    ## 10          9643 Vegan Feast   2017-08-04 22:48:52 Night   Weekend Friday     22
    ## # … with 26 more rows

``` r
# here we see a strange online order at 1:21 am, because I dont have much info on opening hours and delivery I can not say much but it is somehow funny to order a bread at 1:21 am
# I have chosen not to visualize 1 record (online order at 1:21:05 am), which is reported as the 1st row in the tibble below. The reason is to have a more fine visualized plot and because this concerns 1 record, it is fine to have it reported.
df_bakery_wday %>% 
  filter(between(hour, min_hour, 10)) %>% 
  arrange(hour)
```

    ## # A tibble: 5,302 × 7
    ##    TransactionNo Items     DateTime            Daypart DayType wday     hour
    ##            <dbl> <chr>     <dttm>              <chr>   <chr>   <ord>   <int>
    ##  1          4090 Bread     2017-01-01 01:21:05 Morning Weekend Sunday      1
    ##  2           178 Coffee    2016-01-11 07:51:20 Morning Weekday Monday      7
    ##  3           178 Pastry    2016-01-11 07:51:20 Morning Weekday Monday      7
    ##  4           340 Coffee    2016-03-11 07:46:50 Morning Weekday Friday      7
    ##  5           340 Bread     2016-03-11 07:46:50 Morning Weekday Friday      7
    ##  6           341 Medialuna 2016-03-11 07:50:50 Morning Weekday Friday      7
    ##  7           341 Medialuna 2016-03-11 07:50:50 Morning Weekday Friday      7
    ##  8           437 Medialuna 2016-04-11 07:32:33 Morning Weekday Monday      7
    ##  9          1275 Coffee    2016-11-14 07:39:27 Morning Weekday Monday      7
    ## 10          1342 Coffee    2016-11-15 07:42:27 Morning Weekday Tuesday     7
    ## # … with 5,292 more rows

# Question: what are the hours of the week days that deliver high sales?

## Visualization:

-   geom = bars/ rectangle
-   mapping = x maps hour, y maps count of occurrences, fill in Daypart
    (level)
-   position = dodge, bin width = 1 (equal to 1 hour) and border color =
    white
-   facet = wday (week day)
-   theme = minimal
-   scale = continuous, scale_y\_sqrt (Straightening)

``` r
df_bakery_wday %>% 
  filter(hour > 1) %>% 
  ggplot(aes(hour, fill=Daypart), position='dodge') +
  geom_histogram(binwidth = 1, color='white') +
  facet_wrap(vars(wday)) +
  theme_minimal() +
  scale_y_sqrt() +
  labs(
    title='Bakery Sales', 
    subtitle = 'per week day in hours',
    x='hour', 
    y='', 
    caption='Based on data from "The Bread Basket" \n 2016-2017')
```

![](assignment_3_files/figure-gfm/visualization-1.png)<!-- -->
