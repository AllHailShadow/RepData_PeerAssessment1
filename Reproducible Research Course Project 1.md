It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The variables included in this dataset are:

steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

date: The date on which the measurement was taken in YYYY-MM-DD format

interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

**Loading the data**


```r
library(ggplot2)
library(plyr)
activity <- read.csv("activity.csv")
```

**Processing the data**


```r
activity$day <- weekdays(as.Date(activity$date))
activity$DateTime <- as.POSIXct(activity$date,format="%Y-%m-%d")
clean <- activity[!is.na(activity$steps),]
```

**Finding total number of steps per day and creating histogram**


```r
sumTable <- aggregate(activity$steps~activity$date,FUN=sum, )
colnames(sumTable) <- c("Date","Steps")
hist(sumTable$Steps,breaks=5,xlab="Steps",main="Total Steps per Day")
```

![plot of chunk unnamed-chunk-25](figure/unnamed-chunk-25-1.png)

**Calculating mean and median of total steps per day**


```r
as.integer(mean(sumTable$Steps))
```

```
## [1] 10766
```

```r
as.integer(median(sumTable$Steps))
```

```
## [1] 10765
```
The average and median steps taken per day is 10766 and 10765 respectively.

**Finding average daily activity pattern**


```r
intervalTable <- ddply(clean, .(interval), summarize, Avg=mean(steps))
p <- ggplot(intervalTable,aes(x=interval,y=Avg),xlab="Interval",ylab="Average number of steps")
p+geom_line()+xlab("Interval")+ylab("Average number of steps")+ggtitle("Average Number of Steps per Interval")
```

![plot of chunk unnamed-chunk-27](figure/unnamed-chunk-27-1.png)

**Which 5-minute interval, on average and across all days, had the maximum number of steps?**


```r
maxSteps <- max(intervalTable$Avg)
intervalTable[intervalTable$Avg==maxSteps,1]
```

```
## [1] 835
```

The 835th interval had the maximum number of steps.

**Calculating and reporting number of missing values**


```r
nrow(activity[is.na(activity$steps),])
```

```
## [1] 2304
```

The number of missing values is 2304.

**Filling in missing values based on day of the week and recalculating mean and median**


```r
avgTable <- ddply(clean, .(interval,day), summarize, Avg=mean(steps))
nadata <- activity[is.na(activity$steps),]
newdata <- merge(nadata,avgTable,by=c("interval","day"))
newdata2 <- newdata[,c(6,4,1,2,5)]
colnames(newdata2) <- c("steps", "date", "interval", "day", "DateTime")
mergeData <- rbind(clean,newdata2)
sumTable2 <- aggregate(mergeData$steps~mergeData$date,FUN=sum,)
colnames(sumTable2) <- c("Date","Steps")
as.integer(mean(sumTable2$Steps))
```

```
## [1] 10821
```

```r
as.integer(median(sumTable2$Steps))
```

```
## [1] 11015
```

```r
hist(sumTable2$Steps, breaks=5, xlab="Steps", main = "Total Steps per Day with NAs Fixed", col="Black")
hist(sumTable$Steps, breaks=5, xlab="Steps", main = "Total Steps per Day with NAs Fixed", col="Grey", add=T)
legend("topright", c("Imputed Data", "Non-NA Data"), fill=c("black", "grey") )
```

![plot of chunk unnamed-chunk-30](figure/unnamed-chunk-30-1.png)

The corrected mean and median are 10821 and 11015 respectively. This is a difference of 55 and 250 steps respectively compared to the old mean and median, but the distribution looks similar regardless. 

**Looking for differences between weekdays and weekends**


```r
mergeData$DayCategory <- ifelse(mergeData$day %in% c("Saturday", "Sunday"), "Weekend", "Weekday")
library(lattice)
intervalTable2 <- ddply(mergeData, .(interval,DayCategory), summarize, Avg=mean(steps))
xyplot(Avg~interval|DayCategory,data=intervalTable2,type="l",layout=c(1,2),main="Average Steps per Interval based on Type of Day",ylab="Average Number of Steps",xlab="Interval")
```

![plot of chunk unnamed-chunk-31](figure/unnamed-chunk-31-1.png)

The step activity trends are very different on weekends compared to weekdays; this is likely because weekends afford the opportunity to exercise more and get out of the house and workspace.
