---
title: "README"
output: html_document
---

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

```{r}
##  This code sets the library and reads in the initial data from the 
##  unzipped source files 
##  (https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip) 
##  in your local directory.

library(dplyr)
traintable <- read.table("UCI HAR Dataset/train/X_train.txt")
testtable <- read.table("UCI HAR Dataset/test/X_test.txt")
feattable <- read.table("UCI HAR Dataset/features.txt")

##  This code cleans up the column names, removing the (), lower-casing, 
##  and replacing the f or t at the beginning of the name with -freq for 
##  frequency or -time for time.

feattable$V2<- sapply(feattable$V2,
    function (x){
        x <- sub("\\(\\)","",x)
        x <- tolower(x)
    if (substr(x, 1, 1) == "f")
    {y <- paste (x, "-freq", sep = "")
            sub("f","", y)
    }
    else 
    {y <- paste (x, "-time", sep = "")
        sub("t","", y)
    }
})

##  This code binds together the x-data and adds the renamed column headers.

middata <- rbind(traintable,testtable)
names(middata) <- feattable$V2

##  This code removes the unwanted columns that don't include mean or std, 
##  and excludes those that containg meanFreq.

columnnames <- as.character(names(middata))

grep("mean|std",columnnames)
meanstdcol <- grep("-mean|-std",columnnames)
meanstddata <- middata[,meanstdcol]
meanstddata <- select(meanstddata, -contains ("meanFreq"))

##  This code reads in the y-data/row names for the activities from the 
##  dataset and binds it together in the same order as the data above.

acttraintable <- read.table("UCI HAR Dataset/train/y_train.txt")
acttesttable <- read.table("UCI HAR Dataset/test/y_test.txt")
ydata <- rbind(acttraintable, acttesttable)

##  This code converts the activity numbers into the activity names provided
##  in activity_labels.txt

activitynames <- read.table("UCI HAR Dataset/activity_labels.txt", stringsAsFactors=FALSE)

ydatanamed <- sapply(ydata$V1,function(x){activitynames$V2[activitynames$V1 == x]})

##  This code binds together the x-data and the y-data.

neardata <- cbind(ydatanamed, meanstddata)

##  This code renames the activity names column to be readable.

names(neardata)[1] <- "activity"

##  This code reads in the subject data, binds it together
##  and renames it to be user friendly.

subtraintable <- read.table("UCI HAR Dataset/train/subject_train.txt")
subtesttable <- read.table("UCI HAR Dataset/test/subject_test.txt")
subdata <- rbind (subtraintable, subtesttable)
names(subdata) <- "subjects"

##  This code binds all of the data pieces together into one database
##  and turns it into a table for dplyr processing.

alldata <- cbind(subdata, neardata)

finaldata <- tbl_df(alldata)

##  This code groups the data by subjects, then activity, and summarizes
##  all of the following means and stds by subject and activity.

groupedData <- group_by(finaldata, subjects, activity)
groupedMean <- summarize_each(groupedData, funs(mean))
```
