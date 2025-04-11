---
title: "Human Activity Recognition in Weight Lifting Exercise"
author: "Kaung Myat Khant"
date: "2025-04-12"
output: 
    html_document:  
        keep_md: TRUE
---



## Introduction

The quantified self movement, empowered by wearable technology like Fitbits and Jawbone Ups, focuses on tracking the quantity of exercise. However, the quality of movement remains largely unquantified. This research addresses this gap by analyzing the execution of barbell lifts.

I use accelerometer data from six participants performing barbell lifts correctly and incorrectly across five variations. Data was collected from sensors on the belt, forearm, arm, and dumbbell. This detailed dataset allows for a granular analysis of movement mechanics during weight training, promising insights into personalized fitness and injury prevention.

## Method

### Load data and packages

First of all, the data are downloaded directly from the link for [training](%22https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv%22) and [testing](%22https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv%22) datasets. Then, the *tidyverse*, *rpart*, *randomforest* and *caret* packages were loaded.



### Data cleaning

First, the training data is sampled into training and validation sets.



Next, the acceleration measurements of belt, forearm, arm, and dumbbell are selected as predictors. Then, the missing values are checked for the outcome and predictor variables. Columns with missing values are dropped



### Exploratory data analysis

Correlation plot with hierarchical cluster is drawn to see the correlation between the predictors.


``` r
cor <- cor(training[,-53])
corrplot::corrplot(cor, order = 'hclust', diag = F, addrect = 2)
```

![Correlation plot showing the correlation between the numerical predictors](machineLearning_weightLIftingActivity_files/figure-html/explore-1.png)

### Preparing data

The principal component analysis is applied to the predictors with a threshold to explain 80% of variation among the predictors.


``` r
pca <- preProcess(training,method = "pca", thresh = 0.9)
pca
```

```
## Created from 13737 samples and 53 variables
## 
## Pre-processing:
##   - centered (52)
##   - ignored (1)
##   - principal component signal extraction (52)
##   - scaled (52)
## 
## PCA needed 18 components to capture 90 percent of the variance
```

``` r
train.pca <- predict(pca, training)
validation.pca <- predict(pca, validation)
testing.pca <- predict(pca, testing)
```

PCA needed 18 components to capture 90 percent of the variance among the predictors. Pre-processed sets for training, validation and testing data are produced.

### Model training

Decision tree model and random forest model are trained with the processed training set. In random forest model, re-sampling by three-fold cross-validation is used to get the model.



### Validation
The trained models are used to test on the validation set.


``` r
tree.predict <- predict(model.tree, newdata = validation.pca)
confusionMatrix(tree.predict, validation.pca$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1076  530  691  334  456
##          B    0    0    0    0    0
##          C    0    0    0    0    0
##          D   75  161   24  264  101
##          E   21  107    8   55  237
## 
## Overall Statistics
##                                           
##                Accuracy : 0.3809          
##                  95% CI : (0.3661, 0.3959)
##     No Information Rate : 0.2831          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.1693          
##                                           
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9181   0.0000   0.0000  0.40429  0.29849
## Specificity            0.3224   1.0000   1.0000  0.89647  0.94292
## Pos Pred Value         0.3486      NaN      NaN  0.42240  0.55374
## Neg Pred Value         0.9088   0.8072   0.8254  0.88933  0.84995
## Prevalence             0.2831   0.1928   0.1746  0.15773  0.19179
## Detection Rate         0.2599   0.0000   0.0000  0.06377  0.05725
## Detection Prevalence   0.7457   0.0000   0.0000  0.15097  0.10338
## Balanced Accuracy      0.6203   0.5000   0.5000  0.65038  0.62070
```

``` r
rf.predict <- predict(model.rf, newdata = validation.pca)
confusionMatrix(rf.predict, validation.pca$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1172    0    0    0    0
##          B    0  798    0    0    0
##          C    0    0  723    0    0
##          D    0    0    0  653    0
##          E    0    0    0    0  794
## 
## Overall Statistics
##                                      
##                Accuracy : 1          
##                  95% CI : (0.9991, 1)
##     No Information Rate : 0.2831     
##     P-Value [Acc > NIR] : < 2.2e-16  
##                                      
##                   Kappa : 1          
##                                      
##  Mcnemar's Test P-Value : NA         
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            1.0000   1.0000   1.0000   1.0000   1.0000
## Specificity            1.0000   1.0000   1.0000   1.0000   1.0000
## Pos Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Neg Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Prevalence             0.2831   0.1928   0.1746   0.1577   0.1918
## Detection Rate         0.2831   0.1928   0.1746   0.1577   0.1918
## Detection Prevalence   0.2831   0.1928   0.1746   0.1577   0.1918
## Balanced Accuracy      1.0000   1.0000   1.0000   1.0000   1.0000
```

It is clear that the random forest model has far better accuracy, with even 100%. So , it is selected to predict the testing data.

## Result

The random forest model is used to solve the problem given by testing data of 20 observation.


``` r
rf.predict.result <- predict(model.rf,newdata =  testing.pca)
print("Result of the predictions by Random forest model:")
```

```
## [1] "Result of the predictions by Random forest model:"
```

``` r
rf.predict.result
```

```
##  [1] B A A A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```


``` r
ggplot(as.data.frame(table(rf.predict.result)), aes(rf.predict.result,Freq))+
    geom_col(aes(fill = rf.predict.result))+
    scale_fill_manual(values = RColorBrewer::brewer.pal(n = 5, name = "Accent"))+
    labs(x = "Classe", y = "Count")+
    theme_bw()+
    theme(legend.position = "")
```

![Classe prdicted by the random forest model from the testing set](machineLearning_weightLIftingActivity_files/figure-html/bar-1.png)
