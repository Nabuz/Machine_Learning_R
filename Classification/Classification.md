---
title: "Classification"
author: "Hani Nabulsi"
date: "15/11/2020"
output:
  html_document:
    keep_md: true
---

```r
knitr::opts_chunk$set(message=FALSE,warning=FALSE,fig.path = "README_figs/README-" ) 
```
THE STOCK MARKET DATA

We work with Smarket dataset:

Lag1-Lag5 --> Percentage of returns for each of the five previous days
Volume --> Number of shares trades on previous day in billions
Today --> The percentage return on the date in question
Direction -->If the market was up or down on the specific date


```r
library(ISLR) #Load the library plus some basic inspection
names(Smarket)
```

```
## [1] "Year"      "Lag1"      "Lag2"      "Lag3"      "Lag4"      "Lag5"     
## [7] "Volume"    "Today"     "Direction"
```

```r
dim(Smarket)
```

```
## [1] 1250    9
```

```r
summary(Smarket)
```

```
##       Year           Lag1                Lag2                Lag3          
##  Min.   :2001   Min.   :-4.922000   Min.   :-4.922000   Min.   :-4.922000  
##  1st Qu.:2002   1st Qu.:-0.639500   1st Qu.:-0.639500   1st Qu.:-0.640000  
##  Median :2003   Median : 0.039000   Median : 0.039000   Median : 0.038500  
##  Mean   :2003   Mean   : 0.003834   Mean   : 0.003919   Mean   : 0.001716  
##  3rd Qu.:2004   3rd Qu.: 0.596750   3rd Qu.: 0.596750   3rd Qu.: 0.596750  
##  Max.   :2005   Max.   : 5.733000   Max.   : 5.733000   Max.   : 5.733000  
##       Lag4                Lag5              Volume           Today          
##  Min.   :-4.922000   Min.   :-4.92200   Min.   :0.3561   Min.   :-4.922000  
##  1st Qu.:-0.640000   1st Qu.:-0.64000   1st Qu.:1.2574   1st Qu.:-0.639500  
##  Median : 0.038500   Median : 0.03850   Median :1.4229   Median : 0.038500  
##  Mean   : 0.001636   Mean   : 0.00561   Mean   :1.4783   Mean   : 0.003138  
##  3rd Qu.: 0.596750   3rd Qu.: 0.59700   3rd Qu.:1.6417   3rd Qu.: 0.596750  
##  Max.   : 5.733000   Max.   : 5.73300   Max.   :3.1525   Max.   : 5.733000  
##  Direction 
##  Down:602  
##  Up  :648  
##            
##            
##            
## 
```

We use cor() to produce a matrix that contains all the pairwise correlations among the predictors.

```r
library(corrplot)
corrplot(cor(Smarket[,1:8]),method="circle",type= "upper") #We esclude the direction because is qualitative
```

![](README_figs/README-unnamed-chunk-2-1.png)<!-- -->

There is only a (linear) correlation (about 0.5) between Year and Volume predictors. The volume increases over the time, the average number of shares daily increase from 2001 to 2005.

Correlations between the lag vars and today is zero --> no correlation between today return and previous return


```r
boxplot(Volume~Year, data=Smarket, main="Volume(shares trade in billion) over the 2011-2005",xlab="Year",ylab="Volume",col="orange",border ="brown") #The average daily shares (avg volume) traded increase by the year!!! (cor index = 0.5)
```

![](README_figs/README-unnamed-chunk-3-1.png)<!-- -->

LOGISTIC REGRESSION:

Now we fit a logistic regression to predict the Direction using Lag1-Lag5 and Volume.

glm()-->generalized linear models, a class of model that includes the logistic regression.

```r
glm.fits = glm(Direction~Lag1+Lag2*Lag3+Lag4+Lag5*Volume,data = Smarket,family=binomial)
#family=binomial tells R to use logistic regression among the class of generalized linear model.
summary(glm.fits)
```

```
## 
## Call:
## glm(formula = Direction ~ Lag1 + Lag2 * Lag3 + Lag4 + Lag5 * 
##     Volume, family = binomial, data = Smarket)
## 
## Deviance Residuals: 
##    Min      1Q  Median      3Q     Max  
## -1.528  -1.204   1.059   1.145   1.378  
## 
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)
## (Intercept) -0.11385    0.24153  -0.471    0.637
## Lag1        -0.07338    0.05021  -1.462    0.144
## Lag2        -0.03774    0.05060  -0.746    0.456
## Lag3         0.01291    0.05010   0.258    0.797
## Lag4         0.01029    0.05009   0.205    0.837
## Lag5         0.10296    0.22410   0.459    0.646
## Volume       0.12726    0.15894   0.801    0.423
## Lag2:Lag3    0.01822    0.03418   0.533    0.594
## Lag5:Volume -0.06430    0.14767  -0.435    0.663
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 1731.2  on 1249  degrees of freedom
## Residual deviance: 1727.1  on 1241  degrees of freedom
## AIC: 1745.1
## 
## Number of Fisher Scoring iterations: 3
```

We have only a predictor that could be related to the responce: Lag1 (lowest P-value).

The parameter for lag1 is negative --> if the return of previous day is positive than the probability of today to go up (Direction) decreases. But here the P-value is too large (0.144) --> no statistically significance -->no correlation between Lag1 and Direction!!!



```r
glm.probs = predict(glm.fits, type="response") # type="respomce"-->gives probability of the form P(y=1|X).If you dont tell on which data you'll do prediction is calculated on training set used for the fit.
glm.probs[1:10]
```

```
##         1         2         3         4         5         6         7         8 
## 0.5283640 0.4787417 0.4828446 0.5213932 0.5097990 0.5075475 0.4935325 0.5122652 
##         9        10 
## 0.5186681 0.4900873
```

```r
contrasts(Smarket$Direction) #So P(y=1|X), Y=1--> Up
```

```
##      Up
## Down  0
## Up    1
```

Now we need to convert our arrey of prediction (glm.probs) as an array with "up" and "down"


```r
glm.pred=rep("Down",1250) #We create an array with 1250 values, fills with "Down"
glm.pred[glm.probs>0.5] = "Up" #we fill the array with "Up" if the probability of this value in glm.probs array is > 0.5
```

Now we plot the confusion matrix


```r
table(glm.pred,Smarket$Direction)
```

```
##         
## glm.pred Down  Up
##     Down  148 129
##     Up    454 519
```


```r
print("Accuracy on Training Set is: ")
```

```
## [1] "Accuracy on Training Set is: "
```

```r
mean(glm.pred==Smarket$Direction) #Equivalent to : (148 + 519)/1250
```

```
## [1] 0.5336
```

Could be that the logistic regression is better than a "by chance" classificator (Accuracy -->0.5). It's no true!! We are predicting our values on training set-->to optiomistic setting!! Try to setup a more realistic setting (fitting on traing set and test on test set).

```r
print("Error rate on Training set is")
```

```
## [1] "Error rate on Training set is"
```

```r
mean(glm.pred!=Smarket$Direction)
```

```
## [1] 0.4664
```

```r
Smarket.2005 = Smarket[Smarket$Year==2005,] #A subset of Smarket with only the rows with Year = 2005
Direction.2005 = Smarket[Smarket$Year==2005,9] 
dim(Smarket.2005)
```

```
## [1] 252   9
```

```r
Train = Smarket[Smarket$Year !=2005,] #We use as Training set Year 2001-2004
dim(Train)
```

```
## [1] 998   9
```

For a more realistic setting, we fit the logistic model on Subset Year 2001-2004 (Train)
and test it on subset with Year=2005 (Smarket.2005)

```r
glm.fit_Train = glm(Direction~Lag1+Lag2+Lag3+Lag4+Lag5+Volume,data = Train,family=binomial)
glm.probs=predict(glm.fit_Train,Smarket.2005,type="response")
```

```r
glm.pred = rep("Down",252)
glm.pred[glm.probs>0.5] ="Up"
table(glm.pred,Direction.2005)
```

```
##         Direction.2005
## glm.pred Down Up
##     Down   77 97
##     Up     34 44
```

Now we calculate the performance on Test set (Direction.2005)


```r
print("The accuracy on test set is:")
```

```
## [1] "The accuracy on test set is:"
```

```r
mean(glm.pred==Direction.2005)
```

```
## [1] 0.4801587
```

```r
print("The test error rate is:")
```

```
## [1] "The test error rate is:"
```

```r
mean(glm.pred!=Direction.2005)
```

```
## [1] 0.5198413
```

The performance on Test set is worst than "By chance" classifier !

If we fit the logistic regression with all predictors (also predictors with high P-value), so predictors with no relationship with the responce, follows a deterioration in test error rate (predictors with high P-value increase the variance without a corresponding decrease in bias -->check Bias-Variance tradeoff!!!)

We re-fit the logisti regression with two best predictors (smaller P-value) -->Lag1, Lag2


```r
glm.fits_best = glm(Direction~Lag1+Lag2,data=Train,family=binomial)
glm.probs = predict(glm.fits_best,Smarket.2005,type="response")
glm.pred =rep("Down",252)
glm.pred[glm.probs>0.5]="Up"
table(glm.pred,Direction.2005)
```

```
##         Direction.2005
## glm.pred Down  Up
##     Down   35  35
##     Up     76 106
```


```r
print("The accuracy is:")
```

```
## [1] "The accuracy is:"
```

```r
mean(glm.pred==Direction.2005)
```

```
## [1] 0.5595238
```

```r
print("The test error rate is:")
```

```
## [1] "The test error rate is:"
```

```r
mean(glm.pred!=Direction.2005)
```

```
## [1] 0.4404762
```

With fitting the logistic model with predictors with lowest P-value we have better performance!!!

Now we want to predict the direction of the stock with Lag1 = 1.2 and Lag2 = 1.1. We want also predict the direction fo the stock with Lag1 = 1.5 and Lag2 = -0.8 .

```r
predict(glm.fits_best,newdata=data.frame(Lag1=c(1.2,1.5),Lag2=c(1.1,-0.8)),type ="response") #<0.5-->the market goes down in both scenarios
```

```
##         1         2 
## 0.4791462 0.4960939
```

