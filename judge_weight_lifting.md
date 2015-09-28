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

Unfortunately, it performs poorly. So I turn to `rf`.


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

# Examining the out of sample error


```r
confusionMatrix(predict(mod_rf_boot632, subtesting), subtesting$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1648 1134 1019  935  968
##          B    0    1    1    0    0
##          C    2    0    4    2    4
##          D   14    2    2   16   16
##          E   10    2    0   11   94
## 
## Overall Statistics
##                                           
##                Accuracy : 0.2996          
##                  95% CI : (0.2879, 0.3115)
##     No Information Rate : 0.2845          
##     P-Value [Acc > NIR] : 0.005449        
##                                           
##                   Kappa : 0.0256          
##  Mcnemar's Test P-Value : < 2.2e-16       
## 
## Statistics by Class:
## 
##                      Class: A  Class: B  Class: C Class: D Class: E
## Sensitivity           0.98447 0.0008780 0.0038986 0.016598  0.08688
## Specificity           0.03681 0.9997893 0.9983536 0.993091  0.99521
## Pos Pred Value        0.28892 0.5000000 0.3333333 0.320000  0.80342
## Neg Pred Value        0.85635 0.8065613 0.8259833 0.837532  0.82871
## Prevalence            0.28445 0.1935429 0.1743415 0.163806  0.18386
## Detection Rate        0.28003 0.0001699 0.0006797 0.002719  0.01597
## Detection Prevalence  0.96924 0.0003398 0.0020391 0.008496  0.01988
## Balanced Accuracy     0.51064 0.5003336 0.5011261 0.504844  0.54104
```

# Apply the algorithm to 20 test cases 


```
##  [1] A A A A A A C A A A A A A A A A A A A A
## Levels: A B C D E
```
