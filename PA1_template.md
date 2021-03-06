# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
* Set handy options so that numbers are formatted nicely
* Unzip and load the activity data
* Convert dates and times to more useful formats

```r
options(scipen = 1, digits = 2)
unzip("activity.zip")
activity.data <- read.csv("activity.csv")
library(lubridate)
activity.data$date <- ymd(activity.data$date)
activity.data$interval <- parse_date_time(sprintf("%04d", activity.data$interval), "H!M!")
```


## What is mean total number of steps taken per day?
* Explore the total number of steps each day as a histogram, then calculate the mean and median steps per day

```r
data.daily <- aggregate(steps ~ date, data = activity.data, FUN = sum)
hist(data.daily$steps, main="Histogram of steps per day", xlab="steps")
```

![](PA1_template_files/figure-html/meandailysteps-1.png) 

```r
mean.daily.steps <- mean(data.daily$steps)
median.daily.steps <- median(data.daily$steps)
```
* The average number of steps taken per day was: 10766.19
* The median number of steps taken per day was: 10765

## What is the average daily activity pattern?
* Explore the average pattern of steps taken at the same time each day

```r
data.perinterval <- aggregate(steps ~ interval, data = activity.data, FUN = mean)
plot(data.perinterval, type="l",
     main="Average steps per 5 minute interval each day", 
     xlab="5 minute interval")
```

![](PA1_template_files/figure-html/meanactivitypattern-1.png) 

```r
max.interval <- data.perinterval$interval[which.max(data.perinterval$steps)]
```

* The 5 minute interval with the maximum average number of steps was at 08:35


## Imputing missing values
* Explore the extent to which missing values have affected our results.
* First, how many missing values are there?

```r
num.missing <- sum(is.na(activity.data$steps))
num.total <- nrow(activity.data)
```
* There are 2304 missing values out of 17568 observation intervals.

* Next, impute missing values for our data. 
* Use the mean values for each 5 minute interval of a day that we calculated earlier as an estimate of what any missing value would have been.
* Approach for this: split data by day, replace missing values with means for that interval, then recombine the data.
* Check the histogram for the new data set and calculate our mean and median again.


```r
activity.splitbyday <- split(activity.data, factor(activity.data$date))
for(x in seq(along=activity.splitbyday)) {
        activity.splitbyday[[x]]$steps[is.na(activity.splitbyday[[x]]$steps)] <- 
                data.perinterval$steps[is.na(activity.splitbyday[[x]]$steps)]
}
activity.imputed <- unsplit(activity.splitbyday, factor(activity.data$date))
imputed.daily <- aggregate(steps ~ date, data = activity.imputed, FUN = sum)
hist(imputed.daily$steps, main="Histogram of steps per day (imputed)", xlab="steps")
```

![](PA1_template_files/figure-html/imputemissingvalues-1.png) 

```r
mean.imputed.steps <- mean(imputed.daily$steps)
median.imputed.steps <- median(imputed.daily$steps)
```
* The new average number of steps taken per day was: 10766.19
* The new median number of steps taken per day was: 10766.19

* Note that in the new data set with imputed values, the average number of daily steps is exactly the same as for the original data set and the median now matches the average exactly (and remains very close to the old median value). So our strategy for imputing missing values has not made much differenc to our estimates for the centre of the data set. The new histogram shows that the central frequency bucket has around 8 more values in it while the other frequency counts remain the same. 

_A more detailed explanation of this result based on analysis not presented as part of the assignment:_ 
This result is because our strategy was to estimate missing values with the daily averages __combined with the fact that__ when data is missing, it is always missing for the entire day. This means the new daily average was simply including more results exactly matching the old daily average. 
Another side-effect of this is that the median of the new data set now matches the daily average exactly. The original median and mean were very close. The new data set injected 8 additional days with exactly the average number of steps, all close to the old median. The new median turned out to be one of these new days.

## Are there differences in activity patterns between weekdays and weekends?
* Add a factor to classify data gathered on weekdays vs weekends (go back to the original data set, because our imputed data was based on all days of the week, so could hide some of the differences we want to look for).
* Calculate new averages for the number of steps in a given interval of the day, this time keeping weekdays and weekends separate.
* Plot these results on directly comparable time-series plots.


```r
activity.data$daytype <- factor(wday(activity.data$date) %in% c(1,7), 
                                levels=c(TRUE,FALSE), 
                                labels=c("weekend", "weekday"))
data.perinterval.perdaytype <- aggregate(steps ~ interval + daytype, 
                                         data = activity.data, FUN = mean)
par(mfrow=c(2,1), mar=c(3,4,1,1), oma=c(1,0,0,0), cex.main=0.8)
plot(steps ~ interval, data = data.perinterval.perdaytype, type="l", 
     subset = daytype == "weekend", main="weekend", xlab="")
plot(steps ~ interval, data = data.perinterval.perdaytype, type="l", 
     subset = daytype == "weekday", main="weekday", xlab="")
mtext("5 minute interval", side = 1, outer = TRUE)
```

![](PA1_template_files/figure-html/weekendweekdaypatterns-1.png) 

