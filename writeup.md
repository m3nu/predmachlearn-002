HAR machine learning assignment
===

Step 1: Load data
---


```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
pml.testing <- read.csv('pml-testing.csv')
pml.training <- read.csv('pml-training.csv')
```

Step 2: Pick relevant column names
---
This drops all the cols that are mostly NA or empty. We also skip the administrative stuff in cols 1 to 7.


```r
f<-pml.testing[,colSums(is.na(pml.testing)) < nrow(pml.testing)*.6]
p <- colnames(f[8:59])
p
```

```
##  [1] "roll_belt"            "pitch_belt"           "yaw_belt"            
##  [4] "total_accel_belt"     "gyros_belt_x"         "gyros_belt_y"        
##  [7] "gyros_belt_z"         "accel_belt_x"         "accel_belt_y"        
## [10] "accel_belt_z"         "magnet_belt_x"        "magnet_belt_y"       
## [13] "magnet_belt_z"        "roll_arm"             "pitch_arm"           
## [16] "yaw_arm"              "total_accel_arm"      "gyros_arm_x"         
## [19] "gyros_arm_y"          "gyros_arm_z"          "accel_arm_x"         
## [22] "accel_arm_y"          "accel_arm_z"          "magnet_arm_x"        
## [25] "magnet_arm_y"         "magnet_arm_z"         "roll_dumbbell"       
## [28] "pitch_dumbbell"       "yaw_dumbbell"         "total_accel_dumbbell"
## [31] "gyros_dumbbell_x"     "gyros_dumbbell_y"     "gyros_dumbbell_z"    
## [34] "accel_dumbbell_x"     "accel_dumbbell_y"     "accel_dumbbell_z"    
## [37] "magnet_dumbbell_x"    "magnet_dumbbell_y"    "magnet_dumbbell_z"   
## [40] "roll_forearm"         "pitch_forearm"        "yaw_forearm"         
## [43] "total_accel_forearm"  "gyros_forearm_x"      "gyros_forearm_y"     
## [46] "gyros_forearm_z"      "accel_forearm_x"      "accel_forearm_y"     
## [49] "accel_forearm_z"      "magnet_forearm_x"     "magnet_forearm_y"    
## [52] "magnet_forearm_z"
```

Step 3: Split in training- and testset.
---


```r
df = pml.training[c(p, 'classe')]
inTrain <- createDataPartition(y=df$classe, p=.75, list=F)
df.train <- df[inTrain, ]
df.test <- df[-inTrain, ]
```

Step 4: Train algorithm
---
QDA gave the best results, compared to LDA. The problem seems to be non-linear. Running the model on the test set gives almost the same accuracy as in the training set (cross-validation). The expected out-of-sample error is about 10%.


```r
model <- train(df.train$classe ~ ., meth='qda', data=df.train)
```

```
## Loading required package: MASS
```

```r
confusionMatrix(df.test$classe, predict(model, df.test))
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1325   47   16    6    1
##          B   66  776   96    2    9
##          C    0   57  794    3    1
##          D    4    2  101  685   12
##          E    0   30   33   15  823
## 
## Overall Statistics
##                                              
##                Accuracy : 0.898              
##                  95% CI : (0.889, 0.906)     
##     No Information Rate : 0.284              
##     P-Value [Acc > NIR] : <0.0000000000000002
##                                              
##                   Kappa : 0.871              
##  Mcnemar's Test P-Value : <0.0000000000000002
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.950    0.851    0.763    0.963    0.973
## Specificity             0.980    0.957    0.984    0.972    0.981
## Pos Pred Value          0.950    0.818    0.929    0.852    0.913
## Neg Pred Value          0.980    0.966    0.939    0.994    0.994
## Prevalence              0.284    0.186    0.212    0.145    0.173
## Detection Rate          0.270    0.158    0.162    0.140    0.168
## Detection Prevalence    0.284    0.194    0.174    0.164    0.184
## Balanced Accuracy       0.965    0.904    0.874    0.968    0.977
```

Step 5: Solution
---


```r
predict(model, pml.testing[p])
```

```
##  [1] A A B A A E D B A A A C B A E E A B B B
## Levels: A B C D E
```
