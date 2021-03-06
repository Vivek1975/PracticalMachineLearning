# Test
Vivek Nath Penmatsa  
February 19, 2017  



## Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset). 

## Data

The training data for this project are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

## Goal

The goal of the project is to predict the manner in which they did the exercise. There is the "classe" variable in the training set. We will use any of the other variables to predict with. We shall create a report describing how we built our model, how we used cross validation, what we think the expected out of sample error is, and why we made the choices we did. We will also use your prediction model to predict 20 different test cases.

## Summary

In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants and predict the manner they did the exercise which was shown by the outcome "classe". We first load the data and then cleanse the data. Then we do cross validation, build out model, train it and calculate the out of sample error and then predict the outcome of 20 test cases. We are looking for an approach that is both accurate and at the same time less time consuming. We realize our model using pca and random forests and show that the accuracy is almost 0.95 and the out of sample error is almost 0.05 and that the model we build accurately predicts 19 out of 20 testcases in about 13 minutes of time. If we chose to use random forests while considering 45 predictors based on ruling out correlated variates during preprocessing we get accuracy of 0.9927 with out of sample error rate of 0.007 and this predicts all the 20 test cases accurately but it takes more than an hour for the calculation which is rather slow. We also note that using gbm with 45 predictors also results in similar performance but takes about 30 minutes. We however show the fist model in this project as our model as it is a comprimize between accuracy and time. For our course project quiz we use the more accurate ones to achievev 100% accurate results

## Input Data
Read the data from csv files

```r
set.seed(123)
library(caret)
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```r
library(randomForest)
```

```
## randomForest 4.6-12
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```r
Training <- read.csv("pml-training.csv")
Testing <- read.csv("pml-testing.csv")
```
## Cleanse Data
Since the Testing data doesnot contain newwindow as true, we remove those rows from the training data.
The we remove all those columns with missing data as well as NAs. Finally we remove the first seven columns that are not required to be part of covariates

```r
TrainingFilterNW <- Training[!Training$new_window == "yes", ]
TrainingFilterNA <- Filter(function(x) !all(is.na(x)), TrainingFilterNW)
TrainingFilterEmpty <- TrainingFilterNA[!sapply(TrainingFilterNA, function(x) all(x == ""))]
TrainingFiltered <- TrainingFilterEmpty[,8:60]

TestingFilterNA <- Filter(function(x) !all(is.na(x)), Testing)
TestingFilterEmpty <- TestingFilterNA[!sapply(TestingFilterNA, function(x) all(x == ""))]
TestingFiltered <- TestingFilterEmpty[,8:59]
```
## Cross Validation
Then we pick 70% of our Training data set to be the train set and remaining 30% to be our test set

```r
inTrain <- createDataPartition(TrainingFiltered$classe, p = 0.7, list =FALSE)
TrainingTrain <- TrainingFiltered[inTrain,]
TrainingTest <- TrainingFiltered[-inTrain,]
```
## Pre-Processing
Since we are looking for a middleground between accuracy and time taken., we are using pca method in preprocessing and using just ten of the 52 pca components to build our model. The accuracy will be little less but the speed will not be comprimised. If we choose even less pca components the results may be even poor in terms of accuracy so we stick with 10 number of pca components to build our model

```r
preProc <- preProcess(TrainingTrain,method="pca",pcaComp=10)
trainPC <- predict(preProc,TrainingTrain)
testPC <- predict(preProc,TrainingTest)
testPCA <- predict(preProc,TestingFiltered)
```
## Build Model
We chose random forest which is highly accurate but slow. But to increase its speed we use pca method in preprocessing to use top 10 pca components so that resultant model has decent accuracy as well as is fast enough to make predictions

```r
modelFit <- train(classe ~ .,method="rf",data=trainPC)
```
## Prediction
We use the built model on test set to make prediction and show the results through confusion matrix

```r
pred <- predict(modelFit,testPC)
confusionMatrix(pred,testPC$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1603   25   19   15    4
##          B   10 1029   22    4   15
##          C   14   48  941   37    8
##          D   10    8   17  881    8
##          E    4    5    6    7 1023
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9504          
##                  95% CI : (0.9444, 0.9558)
##     No Information Rate : 0.2847          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9372          
##  Mcnemar's Test P-Value : 0.0004103       
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9768   0.9229   0.9363   0.9333   0.9669
## Specificity            0.9847   0.9890   0.9775   0.9911   0.9953
## Pos Pred Value         0.9622   0.9528   0.8979   0.9535   0.9789
## Neg Pred Value         0.9907   0.9816   0.9864   0.9870   0.9926
## Prevalence             0.2847   0.1935   0.1744   0.1638   0.1836
## Detection Rate         0.2782   0.1786   0.1633   0.1529   0.1775
## Detection Prevalence   0.2891   0.1874   0.1818   0.1603   0.1813
## Balanced Accuracy      0.9808   0.9559   0.9569   0.9622   0.9811
```
## Out of Sample Error
Our out of sample error is given by 1 minus the accuracy of the model that is 0.05 approximately.

## Prediction on 20 test cases
The 20 predicted outcomes on the 20 test cases are as follows:

```r
predict(modelFit,testPCA)
```

```
##  [1] B A B A A E D B A A A C B A E E A B B B
## Levels: A B C D E
```


