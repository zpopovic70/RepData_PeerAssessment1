# Reproducible Research: Peer Assessment 1

## 1. Loading and preprocessing the data 
### Pre-requisite
Data file *activity.csv* (uncompressed version of the *[activity.zip](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)*) has to be preset in the working folder. 

Libraries *dplyr* and *lattice* have been loaded


### Loading 
Data is loaded into *activity* data frame.

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
library(lattice)
activity <- read.csv('activity.csv')
head(activity)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

```r
nrow(activity)
```

```
## [1] 17568
```

## 2. What is mean total number of steps taken per day?
The total number of steps for each day is calculated by creating data frame *activity_by_day* ,utilizing *aggregate* function to sum up total steps taken each day 

```r
activity_by_day <- aggregate(steps ~ date, data = activity, FUN = sum, na.rm = TRUE)
head(activity_by_day)
```

```
##         date steps
## 1 2012-10-02   126
## 2 2012-10-03 11352
## 3 2012-10-04 12116
## 4 2012-10-05 13294
## 5 2012-10-06 15420
## 6 2012-10-07 11015
```
The histogram showing total number of steps each day has been created using *hist* plotting function

```r
hist(activity_by_day$steps, main = "The total number of steps per day", xlab = "Number of steps", col = "green")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

The mean is calculated using R's *mean* function

```r
mean(activity_by_day$steps)
```

```
## [1] 10766.19
```
The median is calculated using R's *median* function

```r
median(activity_by_day$steps)
```

```
## [1] 10765
```

## 3. What is the average daily activity pattern?

To show the average daily activity pattern, we make a time series plot of the 5-minute interval and the average number of steps taken, averaged across all days

```r
activity_by_interval <- aggregate(steps ~ interval, data = activity, FUN = mean)

plot(x = activity_by_interval$interval,
     y = activity_by_interval$steps, 
     type = "l",
     main = "Average Daily Activity Pattern",
     xlab = "Interval (mins)",
     ylab = "Number of Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

Next, we find out which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps

```r
max_avg_daily_activity = activity_by_interval[which.max(activity_by_interval$steps),]
max_avg_daily_activity$interval
```

```
## [1] 835
```

We have found that interval 835 has the highest average number of steps

## 4. Imputing missing values

The total number of missing values in the dataset (i.e. the total number of rows with NAs) is found to be 2304, by executing the following command:

```r
nrow(activity[is.na(activity$steps),])
```

```
## [1] 2304
```

The strategy to replace the missing values is to replace them with the mean number of steps taken from the corresponding interval. 

```r
activity_filled <- activity
avg_activity_by_interval <- aggregate(steps ~ interval, data = activity_filled, FUN = mean, rm.na = TRUE)
activity_filled <- merge(activity_filled, avg_activity_by_interval, by="interval", suffixes=c("",".by_interval"))
nas <- is.na(activity_filled$steps)
activity_filled$steps[nas] <- activity_filled$steps.by_interval[nas]
activity_filled <- activity_filled[, c(1:3)]
steps_by_day <- aggregate(steps ~ date, data = activity_filled, FUN = sum)

hist(steps_by_day$steps,breaks=20,labels=unique(steps_by_day$steps[order(steps_by_day$steps)]),main="Histogram of steps by day after Imputing",xlab="Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png) 

```r
mean(steps_by_day$steps)
```

```
## [1] 10766.19
```

```r
median(steps_by_day$steps)
```

```
## [1] 10766.19
```
There does not appear to be the significant difference in te mean and median values. However, the histogram is showing that maxium frequency and distribution of data is different.

## 5. Are there differences in activity patterns between weekdays and weekends?

```r
# Get day from date
activity_filled$day <- weekdays(as.Date(activity_filled$date))
activity_filled$daytype <- factor(activity_filled$day)
levels(activity_filled$daytype) <- list(
  weekday = c("Monday","Tuesday","Wednesday","Thursday","Friday"),
  weekend = c("Saturday","Sunday")
)

avg_steps_weekdays_vs_weekends <- aggregate(steps ~ daytype + interval, activity_filled, mean)

xyplot(
  type = "l",
  data = avg_steps_weekdays_vs_weekends,
  steps ~ interval | daytype,
  xlab = "Interval",
  ylab = "Number of steps",
  layout = c(1,2)
)
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png) 

There are differences in the activity pattern between weekday and weekend.
