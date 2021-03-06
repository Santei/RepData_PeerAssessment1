# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
```
unzip(zipfile="activity.zip")
activity.data <- read.csv("activity.csv", header = TRUE, colClasses = c ("numeric", "Date", "numeric"))
``

## What is mean total number of steps taken per day?
```
total.steps.per.day <- tapply(activity.data$steps, activity.data$date, FUN=sum, na.rm = TRUE)
hist(total.steps.per.day, main = "Number of steps", xlab="Total steps taken per day")
```
### Mean and median of the number of steps taken per day

```
print(mean(total.steps.per.day, na.rm=TRUE))
print(median(total.steps.per.day, na.rm=TRUE))
```

## What is the average daily activity pattern?
```
average.steps <- aggregate(x=list(steps=activity.data$steps), by=list(interval=activity.data$interval),
                             FUN=mean, na.rm=TRUE)
plot(average.steps, type= "l", xlab = "Interval", ylab = "Number of steps")
average.steps[which.max(average.steps$steps),]$interval
```

## Imputing missing values
```
missing.values <- sum(is.na(activity.data$steps))
print(missing.values)
fill.value <- function(steps, interval) {
  filled <- NA
  if (!is.na(steps))
    filled <- c(steps)
  else
    filled <- (average.steps[average.steps$interval==interval, "steps"])
  return(filled)
}
activity.data.filled <- data
activity.data.filled$steps <- mapply(fill.value, activity.data.filled$steps, activity.data.filled$interval)
print(sum(is.na(activity.data.filled$steps)))
total.steps.per.day.filled <- tapply(activity.data.filled$steps, activity.data$date, FUN=sum)
hist(total.steps.per.day.filled, main = "Number of steps", xlab="Total steps taken per day")
print(mean(total.steps.per.day.filled, na.rm=TRUE))
print(median(total.steps.per.day.filled, na.rm=TRUE))
```
## Are there differences in activity patterns between weekdays and weekends?
```
activity.data.filled$weekday = weekdays(as.Date(activity.data.filled$date, format = "%Y-%m-%d"))
activity.data.filled$weekday.type = factor(ifelse(activity.data.filled$weekday == "Sunday" | activity.data.filled$weekday == 
                                                    "Saturday", "weekend", "weekday"), levels = c("weekday", "weekend"))

activity.data.filled.weekdays = activity.data.filled[activity.data.filled$weekday.type == "weekday", ]
activity.data.filled.weekend = activity.data.filled[activity.data.filled$weekday.type == "weekend", ]
average.weekdays.steps = tapply(activity.data.filled.weekdays$steps, activity.data.filled.weekdays$interval, FUN=mean)
average.weekend.steps = tapply(activity.data.filled.weekend$steps, activity.data.filled.weekend$interval, FUN=mean)
```
### Generate the panel plot
```
MakePlot = function() {
  par(mfrow = c(2, 1), mar = c(4, 5, 2, 2))
  plot(average.weekend.steps, type = "l", yaxt = "n", ylim = c(0, 
                                                                        250), xlim = c(0, 300), main = "weekend", ylab = "", xlab = "Interval")
  axis(side = 4, at = seq(0, 250, 50), labels = seq(0, 250, 50))
  plot(average.weekdays.steps, type = "l", ylim = c(0, 250), xlim = c(0, 
                                                                               300), main = "weekday", ylab = "", xlab = "Interval")
  par(mfrow = c(1, 1), mar = c(3, 2, 2, 2))
  mtext(text = "Number of steps", side = 2)
  par(mar = c(5, 5, 5, 2))
}

MakePlot()
```
