---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---
# Reproducible Research: Peer Assessment 1  

## Explore the activity data  


## Loading and preprocessing the data
### 1. Load the data
First we need to load the data. We will use dplyr packages to help manipulate the data

```r
# Import
library(dplyr)

# Convert to dplyr data frame
activity = tbl_df(read.csv("activity.csv"))
```

We will look at the structure and some summary about data

```r
# Structure
str(activity)
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
# Summary
summary(activity)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

### 2. Process/transform the data
`date` variable is now in factor. We should convert it into date

```r
# Convert date to factor
activity$date = as.Date(activity$date, format = "%Y-%m-%d")
```


## What is mean total number of steps taken per day?
### 1. Total of steps taken per day
In this section, we will examine the data of steps per day. First we group the data by day

```r
# Group by date
by_date = group_by(activity, date)
```

The total number of step taken per day is

```r
# Calculate the total number of steps per day
total_steps_per_day = summarise(by_date, total_steps = sum(steps, na.rm=TRUE))
```

### 2. Histogram of the total number of steps taken each day
We'll plot a histogram of total number of steps taken per day

```r
# Plot histogram
hist(total_steps_per_day$total_steps, col="blue", xlab="Total steps", main="Histogram of number of steps taken per day")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

### 3. Mean and median of total number of steps taken
The mean and the median of total number of steps taken per day is:

```r
# Summary of total steps
summary(total_steps_per_day$total_steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       0    6778   10400    9354   12810   21190
```
Mean: `9354`  
Median: `10400`

## What is the average daily activity pattern?
### 1. Time series plot: average number of steps and 5-minute intervals
Now we will examine the data based on the interval. First we create the data group by interval.

```r
# Group by interval
by_interval = group_by(activity, interval)

# Caculate average steps per interval
avg_steps_per_interval = summarise(by_interval, avg_steps = mean(steps, na.rm=TRUE))
```

We make a time series plot of 5-minute interval and the average number of steps taken (across all days)

```r
# Time series plot
plot(avg_steps_per_interval, type="l", xlab="Interval", ylab="Average Steps", main="Average steps taken per 5-minute interval")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png) 

### 2. Maximum average number of steps of 5-minute interval 
The 5-minute interval, on average across all days in the dataset, contains the maximum number of steps is:

```r
# Max average steps
avg_steps_per_interval[which.max(avg_steps_per_interval$avg_steps),]
```

```
## Source: local data frame [1 x 2]
## 
##     interval avg_steps
## 104      835  206.1698
```
Maximum values is `206.1698`


## Imputing missing values
### 1. Total missing values in dataset
In this section, we'll deal with the missing value in the dataset. First we show how many rows in the data have missing values:

```r
# Summary
summary(activity)
```

```
##      steps             date               interval     
##  Min.   :  0.00   Min.   :2012-10-01   Min.   :   0.0  
##  1st Qu.:  0.00   1st Qu.:2012-10-16   1st Qu.: 588.8  
##  Median :  0.00   Median :2012-10-31   Median :1177.5  
##  Mean   : 37.38   Mean   :2012-10-31   Mean   :1177.5  
##  3rd Qu.: 12.00   3rd Qu.:2012-11-15   3rd Qu.:1766.2  
##  Max.   :806.00   Max.   :2012-11-30   Max.   :2355.0  
##  NA's   :2304
```
We see that we have `2034` rows have missing number of steps

### 2. Filling strategy
We will fill those missing value by the median for that 5-minute interval.
First we create a dataset of median steps of each 5-minute inteval

```r
# meadian steps per interval
median_steps_interval = summarise(by_interval, median_steps = median(steps, na.rm=TRUE))
```

### 3. Create a new dataset
We will use the information above to fill the missing values in the original dataset

```r
# First find all the 5-minute interval that has missing data
missing_interval = activity[is.na(activity$steps), "interval"]

# Then find all the index associate with that interval in the  median_step_interval we just created. 
# We will use these indexes to look up the median values
indexes = match(missing_interval, median_steps_interval$interval)

# First create a new copy dataset to reserve the original
activity_filled = activity

# Look up the value in the median_steps_interval dataset to fill the missing value
activity_filled$steps[is.na(activity_filled$steps)] = median_steps_interval[indexes, "median_steps"]
```

Now we compare the original dataset with the new one. Summary of the orginal one

```r
# Original data set
summary(activity)
```

```
##      steps             date               interval     
##  Min.   :  0.00   Min.   :2012-10-01   Min.   :   0.0  
##  1st Qu.:  0.00   1st Qu.:2012-10-16   1st Qu.: 588.8  
##  Median :  0.00   Median :2012-10-31   Median :1177.5  
##  Mean   : 37.38   Mean   :2012-10-31   Mean   :1177.5  
##  3rd Qu.: 12.00   3rd Qu.:2012-11-15   3rd Qu.:1766.2  
##  Max.   :806.00   Max.   :2012-11-30   Max.   :2355.0  
##  NA's   :2304
```
And the new one

```r
# New data set
summary(activity_filled)
```

```
##      steps          date               interval     
##  Min.   :  0   Min.   :2012-10-01   Min.   :   0.0  
##  1st Qu.:  0   1st Qu.:2012-10-16   1st Qu.: 588.8  
##  Median :  0   Median :2012-10-31   Median :1177.5  
##  Mean   : 33   Mean   :2012-10-31   Mean   :1177.5  
##  3rd Qu.:  8   3rd Qu.:2012-11-15   3rd Qu.:1766.2  
##  Max.   :806   Max.   :2012-11-30   Max.   :2355.0
```
We see the steps missing data now is filled. The mean is lower in the new dataset since we introduced more zero.

### 4. Compare old and new dataset
Now, we will calculate the steps taken each day for the new dataset

```r
# Group by date
by_date_filled = group_by(activity_filled, date)

# Calculate the totals per day
total_steps_per_day_filled = summarise(by_date_filled, total_steps = sum(steps))
```

We make the histogram of steps taken per day with the filled dataset

```r
# Plot histogram
hist(total_steps_per_day_filled$total_steps, col="blue", xlab="Total Steps", main="Histogram of number of steps taken per day")
```

![plot of chunk unnamed-chunk-17](figure/unnamed-chunk-17-1.png) 

The mean and the median of the original total steps taken per day

```r
# Original dataset
summary(total_steps_per_day$total_steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       0    6778   10400    9354   12810   21190
```
Mean: `9354`  
Median: `10400`

The mean and the median of the filled total steps taken per day

```r
# Filled missing values dataset
summary(total_steps_per_day_filled$total_steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    6778   10400    9504   12810   21190
```
Mean: `9504`  
Median: `10400`

Compare to the original total steps data, these new values does not differ much. The median for both data set is the same. The mean of the new data set is larger than the original but not much. And as we can see, the two histograms is almost identical.
Therefore, imputing missing data does not have much impact on the estimates of the total daily number of steps.

## Are there differences in activity patterns between weekdays and weekends?

### 1. Create date type variable
In this last section, we will examine the differences in activity patterns between weekdays and weekends

Create new variable to indicate weekday and weekends

```r
# First set the locale in case of any timezone differences problems
Sys.setlocale("LC_ALL", "C")
```

```r
# Create new variable
activity_filled$date_type = weekdays(activity_filled$date)

# Weekdays and weekends vector
weekdays_vec = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
weekends_vec = c("Saturday", "Sunday")

# Change the date_type variable to the two new factors
activity_filled$date_type[activity_filled$date_type %in% weekdays_vec] = "weekday"
activity_filled$date_type[activity_filled$date_type %in% weekends_vec] = "weekend"

activity_filled$date_type = as.factor(activity_filled$date_type)
```

We show the structure of the new dataset and summary information

```r
# Show structure 
str(activity_filled)
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	17568 obs. of  4 variables:
##  $ steps    : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ date     : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval : int  0 5 10 15 20 25 30 35 40 45 ...
##  $ date_type: Factor w/ 2 levels "weekday","weekend": 1 1 1 1 1 1 1 1 1 1 ...
```

```r
# And summary of the new dataset
summary(activity_filled)
```

```
##      steps          date               interval        date_type    
##  Min.   :  0   Min.   :2012-10-01   Min.   :   0.0   weekday:12960  
##  1st Qu.:  0   1st Qu.:2012-10-16   1st Qu.: 588.8   weekend: 4608  
##  Median :  0   Median :2012-10-31   Median :1177.5                  
##  Mean   : 33   Mean   :2012-10-31   Mean   :1177.5                  
##  3rd Qu.:  8   3rd Qu.:2012-11-15   3rd Qu.:1766.2                  
##  Max.   :806   Max.   :2012-11-30   Max.   :2355.0
```

### 2. Plot to compare between weekdays and weekends
Next we want to plot an average number of steps taken of the 5-minute intervals across all weekdays and weekends.

We will prepare a dataset to support the plot.

```r
# Create dataset that support the plot using group_by
by_interval_date_type = group_by(activity_filled, interval, date_type)

# Dataset
avg_steps_by_interval_date_type = summarise(by_interval_date_type, avg_steps = mean(steps))
```

We'll use lattice plot to plot


```r
# Import
library(lattice)

# Plot
xyplot(avg_steps ~ interval | date_type, data=avg_steps_by_interval_date_type, type="l", ylab="Average Steps", main="Average number of steps based on interval for weekdays and weekends", layout=c(1,2))
```

![plot of chunk unnamed-chunk-24](figure/unnamed-chunk-24-1.png) 

We can see the some interesting things at two plot. The plots show us that on weekdays people usually very active around from 8:00 to 9:00. But then become less active throughout the day. It indicates that people may go to work from 8:00 to 9:00 then sit a lot while working.

On the other hand, people tend to get active throughout weekends. It show that people may go out during weekends or do not sit much.
