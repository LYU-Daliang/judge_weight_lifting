# A Model to Tell How Well You Do Weight Lifting
LYU Daliang  
September 27, 2015  

# Background

Using devices such as _Jawbone Up_, _Nike FuelBand_, and _Fitbit_ it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. 

One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this report, I will use data from accelerometers on the _belt_, _forearm_, _arm_, and _dumbell_ of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. Based on the data, I will build a model to predict the manner in which they did the exercise. 

The data come from this source: <http://groupware.les.inf.puc-rio.br/har>.

# Build the model



After loading and cleaning the data, I first try to fit a model with `rpart`.


```r
mod_rp <- train(classe ~ ., data = subtraining, preProcess = 'knnImpute', method = 'rpart')
mod_rp$results
```

```
##          cp  Accuracy     Kappa AccuracySD    KappaSD
## 1 0.1111111 0.4502769 0.3023944 0.07014067 0.08005479
## 2 0.1623932 0.3857174 0.2117931 0.07813745 0.08769933
## 3 0.2136752 0.2971696 0.1054304 0.09169021 0.10211150
```

Unfortunatly, it performs poorly. So I turn to `rf`.


```r
mod_rf <- train(classe ~ ., data = subtraining, preProcess = 'knnImpute', method = 'rf')
mod_rf$results
```

```
##   mtry  Accuracy     Kappa AccuracySD    KappaSD
## 1    2 0.6739042 0.5855988 0.05698339 0.07287320
## 2   59 0.6774873 0.5906897 0.07268575 0.09235784
## 3  117 0.6706013 0.5822370 0.08287332 0.10491615
```

It is much better, but not ideal. I decide to adjust the cross validation. After some testing, `boot632` does the best.


```r
mod_rf_boot632 <- train(classe ~ ., data = subtraining, preProcess = 'knnImpute', method = 'rf', trControl = trainControl(method = 'boot632'))
mod_rf_boot632$results
```

```
##   mtry  Accuracy     Kappa AccuracySD    KappaSD AccuracyApparent
## 1    2 0.7823765 0.7228491 0.05245460 0.06530805                1
## 2   59 0.7821235 0.7231078 0.05477022 0.06792060                1
## 3  117 0.7750329 0.7140049 0.04874132 0.06139024                1
##   KappaApparent
## 1             1
## 2             1
## 3             1
```

# Examping the out of sample error


```r
confusionMatrix(predict(mod_rf_boot632, subtesting), subtesting$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1662 1133 1019  949  996
##          B    0    0    0    0    0
##          C    4    3    7    2    1
##          D    0    0    0   10    7
##          E    8    3    0    3   78
## 
## Overall Statistics
##                                           
##                Accuracy : 0.2986          
##                  95% CI : (0.2869, 0.3104)
##     No Information Rate : 0.2845          
##     P-Value [Acc > NIR] : 0.008791        
##                                           
##                   Kappa : 0.0228          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity           0.99283   0.0000 0.006823 0.010373  0.07209
## Specificity           0.02707   1.0000 0.997942 0.998578  0.99709
## Pos Pred Value        0.28859      NaN 0.411765 0.588235  0.84783
## Neg Pred Value        0.90476   0.8065 0.826346 0.837423  0.82669
## Prevalence            0.28445   0.1935 0.174342 0.163806  0.18386
## Detection Rate        0.28241   0.0000 0.001189 0.001699  0.01325
## Detection Prevalence  0.97859   0.0000 0.002889 0.002889  0.01563
## Balanced Accuracy     0.50995   0.5000 0.502382 0.504475  0.53459
```

# Apply the algorithm to 20 test cases 


```
##  [1] A A A A A A C A A A A A A A A A A A A A
## Levels: A B C D E
```
