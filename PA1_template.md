# Reproducible Research: Peer Assessment 1

#Reproducible Research: Course project 1

### Code for reading in the dataset and/or processing the data
I am using data.table rather than data.frame for performance reason. So, there is the fread instead of read.csv

```r
library(data.table)
dt <- fread("../../_data/activity.csv", stringsAsFactors = TRUE)
```

Data with skipped NA rows

```r
dt.wo.NA <- dt[complete.cases(dt),]
```

Make data with summarized steps by day

```r
dt.sum.by.date <- dt[,list(sum=sum(steps, na.rm = TRUE)), by=date]
```

Make data with average by 5-minute interval

```r
dt.avg.by.5min <- dt.wo.NA[,list(mean=mean(steps)), by=interval]
interval.for.max.mean <- dt.avg.by.5min[, .SD[which.max(mean)], ]$interval
```


### Histogram of the total number of steps taken each day

```r
hist(dt.sum.by.date$sum, main = "Total number of steps taken each day", xlab = "number of steps", breaks = 10, col = "cornflowerblue")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->


```r
steps.median <- median(dt.sum.by.date$sum)
steps.mean <- mean(dt.sum.by.date$sum)
```
Mean 9354.2295082 and median 10395 number of steps taken each day

### Time series plot of the average number of steps taken

```r
plot(dt.avg.by.5min, type = "l")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

The 5-minute interval that, on average, contains the maximum number of steps is 835

### Imputing missing values

```r
rows.w.NA <- nrow(dt) - nrow(dt.wo.NA)
```

The total number of rows with NAs is 2304

Create new dataset with NA filled with average 5-minute values

```r
dt.filled.NA <- dt

for (i in 1:nrow(dt)) {
    if(is.na(dt$steps[i])) {
        dt.filled.NA$steps[i] <- dt.avg.by.5min[interval == dt[i]$interval, mean]
    }
}
```

Make data with summarized steps by day

```r
dt.filled.NA.sum.by.date <- dt.filled.NA[,list(sum=sum(steps)), by=date]
```

Histogram of the total number of steps taken each day

```r
hist(dt.filled.NA.sum.by.date$sum, main = "Total number of steps taken each day", xlab = "number of steps", breaks = 10, col = "coral2")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->


```r
steps.filled.NA.median <- median(dt.filled.NA.sum.by.date$sum)
steps.filled.NA.mean <- mean(dt.filled.NA.sum.by.date$sum)
options(scipen=1, digits=2)
```

Mean 10766.19 and median 10766.19 number of steps taken each day

###Weekdays and weekends
Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
dt.w.weekday <- dt[,weekday := as.factor(ifelse(format(as.Date(date), "%u") %in% c(6,7), "weekend", "weekday")),]
```
Make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
dt.avg.by.5min.weekday <- dt.w.weekday[weekday == "weekday",list(mean=mean(steps, na.rm = TRUE)), by=interval]
dt.avg.by.5min.weekend <- dt.w.weekday[weekday == "weekend",list(mean=mean(steps, na.rm = TRUE)), by=interval]

g_range <- range(0, dt.avg.by.5min.weekday$mean, dt.avg.by.5min.weekend$mean)
plot(dt.avg.by.5min.weekday, type = "l", col = "cornflowerblue", ylim = g_range)
lines(dt.avg.by.5min.weekend, type = "l", col = "coral2")
title(main = "Average number of steps taken \n per 5-minute interval across weekdays and weekends")
legend(1, g_range[2], c("weekday", "weekend"), cex = 0.8, col = c("cornflowerblue", "coral2"), lty = 1)
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png)<!-- -->
