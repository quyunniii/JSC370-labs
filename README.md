Lab 05 - Data Wrangling
================

# Learning goals

- Use the `merge()` function to join two datasets.
- Deal with missings and impute data.
- Identify relevant observations using `quantile()`.
- Practice your GitHub skills.

# Lab description

For this lab we will be dealing with the meteorological dataset `met`.
In this case, we will use `data.table` to answer some questions
regarding the `met` dataset, while at the same time practice your
Git+GitHub skills for this project.

This markdown document should be rendered using `github_document`
document.

# Part 1: Setup a Git project and the GitHub repository

1.  Go to wherever you are planning to store the data on your computer,
    and create a folder for this project

2.  In that folder, save [this
    template](https://github.com/JSC370/jsc370-2023/blob/main/labs/lab05/lab05-wrangling-gam.Rmd)
    as “README.Rmd”. This will be the markdown file where all the magic
    will happen.

3.  Go to your GitHub account and create a new repository of the same
    name that your local folder has, e.g., “JSC370-labs”.

4.  Initialize the Git project, add the “README.Rmd” file, and make your
    first commit.

5.  Add the repo you just created on GitHub.com to the list of remotes,
    and push your commit to origin while setting the upstream.

Most of the steps can be done using command line:

``` sh
# Step 1
cd ~/Documents
mkdir JSC370-labs
cd JSC370-labs

# Step 2
wget https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab05/lab05-wrangling-gam.Rmd
mv lab05-wrangling-gam.Rmd README.Rmd
# if wget is not available,
curl https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab05/lab05-wrangling-gam.Rmd --output README.Rmd

# Step 3
# Happens on github

# Step 4
git init
git add README.Rmd
git commit -m "First commit"

# Step 5
git remote add origin git@github.com:[username]/JSC370-labs
git push -u origin master
```

You can also complete the steps in R (replace with your paths/username
when needed)

``` r
# Step 1
setwd("~/Documents")
dir.create("JSC370-labs")
setwd("JSC370-labs")

# Step 2
download.file(
  "https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab05/lab05-wrangling-gam.Rmd",
  destfile = "README.Rmd"
  )

# Step 3: Happens on Github

# Step 4
system("git init && git add README.Rmd")
system('git commit -m "First commit"')

# Step 5
system("git remote add origin git@github.com:[username]/JSC370-labs")
system("git push -u origin master")
```

Once you are done setting up the project, you can now start working with
the MET data.

## Setup in R

1.  Load the `data.table` (and the `dtplyr` and `dplyr` packages if you
    plan to work with those).

``` r
library(data.table)
library(dtplyr)
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:data.table':
    ## 
    ##     between, first, last

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

2.  Load the met data from
    <https://github.com/JSC370/jsc370-2023/blob/main/labs/lab03/met_all.gz>
    or (Use
    <https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab03/met_all.gz>
    to download programmatically), and also the station data. For the
    latter, you can use the code we used during lecture to pre-process
    the stations data:

``` r
met_url <- "https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab03/met_all.gz"
# Downloading the data to a tempfile (so it is destroyed afterwards)
# you can replace this with, for example, your own data:
# tmp <- tempfile(fileext = ".gz")
tmp <- "met_all.gz"
# We sould be downloading this, ONLY IF this was not downloaded already.
# otherwise is just a waste of time.
if (!file.exists(tmp)) {
 download.file(
 url = met_url,
 destfile = tmp,
 method = "libcurl", timeout = 1000
 )
}
dat = fread(tmp)
```

``` r
head(dat)
```

    ##    USAFID  WBAN year month day hour min  lat      lon elev wind.dir wind.dir.qc
    ## 1: 690150 93121 2019     8   1    0  56 34.3 -116.166  696      220           5
    ## 2: 690150 93121 2019     8   1    1  56 34.3 -116.166  696      230           5
    ## 3: 690150 93121 2019     8   1    2  56 34.3 -116.166  696      230           5
    ## 4: 690150 93121 2019     8   1    3  56 34.3 -116.166  696      210           5
    ## 5: 690150 93121 2019     8   1    4  56 34.3 -116.166  696      120           5
    ## 6: 690150 93121 2019     8   1    5  56 34.3 -116.166  696       NA           9
    ##    wind.type.code wind.sp wind.sp.qc ceiling.ht ceiling.ht.qc ceiling.ht.method
    ## 1:              N     5.7          5      22000             5                 9
    ## 2:              N     8.2          5      22000             5                 9
    ## 3:              N     6.7          5      22000             5                 9
    ## 4:              N     5.1          5      22000             5                 9
    ## 5:              N     2.1          5      22000             5                 9
    ## 6:              C     0.0          5      22000             5                 9
    ##    sky.cond vis.dist vis.dist.qc vis.var vis.var.qc temp temp.qc dew.point
    ## 1:        N    16093           5       N          5 37.2       5      10.6
    ## 2:        N    16093           5       N          5 35.6       5      10.6
    ## 3:        N    16093           5       N          5 34.4       5       7.2
    ## 4:        N    16093           5       N          5 33.3       5       5.0
    ## 5:        N    16093           5       N          5 32.8       5       5.0
    ## 6:        N    16093           5       N          5 31.1       5       5.6
    ##    dew.point.qc atm.press atm.press.qc       rh
    ## 1:            5    1009.9            5 19.88127
    ## 2:            5    1010.3            5 21.76098
    ## 3:            5    1010.6            5 18.48212
    ## 4:            5    1011.6            5 16.88862
    ## 5:            5    1012.7            5 17.38410
    ## 6:            5    1012.7            5 20.01540

``` r
# Download the data
stations <- fread("ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-history.csv")
stations[, USAF := as.integer(USAF)]
```

    ## Warning in eval(jsub, SDenv, parent.frame()): NAs introduced by coercion

``` r
# Dealing with NAs and 999999
stations[, USAF   := fifelse(USAF == 999999, NA_integer_, USAF)]
stations[, CTRY   := fifelse(CTRY == "", NA_character_, CTRY)]
stations[, STATE  := fifelse(STATE == "", NA_character_, STATE)]

# Selecting the three relevant columns, and keeping unique records
stations <- unique(stations[, list(USAF, CTRY, STATE)])

# Dropping NAs
stations <- stations[!is.na(USAF)]

# Removing duplicates
stations[, n := 1:.N, by = .(USAF)]
stations <- stations[n == 1,][, n := NULL]
```

3.  Merge the data as we did during the lecture.

``` r
dat <- merge(
# Data
 x = dat,
 y = stations,
# List of variables to match
 by.x = "USAFID",
 by.y = "USAF",
# Which obs to keep?
 all.x = TRUE,
 all.y = FALSE
 )
```

``` r
met_lz = lazy_dt(dat, immutable = FALSE)
```

``` r
met_lz
```

    ## Source: local data table [2,377,343 x 32]
    ## Call:   `_DT1`
    ## 
    ##   USAFID  WBAN  year month   day  hour   min   lat   lon  elev wind.dir wind.d…¹
    ##    <int> <int> <int> <int> <int> <int> <int> <dbl> <dbl> <int>    <int> <chr>   
    ## 1 690150 93121  2019     8     1     0    56  34.3 -116.   696      220 5       
    ## 2 690150 93121  2019     8     1     1    56  34.3 -116.   696      230 5       
    ## 3 690150 93121  2019     8     1     2    56  34.3 -116.   696      230 5       
    ## 4 690150 93121  2019     8     1     3    56  34.3 -116.   696      210 5       
    ## 5 690150 93121  2019     8     1     4    56  34.3 -116.   696      120 5       
    ## 6 690150 93121  2019     8     1     5    56  34.3 -116.   696       NA 9       
    ## # … with 2,377,337 more rows, 20 more variables: wind.type.code <chr>,
    ## #   wind.sp <dbl>, wind.sp.qc <chr>, ceiling.ht <int>, ceiling.ht.qc <int>,
    ## #   ceiling.ht.method <chr>, sky.cond <chr>, vis.dist <int>, vis.dist.qc <chr>,
    ## #   vis.var <chr>, vis.var.qc <chr>, temp <dbl>, temp.qc <chr>,
    ## #   dew.point <dbl>, dew.point.qc <chr>, atm.press <dbl>, atm.press.qc <int>,
    ## #   rh <dbl>, CTRY <chr>, STATE <chr>, and abbreviated variable name
    ## #   ¹​wind.dir.qc
    ## 
    ## # Use as.data.table()/as.data.frame()/as_tibble() to access results

## Question 1: Representative station for the US

Across all weather stations, what is the median station in terms of
temperature, wind speed, and atmospheric pressure? Look for the three
weather stations that best represent continental US using the
`quantile()` function. Do these three coincide?

``` r
#avg for each station
met_avg = met_lz %>% 
  group_by(USAFID) %>%
  summarise(
    across(
      c(temp, wind.sp, atm.press),
      function(x) mean(x, na.rm = TRUE)
    )
#    temp = mean(temp, na.rm = TRUE),
#    wind.sp = mean(wind.sp, na.rm = TRUE),
#    atm.press = mean(atm.press, na.rm = TRUE)
  )
```

``` r
met_med = met_avg %>%
  summarise(across(
    2:4,
    function(x) quantile(x, probs = .5, na.rm = TRUE)
  ))
```

``` r
met_med
```

    ## Source: local data table [1 x 3]
    ## Call:   `_DT1`[, .(temp = (function (x) 
    ## mean(x, na.rm = TRUE))(temp), wind.sp = (function (x) 
    ## mean(x, na.rm = TRUE))(wind.sp), atm.press = (function (x) 
    ## mean(x, na.rm = TRUE))(atm.press)), keyby = .(USAFID)][, .(temp = (function (x) 
    ## quantile(x, probs = 0.5, na.rm = TRUE))(temp), wind.sp = (function (x) 
    ## quantile(x, probs = 0.5, na.rm = TRUE))(wind.sp), atm.press = (function (x) 
    ## quantile(x, probs = 0.5, na.rm = TRUE))(atm.press))]
    ## 
    ##    temp wind.sp atm.press
    ##   <dbl>   <dbl>     <dbl>
    ## 1  23.7    2.46     1015.
    ## 
    ## # Use as.data.table()/as.data.frame()/as_tibble() to access results

``` r
met_avg %>%
  filter(
    temp == met_med %>% pull(temp) |
    wind.sp == met_med %>% pull(wind.sp) |
    atm.press == met_med %>% pull(atm.press)
  )
```

    ## Source: local data table [1 x 4]
    ## Call:   `_DT1`[, .(temp = (function (x) 
    ## mean(x, na.rm = TRUE))(temp), wind.sp = (function (x) 
    ## mean(x, na.rm = TRUE))(wind.sp), atm.press = (function (x) 
    ## mean(x, na.rm = TRUE))(atm.press)), keyby = .(USAFID)][temp == 
    ##     met_med %>% pull(temp) | wind.sp == met_med %>% pull(wind.sp) | 
    ##     atm.press == met_med %>% pull(atm.press)]
    ## 
    ##   USAFID  temp wind.sp atm.press
    ##    <int> <dbl>   <dbl>     <dbl>
    ## 1 720929  17.4    2.46       NaN
    ## 
    ## # Use as.data.table()/as.data.frame()/as_tibble() to access results

``` r
med_temp_id = met_avg %>%
  mutate(temp_diff = abs(temp - met_med %>% pull(temp))) %>%
  arrange(temp_diff) %>%
  slice(1)
med_windsp_id = met_avg %>%
  mutate(wind_diff = abs(wind.sp - met_med %>% pull(wind.sp))) %>%
  arrange(wind_diff) %>%
  slice(1)
med_atmp_id = met_avg %>%
  mutate(atmp_diff = abs(atm.press - met_med %>% pull(atm.press))) %>%
  arrange(atmp_diff) %>%
  slice(1)
```

``` r
med_temp_id
```

    ## Source: local data table [1 x 5]
    ## Call:   `_DT1`[, .(temp = (function (x) 
    ## mean(x, na.rm = TRUE))(temp), wind.sp = (function (x) 
    ## mean(x, na.rm = TRUE))(wind.sp), atm.press = (function (x) 
    ## mean(x, na.rm = TRUE))(atm.press)), keyby = .(USAFID)][, `:=`(temp_diff = abs(temp - 
    ##     ..met_med %>% pull(temp)))][order(temp_diff)][1[between(1, 
    ##     -.N, .N)]]
    ## 
    ##   USAFID  temp wind.sp atm.press temp_diff
    ##    <int> <dbl>   <dbl>     <dbl>     <dbl>
    ## 1 725515  23.7    2.71       NaN   0.00233
    ## 
    ## # Use as.data.table()/as.data.frame()/as_tibble() to access results

``` r
med_windsp_id 
```

    ## Source: local data table [1 x 5]
    ## Call:   `_DT1`[, .(temp = (function (x) 
    ## mean(x, na.rm = TRUE))(temp), wind.sp = (function (x) 
    ## mean(x, na.rm = TRUE))(wind.sp), atm.press = (function (x) 
    ## mean(x, na.rm = TRUE))(atm.press)), keyby = .(USAFID)][, `:=`(wind_diff = abs(wind.sp - 
    ##     ..met_med %>% pull(wind.sp)))][order(wind_diff)][1[between(1, 
    ##     -.N, .N)]]
    ## 
    ##   USAFID  temp wind.sp atm.press wind_diff
    ##    <int> <dbl>   <dbl>     <dbl>     <dbl>
    ## 1 720929  17.4    2.46       NaN         0
    ## 
    ## # Use as.data.table()/as.data.frame()/as_tibble() to access results

``` r
med_atmp_id
```

    ## Source: local data table [1 x 5]
    ## Call:   `_DT1`[, .(temp = (function (x) 
    ## mean(x, na.rm = TRUE))(temp), wind.sp = (function (x) 
    ## mean(x, na.rm = TRUE))(wind.sp), atm.press = (function (x) 
    ## mean(x, na.rm = TRUE))(atm.press)), keyby = .(USAFID)][, `:=`(atmp_diff = abs(atm.press - 
    ##     ..met_med %>% pull(atm.press)))][order(atmp_diff)][1[between(1, 
    ##     -.N, .N)]]
    ## 
    ##   USAFID  temp wind.sp atm.press atmp_diff
    ##    <int> <dbl>   <dbl>     <dbl>     <dbl>
    ## 1 723200  25.8    1.54     1015.  0.000538
    ## 
    ## # Use as.data.table()/as.data.frame()/as_tibble() to access results

``` r
distinct(met_lz %>%
  filter(
    USAFID  == 725515 |
    USAFID  == 720929 |
    USAFID  == 723200 
  ), USAFID, lat, lon)
```

    ## Source: local data table [5 x 3]
    ## Call:   unique(`_DT1`[USAFID == 725515 | USAFID == 720929 | USAFID == 
    ##     723200, .(USAFID, lat, lon)])
    ## 
    ##   USAFID   lat   lon
    ##    <int> <dbl> <dbl>
    ## 1 720929  45.5 -92.0
    ## 2 723200  34.4 -85.2
    ## 3 723200  34.3 -85.2
    ## 4 723200  34.4 -85.2
    ## 5 725515  40.3 -96.8
    ## 
    ## # Use as.data.table()/as.data.frame()/as_tibble() to access results

Knit the document, commit your changes, and save it on GitHub. Don’t
forget to add `README.md` to the tree, the first time you render it.

## Question 2: Representative station per state

Just like the previous question, you are asked to identify what is the
most representative, the median, station per state. This time, instead
of looking at one variable at a time, look at the euclidean distance. If
multiple stations show in the median, select the one located at the
lowest latitude.

``` r
met_avg_2 = met_lz %>% 
  group_by(USAFID) %>%
  summarise(
    STATE = STATE,
    lon = lon,
    lat = lat,
    across(
      c(temp, wind.sp, atm.press),
      function(x) mean(x, na.rm = TRUE)
    )
  )
    
state_med = met_avg_2 %>% 
  group_by(STATE) %>%
  summarise(
    across(
      c(temp, wind.sp, atm.press),
      function(x) quantile(x, probs = .5, na.rm = TRUE)
    )
  )
# 1 station for 3 variables
```

``` r
met_avg_state_med = merge(x=met_avg_2,y=state_med,by="STATE",all.x=TRUE)
```

``` r
met_avg_state_med 
```

``` r
met_avg_state_med = met_avg_state_med %>%
  mutate(ecl_diff = sqrt((temp.x - temp.y)^2 +(wind.sp.x - wind.sp.y)^2+ (atm.press.x - atm.press.y)^2))
na.omit(met_avg_state_med)
```

    ## Warning in min(ecl_diff, na.rm = TRUE): no non-missing arguments to min;
    ## returning Inf

    ## Warning in min(ecl_diff, na.rm = TRUE): no non-missing arguments to min;
    ## returning Inf

``` r
state_med_final
```

    ## # A tibble: 48 × 2
    ##    STATE ecl_diff
    ##    <chr>    <dbl>
    ##  1 AL       0.192
    ##  2 AR       0.402
    ##  3 AZ       1.72 
    ##  4 CA       0.174
    ##  5 CO       0.966
    ##  6 CT       0.309
    ##  7 DE       0    
    ##  8 FL       0.162
    ##  9 GA       0.388
    ## 10 IA       0.208
    ## # … with 38 more rows

``` r
med_by_state = distinct(merge(x=state_med_final,y=met_avg_state_med,by=c("STATE", "ecl_diff") ,all.x=TRUE))
```

``` r
med_by_state
```

    ##    STATE  ecl_diff USAFID      lon    lat   temp.x wind.sp.x atm.press.x
    ## 1     AL 0.1917347 722286  -87.616 33.212 26.35793  1.675828    1014.909
    ## 2     AR 0.4024542 723407  -90.646 35.831 25.86949  2.208652    1014.575
    ## 3     AR 0.4024542 723407  -90.646 35.832 25.86949  2.208652    1014.575
    ## 4     AZ 1.7210317 723740 -110.720 35.028 27.20925  3.677113    1011.656
    ## 5     AZ 1.7210317 723740 -110.733 35.017 27.20925  3.677113    1011.656
    ## 6     AZ 1.7210317 723740 -110.723 35.022 27.20925  3.677113    1011.656
    ## 7     AZ 1.7210317 723740 -110.721 35.028 27.20925  3.677113    1011.656
    ## 8     CA 0.1736276 722977 -117.866 33.680 22.28589  2.364013    1012.653
    ## 9     CA 0.1736276 722977 -117.868 33.676 22.28589  2.364013    1012.653
    ## 10    CO 0.9658559 724665 -103.666 39.275 20.75472  3.946234    1012.891
    ## 11    CO 0.9658559 724665 -103.666 39.273 20.75472  3.946234    1012.891
    ## 12    CT 0.3085369 725087  -72.651 41.736 22.57539  2.126514    1014.534
    ## 13    CT 0.3085369 725087  -72.649 41.737 22.57539  2.126514    1014.534
    ## 14    DE 0.0000000 724180  -75.606 39.674 24.56026  2.752929    1015.046
    ## 15    DE 0.0000000 724180  -75.607 39.679 24.56026  2.752929    1015.046
    ## 16    FL 0.1623900 722210  -86.517 30.483 27.65745  2.531633    1015.408
    ## 17    FL 0.1623900 722210  -86.525 30.483 27.65745  2.531633    1015.408
    ## 18    GA 0.3884726 723160  -82.507 31.536 26.59746  1.684538    1014.985
    ## 19    IA 0.2075646 725480  -92.401 42.554 21.43686  2.764312    1014.814
    ## 20    IA 0.2075646 725480  -92.400 42.550 21.43686  2.764312    1014.814
    ## 21    IA 0.2075646 725480  -92.400 42.557 21.43686  2.764312    1014.814
    ## 22    ID 0.3397841 722142 -114.215 44.523 20.32324  2.184444    1012.609
    ## 23    ID 0.3397841 722142 -114.218 44.523 20.32324  2.184444    1012.609
    ## 24    IL 0.7078133 725440  -90.523 41.465 22.84806  2.566829    1014.760
    ## 25    IL 0.7078133 725440  -90.500 41.450 22.84806  2.566829    1014.760
    ## 26    IN 0.6447527 725330  -85.206 40.971 21.73189  2.851982    1015.284
    ## 27    IN 0.6447527 725330  -85.200 40.983 21.73189  2.851982    1015.284
    ## 28    KS 0.2503865 724580  -97.651 39.551 24.01181  3.548029    1013.449
    ## 29    KS 0.2503865 724580  -97.650 39.550 24.01181  3.548029    1013.449
    ## 30    KY 0.6859086 724243  -84.077 37.087 23.18690  1.458032    1015.642
    ## 31    KY 0.6859086 724243  -84.085 37.082 23.18690  1.458032    1015.642
    ## 32    LA 0.3471884 722486  -92.041 32.516 28.16413  1.592840    1014.544
    ## 33    LA 0.3471884 722486  -92.038 32.511 28.16413  1.592840    1014.544
    ## 34    MA 0.1373438 725064  -70.729 41.910 21.40933  2.786213    1014.721
    ## 35    MA 0.1373438 725064  -70.729 41.909 21.40933  2.786213    1014.721
    ## 36    MD 0.5535881 724057  -76.170 39.472 25.00877  2.033233    1014.497
    ## 37    MD 0.5535881 724057  -76.169 39.466 25.00877  2.033233    1014.497
    ## 38    ME 0.3784877 726077  -68.367 44.450 18.49969  2.337241    1014.475
    ## 39    ME 0.3784877 726077  -68.361 44.450 18.49969  2.337241    1014.475
    ## 40    MI 0.1561130 726355  -86.428 42.126 20.43892  1.930327    1014.947
    ## 41    MI 0.1561130 726355  -86.427 42.125 20.43892  1.930327    1014.947
    ## 42    MI 0.1561130 726355  -86.428 42.129 20.43892  1.930327    1014.947
    ## 43    MN 0.6873763 726550  -94.051 45.543 19.11831  2.832794    1015.319
    ## 44    MN 0.6873763 726550  -94.060 45.547 19.11831  2.832794    1015.319
    ## 45    MO 0.5361189 723495  -94.495 37.152 24.31621  2.550940    1014.296
    ## 46    MO 0.5361189 723495  -94.498 37.152 24.31621  2.550940    1014.296
    ## 47    MS 0.4784087 723306  -88.450 33.650 26.01216  1.827912    1014.830
    ## 48    MT 0.9793020 726797 -111.160 45.788 18.78980  2.858586    1014.902
    ## 49    MT 0.9793020 726797 -111.153 45.777 18.78980  2.858586    1014.902
    ## 50    MT 0.9793020 726797 -111.161 45.788 18.78980  2.858586    1014.902
    ## 51    NC 0.5531862 723174  -79.477 36.047 24.95288  1.744838    1015.350
    ## 52    ND       Inf     NA       NA     NA       NA        NA          NA
    ## 53    NE 0.3549801 725527  -96.178 41.764 22.20987  3.121762    1014.065
    ## 54    NH 0.6778409 726050  -71.503 43.205 19.86188  1.732752    1014.487
    ## 55    NH 0.6778409 726050  -71.500 43.200 19.86188  1.732752    1014.487
    ## 56    NH 0.6778409 726050  -71.502 43.203 19.86188  1.732752    1014.487
    ## 57    NJ 0.0000000 724075  -75.078 39.366 23.83986  1.949704    1014.825
    ## 58    NJ 0.0000000 724075  -75.072 39.368 23.83986  1.949704    1014.825
    ## 59    NM 1.3632207 723650 -106.617 35.033 26.22686  3.517116    1011.941
    ## 60    NM 1.3632207 723650 -106.615 35.042 26.22686  3.517116    1011.941
    ## 61    NM 1.3632207 723650 -106.609 35.040 26.22686  3.517116    1011.941
    ## 62    NM 1.3632207 723650 -106.616 35.042 26.22686  3.517116    1011.941
    ## 63    NV 0.7535990 725830 -117.808 40.902 23.49863  2.966240    1012.679
    ## 64    NV 0.7535990 725830 -117.800 40.900 23.49863  2.966240    1012.679
    ## 65    NV 0.7535990 725830 -117.806 40.897 23.49863  2.966240    1012.679
    ## 66    NY 0.5007516 725194  -77.056 42.643 20.37207  2.444051    1015.327
    ## 67    NY 0.5007516 725194  -77.053 42.637 20.37207  2.444051    1015.327
    ## 68    OH 0.3601454 725254  -84.429 41.338 22.14885  2.330167    1015.117
    ## 69    OK 0.2737236 723537  -97.414 35.852 27.05520  3.646514    1012.567
    ## 70    OK 0.2737236 723537  -97.416 35.850 27.05520  3.646514    1012.567
    ## 71    OR 1.1661337 720365 -124.290 42.070 16.10729  1.468683    1015.957
    ## 72    PA 0.4460917 725130  -75.727 41.334 21.69177  1.970192    1015.125
    ## 73    PA 0.4460917 725130  -75.717 41.333 21.69177  1.970192    1015.125
    ## 74    PA 0.4460917 725130  -75.723 41.338 21.69177  1.970192    1015.125
    ## 75    RI 0.3375856 725079  -71.283 41.533 22.27697  2.583469    1014.620
    ## 76    RI 0.3375856 725079  -71.282 41.532 22.27697  2.583469    1014.620
    ## 77    SC 0.6778528 723190  -82.710 34.498 25.73726  2.253408    1015.116
    ## 78    SC 0.6778528 723190  -82.709 34.495 25.73726  2.253408    1015.116
    ## 79    SD 0.2428327 726590  -98.413 45.443 19.95928  3.550722    1014.284
    ## 80    SD 0.2428327 726590  -98.417 45.450 19.95928  3.550722    1014.284
    ## 81    SD 0.2428327 726590  -98.422 45.449 19.95928  3.550722    1014.284
    ## 82    TN 0.1240055 723346  -88.917 35.593 24.59407  1.493531    1015.144
    ## 83    TN 0.1240055 723346  -88.916 35.600 24.59407  1.493531    1015.144
    ## 84    TX 0.3727459 722540  -97.700 30.300 29.98977  3.092698    1012.252
    ## 85    TX 0.3727459 722540  -97.680 30.183 29.98977  3.092698    1012.252
    ## 86    TX 0.3727459 722540  -97.670 30.195 29.98977  3.092698    1012.252
    ## 87    UT 0.8310616 725720 -111.969 40.778 26.94480  3.527679    1010.886
    ## 88    UT 0.8310616 725720 -111.950 40.783 26.94480  3.527679    1010.886
    ## 89    UT 0.8310616 725720 -111.978 40.788 26.94480  3.527679    1010.886
    ## 90    VA 0.2118704 724016  -78.455 38.137 24.29327  1.588105    1014.946
    ## 91    VA 0.2118704 724016  -78.453 38.139 24.29327  1.588105    1014.946
    ## 92    VT 0.4686234 726114  -72.614 44.534 17.46999  1.165761    1014.792
    ## 93    VT 0.4686234 726114  -72.614 44.535 17.46999  1.165761    1014.792
    ## 94    WA       Inf     NA       NA     NA       NA        NA          NA
    ## 95    WI 0.6481723 726416  -90.181 43.212 19.12963  1.653207    1014.525
    ## 96    WI 0.6481723 726416  -90.182 43.212 19.12963  1.653207    1014.525
    ## 97    WV 0.2275864 724176  -79.916 39.643 21.94072  1.649151    1015.982
    ## 98    WY 1.1004513 725645 -105.683 41.317 18.39674  4.889281    1012.735
    ## 99    WY 1.1004513 725645 -105.675 41.312 18.39674  4.889281    1012.735
    ##      temp.y wind.sp.y atm.press.y
    ## 1  26.22064  1.543094    1014.926
    ## 2  26.07275  1.861651    1014.591
    ## 3  26.07275  1.861651    1014.591
    ## 4  27.70883  3.023396    1010.144
    ## 5  27.70883  3.023396    1010.144
    ## 6  27.70883  3.023396    1010.144
    ## 7  27.70883  3.023396    1010.144
    ## 8  22.32428  2.524027    1012.708
    ## 9  22.32428  2.524027    1012.708
    ## 10 20.75472  3.087924    1013.334
    ## 11 20.75472  3.087924    1013.334
    ## 12 22.44858  2.077088    1014.810
    ## 13 22.44858  2.077088    1014.810
    ## 14 24.56026  2.752929    1015.046
    ## 15 24.56026  2.752929    1015.046
    ## 16 27.51250  2.531633    1015.335
    ## 17 27.51250  2.531633    1015.335
    ## 18 26.78250  1.426370    1015.208
    ## 19 21.36209  2.633381    1014.957
    ## 20 21.36209  2.633381    1014.957
    ## 21 21.36209  2.633381    1014.957
    ## 22 20.16204  2.168066    1012.908
    ## 23 20.16204  2.168066    1012.908
    ## 24 22.33580  2.078367    1014.760
    ## 25 22.33580  2.078367    1014.760
    ## 26 21.73189  2.246247    1015.063
    ## 27 21.73189  2.246247    1015.063
    ## 28 24.21648  3.679474    1013.389
    ## 29 24.21648  3.679474    1013.389
    ## 30 23.68173  1.732910    1015.254
    ## 31 23.68173  1.732910    1015.254
    ## 32 27.84758  1.459030    1014.593
    ## 33 27.84758  1.459030    1014.593
    ## 34 21.40933  2.648870    1014.721
    ## 35 21.40933  2.648870    1014.721
    ## 36 24.89883  1.600480    1014.824
    ## 37 24.89883  1.600480    1014.824
    ## 38 18.82098  2.137179    1014.475
    ## 39 18.82098  2.137179    1014.475
    ## 40 20.50971  2.069470    1014.947
    ## 41 20.50971  2.069470    1014.947
    ## 42 20.50971  2.069470    1014.947
    ## 43 19.53001  2.357470    1015.042
    ## 44 19.53001  2.357470    1015.042
    ## 45 23.99322  2.187797    1014.522
    ## 46 23.99322  2.187797    1014.522
    ## 47 26.19508  1.385855    1014.830
    ## 48 18.78980  3.378081    1014.072
    ## 49 18.78980  3.378081    1014.072
    ## 50 18.78980  3.378081    1014.072
    ## 51 24.51396  1.415669    1015.420
    ## 52       NA        NA          NA
    ## 53 21.99129  3.121762    1014.345
    ## 54 19.23920  1.556907    1014.689
    ## 55 19.23920  1.556907    1014.689
    ## 56 19.23920  1.556907    1014.689
    ## 57 23.83986  1.949704    1014.825
    ## 58 23.83986  1.949704    1014.825
    ## 59 24.94447  3.517116    1012.404
    ## 60 24.94447  3.517116    1012.404
    ## 61 24.94447  3.517116    1012.404
    ## 62 24.94447  3.517116    1012.404
    ## 63 23.67835  2.966240    1011.947
    ## 64 23.67835  2.966240    1011.947
    ## 65 23.67835  2.966240    1011.947
    ## 66 20.37207  2.204099    1014.887
    ## 67 20.37207  2.204099    1014.887
    ## 68 21.87803  2.368546    1015.351
    ## 69 27.28791  3.646514    1012.711
    ## 70 27.28791  3.646514    1012.711
    ## 71 17.16329  1.942080    1015.813
    ## 72 21.87141  1.759793    1015.474
    ## 73 21.87141  1.759793    1015.474
    ## 74 21.87141  1.759793    1015.474
    ## 75 22.53551  2.583469    1014.837
    ## 76 22.53551  2.583469    1014.837
    ## 77 25.73726  1.592177    1015.265
    ## 78 25.73726  1.592177    1015.265
    ## 79 19.95928  3.665638    1014.497
    ## 80 19.95928  3.665638    1014.497
    ## 81 19.95928  3.665638    1014.497
    ## 82 24.71645  1.513550    1015.144
    ## 83 24.71645  1.513550    1015.144
    ## 84 29.68309  3.150943    1012.456
    ## 85 29.68309  3.150943    1012.456
    ## 86 29.68309  3.150943    1012.456
    ## 87 26.94480  3.361211    1011.701
    ## 88 26.94480  3.361211    1011.701
    ## 89 26.94480  3.361211    1011.701
    ## 90 24.30992  1.588105    1015.158
    ## 91 24.30992  1.588105    1015.158
    ## 92 17.87100  1.408247    1014.792
    ## 93 17.87100  1.408247    1014.792
    ## 94       NA        NA          NA
    ## 95 18.71326  1.986436    1014.893
    ## 96 18.71326  1.986436    1014.893
    ## 97 21.94072  1.617823    1015.757
    ## 98 18.43778  3.873392    1013.157
    ## 99 18.43778  3.873392    1013.157

Knit the doc and save it on GitHub.

## Question 3: In the middle?

For each state, identify what is the station that is closest to the
mid-point of the state. Combining these with the stations you identified
in the previous question, use `leaflet()` to visualize all \~100 points
in the same figure, applying different colors for those identified in
this question.

Knit the doc and save it on GitHub.

## Question 4: Means of means

Using the `quantile()` function, generate a summary table that shows the
number of states included, average temperature, wind-speed, and
atmospheric pressure by the variable “average temperature level,” which
you’ll need to create.

Start by computing the states’ average temperature. Use that measurement
to classify them according to the following criteria:

- low: temp \< 20
- Mid: temp \>= 20 and temp \< 25
- High: temp \>= 25

Once you are done with that, you can compute the following:

- Number of entries (records),
- Number of NA entries,
- Number of stations,
- Number of states included, and
- Mean temperature, wind-speed, and atmospheric pressure.

All by the levels described before.

Knit the document, commit your changes, and push them to GitHub.

## Question 5: Advanced Regression

Let’s practice running regression models with smooth functions on X. We
need the `mgcv` package and `gam()` function to do this.

- using your data with the median values per station, examine the
  association between median temperature (y) and median wind speed (x).
  Create a scatterplot of the two variables using ggplot2. Add both a
  linear regression line and a smooth line.

- fit both a linear model and a spline model (use `gam()` with a cubic
  regression spline on wind speed). Summarize and plot the results from
  the models and interpret which model is the best fit and why.
