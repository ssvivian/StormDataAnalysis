---
title: "Health Harm and Property Damage Caused by Weather Events"
output:
  html_document:
    keep_md: yes
---

## Synopsis
This report aims at describing the impact of natural disasters on population health as well as their economic consequences. To find the most harmful and damaging types of events, we use the U.S. National Oceanic and Atmospheric Administration's (NOAA) storm database. This database includes events from 1950 to 2011 and tracks major storms and weather events in the United States, including estimates of any fatalities, injuries, and property and crop damage. Our analysis shows that Tornado is the event type that has caused the highest total number of fatalities and injuries. Regarding economic consequences, Flood, Hurricane, and Tornado are the types of event that have yielded the highest total property damage. When the average cost per event occurrence is considered, Heavy Rain is also a significantly damaging event type.

## Data Processing

We first download and read in the Storm Data, provided by the [National Weather Service](https://www.weather.gov/).


```r
url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
download.file(url, "data/StormData.csv.bz2")
storm_data <- read.csv("data/StormData.csv.bz2")
dim(storm_data)
```

```
## [1] 902297     37
```

```r
head(storm_data)
```

```
##   STATE__           BGN_DATE BGN_TIME TIME_ZONE COUNTY COUNTYNAME STATE
## 1       1  4/18/1950 0:00:00     0130       CST     97     MOBILE    AL
## 2       1  4/18/1950 0:00:00     0145       CST      3    BALDWIN    AL
## 3       1  2/20/1951 0:00:00     1600       CST     57    FAYETTE    AL
## 4       1   6/8/1951 0:00:00     0900       CST     89    MADISON    AL
## 5       1 11/15/1951 0:00:00     1500       CST     43    CULLMAN    AL
## 6       1 11/15/1951 0:00:00     2000       CST     77 LAUDERDALE    AL
##    EVTYPE BGN_RANGE BGN_AZI BGN_LOCATI END_DATE END_TIME COUNTY_END
## 1 TORNADO         0                                               0
## 2 TORNADO         0                                               0
## 3 TORNADO         0                                               0
## 4 TORNADO         0                                               0
## 5 TORNADO         0                                               0
## 6 TORNADO         0                                               0
##   COUNTYENDN END_RANGE END_AZI END_LOCATI LENGTH WIDTH F MAG FATALITIES
## 1         NA         0                      14.0   100 3   0          0
## 2         NA         0                       2.0   150 2   0          0
## 3         NA         0                       0.1   123 2   0          0
## 4         NA         0                       0.0   100 2   0          0
## 5         NA         0                       0.0   150 2   0          0
## 6         NA         0                       1.5   177 2   0          0
##   INJURIES PROPDMG PROPDMGEXP CROPDMG CROPDMGEXP WFO STATEOFFIC ZONENAMES
## 1       15    25.0          K       0                                    
## 2        0     2.5          K       0                                    
## 3        2    25.0          K       0                                    
## 4        2     2.5          K       0                                    
## 5        2     2.5          K       0                                    
## 6        6     2.5          K       0                                    
##   LATITUDE LONGITUDE LATITUDE_E LONGITUDE_ REMARKS REFNUM
## 1     3040      8812       3051       8806              1
## 2     3042      8755          0          0              2
## 3     3340      8742          0          0              3
## 4     3458      8626          0          0              4
## 5     3412      8642          0          0              5
## 6     3450      8748          0          0              6
```

### Subsetting the Data

We are interested in statistics regarding population health harm and economic consequences. In the first case, the data is in the columns `FATALITIES` and `INJURIES`, which contain the total number of fatalities and injuries, respectively, for a given occurrence of an event of some type. We can then subset only the columns of interest.


```r
fatalities <- subset(storm_data, select = c(EVTYPE, FATALITIES, INJURIES))
head(fatalities)
```

```
##    EVTYPE FATALITIES INJURIES
## 1 TORNADO          0       15
## 2 TORNADO          0        0
## 3 TORNADO          0        2
## 4 TORNADO          0        2
## 5 TORNADO          0        2
## 6 TORNADO          0        6
```

Regarding economic consequences, the important columns are `PROPDMG` and `CROPDMG`, which contain the estimates, in American dollars, of property and crop damages, respectively; and `PROPDMGEXP` and `CROPDMGEXP`, which indicate the magnitude of the values in the `PROPDMG` and `CROPDMG` columns, respectively. We then subset these columns.


```r
damages <- subset(storm_data, select = c(EVTYPE, PROPDMG, PROPDMGEXP, CROPDMG, CROPDMGEXP))
head(damages)
```

```
##    EVTYPE PROPDMG PROPDMGEXP CROPDMG CROPDMGEXP
## 1 TORNADO    25.0          K       0           
## 2 TORNADO     2.5          K       0           
## 3 TORNADO    25.0          K       0           
## 4 TORNADO     2.5          K       0           
## 5 TORNADO     2.5          K       0           
## 6 TORNADO     2.5          K       0
```

### Normalizing monetary values

Since data is entered manually by different people, values in the columns `PROPDMGEXP` and `CROPDMGEXP` may not be consistent across all observations. Valid values are H (hundreds), K (thousands), M (millions), and B (billions), so we need to check whether these are the only values in these columns.


```r
unique(damages$PROPDMGEXP)
```

```
##  [1] K M   B m + 0 5 6 ? 4 2 3 h 7 H - 1 8
## Levels:  - ? + 0 1 2 3 4 5 6 7 8 B h H K m M
```

```r
unique(damages$CROPDMGEXP)
```

```
## [1]   M K m B ? 0 k 2
## Levels:  ? 0 2 B k K m M
```

There are many values besides those listed above, so let's check how much they represent in the dataset.


```r
mean(!damages$PROPDMGEXP %in% c("H", "h", "K", "k", "M", "m", "B", "b"))
```

```
## [1] 0.5167345
```

```r
mean(!damages$CROPDMGEXP %in% c("H", "h", "K", "k", "M", "m", "B", "b"))
```

```
## [1] 0.6854062
```

The non-standard values represent more than 50% of the observations, so they can't be ignored. For normalizing the values in the columns `PROPDMG` and `CROPDMG`, we'll use the conversion proposed in [this](https://rstudio-pubs-static.s3.amazonaws.com/58957_37b6723ee52b455990e149edde45e5b6.html) analysis of the data for treating the values in the columns `PROPDMGEXP` and `CROPDMGEXP`.


```r
#Map the values in columns PROPDMGEXP and CROPDMGEXP to their meanings
conversion <- data.frame(value=c("[0-9]", "-|\\?", "\\+", "H|h", "K|k", "M|m", "B|b"), 
                         factor=c(10, 0, 1, 100, 1000, 1000000, 1000000000))

for(i in seq(nrow(conversion))){
    damages$PROPDMGEXP <- gsub(conversion[i, "value"], conversion[i, "factor"], damages$PROPDMGEXP)
    damages$CROPDMGEXP <- gsub(conversion[i, "value"], conversion[i, "factor"], damages$CROPDMGEXP)
}

#Normalize the property and crop damage values
damages$NORM_PROPDMG <- damages$PROPDMG * as.numeric(damages$PROPDMGEXP)
damages$NORM_CROPDMG <- damages$CROPDMG * as.numeric(damages$CROPDMGEXP)
```

## Results

### Population Health Harm

To find which types of events are most harmful with respect to population health across the United States, we sum the number of fatalities and injuries grouped by event type.


```r
total_fatals <- t(sapply(split(fatalities[,2:3], fatalities$EVTYPE), colSums))
total_fatals <- total_fatals[order(total_fatals[,"FATALITIES"], decreasing = TRUE),]
total_fatals <- as.data.frame(total_fatals)
total_fatals <- tibble::rownames_to_column(total_fatals, "EVTYPE")
head(total_fatals)
```

```
##           EVTYPE FATALITIES INJURIES
## 1        TORNADO       5633    91346
## 2 EXCESSIVE HEAT       1903     6525
## 3    FLASH FLOOD        978     1777
## 4           HEAT        937     2100
## 5      LIGHTNING        816     5230
## 6      TSTM WIND        504     6957
```

We'll use the number of fatalities as the parameter for determining the most harmful event types.


```r
summary(total_fatals$FATALITIES)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    0.00    0.00    0.00   15.38    0.00 5633.00
```

By the summary, we can see that at least 75% of the event types have no fatalities. We will, then, use the mean number of fatalities for all the events that had at least one fatality, and plot the events for which the number of fatalities is above the mean.


```r
mean_fatals <- mean(total_fatals$FATALITIES[total_fatals$FATALITIES > 0])
mean_fatals
```

```
## [1] 90.14881
```

```r
#Plot
par(mar=c(5,10,4,2))
barplot(cbind(FATALITIES, INJURIES) ~ EVTYPE, data = total_fatals, subset = FATALITIES > mean_fatals, horiz = TRUE, las = 2, beside = TRUE, cex.names=0.7, ylab = "", col=c("red4","red2"), border = NA, xaxt = "n", legend = c("Fatalities", "Injuries"))
axis(1, at = seq(0, 90000, by = 10000), las = 2)
title(main = "Total Number of Fatalities and Injuries by Event Type")
```

![](storm_data_analysis_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

We can see that Tornado is by far the most harmful event type, with the highest number of both fatalities and injuries, followed by heat-related events. Lightning, Thunderstorm (TSTM) Wind, and Flood are other event types resulting in a high number of injuries.

### Economic Consequences

For finding the types of events that have the greatest economic consequences across the United States, we sum the property and crop damage costs, grouped by event type.


```r
total_damages <- t(sapply(split(damages[,6:7], damages$EVTYPE), colSums, na.rm = TRUE))
total_damages <- total_damages[order(total_damages[,"NORM_PROPDMG"], decreasing = TRUE),]
total_damages <- as.data.frame(total_damages)
total_damages <- tibble::rownames_to_column(total_damages, "EVTYPE")
head(total_damages)
```

```
##              EVTYPE NORM_PROPDMG NORM_CROPDMG
## 1             FLOOD 144657709800   5661968450
## 2 HURRICANE/TYPHOON  69305840000   2607872800
## 3           TORNADO  56937162897    414954710
## 4       STORM SURGE  43323536000         5000
## 5       FLASH FLOOD  16140815011   1421317100
## 6              HAIL  15732269877   3025954650
```

To determine the most economically harmful event types, we'll use the property damage costs as the primary parameter.


```r
summary(total_damages$NORM_PROPDMG)
```

```
##      Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
## 0.000e+00 0.000e+00 0.000e+00 4.338e+08 5.105e+04 1.447e+11
```

At least 50% of the event types caused no property damages, so let's compute the mean total cost for all the events that caused some damage to properties.


```r
mean_damages <- mean(total_damages$NORM_PROPDMG[total_damages$NORM_PROPDMG > 0])
mean_damages
```

```
## [1] 1055107951
```

The mean value is of the order of billions, so let's plot the total cost in billions of US dollars, for all the event types for which the total property damage cost was above the mean.


```r
#Plot
par(mar=c(5,10,4,2))
barplot(cbind((NORM_PROPDMG/1000000000), (NORM_CROPDMG/1000000000)) ~ EVTYPE, data = total_damages, subset = NORM_PROPDMG > mean_damages, horiz = TRUE, las = 2, beside = TRUE, cex.names=0.7, ylab = "", col=c("skyblue4","skyblue2"), border = NA, xaxt = "n", legend = c("Property damage", "Crop damage"))
axis(1, at = seq(0, 150, by = 15), las = 2)
title(main = "Total Cost of Property and Crop Damages by Event Type", xlab = "Cost in $bi")
```

![](storm_data_analysis_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

Since an event can cause much damage but be very infrequent or, conversely, cause moderate damage but occur very often, another way to verify the economic consequence of event types is by computing the mean cost of property and crop damages.


```r
mean_damage_cost <- t(sapply(split(damages[,6:7], damages$EVTYPE), colMeans, na.rm = TRUE))
mean_damage_cost[is.nan(mean_damage_cost)] = 0 #colMeans may introduce NaN entries, so replace then by 0
mean_damage_cost <- mean_damage_cost[order(mean_damage_cost[,"NORM_PROPDMG"], decreasing = TRUE),]
mean_damage_cost <- as.data.frame(mean_damage_cost)
mean_damage_cost <- tibble::rownames_to_column(mean_damage_cost, "EVTYPE")
head(mean_damage_cost)
```

```
##                       EVTYPE NORM_PROPDMG NORM_CROPDMG
## 1  HEAVY RAIN/SEVERE WEATHER   2500000000 0.000000e+00
## 2 TORNADOES, TSTM WIND, HAIL   1600000000 2.500000e+06
## 3          HURRICANE/TYPHOON    990083429 7.902645e+07
## 4             HURRICANE OPAL    396605750 6.333333e+06
## 5                STORM SURGE    247563063 7.142857e+02
## 6        SEVERE THUNDERSTORM    200893333 2.000000e+05
```

```r
mean_avg_dmg_cost <- mean(mean_damage_cost$NORM_PROPDMG[mean_damage_cost$NORM_PROPDMG > 0], na.rm = TRUE)
mean_avg_dmg_cost
```

```
## [1] 17450179
```

```r
#Plot
par(mar=c(5,10,4,2))
barplot(cbind((NORM_PROPDMG/1000000), (NORM_CROPDMG/1000000)) ~ EVTYPE, data = mean_damage_cost, subset = NORM_PROPDMG > mean_avg_dmg_cost, horiz = TRUE, las = 2, beside = TRUE, cex.names=0.7, ylab = "", col=c("seagreen4","seagreen2"), border = NA, xaxt = "n", legend = c("Property damage", "Crop damage"))
axis(1, at = seq(0, 3000, by = 250), las = 2)
title(main = "Average Cost of Property and Crop Damages by Event Type", xlab = "Cost in $mi")
```

![](storm_data_analysis_files/figure-html/unnamed-chunk-14-1.png)<!-- -->

From the plots we can see that Flood is the event type that caused the greatest economic consequence, having the highest total damage cost, followed by Hurricane, Tornado and Storm Surge. Nevertheless, when the average damage cost per event is considered, we can see that Heavy Rain is a type of event that also has great economic consequences.
