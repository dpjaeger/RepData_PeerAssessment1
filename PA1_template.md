# Reproducible Research: Peer Assessment 1

This is my submission for Peer Assignment 1.

Ensure that you have the activity.csv file in the RepData_PeerAssessment1 directory,
which should be in your working directory.

Make sure you are using the .csv version, NOT the .zip version (i.e. unzip the file)

First, we'll  need the lubridate package later, so we will install it before beginning:

```r
install.packages("lubridate", repos="http://cran.rstudio.com/")
```

```
## 
## The downloaded binary packages are in
## 	/var/folders/mk/8jz407lx50j4fhdz6wnr4pbw0000gn/T//Rtmprupvsy/downloaded_packages
```

```r
library(lubridate)
```

## Loading and preprocessing the data
The data is pretty clean, so we can just load without adjustment to get started:

```r
activity <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?
We add a field called "day" and then use the "aggregate" function

```r
activity$day <- as.numeric(activity[ , 2], format='%d')
mean_day <- aggregate(activity$steps, by=list(activity$day), FUN=sum)
```

below are the histograms and mean/medians from our calculations:

```r
hist(mean_day[ ,2], main = "Steps per day", xlab = "Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

```r
mean(mean_day[ ,2], na.rm= TRUE)
```

```
## [1] 10766.19
```

```r
median(mean_day[ ,2], na.rm = TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
we will use the "aggregate" function again but first we need to use remove the NA's:

```r
activity_complete <- activity[complete.cases(activity), ]
mean_interval <- aggregate(activity_complete$steps, by=list(activity_complete$interval), FUN=mean, na.action=na.omit)
```

Here is the plot and the interval with maximum steps.  It is in the morning, around 830am:

```r
plot(mean_interval, type = 'l', main = "steps by interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

```r
mean_interval <- mean_interval[order(-mean_interval[ ,2]), ]
mean_interval[1,]
```

```
##     Group.1        x
## 104     835 206.1698
```
Note that the maximum step interval is above!!

## Inputing missing values
For this one, we will use the average steps for a given interval (from above) to fill this in.  We will use a "for" loop to fill in the data.

```r
interval_values <- unique(mean_interval[ ,1])

activity_fill <- activity

for (x in interval_values) {
  activity_fill[is.na(activity_fill$steps) & activity_fill$interval == x, 1] <-
  mean_interval[mean_interval$Group.1 == x,2]
}

mean_day_fill <- aggregate(activity_fill$steps, by=list(activity_fill$day), FUN=sum)
```

Once again, here's a histogram and mean/medians, now with NAs filled in:

```r
hist(mean_day_fill[ ,2], main = "Steps per day, NAs filled", xlab = "steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png) 

```r
mean(mean_day_fill[ ,2], na.rm= TRUE)
```

```
## [1] 10766.19
```

```r
median(mean_day_fill[ ,2], na.rm = TRUE)
```

```
## [1] 10766.19
```

## Are there differences in activity patterns between weekdays and weekends?
We will create a new field with a we/wd designation:

```r
activity_fill$dow <- wday(activity_fill$date, label = TRUE)
activity_fill$wewd <- "wd"
activity_fill[activity_fill$dow == "Sat" | activity_fill$dow == "Sun", 6] <- "we"
mean_day_wewd <- aggregate(activity_fill$steps, by=list(activity_fill$wewd), FUN=mean,
  na.action = na.omit)
activity_fill_we <- activity_fill[activity_fill$wewd == "we", ]
activity_fill_wd <- activity_fill[activity_fill$wewd == "wd", ]
mean_interval_we <- aggregate(activity_fill_we$steps, by=list(activity_fill_we$interval), FUN=mean, na.action=na.omit)
mean_interval_wd <- aggregate(activity_fill_wd$steps, by=list(activity_fill_wd$interval), FUN=mean, na.action=na.omit)
```

Here are the plots.  As can be seen, there are differences.  The weekend has more activity spread out through the day while the mornig has a spike in the morning and then some mini-spikes, but not near the weekend volume:

```r
par(mfcol = c(1, 2))
plot(mean_interval_we, type = 'l', main = "weekend", xlab = "time", ylab= "steps")
plot(mean_interval_wd, type = 'l', main = "weekday", xlab = "time", ylab= "steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png) 

