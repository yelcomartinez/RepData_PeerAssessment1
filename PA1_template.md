---
title: "Reproducible Research: Peer Assessment 1"

output: html_document
---

## Loading and preprocessing the data

```{r cache=TRUE}

#Download data

dir.create("./activity")
urlzip <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(urlzip, destfile = "./activity.zip" )
unzip("./activity.zip", exdir = "./activity" )

#Read Data

dat <- read.csv("./activity/activity.csv") 

#Change data format

dates <- strptime(dat$date, "%Y-%m-%d")
dat$date <- dates

#Show unique dates and intervals

uniqueDates <- unique(dates)
uniqueIntervals <- unique(dat$interval)
```


## What is mean total number of steps taken per day?

```{r cache=TRUE, fig.width=11, fig.height=6}

# Making the data frame: steps by day
stepsSplit <- split(dat$steps, dates$yday)

# Total number of steps by day
totalStepsPerDay <- sapply(stepsSplit, sum, na.rm=TRUE)

# Plot histogram
plot(uniqueDates, totalStepsPerDay, main="Histogram of steps / day", 
     xlab="Date (October to November 2012)", ylab="Frequency", type="h", lwd=4, col="blue")
```

Mean steps per day are:

```{r cache=TRUE}
meanStepsPerDay <- sapply(stepsSplit, mean, na.rm=TRUE)
meanDataFrame <- data.frame(date=uniqueDates, meanStepsPerDay=meanStepsPerDay, row.names=NULL)
meanDataFrame
```

Median steps per day are:
```{r cache=TRUE}
medianStepsPerDay <- sapply(stepsSplit, median, na.rm=TRUE)
medianDataFrame <- data.frame(date=uniqueDates, medianStepsPerDay=medianStepsPerDay, row.names=NULL)
medianDataFrame
```


## What is the average daily activity pattern?

```{r cache=TRUE, fig.width=11, fig.height=6}

# Split up the data depending the interval
intervalSplit <- split(dat$steps, dat$interval)

# Average amount of steps per time interval
averageStepsPerInterval <- sapply(intervalSplit, mean, na.rm=TRUE)

# Plot the time-series graph
plot(uniqueIntervals, averageStepsPerInterval, type="l",
     main="Average number of steps per interval across all days", 
     xlab="Interval", ylab="Average # of steps across all days", 
     lwd=2, col="blue")

# Plot max
maxIntervalDays <- max(averageStepsPerInterval, na.rm=TRUE)
maxIndex <- as.numeric(which(averageStepsPerInterval == maxIntervalDays))
maxInterval <- uniqueIntervals[maxIndex]
abline(v=maxInterval, col="red", lwd=3)

```

Maximum number of steps:

```{r cache=TRUE}
maxInterval
```

## Imputing missing values

```{r cache=TRUE}
#Calculate total amount of missing values in the data set

completeRowsBool <- complete.cases(dat$steps)
numNA <- sum(as.numeric(!completeRowsBool))
numNA
```

Filling in all of the missing values in the dataset

```{r cache=TRUE}
# Remove NaN values and replace with 0.  

meanStepsPerDay[is.nan(meanStepsPerDay)] <- 0

# Create a replicated vector 288 times

meanColumn <- rep(meanStepsPerDay, 288)

# Steps before replacement

rawSteps <- dat$steps

# NA values in the raw steps data

stepsNA <- is.na(rawSteps)

# Replacing NA values with their mean

rawSteps[stepsNA] <- meanColumn[stepsNA]

datNew <- dat
datNew$steps <- rawSteps
```

Histogram of the new data:

```{r cache=TRUE, fig.width=11, fig.height=12}

#Repeat the process

stepsSplitNew <- split(datNew$steps, dates$yday)

totalStepsPerDayNew <- sapply(stepsSplitNew, sum)

par(mfcol=c(2,1))

# Original histogram first

plot(uniqueDates, totalStepsPerDay, main="Histogram of steps taken each day before imputing", 
     xlab="Date (October to November 2012)", ylab="Frequency", type="h", lwd=4, col="blue")

# Modified histogram after

plot(uniqueDates, totalStepsPerDayNew, main="Histogram of steps taken each day after imputing", 
     xlab="Date (October to November 2012)", ylab="Frequency", type="h", lwd=4, col="blue")
```

The mean steps per day of the new data are:

```{r cache=TRUE}
meanStepsPerDayNew <- sapply(stepsSplitNew, mean)
meanDataFrameNew <- data.frame(date=uniqueDates, meanStepsPerDay=meanStepsPerDay, 
                               meanStepsPerDayNew=meanStepsPerDayNew, row.names=NULL)
meanDataFrameNew
```

The median steps per day are:
```{r cache=TRUE}
medianStepsPerDayNew <- sapply(stepsSplitNew, median)
medianDataFrameNew <- data.frame(date=uniqueDates, medianStepsPerDay=medianStepsPerDay, 
                                 medianStepsPerDayNew=medianStepsPerDayNew, row.names=NULL)
medianDataFrameNew
```

Only have changed are those days where all of the observations were missing.

## Are there differences in activity patterns between weekdays and weekends?

```{r cache=TRUE}
# Split up the data between sorted by weekday or weekend
# wday is an integer from 0 to 6 depending the week

wdays <- dates$wday

# Sort out the day as either a weekday or weekend

classifywday <- rep(0, 17568) # 17568 observations overall

classifywday[wdays >= 1 & wdays <= 5] <- 1
classifywday[wdays == 6 | wdays == 0] <- 2


daysFactor <- factor(classifywday, levels=c(1,2), labels=c("Weekdays", "Weekends"))
datNew$typeOfDay <- daysFactor

# Making a dataset for week days and other for weekend days

datWeekdays <- datNew[datNew$typeOfDay == "Weekdays", ]
datWeekends <- datNew[datNew$typeOfDay == "Weekends", ]
```


```{r cache=TRUE, fig.width=11, fig.height=12}

# Split up the Weekdays and Weekends into intervals

datSplitWeekdays <- split(datWeekdays$steps, datWeekdays$interval)
datSplitWeekends <- split(datWeekends$steps, datWeekends$interval)

# Average per interval

meanStepsPerWeekdayInterval <- sapply(datSplitWeekdays, mean)
meanStepsPerWeekendInterval <- sapply(datSplitWeekends, mean)
par(mfcol=c(2,1))

#Weekdays

plot(uniqueIntervals, meanStepsPerWeekdayInterval, type="l",
     main="Average number of steps per interval across all weekdays", 
     xlab="Interval", ylab="Average # of steps across all weekdays", 
     lwd=2, col="blue")

#Weekends

plot(uniqueIntervals, meanStepsPerWeekendInterval, type="l",
     main="Average number of steps per interval across all weekends", 
     xlab="Interval", ylab="Average # of steps across all weekends", 
     lwd=2, col="blue")
```