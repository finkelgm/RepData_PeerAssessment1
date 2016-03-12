# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
To prepricess the data I converted date to date format


```r
activity <- read.csv("activity.csv", stringsAsFactors=FALSE)
activity$date<-as.Date(activity$date)
```


## What is mean total number of steps taken per day?
- Calculate the total number of steps taken per day (daysteps data frame)

```r
library(dplyr,warn.conflicts = FALSE)
daysteps<- activity %>% filter(!is.na(steps)) %>% group_by(date) %>% summarize(sumsteps=sum(steps))
```

- Make a histogram of the total number of steps taken each day


```r
hist(daysteps$sumsteps)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)

- Calculate and report the mean and median of the total number of steps taken per day

```r
mean(daysteps$sumsteps)
```

```
## [1] 10766.19
```

```r
median(daysteps$sumsteps)
```

```
## [1] 10765
```


## What is the average daily activity pattern?
Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
intsteps<- activity %>% filter(!is.na(steps)) %>% group_by(interval) %>% summarize(meansteps=mean(steps))
plot(x=intsteps$interval,y=intsteps$meansteps,type = "l")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)

```r
#max interval
intsteps$interval[which.max(intsteps$meansteps)]
```

```
## [1] 835
```


## Imputing missing values
Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
nrow(activity)-nrow(na.omit(activity))
```

```
## [1] 2304
```

Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. I use the mean for that 5-minute interval.
Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
act<-activity
act<-right_join(act,intsteps)
```

```
## Joining by: "interval"
```

```r
act$steps<-ifelse(is.na(act$steps),act$meansteps,act$steps)
```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? 

```r
daysteps<- act %>%  group_by(date) %>% summarize(sumsteps=sum(steps))
hist(daysteps$sumsteps)
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)

```r
mean(daysteps$sumsteps)
```

```
## [1] 10766.19
```

```r
median(daysteps$sumsteps)
```

```
## [1] 10766.19
```
What is the impact of imputing missing data on the estimates of the total daily number of steps?
Median now is equal to mean

## Are there differences in activity patterns between weekdays and weekends?
Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
act$weekday<-factor(ifelse((as.POSIXlt(act$date)$wday==0) | (as.POSIXlt(act$date)$wday==6), "weekend","weekday"))
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
intsteps<- act %>% group_by(interval,weekday) %>% summarize(meansteps=mean(steps))
library(ggplot2,warn.conflicts = FALSE)
ggplot(data=intsteps,aes(x=interval,y=meansteps))+geom_line()+facet_grid(weekday~.)
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)

