Processing the bike-share data
================
Anne Wilson
2024-02-29

## Loading required packages

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.4.4     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
bike_share_2023_cleaned <- read_rds("bike_share_2023_cleaned.RDS")
```

## Transforming the data

First some calculated columns are added to the data. These give the ride
length, day of week the ride started and the month the ride started. The
ride lengths are rounded to the nearest minute.

``` r
bike_share_2023_processed <-  bike_share_2023_cleaned %>% 
  add_column(ride_length = round(difftime(bike_share_2023_cleaned$ended_at, bike_share_2023_cleaned$started_at, units = "mins"))) %>% 
  add_column(day_of_week = wday(bike_share_2023_cleaned$started_at)) %>% 
  add_column(month = month(bike_share_2023_cleaned$started_at))
```

Look at the data.

``` r
glimpse(bike_share_2023_processed)
```

    ## Rows: 4,331,628
    ## Columns: 16
    ## $ ride_id            <chr> "D8EEE72183269F07", "E5AD797A579842F8", "8FBD2AD70B…
    ## $ rideable_type      <chr> "classic_bike", "electric_bike", "classic_bike", "e…
    ## $ started_at         <dttm> 2023-01-01 00:02:06, 2023-01-01 00:03:26, 2023-01-…
    ## $ ended_at           <dttm> 2023-01-01 00:29:46, 2023-01-01 00:07:23, 2023-01-…
    ## $ start_station_name <chr> "Fairbanks Ct & Grand Ave", "Sheridan Rd & Loyola A…
    ## $ start_station_id   <chr> "TA1305000003", "RP-009", "TA1309000015", "KA150300…
    ## $ end_station_name   <chr> "New St & Illinois St", "Sheridan Rd & Loyola Ave",…
    ## $ end_station_id     <chr> "TA1306000013", "RP-009", "13108", "KA1503000022", …
    ## $ start_lat          <dbl> 41.89185, 42.00114, 41.96889, 41.96154, 41.88462, 4…
    ## $ start_lng          <dbl> -87.62058, -87.66126, -87.68400, -87.66619, -87.627…
    ## $ end_lat            <dbl> 41.89085, 42.00104, 41.97382, 41.96159, 41.86789, 4…
    ## $ end_lng            <dbl> -87.61862, -87.66120, -87.65966, -87.66604, -87.623…
    ## $ member_casual      <chr> "member", "casual", "casual", "member", "member", "…
    ## $ ride_length        <drtn> 28 mins, 4 mins, 10 mins, 12 mins, 27 mins, 16 min…
    ## $ day_of_week        <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
    ## $ month              <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …

Now a summary of the ride lengths is generated to look at the mean,
median, minimum and maximum.

``` r
bike_share_2023_processed %>% 
  summarise(mean_ride_length = mean(ride_length), median_ride_length = median(ride_length), min_ride_length = min(ride_length), max_ride_length = max(ride_length))
```

    ## # A tibble: 1 × 4
    ##   mean_ride_length median_ride_length min_ride_length max_ride_length
    ##   <drtn>           <drtn>             <drtn>          <drtn>         
    ## 1 15.95108 mins    10 mins            0 mins          12136 mins

There is at least one ride of length zero and there is a ride of length
12136 minutes, that is over 8 days - this might be an outlier and may
cause problems with the analysis. Rides of less than one minute are
unlikely to be genuine rides. Filter to look at the rides that are less
than one minute.

``` r
bike_share_2023_processed %>% 
  filter(ride_length < 1)
```

    ## # A tibble: 55,896 × 16
    ##    ride_id          rideable_type started_at          ended_at           
    ##    <chr>            <chr>         <dttm>              <dttm>             
    ##  1 C56965D2AE5D234E electric_bike 2023-01-01 00:22:03 2023-01-01 00:22:22
    ##  2 BAC999B11574C4FE classic_bike  2023-01-01 00:31:04 2023-01-01 00:31:25
    ##  3 B429DA4976E3859C classic_bike  2023-01-01 00:47:14 2023-01-01 00:47:24
    ##  4 4259FDA946A26352 classic_bike  2023-01-01 00:56:24 2023-01-01 00:56:38
    ##  5 D3A3A18D24281BAC electric_bike 2023-01-01 01:18:59 2023-01-01 01:19:19
    ##  6 4F28A430382131D6 classic_bike  2023-01-01 01:21:01 2023-01-01 01:21:03
    ##  7 9C7651EE0C424805 electric_bike 2023-01-01 01:40:42 2023-01-01 01:41:00
    ##  8 AB2F052E4E79EE00 electric_bike 2023-01-01 01:49:36 2023-01-01 01:50:03
    ##  9 C05839E0454A98CB electric_bike 2023-01-01 01:51:56 2023-01-01 01:52:15
    ## 10 B4321D71C1489037 electric_bike 2023-01-01 01:55:32 2023-01-01 01:55:49
    ## # ℹ 55,886 more rows
    ## # ℹ 12 more variables: start_station_name <chr>, start_station_id <chr>,
    ## #   end_station_name <chr>, end_station_id <chr>, start_lat <dbl>,
    ## #   start_lng <dbl>, end_lat <dbl>, end_lng <dbl>, member_casual <chr>,
    ## #   ride_length <drtn>, day_of_week <dbl>, month <dbl>

There are 55907 of these rows with ride length less than one minute.
These are deleted to avoid them skewing the final analysis.

``` r
bike_share_2023_processed <- bike_share_2023_processed %>% 
  filter(ride_length > 1)
```

Next the data is filtered to see how many rides there are that are more
than 24 hours (1440 minutes).

``` r
bike_share_2023_processed %>% 
  arrange(-ride_length) %>% 
  filter(ride_length > 1440)
```

    ## # A tibble: 133 × 16
    ##    ride_id          rideable_type started_at          ended_at           
    ##    <chr>            <chr>         <dttm>              <dttm>             
    ##  1 59AD7EE868FC6588 classic_bike  2023-05-30 12:48:08 2023-06-07 23:04:26
    ##  2 FA287922CA358CE0 classic_bike  2023-06-03 17:52:15 2023-06-11 11:44:31
    ##  3 47158A16C754A9F4 classic_bike  2023-08-10 22:17:49 2023-08-15 17:09:02
    ##  4 4031082BC503CC84 classic_bike  2023-08-02 17:28:57 2023-08-06 11:07:57
    ##  5 3BC5FFFDF7503DAA classic_bike  2023-06-15 13:28:59 2023-06-18 23:12:06
    ##  6 6786F74C5A6183FB classic_bike  2023-06-18 19:21:22 2023-06-21 10:59:46
    ##  7 D2273A0F45CDD4CC classic_bike  2023-08-18 09:13:48 2023-08-20 16:14:38
    ##  8 A795B5420E15A65B classic_bike  2023-05-10 18:42:11 2023-05-13 00:47:58
    ##  9 280CB8109510E280 classic_bike  2023-07-06 21:34:55 2023-07-08 14:32:46
    ## 10 D88D5192DF6A4536 classic_bike  2023-04-13 17:46:11 2023-04-15 08:55:32
    ## # ℹ 123 more rows
    ## # ℹ 12 more variables: start_station_name <chr>, start_station_id <chr>,
    ## #   end_station_name <chr>, end_station_id <chr>, start_lat <dbl>,
    ## #   start_lng <dbl>, end_lat <dbl>, end_lng <dbl>, member_casual <chr>,
    ## #   ride_length <drtn>, day_of_week <dbl>, month <dbl>

While there are not many rides longer than 24 hours, there are over 100
of them and they are likely to be real data, so these will be kept but
they may need to be handled separately for some of the analysis.

Now the data is ready to be analysed, so the processed data in the data
frame bike_share_2023_processed is saved to RDS and CSV files.

``` r
write_rds(bike_share_2023_processed, file = "bike_share_2023_processed.RDS")
write_csv(bike_share_2023_processed, "bike_share_2023_processed.csv")
```
