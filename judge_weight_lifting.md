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
mod_rp <- train(classe ~ ., data = subtraining, method = 'rpart', 
                preProcess = c('center', 'scale'))
mod_rp$results
```


```
##           cp  Accuracy      Kappa AccuracySD    KappaSD
## 1 0.02715276 0.5457157 0.41362608 0.02374227 0.03856742
## 2 0.04281781 0.4989586 0.34790145 0.03479323 0.05986992
## 3 0.11478211 0.3236957 0.05940286 0.04143162 0.06314359
```

Unfortunately, it performs poorly. So I turn to `rf`, and plug in cross validation `cv`.


```r
mod_rf <- train(classe ~ ., data = subtraining, method = 'rf', 
                preProcess = c('center', 'scale'), 
                trControl = trainControl(method = 'cv', number = 5))
mod_rf$results
```



```
##   mtry  Accuracy     Kappa  AccuracySD     KappaSD
## 1    2 0.9914387 0.9891691 0.002420505 0.003064081
## 2   27 0.9910313 0.9886538 0.002103184 0.002661357
## 3   52 0.9865471 0.9829801 0.003128190 0.003957831
```

It is ideal. 



# Examining the out of sample error



The error rate is 0.01, quite small.


```
##           Reference
## Prediction    A    B    C    D    E
##          A 1393    4    0    0    0
##          B    2  942    4    0    0
##          C    0    3  850   12    1
##          D    0    0    1  792    2
##          E    0    0    0    0  898
```

```
##       Accuracy          Kappa  AccuracyLower  AccuracyUpper   AccuracyNull 
##      0.9940865      0.9925197      0.9915181      0.9960361      0.2844617 
## AccuracyPValue  McnemarPValue 
##      0.0000000            NaN
```

# Apply the algorithm to 20 test cases 


```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
