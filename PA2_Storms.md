# Reproducible Research: Peer Assessment 2, Storms


## Loading packages

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
## 
## The following objects are masked from 'package:stats':
## 
##     filter, lag
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(lubridate)
```


### Loading and Processing the Raw data

We download the data from the course website for Reproducible Research. Note that https:// URLs are not supported by download.file by default, but using method = "curl" permits the download. Also we use "bzfile" in the read.csv argument since the file is a bz2 file.


```r
temp <- tempfile()
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2", temp, method = "curl")
data <- read.csv(bzfile(temp, "repdata%2Fdata%2FStormData.csv.bz2"), header = TRUE)
unlink(temp)
```

Let's have a look at the data. First of all I prefer having variables in lowercase

```r
data2 <- data ## remove this later, just so I don't have to keep reading in the file
names(data2) <- tolower(names(data2))
data2 <- data2[,c("bgn_date", "state", "evtype",  "fatalities", "injuries", "propdmg", "propdmgexp", "cropdmg", "cropdmgexp")]


names(data2)
```

```
## [1] "bgn_date"   "state"      "evtype"     "fatalities" "injuries"  
## [6] "propdmg"    "propdmgexp" "cropdmg"    "cropdmgexp"
```

```r
data2$evtype <- tolower(data2$evtype)
length(unique(data2$evtype))
```

```
## [1] 898
```



```r
## remove extraneous variables


head(data2)
```

```
##             bgn_date state  evtype fatalities injuries propdmg propdmgexp
## 1  4/18/1950 0:00:00    AL tornado          0       15    25.0          K
## 2  4/18/1950 0:00:00    AL tornado          0        0     2.5          K
## 3  2/20/1951 0:00:00    AL tornado          0        2    25.0          K
## 4   6/8/1951 0:00:00    AL tornado          0        2     2.5          K
## 5 11/15/1951 0:00:00    AL tornado          0        2     2.5          K
## 6 11/15/1951 0:00:00    AL tornado          0        6     2.5          K
##   cropdmg cropdmgexp
## 1       0           
## 2       0           
## 3       0           
## 4       0           
## 5       0           
## 6       0
```

```r
## remove extraneous event types - working
event_pattern <- "(summary|\\?|other|none|month|no severe)"
data2$index <- grepl(event_pattern, data2$evtype)
data2 <- data2[!data2$index == TRUE,] # remove daily summaries and unknown types


## group thunderstorm types together - working
length(unique(data2$evtype))
```

```
## [1] 818
```

```r
data2$evtype <- gsub(" ", "_", data2$evtype)
data2$evtype2 <- data2$evtype
length(unique(data2$evtype2))
```

```
## [1] 818
```

```r
thunder_pattern <- "(thund|microburst|tstm|tunder|thuder)"
data2$evtype2 <- ifelse(grepl(thunder_pattern, data2$evtype) == TRUE, "thunderstorm", data2$evtype2)

length(unique(data2$evtype2))
```

```
## [1] 688
```

```r
hurricane_pattern <- "(hurricane|typhoon)"
data2$evtype2 <- ifelse(grepl(hurricane_pattern, data2$evtype) == TRUE, "hurricane", data2$evtype2)

tropical_storm_pattern <- "tropical storm"
data2$evtype2 <- ifelse(grepl(tropical_storm_pattern, data2$evtype) == TRUE, "tropical storm", data2$evtype2)

drought_pattern <- "(dry|driest|record_low_rainfall)"
data2$evtype2 <- ifelse(grepl(drought_pattern, data2$evtype) == TRUE, "drought", data2$evtype2)

cold_pattern <- "(cold|wind_chill|windchill|record_low|low_temp)"
data2$evtype2 <- ifelse(grepl(cold_pattern, data2$evtype) == TRUE, "cold_windchill", data2$evtype2)

heat_pattern <- "(heat|hot|warm|record_temperature|hyperthermia|temperature_record|record_high)"
data2$evtype2 <- ifelse(grepl(heat_pattern, data2$evtype) == TRUE, "heat", data2$evtype2)

surf_pattern <- "surf"
data2$evtype2 <- ifelse(grepl(surf_pattern, data2$evtype) == TRUE, "high_surf", data2$evtype2)

tornado_pattern <- "nado|torn"
data2$evtype2 <- ifelse(grepl(tornado_pattern, data2$evtype) == TRUE, "tornado", data2$evtype2)

hail_pattern <- "(hail|ice_pellets)"
data2$evtype2 <- ifelse(grepl(hail_pattern, data2$evtype) == TRUE, "hail", data2$evtype2)

flood_pattern <- "(flood|fld)"
data2$evtype2 <- ifelse(grepl(flood_pattern, data2$evtype) == TRUE, "flood", data2$evtype2)

rain_pattern <- "(rain|record_precipitation|wet)"
data2$evtype2 <- ifelse(grepl(rain_pattern, data2$evtype) == TRUE, "heavy rain", data2$evtype2)

avalanche_pattern <- "avalan"
data2$evtype2 <- ifelse(grepl(avalanche_pattern, data2$evtype) == TRUE, "avalanche", data2$evtype2)

blizzard_pattern <- "blizz"
data2$evtype2 <- ifelse(grepl(blizzard_pattern, data2$evtype) == TRUE, "blizzard", data2$evtype2)


fire_pattern <- "fire"
data2$evtype2 <- ifelse(grepl(fire_pattern, data2$evtype) == TRUE, "fire", data2$evtype2)

winter_pattern <- "winter"
data2$evtype2 <- ifelse(grepl(winter_pattern, data2$evtype) == TRUE, "winter weather", data2$evtype2)

frost_pattern <- "(frost|freez)"
data2$evtype2 <- ifelse(grepl(frost_pattern, data2$evtype) == TRUE, "frost_freeze", data2$evtype2)

sleet_pattern <- "sleet"
data2$evtype2 <- ifelse(grepl(sleet_pattern, data2$evtype) == TRUE, "sleet", data2$evtype2)

fog_pattern <- "fog"
data2$evtype2 <- ifelse(grepl(fog_pattern, data2$evtype) == TRUE, "fog", data2$evtype2)

lightning_pattern <- "lightning"
data2$evtype2 <- ifelse(grepl(lightning_pattern, data2$evtype) == TRUE, "lightning", data2$evtype2)

snow_ice_pattern <- "(snow|ice)" # note many events list both so hard to distinguish
data2$evtype2 <- ifelse(grepl(snow_ice_pattern, data2$evtype) == TRUE, "snow_ice", data2$evtype2)

tide_pattern <- "(high_tide|surge)"
data2$evtype2 <- ifelse(grepl(tide_pattern, data2$evtype) == TRUE, "tide_surge", data2$evtype2)

length(unique(data2$evtype2))
```

```
## [1] 180
```

```r
data2 %>% group_by(evtype2) %>% summarise(fatalities = sum(fatalities)) %>% arrange(desc(fatalities))
```

```
## Source: local data frame [180 x 2]
## 
##           evtype2 fatalities
## 1         tornado       5636
## 2            heat       3179
## 3           flood       1552
## 4       lightning        817
## 5    thunderstorm        725
## 6  cold_windchill        459
## 7     rip_current        368
## 8  winter weather        278
## 9        snow_ice        266
## 10      high_wind        248
## ..            ...        ...
```


# Question 1:
## Across the United States, which types of events (as indicated in the EVTYPE variable) are most harmful with respect to population health?




# Question 2:
## Across the United States, which types of events have the greatest economic consequences?
