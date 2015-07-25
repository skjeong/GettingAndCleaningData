# CodeBook

## Data
* Source: https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip 
* Data set information: http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones 


## Run_analysis.R

* Set file path
```
path = file.path(getwd(), "UCI HAR Dataset")
```

* Read data set
```
subjectTrain = read.csv(file.path(path, "train", "subject_train.txt"), header = F)
subjectTest = read.csv(file.path(path, "test", "subject_test.txt"), header = F)

measurementTrain = read.table(file.path(path, "train", "X_train.txt"), header = F, colClasses = "numeric")
measurementTest = read.table(file.path(path, "test", "X_test.txt"), header = F, colClasses = "numeric")

labelTrain = read.csv(file.path(path, "train", "y_train.txt"), header = F)
labelTest = read.csv(file.path(path, "test", "y_test.txt"), header = F)
```

* Merge the training and the test sets to create one data set 1
```
subject = rbind(subjectTrain, subjectTest)
measurement = rbind(measurementTrain, measurementTest)
label = rbind(labelTrain, labelTest)
```

* Add column names
```
featureNames = read.table(file.path(path, "features.txt"), header = F)
names(subject) = "subject"
names(label) = "label"
names(measurement) = featureNames$V2
```

* Merge the training and the test sets to create one data set 2
```
data = cbind(subject, label, measurement)
```

* Clear unnecessary variables
```
rm(subjectTrain, subjectTest, measurementTrain, measurementTest, labelTrain, labelTest, featureNames)
```

* Extract only the measurements on the mean and standard deviation for each measurement
```
extractedFeatures = as.character(featureNames$V2[grep("mean|std", featureNames$V2)])
data2 = subset(data, select=c("subject", "label", extractedFeatures))
```

* Use descriptive activity names to name the activities in the data set
```
labelActivity = read.table(file.path(path, "activity_labels.txt"), header =F)
data2$label <- factor(data2$label, levels = c(1,2,3,4,5,6), labels = labelActivity$V2)
```

* Appropriately label the data set with descriptive variable names
```
names(data2)<-gsub("^t", "Time", names(data2))
names(data2)<-gsub("^f", "FastFourierTransform", names(data2))
```

* Create a second, independent tidy data set with the average of each variable for each activity and each subject.
```
library(reshape2)
temp1 <- melt(data2, id.vars = c("subject", "label"))
temp2 <- dcast(temp1, subject + label ~ variable, mean)
write.table(temp2, file = "tidy_data.txt", row.names = F)
```
