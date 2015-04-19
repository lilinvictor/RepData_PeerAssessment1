# Reproducible Research: Peer Assessment 1

## Loading and preprocessing the data

Make sure the data ZIP file "activity.zip" is put under current working directory. The code will unzip the compressed file and load "activity.csv" to data object.


```r
unzip("activity.zip")
data <- read.csv("activity.csv")
```

Here is the first couple of rows of loaded activity data:


```r
head(data)
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

## What is mean total number of steps taken per day?

Calculate total steps per day and present the histogram:


```r
totalSteps <- aggregate(steps ~ date, data, sum)
hist(totalSteps$steps,
	 main = "Histogram of total steps per day",
	 xlab = "Total steps taken each day",
	 breaks = 20)
```

![](PA1_template_files/figure-html/mean_steps-1.png) 

And here are the mean and median of total number of steps taken per day:

- **Mean** = 1.0766189\times 10^{4}
- **Median** = 10765

## What is the average daily activity pattern?

Below is the time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis):


```r
avgSteps <- aggregate(steps ~ interval, data, mean)
plot(avgSteps,
	 type = "l",
	 main = "Average activity across all days",
	 xlab = "5 minute interval",
	 ylab = "Averaged steps")
```

![](PA1_template_files/figure-html/avarage_activity-1.png) 

And the following 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps:


```r
avgSteps[order(avgSteps$steps, decreasing = TRUE),][1,]
```

```
##     interval    steps
## 104      835 206.1698
```

## Imputing missing values

Here is the total number of missing values in the dataset (i.e. the total number of rows with NAs):


```r
nrow(data[is.na(data$steps),])
```

```
## [1] 2304
```

To fill in these missing values, we choose the mean value for that interval across all days. 

Here is the code to create the new dataset which is equal to the original dataset but with the missing data filled in:


```r
fixedData <- data
fixedData$steps[is.na(fixedData$steps)] <- avgSteps$steps[is.na(fixedData$steps)]
```

Now we re-calculate the total steps per day and present the updated histogram:


```r
totalStepsFixed <- aggregate(steps ~ date, fixedData, sum)
hist(totalStepsFixed$steps,
	 main = "Histogram of total steps per day (with fixed dataset)",
	 xlab = "Total steps taken each day",
	 breaks = 20)
```

![](PA1_template_files/figure-html/mean_steps_fixed-1.png) 

And here are the updated mean and median of total number of steps taken per day:

- **Mean** = 1.0766189\times 10^{4}
- **Median** = 1.0765594\times 10^{4}

Comparing with previous reports for original dataset, there is no obvious difference: mean is same, and median is changed with 0.006%.So the impact of imputing missing data on the estimates of the total daily number of steps is quite tiny here.


## Are there differences in activity patterns between weekdays and weekends?

Here is comparison of patterns between weekdays and weekends for avaraged activities of intervals across all days:


```r
# Add factor for weekday vs weekend
fixedData$weekday <- as.factor(ifelse(weekdays(as.POSIXct(fixedData$date)) %in% c("Saturday","Sunday"), "Weekend", "Weekday"))

# Aggregate by weekday
avgSteps <- aggregate(steps ~ interval + weekday, fixedData, mean)

# Draw panels to compare average activities
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.1.3
```

```r
qplot(x = interval,
	  y = steps,
	  data = avgSteps,
	  geom = "line",
	  facets = weekday ~ .,
	  color = weekday,
	  main = "Compare average activities between weekday and weekend",
	  xlab = "5-minute interval",
	  ylab = "Averaged number of steps")
```

![](PA1_template_files/figure-html/weekday_comparison-1.png) 
