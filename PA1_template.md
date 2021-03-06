---
title: "Peer-graded Assignment: Course Project 1"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Loading and preprocessing the data

Unzip file and load data, change "character"" to "Date".


```{r}
setwd("D:/R Coursera/Reproducible Research/Activity Monitoring Data")
unzip("repdata%2Fdata%2Factivity.zip", exdir = "Activity Monitoring Data")
AMD <- read.csv("activity.csv", colClasses = c("date" = "Date" ))

```

Calculate the mean and median value for steps taken each day.

```{r}
stepsDay     <- tapply(AMD$steps, format(AMD$date, '%m-%d'), sum) 
stepsMean    <- mean(stepsDay, na.rm = TRUE)
stepsMedian  <- median(stepsDay, na.rm = TRUE)
```

Here they are:

```{r}
stepsMean
stepsMedian

```

```{r graphic, echo=FALSE}
hist(stepsDay, breaks = c(0,5000,10000,15000,20000,25000,30000), col = "darkcyan", xlab = "Steps per Day", main = "Frequency of Steps per Day")
abline(v = stepsMedian, col = "slateblue4")

```

## What is the average daily activity pattern?

Load dplyr package, create a variable that counts steps for each interval.

```{r load pack, echo=FALSE}
library(dplyr)
```

```{r}
IntervalSteps     <- group_by(AMD, AMD$interval)
SumStepsInterval  <- summarise(IntervalSteps, count = n(), 
                          steps  = mean(steps, na.rm = T))
```

Plot the time series with mean steps per interval

```{r time series}
plot(SumStepsInterval$`AMD$interval`, SumStepsInterval$steps, type = 'l', 
       col =   "darkcyan", main = "Mean Steps per Interval", xlab = "Interval", 
      ylab = "Mean Steps")

abline(h = mean(SumStepsInterval$steps), col = "slateblue4")
```

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```{r}
StepsMax <- SumStepsInterval[which.max(SumStepsInterval$steps), ]$`AMD$interval`
StepsMax
```

##Imputing missing values

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```{r}
na_count <- sum(is.na(AMD))
na_count 
```

Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

Create a new dataset that is equal to the original dataset but with the missing data filled in.

```{r}
AMD2          <- AMD
allNA         <- is.na(AMD2$steps)
stepsmean     <- tapply(AMD2$steps, AMD2$interval, mean, na.rm=TRUE, simplify = TRUE)
AMD2$steps[allNA]   <- stepsmean
sum(is.na(AMD2))
```

Same code as for the first plot.

```{r plot2, echo=FALSE}
stepsDay2     <- tapply(AMD2$steps, format(AMD2$date, '%m-%d'), sum) 
stepsMean2    <- mean(stepsDay2, na.rm = TRUE)
hist(stepsDay2, breaks = c(0,5000,10000,15000,20000,25000,30000), col = "darkcyan", xlab = "Steps per Day", main = "Frequency of Steps per Day")
abline(v = stepsMean2, col = "slateblue4")
```

##Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```{r}
AMD3 <- AMD2%>%
    mutate(typeofday = ifelse(weekdays(AMD2$date) == "Sonntag" | weekdays(AMD2$date)     == "Samstag", "Weekend", "Weekday"))

summary(AMD3)
```

```{r package, echo=FALSE}
library(ggplot2)
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```{r}
IntervalStepsMean <- aggregate(steps ~ interval, data = AMD3, FUN = mean, na.rm = TRUE)
g <- ggplot(AMD3, aes(x = interval , y = steps, color = typeofday)) 
g <- g +  geom_line() +
     labs(title = "Average steps by type of day", 
           x = "Interval", 
           y = "Mean number of Steps") 
g <- g +   facet_wrap( ~ typeofday, ncol = 1, nrow = 2) 
  plot(g)
```