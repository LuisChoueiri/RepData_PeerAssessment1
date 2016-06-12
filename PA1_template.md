# RepData_PeerAssessment1
Luis Felipe Choueiri  
June 11, 2016  

Set WD and load in Data


```r
setwd("C:/Desktop/Programming/R")
library("ggplot2")
```

```
## Warning: package 'ggplot2' was built under R version 3.2.5
```

```r
library("knitr")
Data <- read.csv("activity.csv")
Data$date <- as.Date(Data$date)

CompData <- Data[complete.cases(Data[,1]),]
```


Calculate the daily total, mean and median steps


```r
CompAggSum <- aggregate(list(Steps = CompData$steps),
                     list(Date = CompData$date),
                     FUN= sum)
```

The total Daily steps are


```r
qplot(Date, data= CompAggSum, weight= Steps, geom = "histogram", binwidth= 1, ylab = "Steps")
```

![](PA1_template_files/figure-html/DlyStepsSum-1.png)<!-- -->

And the mean value is:


```r
mean(CompAggSum$Steps)
```

```
## [1] 10766.19
```

with a median of:


```r
median(CompAggSum$Steps)
```

```
## [1] 10765
```

We then continue our analysis at the interval level, as opposed to the day level


```r
CompIntAggMean <- aggregate(list(Steps = CompData$steps),
                     list(Interval = CompData$interval),
                     FUN= mean)
```

With the linear plot of median steps by interval


```r
qplot(Interval,Steps, data= CompIntAggMean,  geom = "line", ylab = "Steps")
```

![](PA1_template_files/figure-html/IntStepsMean-1.png)<!-- -->

With max at interval:


```r
CompIntAggMean[CompIntAggMean$Steps == max(CompIntAggMean$Steps), 1]
```

```
## [1] 835
```

We then calculate the number of rows in the original data with NAs



```r
sum(is.na(Data$steps))
```

```
## [1] 2304
```

We can then fill in these values, using the previously calculated data we can bring in the interval mean values, as some days are missing whole.


```r
CorrData <- merge(CompIntAggMean, Data, by.x = "Interval", by.y = "interval", all.y = T)

CorrData[is.na(CorrData$steps), 3] <- CorrData[is.na(CorrData$steps),2]

CorrData <- CorrData[, c(1,3,4)]
```

We now recalculate our earlier analysis with the NAs replaced



```r
CorrAggSum <- aggregate(list(Steps = CorrData$steps),
                     list(Date = CorrData$date),
                     FUN= sum)
```

The total Daily steps are


```r
qplot(Date, data= CorrAggSum, weight= Steps, geom = "histogram", binwidth= 1, ylab = "Steps")
```

![](PA1_template_files/figure-html/CorrDlyStepsSum-1.png)<!-- -->

And the mean value is:


```r
mean(CorrAggSum$Steps)
```

```
## [1] 10766.19
```

with a median of:


```r
median(CorrAggSum$Steps)
```

```
## [1] 10766.19
```

Showing a small impact to the final calculated values.


Continuing our Analysis but focusing on the difference between weekdays and weekends:


```r
CorrData$Weekn <- as.character(weekdays(CorrData$date))

CorrData[CorrData$Weekn %in% c("Saturday","Sunday"), "Weekn"] <- "Weekend"
CorrData[CorrData$Weekn %in% c("Monday","Tuesday","Wednesday","Thursday","Friday"), "Weekn"] <- "Weekday"

CorrData$Weekn <- as.factor(CorrData$Weekn)

CorrAggNData <- aggregate(list(Steps=CorrData$steps),
                         list(Interval=CorrData$Interval,WeekN=CorrData$Weekn),
                         FUN=mean)
```

Plot of weekend data by interval


```r
qplot(Interval,Steps, data= CorrAggNData[CorrAggNData$WeekN == "Weekend",]
      ,  geom = "line", ylab = "Steps", main = "Weekend")
```

![](PA1_template_files/figure-html/IntWkEndSteps-1.png)<!-- -->

Followed by a plot of Weekday data by interval


```r
qplot(Interval,Steps, data= CorrAggNData[CorrAggNData$WeekN == "Weekday",]
      ,  geom = "line", ylab = "Steps", main = "Weekday")
```

![](PA1_template_files/figure-html/IntWkDaySteps-1.png)<!-- -->


