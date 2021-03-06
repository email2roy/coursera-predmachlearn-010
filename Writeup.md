---
title: "Building a model to predict \"how (well)\" an activity was performed"
author: "Roy Chan"
date: "25 January, 2015"
output: html_document
---

##Abstract
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - sa group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

##Data
We first download and read the training and test set

```r
training_set_url  <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
training_set_file <- "pml-training.csv"
test_set_url      <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
test_set_file     <- "pml-testing.csv"
download.file(training_set_url, training_set_file, method="wget")
download.file(test_set_url, test_set_file, method="wget")
```

##Exploratory Analysis
Now, load the data we downloaded

```r
readdata     <- function(file){read.csv(file, na.strings=c("","NA","#DIV/0!"))}
training_set <- readdata(training_set_file)
test_set     <- readdata(test_set_file)
```
Only load those that we are interested in

```r
columns <- c("roll_belt", "pitch_belt", "yaw_belt", "total_accel_belt", 
    "gyros_belt_x", "gyros_belt_y", "gyros_belt_z", "accel_belt_x", "accel_belt_y", 
    "accel_belt_z", "magnet_belt_x", "magnet_belt_y", "magnet_belt_z", "roll_arm", 
    "pitch_arm", "yaw_arm", "total_accel_arm", "gyros_arm_x", "gyros_arm_y", 
    "gyros_arm_z", "accel_arm_x", "accel_arm_y", "accel_arm_z", "magnet_arm_x", 
    "magnet_arm_y", "magnet_arm_z", "roll_dumbbell", "pitch_dumbbell", "yaw_dumbbell", 
    "total_accel_dumbbell", "gyros_dumbbell_x", "gyros_dumbbell_y", "gyros_dumbbell_z", 
    "accel_dumbbell_x", "accel_dumbbell_y", "accel_dumbbell_z", "magnet_dumbbell_x", 
    "magnet_dumbbell_y", "magnet_dumbbell_z", "roll_forearm", "pitch_forearm", 
    "yaw_forearm", "total_accel_forearm", "gyros_forearm_x", "gyros_forearm_y", 
    "gyros_forearm_z", "accel_forearm_x", "accel_forearm_y", "accel_forearm_z", 
    "magnet_forearm_x", "magnet_forearm_y", "magnet_forearm_z")
test_set <- test_set [,columns]
```

And for training set we need the "classe" which is the column indicating the activity

```r
columns <- c(columns,"classe")
training_set <- training_set[,columns]
```

Check if all variables are good to use

```r
correlation <- cor(training_set[, names(training_set) != "classe"])
palette <- colorRampPalette(c("blue", "white", "red"))(n = 199)
heatmap(correlation, col = palette)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

##Predictive Model
Use Random forests algorithm as predictive model.
First set the random seed to make sure we can reproduce the same result

```r
set.seed(28)
```

Then train the training set with 2048 trees.

```r
library(randomForest)
```

```
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
## Need help? Try the ggplot2 mailing list: http://groups.google.com/group/ggplot2.
```

```r
model <- randomForest(classe ~ ., data = training_set, ntree = 2048)
```

Let check to make sure the model predicts properly with low error value

```r
model$confusion
```

```
##      A    B    C    D    E class.error
## A 5577    2    0    0    1   0.0005376
## B    9 3785    3    0    0   0.0031604
## C    0   11 3411    0    0   0.0032145
## D    0    0   21 3193    2   0.0071517
## E    0    0    1    3 3603   0.0011090
```

The confusion matrix also looks good, as there isn't any high error rate (class.error numbers are low). Now we can apply the model to the test set.

```r
answer <- predict(model, test_set)
answer
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```

##Conclusion
The model we got by using the random forest is able to get a 100% accuracy on test set provided in the submission section. With the low error rate in the confusion martix we believe that the model is a good one.


##Reference
http://en.wikipedia.org/wiki/Random_forest




