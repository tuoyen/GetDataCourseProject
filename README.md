*DESCRIPTION OF THE REPOSITORY*
===============================

This repo contains the following files: - summarydata.txt: a dataset of within-subject mean and standard deviations for six different activity measurements - run\_analysis.R: R script for generating the dataset "summarydata.txt"" - README.md: a read me file describing how the script "run\_analysis.R" works - CodeBook.md: a code book describing variables in the dataset "summarydata.txt"

*DESCRIPTION OF THE R SCRIPT*
=============================

HOUSEKEEPING
------------

``` r
getwd() ### check working directory
library(dplyr)
```

DOWNLOAD DATA
-------------

Download data collected from Samsung Galaxy S smartphone accelerometers. See [full description](http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones) of the data.

``` r
fileURL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
if (!file.exists("HARdata.zip")) {
  download.file(fileURL, destfile="./HARdata.zip")
}
list.files("./")
if (!file.exists("UCI HAR Dataset")) {
  unzip("HARdata.zip")
}
list.files('./UCI HAR Dataset/')
```

TRAINING SET
------------

``` r
# Read activity name class labels

activitylb <- read.table("./UCI HAR Dataset/activity_labels.txt")
head(activitylb)

# Read features
features <- read.table("./UCI HAR Dataset/features.txt")
head(features)

# Read training labels
trainlb <- read.table("./UCI HAR Dataset/train/y_train.txt")
head(trainlb)
trainlb$V1 <- factor(trainlb$V1, levels = activitylb$V1, labels = activitylb$V2)
summary(trainlb)

# Read training subject
trainID <- read.table("./UCI HAR Dataset/train/subject_train.txt")
head(trainID)

# Read training set
trainset <- read.table("./UCI HAR Dataset/train/X_train.txt", col.names = features$V2)
head(trainset)

# Add activity and subject labels to training data
trainset$activity <- trainlb$V1
trainset$subject <-trainID$V1
head(trainset$activity)
head(trainset$subject)

# Add data type variable
trainset$datatype <- rep("training", nrow(trainset))
head(trainset$datatype)
```

TEST SET
--------

``` r
# Read test labels
testlb <- read.table("./UCI HAR Dataset/test/y_test.txt")
head(testlb)
testlb$V1 <- factor(testlb$V1, levels = activitylb$V1, labels = activitylb$V2)
summary(testlb)

# Read test subject
testID <- read.table("./UCI HAR Dataset/test/subject_test.txt")
head(testID)

# Read test set
testset <- read.table("./UCI HAR Dataset/test/X_test.txt", col.names = features$V2)
head(testset)

# Add activity and subject labels to training data
testset$activity <- testlb$V1
testset$subject <-testID$V1
head(testset$activity)
head(testset$subject)

# Add data type variable
testset$datatype <- rep("test", nrow(testset))
head(testset$datatype)
```

1. Merge the training and the test sets to create one data set
--------------------------------------------------------------

``` r
total <- rbind(trainset, testset)

# check data and activity types in the data
table(total$datatype)
table(total$activity)
table(total$datatype, total$activity)

# check subjects in the data
unique(total$subject)
length(unique(total$subject))
```

2. Extract only the measurements on the mean and standard deviation for each measurement
----------------------------------------------------------------------------------------

``` r
extract <- grep("mean\\()|std\\()", features$V2)
length(extract)
features$V2[grepl("mean\\()|std\\()", features$V2)] ### check

subset <- select(total, subject, activity, extract)
dim(subset) ### check
```

3. Descriptive activity names are stored in the "activity" variable
-------------------------------------------------------------------

4. All variable names are appropriately labeled
-----------------------------------------------

5. Create a second, independent tidy data set with the average of each variable for each activity and each subject
------------------------------------------------------------------------------------------------------------------

``` r
summarydata <- subset %>% group_by(activity, subject) %>% summarize_all(mean)
write.table(summarydata, "summarydata.txt", row.names = FALSE)
```
