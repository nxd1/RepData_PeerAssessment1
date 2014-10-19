---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---
## Reproducible Research: Peer Assessment 1
This assignment requires to write a report that answers the questions detailed below. The entire assignment is to be completed in a **single R markdown** document that can be processed by **knitr** and transformed into an HTML file.

### PART 1: Loading and preprocessing the data

```r
library(data.table)
library(ggplot2)
library(knitr)
library(plyr)
```


```r
# Unzip and read the data file from current working directory
unzip("activity.zip", overwrite = T, exdir = ".")
df <- read.csv("activity.csv", header=T, colClasses=c("numeric", "character", "numeric"))

# Convert date format for processing weekdays
df$date <- as.Date(df$date, format="%Y-%m-%d")

# Make data table to process/transform the dataset
dt <- data.table(df, key="date")

# Calculate sum and mean for steps per day, ignoring missing values
d.steps <- dt[, list(d.sum=sum(steps, na.rm=T), d.mean=mean(steps, na.rm=T)), by=list(date)]
```

### PART 2: What is mean total number of steps taken per day?
- Histogram of steps taken per day  


```r
hist(d.steps$d.sum, breaks=10, freq=T, density=30, col="blue", border="grey",
     main="", xlab="Steps", xlim=c(0, 25000), ylim=c(0,20))
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

```r
mean.orig <- mean(d.steps$d.sum, na.rm=T)
median.orig <- median(d.steps$d.sum, na.rm=T)
```

- **mean** = 9354.23 and **median** = 10395.00 for total number of steps per day.

### PART 3: What is the average daily activity pattern?
- Plot of the 5-minute interval and average number of steps across all days:  


```r
i.steps <- dt[, list(i.mean=mean(steps, na.rm=T)), by=list(interval)]
max.idx <- which.max(i.steps$i.mean)
interval <- i.steps[max.idx, interval]
m.max <- i.steps[max.idx, i.mean]

plot(i.mean ~ interval, i.steps, type="l", xlab="Interval", ylab="Avg. Number of Steps")
```

![plot of chunk timeplot](figure/timeplot.png) 

- Interval **835** contains the maximun avg. number of steps of 206.17

### PART 4: Imputing Missing Values

```r
# Count complete cases (no NA's)
na.count <- sum(complete.cases(dt) == FALSE)
ok.count <- sum(complete.cases(dt) == TRUE)

# Merge dataset of computed interval averages with original data, joining by the interval
dt2 <- merge(dt, i.steps, by="interval")

# Create new dataset with the missing data filled in:
# Add new column containing either: a) the mean # of steps for the interval, when steps
# are missing; otherwise, b) the original number of steps for the interval
dt2 <- dt2[, fill:={ifelse(is.na(steps), i.mean, steps)}]
```
- Total number of missing values in the dataset (i.e. rows with `NA`s): 2304

    - 15264 good counts / 17568 total observations

- Filled in missing values with the mean number of steps for the interval
- Histogram of the estimated total number of steps taken each day
  

```r
# Calculate sum and mean for steps per day using values filled in for missing data
d2.steps <- dt2[, list(d.sum=sum(fill), d.mean=mean(fill)), by=list(date)]

hist(d2.steps$d.sum,
     breaks=c(10), freq=T, density=30, col="blue",
     main="", xlab="Steps", xlim=c(0, 25000), ylim=c(0,20))
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

```r
mean.est <- mean(d2.steps$d.sum)
median.est <- median(d2.steps$d.sum)
mean.diff <- mean.est - mean.orig
median.diff <- median.est - median.orig
mean.pct.diff <- sprintf("%.1f%%", mean.diff/mean.orig*100)
median.pct.diff <- sprintf("%.1f%%", median.diff/median.orig*100)
```

- New **mean** = 10766.19, **median** = 10766.19 for  estimated steps per day

- These estimated values differ from the first part of the assignment, as follows:

    + estimated mean differs by 1411.96 (15.1%) from original
    + estimated median differs by 371.19 (3.6%) from original

- Impact of imputing missing data on the estimates of the total daily number of steps:

    - flattens the distribution over the intervals, spiked at the mean
    - tends to underestimate variance

### PART 5: Differences in activity patterns between weekdays and weekends

- Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
dt2$d.type <- ifelse(weekdays(dt2$date) %in% c("Saturday", "Sunday"), "weekend", "weekday")
```

- Make a panel plot containing a time series of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
i2.steps <- dt2[, list(d.type.mean=mean(fill)), by=list(d.type, interval)]
par(mfrow = c(2, 1))
g <- ggplot(i2.steps, aes(interval, d.type.mean))
g + geom_line() + facet_grid(. ~ d.type) + facet_wrap(~ d.type, nrow=2, ncol=1) +
  xlab("Interval") + ylab("Avg Number of Steps")
```

![plot of chunk timeplot2](figure/timeplot2.png) 
