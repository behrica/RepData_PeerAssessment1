# Reproducible Research: Peer Assessment 1


```r
## numbers >= 10^5 will be denoted in scientific notation, and rounded to 2
## digits
options(scipen = 1, digits = 2)
```



## Loading and preprocessing the data

```r
activity <- read.csv(unz("activity.zip", "activity.csv"))
activity$date <- as.Date(activity$date)
```




## What is mean total number of steps taken per day?
The distribution of the mean total number of steps per day can be seen in the following histogram.


```r
sum.per.day <- sapply(split(activity$steps, f = activity$date), sum)
hist(sum.per.day)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

```r
mean.sum.per.day <- mean(sum.per.day, na.rm = T)
median.sum.per.day <- median(sum.per.day, na.rm = T)
```



The __mean__ total number of steps per day is 10766.19 and the __median__ total number of steps per day is 10765


## What is the average daily activity pattern?

```r
mean.per.interval <- sapply(split(activity$steps, activity$interval), mean, 
    na.rm = T)
plot(mean.per.interval, type = "l", xaxt = "n", xlab = "interval", ylab = "average steps")
labels <- seq(from = 1, to = 288, length.out = 5)
axis(1, labels = names(mean.per.interval)[labels], at = labels)
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 



```r
max.index <- which.max(mean.per.interval)[[1]]
```


The maximum number of steps (on average) is in the interval 835.
This interval has 206.17 steps on average.


## Imputing missing values
The activity data set contains ´r nrow(activity[is.na(activity$steps),])´ rows where data from the number of steps is missing.

We will replace those NAs with the average number of steps for its interval.

```r
steps.imputted <- apply(X = activity[is.na(activity), ], MARGIN = 1, FUN = function(x) {
    
    interval <- as.character(as.integer(x[3]))
    mean.per.interval[interval]
})
activity.imputted <- data.frame(activity)
activity.imputted[is.na(activity), ]$steps <- steps.imputted
```



```r
sum.per.day.imputted <- sapply(split(activity.imputted$steps, f = activity$date), 
    sum)
hist(sum.per.day.imputted)
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 

```r
mean.sum.per.day.imputted <- mean(sum.per.day.imputted)
median.sum.per.day.imputted <- median(sum.per.day.imputted)
```



The __imputted mean__ total number of steps per day is 10766.19 and the __imputted median__ total number of steps per day is 10766.19

The imputing of missing values with daily averages resulted in a slightly more centralized distribution of the data, as expected. Instead of ~26 days where the sum of steps is between 10000 and 15000, we have now ~ 35 days in this range.

The mean and median of the data were not affected by this.


## Are there differences in activity patterns between weekdays and weekends?

```r
activity.imputted$weekday <- factor(weekdays(activity.imputted$date) %in% c("Sunday", 
    "Saturday"), levels = c(TRUE, FALSE), labels = c("weekend", "weekday"))
mean.per.interval.imputted.weekend <- sapply(split(activity.imputted[activity.imputted$weekday == 
    "weekend", ]$steps, activity.imputted[activity.imputted$weekday == "weekend", 
    ]$interval), mean, na.rm = T)
mean.per.interval.imputted.weekday <- sapply(split(activity.imputted[activity.imputted$weekday == 
    "weekday", ]$steps, activity.imputted[activity.imputted$weekday == "weekday", 
    ]$interval), mean, na.rm = T)

par(mfcol = c(2, 1), mar = c(2, 3, 2, 2))
plot(mean.per.interval.imputted.weekend, type = "l", xaxt = "n", xlab = "", 
    ylab = "steps", ylim = c(0, 250), main = "weekend")
labels <- seq(from = 1, to = 288, length.out = 5)
axis(1, labels = names(mean.per.interval.imputted.weekend)[labels], at = labels)

plot(mean.per.interval.imputted.weekday, type = "l", xaxt = "n", xlab = "interval", 
    ylab = "steps", ylim = c(0, 250), main = "weekday")
labels <- seq(from = 1, to = 288, length.out = 5)
axis(1, labels = names(mean.per.interval.imputted.weekday)[labels], at = labels)
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 

The activity pattern is different during weekdays and weekends.

The plots show, that in general, the activity of the person , measured in number of steps, is more homogeneous during the weekend. There is more "continuous activity" during the day.

The weekday patterns shows a prominent high peek activity around interval 800, which is even higher then any peek during then weekend, but then the activity slows down and stays lower then during the weekend.
