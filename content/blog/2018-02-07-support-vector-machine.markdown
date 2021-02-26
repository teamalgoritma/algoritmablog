---
title: Support Vector Machine
author: Efa Hazna Latiefah
date: '2018-02-07'
slug: support-vector-machine
categories:
  - R
tags:
  - SVM
  - Machine Learning
description: ''
featured: 'banner_SVM.jpg'
featuredalt: ''
featuredpath: 'date'
linktitle: ''
type: post
---




## Support Vector Machine (SVM)

Support Vector Machine is a Supervised Machine Learning Algorithm which can be used both classification and regression. In this algorithm, each data item is plotted as point in n-dimensional space with the value of each feature being the value of a particular coordinate. Then, the algorithm perform classification by finding the hyper-plane that differentiate the two classes very well.

So how does SVM find the right hyperplane?. Let me give you example to understand how it works.

**Identify the right hyper-plane (Scenario-1)**

> Which line do you think separates the data?

<center> <img src="/img/svm/gambar1.jpg"> </center>

> And obviously the line B is the one that segregates the two classes better.

**Identify the right hyper-plane (Scenario-2)**

  Check the line that you think best separates the data. Obviously all three of them seem   to separates the data, but one is the best?

<center> <img src="/img/svm/gambar2.jpg"> </center>

  And SVM would choose the line C as the best separator. What this line does that the     other ones don't do, it maximizes the distance to the nearest point and it does this     relative to both classes. It's a line that maximizes the distance to the nearest points   in either class, that distance is often called margin.

**Identify the right hyper-plane (Scenario-3)**

<center> <img src="/img/svm/gambar3.jpg"> </center>


  SVM selects the hyper-plane which classifies the classes accurately prior to maximizing   margin.So For SVM , you are triying to classify correctly and subject to that         constraint, you maximize the margin. Here, hyper-plane B has a classification error and A   has classified all correctly. Therefore, the right hyper-plane is A.

**Identify the right hyper-plane (Scenario-4)**

  So sometimes for SVM, it seems impossible to do right job. For example, you might have   a data set just like this. In which clearly no decision surface exist that would   separate the two classes, you can think of the point down here as an outlier


<center> <img src="/img/svm/gambar4.jpg"> </center>


SVM has a feature to ignore outliers and find the hyper-plane that has maximum margin. Hence, we can say, SVM is robust to outliers.


<center> <img src="/img/svm/gambar5.jpg"> </center>


### The Kernel Trick

In the scenario below, we can't have linear hyper-plane between the two classes, so how does SVM classify these two classes? Till now, we have only looked at the linear hyper-plane.

<center> <img src="/img/svm/gambar6.jpg"> </center>


SVM can solve this problem Easily! It solves this problem by introducing additional feature. Here, we will add a new feature z=x^2+y^2. Now, let's plot the data points on axis x and z.


<center> <img src="/img/svm/gambar7.jpg"> </center>


When we look at the hyper-plane in original input space it looks like a circle.


<center> <img src="/img/svm/gambar10.jpg"> </center>

SVM has a technique called the kernel trick. These are functions which takes low dimensional input space and transform it to a higher dimensional space i.e. it converts not separable problem to separable problem, these functions are called kernels.

In R, there are 4 various options available with kernel. They are linear, radial, polynomial and sigmoid.

### Parameters in SVM

Parameters in SVM are kernel, c (cost function) and gamma. C is a parameter that controls tradeoff between smooth decision boundary and classifying training points correctly. Small C makes the cost of misclassificaiton low ("soft margin"), thus allowing more of them for the sake of wider cushion and conversely Large C makes the cost of misclassification high ('hard margin"), thus forcing the algorithm to explain the input data stricter and potentially overfit. In the other side, gamma parameter  defines how far the influence of a single training example reaches, if gamma has a low value, then that means that every point has a far reach, and conversely high values means that each training example only has a close reach.

### Strengths and Weaknesses of SVM 

**Strengths**

  1. Tradeoff between classifier complexity and error can be controlled explicitly
  2. SVMs work very well in practice, even with very small training
  3. It uses a subset of training points in the decision function (called support vectors), so it is also memory efficient

**Weaknesses**

  Need to choose a "good" kernel function
  
### How to determine the optimal parameters?

there's no way to figure out which kernel would do the best for a particular problem. The only way to choose the best kernel is to actually try out all possible kernels, and choose the one that does the best empirically. However, we can still look at some differences between various kernel functions, to have some rules of thumb. When data is not linearly separable the first choice is always a Radial kernels because the Radial kernel have the properties that tend to make Radial kernel better in general, for most problems.They are:

1. Translation invariance: The radial kernel is the only non-linear kernel that is translation invariant, 
2. Radial kernel is a function of the Euclidean distance between the points, whereas all other kernels are functions of inner product of the points.
3. Normalized: A kernel is said to be normalized if K(x,x)=1 for all x. This is true for only RBF kernel in the above list.

In order to find the optimum values of C and gamma parameter, we can use grid Search along with Cross Validation. In R , you can use tune.svm for finding the best model.

## SVM in R

The e1071 package in R is used to create Support Vector Machines with ease. 

### SVM for Classification

Let's apply what we learn above to do classification using 1000 past records of bank loans, each with a variable `default` that indicates whether the applicant did default on the loan.


```r
loans <- read.csv("data_input/loan.csv")
colnames(loans)
```

```
##  [1] "checking_balance"     "months_loan_duration" "credit_history"      
##  [4] "purpose"              "amount"               "savings_balance"     
##  [7] "employment_duration"  "percent_of_income"    "years_at_residence"  
## [10] "age"                  "other_credit"         "housing"             
## [13] "existing_loans_count" "job"                  "dependents"          
## [16] "phone"                "default"
```

```r
set.seed(10)
# select 900 values randomly from 1:1000
rand.vec <- sample(1:1000,900)
loans.train <- loans[rand.vec, ] 
loans.test <- loans[-rand.vec, ]
```



```r
library(e1071)
library(gmodels)
modelsvm1<-svm(default~., data=loans.train, kernel="linear")
modelsvm2<-svm(default~., data=loans.train, kernel="radial")

preds1<-predict(modelsvm1,loans.test[,-17])
preds2<-predict(modelsvm2,loans.test[,-17])
CrossTable(preds1,loans.test$default)
```

```
## 
##  
##    Cell Contents
## |-------------------------|
## |                       N |
## | Chi-square contribution |
## |           N / Row Total |
## |           N / Col Total |
## |         N / Table Total |
## |-------------------------|
## 
##  
## Total Observations in Table:  100 
## 
##  
##              | loans.test$default 
##       preds1 |        no |       yes | Row Total | 
## -------------|-----------|-----------|-----------|
##           no |        63 |        26 |        89 | 
##              |     0.640 |     1.139 |           | 
##              |     0.708 |     0.292 |     0.890 | 
##              |     0.984 |     0.722 |           | 
##              |     0.630 |     0.260 |           | 
## -------------|-----------|-----------|-----------|
##          yes |         1 |        10 |        11 | 
##              |     5.182 |     9.213 |           | 
##              |     0.091 |     0.909 |     0.110 | 
##              |     0.016 |     0.278 |           | 
##              |     0.010 |     0.100 |           | 
## -------------|-----------|-----------|-----------|
## Column Total |        64 |        36 |       100 | 
##              |     0.640 |     0.360 |           | 
## -------------|-----------|-----------|-----------|
## 
## 
```

```r
CrossTable(preds2,loans.test$default)
```

```
## 
##  
##    Cell Contents
## |-------------------------|
## |                       N |
## | Chi-square contribution |
## |           N / Row Total |
## |           N / Col Total |
## |         N / Table Total |
## |-------------------------|
## 
##  
## Total Observations in Table:  100 
## 
##  
##              | loans.test$default 
##       preds2 |        no |       yes | Row Total | 
## -------------|-----------|-----------|-----------|
##           no |        63 |        29 |        92 | 
##              |     0.288 |     0.513 |           | 
##              |     0.685 |     0.315 |     0.920 | 
##              |     0.984 |     0.806 |           | 
##              |     0.630 |     0.290 |           | 
## -------------|-----------|-----------|-----------|
##          yes |         1 |         7 |         8 | 
##              |     3.315 |     5.894 |           | 
##              |     0.125 |     0.875 |     0.080 | 
##              |     0.016 |     0.194 |           | 
##              |     0.010 |     0.070 |           | 
## -------------|-----------|-----------|-----------|
## Column Total |        64 |        36 |       100 | 
##              |     0.640 |     0.360 |           | 
## -------------|-----------|-----------|-----------|
## 
## 
```

the svm radial kernel perform better with the 79% accuracy. Let's tune the c and gamma parameter using 'tune.svm()' function.


```r
svmmodel<-tune.svm(default~., data = loans.test, kernel="radial",gamma =seq(0.01,0.1,len=10), cost=seq(5,50,len=10),tunecontrol=tune.control(cross=5))
summary(svmmodel)
```

```
## 
## Parameter tuning of 'svm':
## 
## - sampling method: 5-fold cross validation 
## 
## - best parameters:
##  gamma cost
##   0.04    5
## 
## - best performance: 0.28 
## 
## - Detailed performance results:
##     gamma cost error dispersion
## 1    0.01    5  0.35 0.13228757
## 2    0.02    5  0.34 0.11401754
## 3    0.03    5  0.31 0.12942179
## 4    0.04    5  0.28 0.14832397
## 5    0.05    5  0.30 0.11180340
## 6    0.06    5  0.30 0.09354143
## 7    0.07    5  0.29 0.12449900
## 8    0.08    5  0.30 0.12247449
## 9    0.09    5  0.29 0.11401754
## 10   0.10    5  0.30 0.09354143
## 11   0.01   10  0.35 0.12747549
## 12   0.02   10  0.30 0.16201852
## 13   0.03   10  0.32 0.10954451
## 14   0.04   10  0.35 0.11180340
## 15   0.05   10  0.34 0.13874437
## 16   0.06   10  0.34 0.13874437
## 17   0.07   10  0.32 0.10368221
## 18   0.08   10  0.31 0.08944272
## 19   0.09   10  0.31 0.08944272
## 20   0.10   10  0.30 0.07905694
## 21   0.01   15  0.34 0.12449900
## 22   0.02   15  0.31 0.12942179
## 23   0.03   15  0.36 0.09617692
## 24   0.04   15  0.35 0.14142136
## 25   0.05   15  0.35 0.11726039
## 26   0.06   15  0.33 0.10954451
## 27   0.07   15  0.32 0.08366600
## 28   0.08   15  0.31 0.08944272
## 29   0.09   15  0.31 0.08944272
## 30   0.10   15  0.30 0.07905694
## 31   0.01   20  0.34 0.14747881
## 32   0.02   20  0.34 0.10839742
## 33   0.03   20  0.37 0.12549900
## 34   0.04   20  0.33 0.13509256
## 35   0.05   20  0.34 0.11937336
## 36   0.06   20  0.33 0.10954451
## 37   0.07   20  0.32 0.08366600
## 38   0.08   20  0.31 0.08944272
## 39   0.09   20  0.31 0.08944272
## 40   0.10   20  0.30 0.07905694
## 41   0.01   25  0.33 0.18234583
## 42   0.02   25  0.35 0.11726039
## 43   0.03   25  0.36 0.15572412
## 44   0.04   25  0.35 0.11726039
## 45   0.05   25  0.34 0.11937336
## 46   0.06   25  0.33 0.10954451
## 47   0.07   25  0.32 0.08366600
## 48   0.08   25  0.31 0.08944272
## 49   0.09   25  0.31 0.08944272
## 50   0.10   25  0.30 0.07905694
## 51   0.01   30  0.35 0.16201852
## 52   0.02   30  0.37 0.09746794
## 53   0.03   30  0.34 0.13874437
## 54   0.04   30  0.33 0.11510864
## 55   0.05   30  0.34 0.11937336
## 56   0.06   30  0.33 0.10954451
## 57   0.07   30  0.32 0.08366600
## 58   0.08   30  0.31 0.08944272
## 59   0.09   30  0.31 0.08944272
## 60   0.10   30  0.30 0.07905694
## 61   0.01   35  0.34 0.14747881
## 62   0.02   35  0.37 0.09746794
## 63   0.03   35  0.35 0.14577380
## 64   0.04   35  0.33 0.11510864
## 65   0.05   35  0.34 0.11937336
## 66   0.06   35  0.33 0.10954451
## 67   0.07   35  0.32 0.08366600
## 68   0.08   35  0.31 0.08944272
## 69   0.09   35  0.31 0.08944272
## 70   0.10   35  0.30 0.07905694
## 71   0.01   40  0.32 0.13038405
## 72   0.02   40  0.38 0.10954451
## 73   0.03   40  0.36 0.12449900
## 74   0.04   40  0.33 0.11510864
## 75   0.05   40  0.34 0.11937336
## 76   0.06   40  0.33 0.10954451
## 77   0.07   40  0.32 0.08366600
## 78   0.08   40  0.31 0.08944272
## 79   0.09   40  0.31 0.08944272
## 80   0.10   40  0.30 0.07905694
## 81   0.01   45  0.30 0.11726039
## 82   0.02   45  0.36 0.13416408
## 83   0.03   45  0.35 0.12247449
## 84   0.04   45  0.33 0.11510864
## 85   0.05   45  0.34 0.11937336
## 86   0.06   45  0.33 0.10954451
## 87   0.07   45  0.32 0.08366600
## 88   0.08   45  0.31 0.08944272
## 89   0.09   45  0.31 0.08944272
## 90   0.10   45  0.30 0.07905694
## 91   0.01   50  0.31 0.09617692
## 92   0.02   50  0.37 0.14404860
## 93   0.03   50  0.35 0.12247449
## 94   0.04   50  0.33 0.11510864
## 95   0.05   50  0.34 0.11937336
## 96   0.06   50  0.33 0.10954451
## 97   0.07   50  0.32 0.08366600
## 98   0.08   50  0.31 0.08944272
## 99   0.09   50  0.31 0.08944272
## 100  0.10   50  0.30 0.07905694
```

The methods above give gamma= 0.03 and c=5 as optimal paramters. lets apply this result to our svm model to know wether the model would perform better or not.


```r
svmmodelrad<-svm(default~., data=loans.train, kernel="radial", gamma=0.03,cost=5)
predsrad<-predict(svmmodelrad,loans.test[,-17])

CrossTable(predsrad,loans.test$default)
```

```
## 
##  
##    Cell Contents
## |-------------------------|
## |                       N |
## | Chi-square contribution |
## |           N / Row Total |
## |           N / Col Total |
## |         N / Table Total |
## |-------------------------|
## 
##  
## Total Observations in Table:  100 
## 
##  
##              | loans.test$default 
##     predsrad |        no |       yes | Row Total | 
## -------------|-----------|-----------|-----------|
##           no |        61 |        23 |        84 | 
##              |     0.975 |     1.733 |           | 
##              |     0.726 |     0.274 |     0.840 | 
##              |     0.953 |     0.639 |           | 
##              |     0.610 |     0.230 |           | 
## -------------|-----------|-----------|-----------|
##          yes |         3 |        13 |        16 | 
##              |     5.119 |     9.100 |           | 
##              |     0.188 |     0.812 |     0.160 | 
##              |     0.047 |     0.361 |           | 
##              |     0.030 |     0.130 |           | 
## -------------|-----------|-----------|-----------|
## Column Total |        64 |        36 |       100 | 
##              |     0.640 |     0.360 |           | 
## -------------|-----------|-----------|-----------|
## 
## 
```
Although the tune method doesn't give the higher accuracy, but it reduce the number of false negatives.
