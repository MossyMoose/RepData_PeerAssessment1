---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

This is the assignment for the Coursera student with the GitHub username of *MossyMoose*.

Steps are indicated in Heading 2 format. Questions we are displaying in this document are in Heading 4 format, and subquestions are in Heading 5 format.

## Loading and preprocessing the data

First, we load the data and then process/transfer it into a format suitable for our analysis. We'll assume that the zip file is in our working directory, since it was included in the repository.


```r
unzip("activity.zip")
activity<-read.csv("activity.csv")
```

The resulting data frame has three headings:


```r
names(activity)
```

```
## [1] "steps"    "date"     "interval"
```

As noted in the assignment, the variables included in this dataset are:

`steps`: Number of steps taking in a 5-minute interval (missing values are coded as `NA`)
`date`: The date on which the measurement was taken in YYYY-MM-DD format
`interval`: Identifier for the 5-minute interval in which measurement was taken

We'll convert the date column to a Date format:


```r
activity$date<-as.Date(activity$date, "%Y-%m-%d")
```

`steps` and `interval` are integers.

Let's take a look at the summary information for `activity`.


```r
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

We're all set to proceed with our analysis.

## What is mean total number of steps taken per day?

For this part of the assignment, we are ignoring the missing values in the dataset (i.e., remove NA values).

#### Calculate the total number of steps taken per day

We can summarize the data using the `aggregate()` function.


```r
stepsDay<-aggregate(steps~date, data=activity, FUN=sum, rm.na=TRUE)
```

#### Make a histogram of the total number of steps taken each day


```r
hist(stepsDay$steps, xlab="Steps per Day", main="Frequency of Steps per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->


#### Calculate and report the mean and median of the total number of steps taken per day

To calculate the mean and median, we can use the following code.


```r
mean(stepsDay$steps)
median(stepsDay$steps)
```

The mean number of steps per day is **10767.19** (rounded to 2 decimal places). The median number of steps per day is **10766**.

## What is the average daily activity pattern?

Here we will look at the number of steps each day in 5-minute intervals.

#### Make a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

To create this plot, we'll first use `aggregate()` to calculate the average number of steps for each interval, across all days. Then, we'll construct the plot.


```r
stepsInterval<-aggregate(steps~interval, data=activity, FUN=mean, rm.na=TRUE)
plot(stepsInterval$interval, stepsInterval$steps, type="l", xlab="5-Minute Interval of the Day", ylab="Average Number of Steps", main="Average Daily Activity Pattern")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

#### Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

To answer this question, we'll index the interval variable using the steps value equal to the maximum.


```r
stepsInterval$interval[stepsInterval$steps==max(stepsInterval$steps)]
```

```
## [1] 835
```

Looks like the interval starting at 8:35 AM is the most active time of this data set, with 206.17 steps.

Just for fun (outside the assignment), let's redraw the plot with that time shown by a line.


```r
mostActive<-stepsInterval$interval[stepsInterval$steps==max(stepsInterval$steps)]
plot(stepsInterval$interval, stepsInterval$steps, type="l", xlab="5-Minute Interval of the Day", ylab="Average Number of Steps", main="Average Daily Activity Pattern")
abline(v=mostActive, col="blue")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

## Imputing missing values

As noted in the assignment, there are a number of days/intervals where there are missing values in the data set (coded as NA). They have been retained in the `activity` data set. The presence of missing days may introduce bias into some calculations or summaries of the data. Here, we impute those missing values.

#### Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with `NA`s)

We saw this in the summary statistics earlier. There are 2,304 NAs in the `steps` column.

We can count them again with this piece of code. `is.na()` makes it easy!


```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

#### Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

As suggested in the text, an easy approach to imputing missing values might be to simply find the NAs and fill them in with the mean values from the `stepsIntervals` data set.

Before we do that, though, is there a pattern to the missing data? Let's grab the subset of `activity` that is missing and see what it looks like in a histogram.


```r
activityMissing<-subset(activity, is.na(steps))
hist(activityMissing$interval, xlab="5-Minute Interval", main="Frequency of Missing Steps per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

As we can see from the plot, it's pretty consistent all through the day. We don't have any other variables to suggest a pattern of `NA`s. Therefore, we'll proceed with the plan!

#### Create a new dataset that is equal to the original dataset but with the missing data filled in.

We'll create `activityImputed` using the `stepsInterval` data set to fill in wherever we have missing values in the original data.

To flag and fill in missing values, we'll create a vector `activityNA` indicating where the NAs are located. We'll also make a copy of `activity` and call it `activityImputed` to work with.


```r
activityNA<-is.na(activity$steps)
activityImputed<-activity
```

Now, we will go through the values and fill in the missing ones. There are a number of ways to do this, but for me, it made sense to loop through the values and "look up" the mean number of steps we had already calculated as an average for each interval, stored in `stepsInterval`. We'll use the `activityNA` we just created to find the missing data in the `activityImputed` data set and replace it with the average for that interval, using `match()` to do the lookup. One other step is to round off the results to 0 decimal places, because the original data only had whole numbers for the step count and we don't want the imputed values to stand out.


```r
activityImputed$steps <- ifelse(activityNA,
  stepsInterval$steps[match(activityImputed$interval, stepsInterval$interval)],
  activityImputed$steps)
activityImputed$steps<-round(activityImputed$steps,0)
```

#### Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.

We'll reuse the code from before, using the `activityImputed` data set.


```r
stepsDayImputed<-aggregate(steps~date, data=activityImputed, FUN=sum, rm.na=TRUE)
hist(stepsDayImputed$steps, xlab="Steps per Day", main="Frequency of Steps per Day (with Imputed Data)")
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

```r
mean(stepsDayImputed$steps)
```

```
## [1] 10766.64
```

```r
median(stepsDayImputed$steps)
```

```
## [1] 10763
```

The mean number of steps per day in the imputed data set is 10766.64 (rounded to 2 decimal places) and the median is 1.0763\times 10^{4}.

##### Do these values differ from the estimates from the first part of the assignment?

The values each went down a little bit due to filling in missing values.

As a reminder, we can calculate the mean and median using the following code.


```r
mean(stepsDayImputed$steps)
median(stepsDayImputed$steps)
```

The mean went from 10767.19 to 10766.64, a decrease of 0.549335.

The median went from 10766 to 10763, a decrease of 3.

##### What is the impact of imputing missing data on the estimates of the total daily number of steps?

The total number of steps went up substantially. While the mean and median were weighed down by the imputed values (which were small), the total just adds up all the values.

As a reminder, we can calculate the total steps per day using the following code.


```r
sum(stepsDayImputed$steps)
```

Across all days, the original data set had 570661 recorded steps. The imputed data increased that to 6.56765\times 10^{5} steps, an increase of 8.6104\times 10^{4}.

## Are there differences in activity patterns between weekdays and weekends? Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day. Make a panel plot containing a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

As directed, we'll use the imputed data set to address this question. Using the `weekday()` function, we'll create a new factor variable called `daytype`.


```r
activityImputed$daytype<-ifelse(weekdays(activityImputed$date)=="Saturday" | weekdays(activityImputed$date)=="Sunday", "weekend", "weekday")
activityImputed$daytype<-as.factor(activityImputed$daytype)
```

Next, we calculate the mean number of steps per interval, grouped by weekends and weekdays. Instead of aggregating over just the `interval`, we'll use the `weekend` factor, as well.


```r
stepsIntDay<-aggregate(steps~interval+daytype, data=activityImputed, FUN=mean)
```

Finally, we'll plot them for comparison. This will be a lot easier and look a lot nicer with `ggplot2`, so we'll load that now and construct the plots with `ggplot()`.


```r
library(ggplot2)
ggplot(stepsIntDay, aes(interval, steps)) +
  geom_line() +
  facet_grid(daytype~.) +
  labs(x="5-Minute Interval of the Day",
       y="Average Number of Steps",
       title="Average Daily Activity Pattern")
```

![](PA1_template_files/figure-html/unnamed-chunk-20-1.png)<!-- -->

As we see, during weekdays, there is a spike of activity around 8:35 AM (as we saw in the original data set). That's probably when people are headed into work. There are smaller increases during breaks, after work, and in the evening. People are less activity later in the evening. During the weekends, however it appears that most people start to get up and moving around the time they do on weekdays, but then the activity level is all over the map, depending on what people are doing.
