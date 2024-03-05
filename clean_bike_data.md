Cleaning the bike-share data
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
    ## 
    ## Attaching package: 'janitor'
    ## 
    ## 
    ## The following objects are masked from 'package:stats':
    ## 
    ##     chisq.test, fisher.test

## Reading in the data

``` r
bike_share_2023 <- read_rds("bike_share_2023_raw.RDS")
```

Check which columns have missing data and how many rows are affected.

``` r
for (i in 1:13) {
  print(paste("Column ", i, ": ", sum(is.na(bike_share_2023[,i]))))
}
```

    ## [1] "Column  1 :  0"
    ## [1] "Column  2 :  0"
    ## [1] "Column  3 :  0"
    ## [1] "Column  4 :  0"
    ## [1] "Column  5 :  875716"
    ## [1] "Column  6 :  875848"
    ## [1] "Column  7 :  929202"
    ## [1] "Column  8 :  929343"
    ## [1] "Column  9 :  0"
    ## [1] "Column  10 :  0"
    ## [1] "Column  11 :  6990"
    ## [1] "Column  12 :  6990"
    ## [1] "Column  13 :  0"

Remove all rows with na anywhere since the data in those rows is
probably not reliable. Save the data to a new data frame -
bike_share_2023_cleaned.

``` r
bike_share_2023_cleaned <- bike_share_2023 %>% 
  drop_na()
```

Check that all the na values have been removed and look at the data
frame.

``` r
for (i in 1:13) {
  print(paste("Column ", i, ": ", sum(is.na(bike_share_2023_cleaned[,i]))))
}
```

    ## [1] "Column  1 :  0"
    ## [1] "Column  2 :  0"
    ## [1] "Column  3 :  0"
    ## [1] "Column  4 :  0"
    ## [1] "Column  5 :  0"
    ## [1] "Column  6 :  0"
    ## [1] "Column  7 :  0"
    ## [1] "Column  8 :  0"
    ## [1] "Column  9 :  0"
    ## [1] "Column  10 :  0"
    ## [1] "Column  11 :  0"
    ## [1] "Column  12 :  0"
    ## [1] "Column  13 :  0"

``` r
glimpse(bike_share_2023_cleaned)
```

    ## Rows: 4,331,707
    ## Columns: 13
    ## $ ride_id            <chr> "F96D5A74A3E41399", "13CB7EB698CEDB88", "BD88A2E670…
    ## $ rideable_type      <chr> "electric_bike", "classic_bike", "electric_bike", "…
    ## $ started_at         <dttm> 2023-01-21 20:05:42, 2023-01-10 15:37:36, 2023-01-…
    ## $ ended_at           <dttm> 2023-01-21 20:16:33, 2023-01-10 15:46:05, 2023-01-…
    ## $ start_station_name <chr> "Lincoln Ave & Fullerton Ave", "Kimbark Ave & 53rd …
    ## $ start_station_id   <chr> "TA1309000058", "TA1309000037", "RP-005", "TA130900…
    ## $ end_station_name   <chr> "Hampden Ct & Diversey Ave", "Greenwood Ave & 47th …
    ## $ end_station_id     <chr> "202480.0", "TA1308000002", "599", "TA1308000002", …
    ## $ start_lat          <dbl> 41.92407, 41.79957, 42.00857, 41.79957, 41.79957, 4…
    ## $ start_lng          <dbl> -87.64628, -87.59475, -87.69048, -87.59475, -87.594…
    ## $ end_lat            <dbl> 41.93000, 41.80983, 42.03974, 41.80983, 41.80983, 4…
    ## $ end_lng            <dbl> -87.64000, -87.59938, -87.69941, -87.59938, -87.599…
    ## $ member_casual      <chr> "member", "member", "casual", "member", "member", "…

There are 4,331,707 rows left out of the original 5,719,877 so about one
quarter of the original data has now been deleted.

Next, the data is checked to make sure there are no duplicates. This is
done by checking the ride_id column, which should be unique.

``` r
bike_share_2023_cleaned %>% 
  get_dupes(ride_id)
```

    ## No duplicate combinations found of: ride_id

    ## # A tibble: 0 × 14
    ## # ℹ 14 variables: ride_id <chr>, dupe_count <int>, rideable_type <chr>,
    ## #   started_at <dttm>, ended_at <dttm>, start_station_name <chr>,
    ## #   start_station_id <chr>, end_station_name <chr>, end_station_id <chr>,
    ## #   start_lat <dbl>, start_lng <dbl>, end_lat <dbl>, end_lng <dbl>,
    ## #   member_casual <chr>

There are no duplicates. Next the data is sorted by the started_at time.

``` r
bike_share_2023_cleaned <- bike_share_2023_cleaned %>%
  arrange(started_at)
head(bike_share_2023_cleaned)
```

    ## # A tibble: 6 × 13
    ##   ride_id          rideable_type started_at          ended_at           
    ##   <chr>            <chr>         <dttm>              <dttm>             
    ## 1 D8EEE72183269F07 classic_bike  2023-01-01 00:02:06 2023-01-01 00:29:46
    ## 2 E5AD797A579842F8 electric_bike 2023-01-01 00:03:26 2023-01-01 00:07:23
    ## 3 8FBD2AD70B0F6A6F classic_bike  2023-01-01 00:04:07 2023-01-01 00:13:56
    ## 4 B05BD052B9EBB767 electric_bike 2023-01-01 00:04:27 2023-01-01 00:16:52
    ## 5 F9EA7B9E6C243CFC classic_bike  2023-01-01 00:04:54 2023-01-01 00:31:52
    ## 6 27C2A67184C49D01 electric_bike 2023-01-01 00:05:43 2023-01-01 00:21:37
    ## # ℹ 9 more variables: start_station_name <chr>, start_station_id <chr>,
    ## #   end_station_name <chr>, end_station_id <chr>, start_lat <dbl>,
    ## #   start_lng <dbl>, end_lat <dbl>, end_lng <dbl>, member_casual <chr>

The values in the member_casual column should be ‘member’ or ‘casual’.
The data is checked to make sure there are no other values.

``` r
bike_share_2023_cleaned %>% 
  filter((member_casual != "member") & (member_casual != "casual"))
```

    ## # A tibble: 0 × 13
    ## # ℹ 13 variables: ride_id <chr>, rideable_type <chr>, started_at <dttm>,
    ## #   ended_at <dttm>, start_station_name <chr>, start_station_id <chr>,
    ## #   end_station_name <chr>, end_station_id <chr>, start_lat <dbl>,
    ## #   start_lng <dbl>, end_lat <dbl>, end_lng <dbl>, member_casual <chr>

The member-casual column is good. Now the values in the rideable_type
column are checked - these should be ‘classic_bike’ or ‘electric_bike’.
Inspect the rideable_type to see what values this takes, and how many
occurrences of each.

``` r
bike_share_2023_cleaned %>% 
  group_by(rideable_type) %>% 
  summarise(count = n())
```

    ## # A tibble: 3 × 2
    ##   rideable_type   count
    ##   <chr>           <int>
    ## 1 classic_bike  2690744
    ## 2 docked_bike     76124
    ## 3 electric_bike 1564839

This shows there are some bikes listed as ‘docked_bikes’. Some internet
research shows that classic bikes were called docked bikes before
electric bikes were introduced to the scheme in July 2020 Since the
docked bikes in this data are classic bikes that have not been
re-labelled, ‘docked_bike’ can now be replaced with ‘classic_bike’.

``` r
bike_share_2023_cleaned <-  bike_share_2023_cleaned %>% 
  mutate(rideable_type = ifelse(rideable_type == "docked_bike", "classic_bike", rideable_type))
```

Check that there are now only ‘classic_bike’ and ‘electric_bike’ in the
rideable_type column.

``` r
bike_share_2023_cleaned %>% 
  group_by(rideable_type) %>% 
  summarise(count = n())
```

    ## # A tibble: 2 × 2
    ##   rideable_type   count
    ##   <chr>           <int>
    ## 1 classic_bike  2766868
    ## 2 electric_bike 1564839

Look at a summary of the data to check for potentially problematic
values.

``` r
summary(bike_share_2023_cleaned)
```

    ##    ride_id          rideable_type        started_at                    
    ##  Length:4331707     Length:4331707     Min.   :2023-01-01 00:02:06.00  
    ##  Class :character   Class :character   1st Qu.:2023-05-20 13:02:18.00  
    ##  Mode  :character   Mode  :character   Median :2023-07-20 15:12:22.00  
    ##                                        Mean   :2023-07-15 19:09:13.49  
    ##                                        3rd Qu.:2023-09-16 16:19:20.50  
    ##                                        Max.   :2023-12-31 23:58:55.00  
    ##     ended_at                     start_station_name start_station_id  
    ##  Min.   :2023-01-01 00:07:23.0   Length:4331707     Length:4331707    
    ##  1st Qu.:2023-05-20 13:23:20.5   Class :character   Class :character  
    ##  Median :2023-07-20 15:29:43.0   Mode  :character   Mode  :character  
    ##  Mean   :2023-07-15 19:25:10.5                                        
    ##  3rd Qu.:2023-09-16 16:39:39.0                                        
    ##  Max.   :2024-01-01 14:20:23.0                                        
    ##  end_station_name   end_station_id       start_lat       start_lng     
    ##  Length:4331707     Length:4331707     Min.   :41.65   Min.   :-87.84  
    ##  Class :character   Class :character   1st Qu.:41.88   1st Qu.:-87.66  
    ##  Mode  :character   Mode  :character   Median :41.90   Median :-87.64  
    ##                                        Mean   :41.90   Mean   :-87.64  
    ##                                        3rd Qu.:41.93   3rd Qu.:-87.63  
    ##                                        Max.   :42.06   Max.   :-87.53  
    ##     end_lat         end_lng       member_casual     
    ##  Min.   : 0.00   Min.   :-87.84   Length:4331707    
    ##  1st Qu.:41.88   1st Qu.:-87.66   Class :character  
    ##  Median :41.90   Median :-87.64   Mode  :character  
    ##  Mean   :41.90   Mean   :-87.64                     
    ##  3rd Qu.:41.93   3rd Qu.:-87.63                     
    ##  Max.   :42.06   Max.   :  0.00

This summary shows that there is a minimum of 0.00 for the end
latititude (end_lat column) and a maximum of 0.00 for the end longitude
(end_lng). These values are obviously wrong so need investigating. First
the data is sorted by the end_lat column.

``` r
bike_share_2023_cleaned %>% 
  arrange(end_lat)
```

    ## # A tibble: 4,331,707 × 13
    ##    ride_id          rideable_type started_at          ended_at           
    ##    <chr>            <chr>         <dttm>              <dttm>             
    ##  1 ADFF57D27B5BF9D2 classic_bike  2023-06-15 09:38:07 2023-06-15 09:42:57
    ##  2 873D50153BBC0686 electric_bike 2023-06-15 12:38:05 2023-06-15 12:38:41
    ##  3 43107577DF9B498D classic_bike  2023-08-21 18:43:22 2023-08-21 22:05:55
    ##  4 43E7A9708C3CBBDA electric_bike 2023-01-11 15:23:39 2023-01-11 15:33:27
    ##  5 041B44FC0477EA71 classic_bike  2023-01-18 15:38:26 2023-01-18 15:43:20
    ##  6 48B846D268078D39 electric_bike 2023-01-20 15:21:36 2023-01-20 15:26:02
    ##  7 0AED4FDA2D51C727 electric_bike 2023-01-25 15:12:36 2023-01-25 15:18:14
    ##  8 55ADC86FBDED24A6 electric_bike 2023-02-01 15:12:58 2023-02-01 15:17:51
    ##  9 9868ABFA89FBEAA2 classic_bike  2023-02-02 15:14:00 2023-02-02 15:20:11
    ## 10 58B3B6898CD49146 classic_bike  2023-02-06 07:07:08 2023-02-06 07:13:25
    ## # ℹ 4,331,697 more rows
    ## # ℹ 9 more variables: start_station_name <chr>, start_station_id <chr>,
    ## #   end_station_name <chr>, end_station_id <chr>, start_lat <dbl>,
    ## #   start_lng <dbl>, end_lat <dbl>, end_lng <dbl>, member_casual <chr>

There are three rows with end_lat and end_lng equal to zero. The first
two seem to be some sort of test, with the ride time at less than five
minutes and a test end station. These rows can be removed. The third row
has a valid end_station_name and end_station_id (653B) so it should be
possible to replace the end_lat and end_lng with the correct values.
Filter the data to find the correct latitude and longitude for station
653B.

``` r
bike_share_2023_cleaned %>% 
  filter(end_station_id == "653B")
```

    ## # A tibble: 296 × 13
    ##    ride_id          rideable_type started_at          ended_at           
    ##    <chr>            <chr>         <dttm>              <dttm>             
    ##  1 7B974C5BD11611D6 electric_bike 2023-03-02 16:23:09 2023-03-02 16:31:35
    ##  2 DE1A793BD09DDD0C electric_bike 2023-03-04 16:03:08 2023-03-04 16:11:20
    ##  3 5A4270332896C439 classic_bike  2023-03-05 18:46:14 2023-03-05 19:18:07
    ##  4 D0B96B32A34EC340 classic_bike  2023-03-13 19:35:45 2023-03-13 19:49:12
    ##  5 9FE2BD2EF5871C1C classic_bike  2023-03-16 08:06:23 2023-03-16 08:15:15
    ##  6 1E8EC92FB7FB4F8C classic_bike  2023-03-21 11:35:36 2023-03-21 11:42:30
    ##  7 99B1617F0EE74185 classic_bike  2023-03-28 11:32:50 2023-03-28 11:38:59
    ##  8 D308A0A11EEE19BB classic_bike  2023-03-29 11:02:50 2023-03-29 11:18:28
    ##  9 4608A431D7F79D97 classic_bike  2023-03-30 17:16:50 2023-03-30 17:30:25
    ## 10 18AA4CF5C227304F classic_bike  2023-04-03 17:13:19 2023-04-03 17:25:13
    ## # ℹ 286 more rows
    ## # ℹ 9 more variables: start_station_name <chr>, start_station_id <chr>,
    ## #   end_station_name <chr>, end_station_id <chr>, start_lat <dbl>,
    ## #   start_lng <dbl>, end_lat <dbl>, end_lng <dbl>, member_casual <chr>

The end_lat for this station appears to be 41.78000 and the end_lng
-87.59000. However we need to check that the end_lat and and end_lng are
the same in all rows for this station (excluding the single row for
which they are zero) - to do this we look at the maximum and minimum
values.

``` r
df <- bike_share_2023_cleaned %>% 
  filter((end_station_id == "653B") & (end_lat != 0)) %>% 
  summarise(max_end_lat = max(end_lat), min_end_lat = min(end_lat),
            max_end_lng = max(end_lng), min_end_lng = min(end_lng))
df
```

    ## # A tibble: 1 × 4
    ##   max_end_lat min_end_lat max_end_lng min_end_lng
    ##         <dbl>       <dbl>       <dbl>       <dbl>
    ## 1        41.8        41.8       -87.6       -87.6

A quick inspection of the data shows that two values are used for
end_lat and end_long, but this is just due to rounding. The zero values
for ride_id 43107577DF9B498D can be set to either of these values but we
use the less rounded values, given here as max_end_lat and max_end_lng.

``` r
bike_share_2023_cleaned$end_lat[bike_share_2023_cleaned$ride_id == "43107577DF9B498D"] <- df$max_end_lat[1]
bike_share_2023_cleaned$end_lng[bike_share_2023_cleaned$ride_id == "43107577DF9B498D"] <- df$max_end_lng[1]
```

The two remaining rows with zero for end_lat and end_lng are now
removed.

``` r
bike_share_2023_cleaned <- bike_share_2023_cleaned %>% 
  filter(end_lat != 0)
```

Next we need to check if the test station appears elsewhere in the data.

``` r
bike_share_2023_cleaned %>% 
  filter(start_station_id == "OH Charging Stx - Test" | end_station_id == "OH Charging Stx - Test")
```

    ## # A tibble: 12 × 13
    ##    ride_id          rideable_type started_at          ended_at           
    ##    <chr>            <chr>         <dttm>              <dttm>             
    ##  1 3AA20CC3FE43F678 classic_bike  2023-06-28 10:56:35 2023-06-28 10:56:40
    ##  2 12A36ED2AAE587FD electric_bike 2023-06-28 15:32:11 2023-06-28 15:32:27
    ##  3 1EC494994DFD4553 electric_bike 2023-06-28 15:32:50 2023-06-28 15:33:07
    ##  4 0D77713ADEE7A4ED electric_bike 2023-06-28 15:34:27 2023-06-28 15:34:33
    ##  5 B6A90F07CCAEEB50 electric_bike 2023-06-28 15:38:05 2023-06-28 15:38:13
    ##  6 103567010777D572 classic_bike  2023-06-28 15:43:40 2023-06-28 15:43:44
    ##  7 E3AC9546FB4F0BEB classic_bike  2023-06-28 15:44:00 2023-06-28 15:44:06
    ##  8 A34B8C56E7692CB5 electric_bike 2023-06-29 14:29:06 2023-06-29 14:29:13
    ##  9 89D9BB1625C66CE3 classic_bike  2023-06-29 14:36:06 2023-06-29 14:36:13
    ## 10 17E7201CD21C0085 classic_bike  2023-06-29 14:36:38 2023-06-29 14:36:51
    ## 11 465EA70E1D719562 electric_bike 2023-06-29 14:41:13 2023-06-29 14:41:19
    ## 12 21F3D75A63DC5D8B electric_bike 2023-06-29 14:41:41 2023-06-29 14:41:45
    ## # ℹ 9 more variables: start_station_name <chr>, start_station_id <chr>,
    ## #   end_station_name <chr>, end_station_id <chr>, start_lat <dbl>,
    ## #   start_lng <dbl>, end_lat <dbl>, end_lng <dbl>, member_casual <chr>

There are 12 rows with this test station and all the rides are less than
a minute long, so these rows can be deleted.

``` r
bike_share_2023_cleaned <-  bike_share_2023_cleaned %>% 
  filter(start_station_id != "OH Charging Stx - Test" | end_station_id != "OH Charging Stx - Test")
```

Next the times are checked to make sure that the ended_at times are all
later than the corresponding started_at times. This should only be
possible on 2023-11-05 when the daylight saving time ended and the
clocks went back.

``` r
bike_share_2023_cleaned %>% 
  filter(ended_at < started_at)
```

    ## # A tibble: 66 × 13
    ##    ride_id          rideable_type started_at          ended_at           
    ##    <chr>            <chr>         <dttm>              <dttm>             
    ##  1 7A4D237E2C99D424 electric_bike 2023-04-04 17:15:08 2023-04-04 17:15:05
    ##  2 81E1C5175FA5A23D classic_bike  2023-04-19 14:47:18 2023-04-19 14:47:14
    ##  3 0063C3704F56EC55 electric_bike 2023-04-27 07:51:14 2023-04-27 07:51:09
    ##  4 00AC4040E25E347E classic_bike  2023-05-07 15:54:58 2023-05-07 15:54:47
    ##  5 97BF63D06721A3B9 classic_bike  2023-05-13 18:08:15 2023-05-13 18:08:09
    ##  6 579596DD4C7C7538 classic_bike  2023-05-23 17:39:38 2023-05-23 17:39:35
    ##  7 A769AB597DEA18C4 classic_bike  2023-05-27 05:31:51 2023-05-27 05:31:37
    ##  8 934174DB8E2AD791 classic_bike  2023-05-29 17:34:21 2023-05-29 17:34:09
    ##  9 FAC4E90497237BD1 classic_bike  2023-06-01 16:47:50 2023-06-01 16:47:49
    ## 10 0F2C8AC039F63D8F electric_bike 2023-06-08 16:31:42 2023-06-08 16:31:38
    ## # ℹ 56 more rows
    ## # ℹ 9 more variables: start_station_name <chr>, start_station_id <chr>,
    ## #   end_station_name <chr>, end_station_id <chr>, start_lat <dbl>,
    ## #   start_lng <dbl>, end_lat <dbl>, end_lng <dbl>, member_casual <chr>

This shows that 66 rows have ended_at times earlier than the started_at
times. Most, but not all, of these are related to the clocks changing on
2023-11-05. Since these are only a few rows out of a large data set,
they can be deleted.

``` r
bike_share_2023_cleaned <- bike_share_2023_cleaned %>% 
  filter(ended_at >= started_at)
```

Now look at the summary again.

``` r
summary(bike_share_2023_cleaned)
```

    ##    ride_id          rideable_type        started_at                    
    ##  Length:4331628     Length:4331628     Min.   :2023-01-01 00:02:06.00  
    ##  Class :character   Class :character   1st Qu.:2023-05-20 13:01:52.50  
    ##  Mode  :character   Mode  :character   Median :2023-07-20 15:11:52.00  
    ##                                        Mean   :2023-07-15 19:07:48.04  
    ##                                        3rd Qu.:2023-09-16 16:18:19.25  
    ##                                        Max.   :2023-12-31 23:58:55.00  
    ##     ended_at                      start_station_name start_station_id  
    ##  Min.   :2023-01-01 00:07:23.00   Length:4331628     Length:4331628    
    ##  1st Qu.:2023-05-20 13:22:26.00   Class :character   Class :character  
    ##  Median :2023-07-20 15:29:12.00   Mode  :character   Mode  :character  
    ##  Mean   :2023-07-15 19:23:45.19                                        
    ##  3rd Qu.:2023-09-16 16:38:27.50                                        
    ##  Max.   :2024-01-01 14:20:23.00                                        
    ##  end_station_name   end_station_id       start_lat       start_lng     
    ##  Length:4331628     Length:4331628     Min.   :41.65   Min.   :-87.84  
    ##  Class :character   Class :character   1st Qu.:41.88   1st Qu.:-87.66  
    ##  Mode  :character   Mode  :character   Median :41.90   Median :-87.64  
    ##                                        Mean   :41.90   Mean   :-87.64  
    ##                                        3rd Qu.:41.93   3rd Qu.:-87.63  
    ##                                        Max.   :42.06   Max.   :-87.53  
    ##     end_lat         end_lng       member_casual     
    ##  Min.   :41.65   Min.   :-87.84   Length:4331628    
    ##  1st Qu.:41.88   1st Qu.:-87.66   Class :character  
    ##  Median :41.90   Median :-87.64   Mode  :character  
    ##  Mean   :41.90   Mean   :-87.64                     
    ##  3rd Qu.:41.93   3rd Qu.:-87.63                     
    ##  Max.   :42.06   Max.   :-87.53

The data now looks good so the bike_share_2023_cleaned data frame is
saved to an RDS file and to a CSV file, ready for further processing.

``` r
write_rds(bike_share_2023_cleaned, file = "bike_share_2023_cleaned.RDS")
write_csv(bike_share_2023_cleaned, "bike_share_2023_cleaned.csv")
```
