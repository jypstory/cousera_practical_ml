# cousera_practical_ml
coursera_practical_ml

https://rpubs.com/skyheart/coursear_practicalML





---
title: "Practica Machine Learning"
author: "joshua_park"
date: "2016년 8월 21일"
output: pdf_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Practical Machine Learning Final Report

###Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset). 


###Data

The training data for this project are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

## [step 1] load library
```{r}
library('caret')
library('rpart')
```

## [step 2]  load training data and testing data
```{r}
pml_training_data <- read.csv('pml-training.csv', header=T)
pml_predict_data <- read.csv('pml-testing.csv', header=T)
```

### set seed value
```{r}
set.seed(1004)
training_partition <- createDataPartition(y=pml_training_data$classe, p=0.7, list=FALSE)
```

### data partitioning training set and testing set  by 70% 
```{r}
training <- pml_training_data[training_partition, ]
testing <- pml_training_data[-training_partition, ]
```

### check demestion 
```{r}
dim(training)
dim(testing)
```

### check data.frame 
```{r }
str(training)
```

## [step 3] preprocessing for remove NA value
### remove near zero variance(NZV)
```{r}
NZV <- nearZeroVar(training)
training <- training[, -NZV]
testing  <- testing[, -NZV]
dim(training)
dim(testing)
```

### remove na value
```{r}
NoneNACol <- sapply(training, function(x) mean(is.na(x))) > 0.95
names(NoneNACol)
training <- training[, NoneNACol==FALSE]
testing  <- testing[, NoneNACol==FALSE]
dim(training)
dim(testing)
```

### remove column not used 
```{r}
training <- training[,-(1:6)]
testing <- testing[,-(1:6)]
dim(training)
dim(testing)
names(training)
```

## [step 4] building model
### Decision trees CART
```{r}
cart_model <- train(
  classe ~ ., 
  data=training,
  trControl=trainControl(method='cv', number = 3),
  method='rpart'
)
cart_model
predict_cart <- predict(cart_model, newdata=testing)
confusion_cart <- confusionMatrix(predict_cart, testing$classe)
confusion_cart
```


### Random forest
```{r}
rf_model <- train(
  classe ~ ., 
  data=training,
  trControl=trainControl(method='cv', number = 3),
  method='rf',
  ntree=100
)
rf_model
predict_rf <- predict(rf_model, newdata=testing)
confusion_rf <- confusionMatrix(predict_rf, testing$classe)
confusion_rf
```


###gradient boosting model (gbm)
```{r}
gbm_model <- train(
  classe ~ ., 
  data=training,
  trControl=trainControl(method='cv', number = 3),
  method='gbm'
)
gbm_model
predict_gbm <- predict(gbm_model, newdata=testing)
confusion_gbm <- confusionMatrix(predict_gbm, testing$classe)
confusion_gbm
```

when we check the Accuracy 
we can find that The Random Forest model is best model

## [step 5] predict pml data 
```{r}
predict_pml <- predict(rf_model, newdata=pml_predict_data)
predict_pml
```

