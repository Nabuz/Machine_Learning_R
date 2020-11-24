---
title: "Resampling_methods"
author: "Hani Nabulsi"
date: "24/11/2020"
output:
  html_document:
    keep_md: true
---

```r
knitr::opts_chunk$set(message=FALSE,warning=FALSE,fig.path = "README_figs/README-" ) 
```

The Validation Set Approach

We explore the use of the validation set approach in order to estimate the test error rate that result from fitting various linear models on Auto data set.


```r
library(ISLR)
set.seed(1) #To reproduce the randomness in same way more times
train= sample(392,196) #We take randomly 196 obs from a dataset with 392 obs (1:392)
```

```r
lm.fit=lm(mpg~horsepower,data=Auto,subset=train) #we fit the linear regr model on training set. Remeber that subset takes an array with indexes
```

```r
prediction_test = predict(lm.fit,Auto[-train,]) #Prediction on complementary of Training set--> Test sets 
mpg_test = Auto$mpg[-train]  #Values of responce of test set
mean((mpg_test-prediction_test)^2) #Test error rate (MSE)
```

```
## [1] 26.14142
```
Now we calculate the Test Error Rate on with a quadratic and cube polinomio of the features.


```r
lm.fit2 = lm(mpg~poly(horsepower,2),data=Auto,subset=train)
prediction_test_2 = predict(lm.fit2,Auto[-train,])
mean((mpg_test-prediction_test_2)^2)
```

```
## [1] 19.82259
```


```r
lm.fit3 = lm(mpg~poly(horsepower,3),data=Auto,subset=train)
prediction_test_3 = predict(lm.fit3,Auto[-train,])
mean((mpg_test-prediction_test_3)^2)
```

```
## [1] 19.78252
```

Now we calculate the Test Error Rate with another random split in Training Set/Test Set


```r
#For linear Regression
set.seed(2)
train= sample(392,196)

lm.fit=lm(mpg~horsepower,data=Auto,subset=train) 
prediction_test = predict(lm.fit,Auto[-train,])
mpg_test = Auto$mpg[-train]  
mean((mpg_test-prediction_test)^2) 
```

```
## [1] 23.29559
```

```r
#For quadratic polynomio
lm.fit2 = lm(mpg~poly(horsepower,2),data=Auto,subset=train)
prediction_test_2 = predict(lm.fit2,Auto[-train,])
mean((mpg_test-prediction_test_2)^2)
```

```
## [1] 18.90124
```


```r
#For cubic polynomio
lm.fit3 = lm(mpg~poly(horsepower,3),data=Auto,subset=train)
prediction_test_3 = predict(lm.fit3,Auto[-train,])
mean((mpg_test-prediction_test_3)^2)
```

```
## [1] 19.2574
```


If we change the random split in Training Set and Test Set and recalculate the Test Error Rate with the different models we can see that quadratic model is better than linear model!!

MOST IMPORTANT: if we change the random splitting in Training Set/Test Set, there is a variability in Test Error Rate among the models!

Leave-One-Out Cross-Validation


```r
library(boot) # to load cv.glm()
glm.fit = glm(mpg~horsepower,data=Auto) #if we don't explicit family= bynomial, we have a linear regression. We need glm() because so we can use cv.glm
cv.err = cv.glm(Auto,glm.fit)
cv.err$delta #delta-->cross-validation results. We have the two value equal (we see a situation where there are different). It represents the mean of all test errors on a single row.
```

```
## [1] 24.23151 24.23114
```

We use LOOCV with more type of polinomial regression


```r
cv.error =rep(0,5)
for (i in 1:5){ #We use for loop for fit many polynomial model at the same time
glm.fit=glm(mpg~poly(horsepower,i),data=Auto)
cv.error[i]=cv.glm(Auto,glm.fit)$delta[1] #We memorize the vector of all means of all test errors.
}
```

```r
cv.error  #We estimate the test MSE. We can see the improve from linear to quadratic is relevant. It's no so relevant with high-order polynomials.
```

```
## [1] 24.23151 19.24821 19.33498 19.42443 19.03321
```

k_Fold Cross-Validation


```r
set.seed(17) #We need to split randomly the dataset in 10 subsets
cv.error.10 = rep(0,10)
for (i in 1:10){ #We use for loop for fit many polynomial model at the same time
glm.fit=glm(mpg~poly(horsepower,i),data=Auto)
cv.error.10[i]=cv.glm(Auto,glm.fit,K=10)$delta[1] #We memorize the vector of all means of all test errors.  cv.glm$delta[1]=test MSE, cv.glm$delta[2] = bias corrected version of first one. It's very similar to first one.
}
```

```r
cv.error.10  #As before, from linear to quadratic fit there is a good improvement in test MSE. Not so good improvement with high-order polynomials.
```

```
##  [1] 24.20520 19.18924 19.30662 19.33799 18.87911 19.02103 18.89609 19.71201
##  [9] 18.95140 19.50196
```








