# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

Load csv file into data frame.


```r
activity <- read.csv("activity.csv")
```


## What is mean total number of steps taken per day?

We use tapply to group the data frame by date and calculate the sum for each to get the total number of steps per day.



```r
steps_per_day <- tapply(activity$steps, activity$date, sum)
hist(steps_per_day, breaks = 8)
```

![plot of chunk unnamed-chunk-2](./PA1_template_files/figure-html/unnamed-chunk-2.png) 

With stripping NA values let's calculate mean and median here:


```r
mean(steps_per_day, na.rm=TRUE)
```

```
## [1] 10766
```

```r
median(steps_per_day, na.rm=TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

We group activity data by interval and calculate the mean stripping NA values.


```r
steps_per_interval <- tapply(activity$steps, activity$interval, function(x) mean(x, na.rm = TRUE))
```


For nicer display let's create a vector containing the hour of day based on the interval. We divide by 12 since it's 12 intervals per hour.


```r
intervals_per_hour <- 12
hour <- 1:288 / intervals_per_hour

plot(hour, steps_per_interval, type = 'l')
```

![plot of chunk unnamed-chunk-5](./PA1_template_files/figure-html/unnamed-chunk-5.png) 

Which interval contains the maximum number of steps?


```r
which.max(steps_per_interval)
```

```
## 835 
## 104
```


## Imputing missing values

We count the number of cases that are not complete by summing a logical vector of complete cases:


```r
sum(!complete.cases(activity))
```

```
## [1] 2304
```

NAs only occur in the steps column. To fix that we take the vector with mean steps per interval and replace the whole column using the `ifelse` function which selects the mean for each NA value and the actual value otherwise. Luckily enough the `ifelse` function recycles the mean vector so it will cycler over the days.


```r
complete_activity <- data.frame(activity)
complete_activity$steps <- ifelse(is.na(activity$steps), steps_per_interval, activity$steps)
```

Let's see how this affects the histogram:


```r
complete_steps_per_day <- tapply(complete_activity$steps, complete_activity$date, sum)
hist(complete_steps_per_day, breaks = 8)
```

![plot of chunk unnamed-chunk-9](./PA1_template_files/figure-html/unnamed-chunk-9.png) 

Apparently the median bucket is now more emphasized whcih is not a surprise because we have more days with the mean number of steps since we created them artificially.

What happens with the mean and median?


```r
mean(complete_steps_per_day)
```

```
## [1] 10766
```

```r
median(complete_steps_per_day)
```

```
## [1] 10766
```

Barely any change which is not a surprise again since generating average days will not change the average.

## Are there differences in activity patterns between weekdays and weekends?

We first create a vector containing the week day and then use `ifelse` to assign the right label for each row.


```r
wday <- weekdays(as.Date(activity$date))
activity$day <- ifelse(wday == "Saturday" | wday == "Sunday", c("weekend"), c("weekday"))
```

Now we create two new datasets, one for the weekend and one for the weekdays:


```r
weekday <- activity[which(activity$day == "weekday"),]
weekend <- activity[which(activity$day == "weekend"),]
```

For each data set calculate the mean steps per interval.


```r
weekday_steps <- tapply(weekday$steps, weekday$interval, function(x) mean(x, na.rm = TRUE))
weekend_steps <- tapply(weekend$steps, weekend$interval, function(x) mean(x, na.rm = TRUE))
```



```r
par(mfrow=c(2,1), pin=c(4,2))
plot(hour, weekday_steps, type = 'l')
title("Weekday")
plot(hour, weekend_steps, type = 'l')
title("Weekend")
```

![plot of chunk unnamed-chunk-14](./PA1_template_files/figure-html/unnamed-chunk-14.png) 


