# Reproducible Research: Peer Assessment 1


## Introduction
It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a **Fitbit**, **Nike Fuelband**, or **Jawbone Up**. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

## The Data

The data for this assignment can be downloaded from the course web site:

Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

The variables included in this dataset are:

**steps**: Number of steps taking in a 5-minute interval (missing values are coded as NA)

**date**: The date on which the measurement was taken in YYYY-MM-DD format

**interval**: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (*CSV*) file and there are a total of 17,568 observations in this dataset.


## Loading and preprocessing the data

First step we will load the data, and transform it according to its definition. We will convert *date* into a DateTime value.
To make it more efficient, we will only unzip the file if it has not yet beem unzipped before.

```r
#unzip, if we need to
if (!file.exists('activity.csv')) {
  unzip('activity.zip')   # Unzip file
}
```

Now, we can read adn convert date into DateTime.



```r
#read file
data<-read.csv('activity.csv')
data$dateTime<-as.Date(data$date, "%Y-%m-%d")
```

## What is mean total number of steps taken per day?

First we will calculate the number of steps taken per day, and store into ```summary_data```



```r
library(plyr)
summary_data<-ddply(data,~date,summarise,total=sum(steps))
```

Once we have gropued by date and counted the steps, we can render the histogram with ```hist```


```r
#render the histogram
hist(summary_data$total, main="Steps per day", xlab="Number of steps", breaks=10)
```

![](PA1_template_files/figure-html/histogram-1.png) 

Finally, let's calulate median, and mean


```r
#calculate and output mean/median
steps_mean<-mean(summary_data$total, na.rm=TRUE)
steps_mean
```

```
## [1] 10766.19
```

```r
steps_median<-median(summary_data$total, na.rm=TRUE)
steps_median
```

```
## [1] 10765
```

## What is the average daily activity pattern?

To study the daily activity pattern, we first will summarize by interval, and the represent the number of steps taken, averaged across all days.

We will represent the time series using plot.


```r
#calcualte the mena
data_interval<-ddply(data,~interval,summarise,mean=mean(steps,na.rm = TRUE))
#plto the intervals mean
plot( mean ~ interval, data_interval, type = "l", main="Daily pattern")
```

![](PA1_template_files/figure-html/daily_pattern-1.png) 

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
#get maximum
max<-data_interval$interval[data_interval$mean==max(data_interval$mean)]
max
```

```
## [1] 835
```


## Imputing missing values

Let's calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
#findout total NA
numnas<-sum(is.na(data$steps))
numnas
```

```
## [1] 2304
```
For missing values, we will need to fill them using  defines specific strategy. The strategy we will apply, will be to assign the mean value of that 5-minutes interval, for missing values.
As we already have the means by interval calculated, we can use that data.

```data2``` will hold teh new dataset with NA repalced by the mean of the interval.


```r
#impute missing values as mean of the interval
data2<-data
data2$steps[is.na(data2$steps)]<-data_interval$mean[is.na(data2$steps)]
data2$steps<-as.numeric(data2$steps)
```

Now, let's make an histogram again with the fixed data, so we can compare with the initial one. 
We will also re-calculate the median and mean, so we can compare.


```r
#calculate total
summary_data2<-ddply(data2,~date,summarise,total=sum(steps))
#render the histogram
hist(summary_data2$total, main="Steps per day", xlab="Number of steps", breaks=10)
```

![](PA1_template_files/figure-html/histogram_2-1.png) 

Finally, let's calulate median, and mean


```r
#calcualte mean/median and output
steps_mean2<-mean(summary_data2$total, na.rm=TRUE)
steps_mean2
```

```
## [1] 10766.19
```

```r
steps_median2<-median(summary_data2$total, na.rm=TRUE)
steps_median2
```

```
## [1] 10765.59
```

What is the impact of imputing missing data on the estimates of the total daily number of steps?
The mean of daily steps isthe same in both cases. However, the median is different. 

Due to the way we have imputed missing values, that behabiour is expected, as we are filling missing values iwth the mean for that itnerval, so it should not change the mean, but will affect to median.


## Are there differences in activity patterns between weekdays and weekends?


Now, we will enhabce the dataset with a new factor, with 2 levels( “weekday” and “weekend” ), indicating if the days is a week day or a weekend day.

```r
data2$weekday<-factor(weekdays(as.Date(data2$date, "%Y-%m-%d")) %in% c("Saturday", "Sunday"),labels=c("weekday","weekend"))
```

Finally, to compare the differences between weekday and weekend days, we will plot both data, head to head, so we can compare the steps on both type of  days:


```r
#let's split intos weekday and weekend
data2_weekday <- data2[data2$weekday == "weekday",]
data2_weekend <- data2[data2$weekday == "weekend",]

#calculate means
data_interval_weekday<-ddply(data2_weekday,~interval,summarise,mean=mean(steps,na.rm = TRUE))
data_interval_weekend<-ddply(data2_weekend,~interval,summarise,mean=mean(steps,na.rm = TRUE))


#Let's render both on teh same graphic
par(mfrow=c(2,1))
plot( mean ~ interval, data_interval_weekend, type = "l", main="Weekend")
plot( mean ~ interval, data_interval_weekday, type = "l", main="Weekday")
```

![](PA1_template_files/figure-html/h2h-1.png) 
