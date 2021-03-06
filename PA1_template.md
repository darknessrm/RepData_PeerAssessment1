## Loading and preprocessing the data

library(lattice)
library(knitr)
library(ggplot2)
library(markdown)
library(dplyr)
opts_chunk$set(echo=TRUE, results="hold")
Sys.setlocale("LC_TIME", "English")
dtRAW <- read.csv('activity.csv', header = TRUE, na.strings = "NA", stringsAsFactors=FALSE)
str(dtRAW)
head(dtRAW)
tail(dtRAW)
dtRAW$date <- as.Date(dtRAW$date, "%Y-%m-%d")
str(dtRAW)
head(dtRAW)
tail(dtRAW)

## What is mean total number of steps taken per day?

totalSPD <- aggregate(steps~date, data=dtRAW, FUN=sum, na.rm=TRUE)
str(totalSPD)
head(totalSPD)
tail(totalSPD)
hist(totalSPD$steps, breaks=20, col="grey",
     main="Total number of steps taken per day \n (missing data removed)",
     xlab="Number of steps")
muTSPD <- mean(totalSPD$steps)
meTSPD <- median(totalSPD$steps)

## What is the average daily activity pattern?

muSPI <- aggregate(steps~interval, data=dtRAW, FUN=mean, na.rm=TRUE)
plot(muSPI, type="l",
     main="Average number of steps per 5-minute interval",
     xlab="5-minute intervals",
     ylab="Average numberof steps")
numMaxStep <- muSPI[which(muSPI$steps==max(muSPI$steps)), ]$interval

## Imputing missing values

numNA <- nrow(dtRAW[which(is.na(dtRAW$steps)),])
dtNEW <- transform(dtRAW,
                   steps=ifelse(is.na(dtRAW$steps),
                                muSPI[match(muSPI$interval, dtRAW$interval), ]$steps,
                                dtRAW$steps))
sum(is.na(dtNEW))
str(dtNEW)
head(dtNEW)
tail(dtNEW)
newTSPD <- aggregate(steps~date, data=dtNEW, FUN=sum, na.rm=TRUE)
muNTSPD <- mean(newTSPD$steps)
meNTSPD <- median(newTSPD$steps)
hist(newTSPD$steps, breaks=20, col="grey",
     main="Total number of steps taken per day \n (missing data filled in)",
     xlab="Number of steps")
mumeComp <- data.frame("mean"=c(muTSPD, muNTSPD), "median"=c(meTSPD, meNTSPD))
rownames(mumeComp) <- c("naRrmoved", "naFilled")
mumeComp
par(mfrow=c(1, 2))
hist(totalSPD$steps, breaks=20, col="grey",
     main="Total number of steps taken per day \n (missing data removed)",
     xlab="Number of steps")
hist(newTSPD$steps, breaks=20, col="grey",
     main="Total number of steps taken per day \n (missing data filled in)",
     xlab="Number of steps")
par(mfrow=c(1, 1))


## Are there differences in activity patterns between weekdays and weekends?

dtNEW$weekdays <- as.factor(ifelse(weekdays(dtNEW$date) %in% c("Saturday", "Sunday"),
                                  "weekend", "weekday"))
str(dtNEW)
head(dtNEW)
tail(dtNEW)
weekSPI <- aggregate(steps ~ weekdays + interval, dtNEW, FUN=mean, na.rm=TRUE)
xyplot(steps ~ interval | weekdays, data=weekSPI, type="l", layout=c(1, 2),
       xlab="Interval", ylab="Number of steps")
