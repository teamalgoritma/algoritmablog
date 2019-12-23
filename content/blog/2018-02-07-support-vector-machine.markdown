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
featured: ''
featuredalt: ''
featuredpath: ''
linktitle: ''
type: post
---




## Support Vector Machine (SVM)

Support Vector Machine is a Supervised Machine Learning Algorithm which can be used both classification and regression. In this algorithm, each data item is plotted as point in n-dimensional space with the value of each feature being the value of a particular coordinate. Then, the algorithm perform classification by finding the hyper-plane that differentiate the two classes very well.

So how does SVM find the right hyperplane?. Let me give you example to understand how it works.

**Identify the right hyper-plane (Scenario-1)**

> Which line do you think separates the data?

<img src="/img/svm/gambar1.jpg" style="display: block; margin: auto;" />

> And obviously the line B is the one that segregates the two classes better.

**Identify the right hyper-plane (Scenario-2)**

  Check the line that you think best separates the data. Obviously all three of them seem   to separates the data, but one is the best?

<img src="/img/svm/gambar2.jpg" style="display: block; margin: auto;" />

  And SVM would choose the line C as the best separator. What this line does that the     other ones don't do, it maximizes the distance to the nearest point and it does this     relative to both classes. It's a line that maximizes the distance to the nearest points   in either class, that distance is often called margin.

**Identify the right hyper-plane (Scenario-3)**

<img src="/img/svm/gambar3.jpg" style="display: block; margin: auto;" />


  SVM selects the hyper-plane which classifies the classes accurately prior to maximizing   margin.So For SVM , you are triying to classify correctly and subject to that         constraint, you maximize the margin. Here, hyper-plane B has a classification error and A   has classified all correctly. Therefore, the right hyper-plane is A.

**Identify the right hyper-plane (Scenario-4)**

  So sometimes for SVM, it seems impossible to do right job. For example, you might have   a data set just like this. In which clearly no decision surface exist that would   separate the two classes, you can think of the point down here as an outlier


<img src="/img/svm/gambar4.jpg" style="display: block; margin: auto;" />


SVM has a feature to ignore outliers and find the hyper-plane that has maximum margin. Hence, we can say, SVM is robust to outliers.


<img src="/img/svm/gambar5.jpg" style="display: block; margin: auto;" />

### The Kernel Trick

In the scenario below, we can't have linear hyper-plane between the two classes, so how does SVM classify these two classes? Till now, we have only looked at the linear hyper-plane.


<img src="/img/svm/gambar6.jpg" style="display: block; margin: auto;" />

SVM can solve this problem Easily! It solves this problem by introducing additional feature. Here, we will add a new feature z=x^2+y^2. Now, let's plot the data points on axis x and z.


<img src="/img/svm/gambar7.jpg" style="display: block; margin: auto;" />

When we look at the hyper-plane in original input space it looks like a circle.


<img src="/img/svm/gambar10.jpg" style="display: block; margin: auto;" />

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
```

```
## Warning: package 'e1071' was built under R version 3.5.3
```

```r
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
##           no |        61 |        12 |        73 | 
##              |     1.115 |     3.016 |           | 
##              |     0.836 |     0.164 |     0.730 | 
##              |     0.836 |     0.444 |           | 
##              |     0.610 |     0.120 |           | 
## -------------|-----------|-----------|-----------|
##          yes |        12 |        15 |        27 | 
##              |     3.016 |     8.154 |           | 
##              |     0.444 |     0.556 |     0.270 | 
##              |     0.164 |     0.556 |           | 
##              |     0.120 |     0.150 |           | 
## -------------|-----------|-----------|-----------|
## Column Total |        73 |        27 |       100 | 
##              |     0.730 |     0.270 |           | 
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
##           no |        69 |        17 |        86 | 
##              |     0.616 |     1.666 |           | 
##              |     0.802 |     0.198 |     0.860 | 
##              |     0.945 |     0.630 |           | 
##              |     0.690 |     0.170 |           | 
## -------------|-----------|-----------|-----------|
##          yes |         4 |        10 |        14 | 
##              |     3.786 |    10.235 |           | 
##              |     0.286 |     0.714 |     0.140 | 
##              |     0.055 |     0.370 |           | 
##              |     0.040 |     0.100 |           | 
## -------------|-----------|-----------|-----------|
## Column Total |        73 |        27 |       100 | 
##              |     0.730 |     0.270 |           | 
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
##   0.03    5
## 
## - best performance: 0.27 
## 
## - Detailed performance results:
##     gamma cost error dispersion
## 1    0.01    5  0.32 0.07582875
## 2    0.02    5  0.28 0.08366600
## 3    0.03    5  0.27 0.07582875
## 4    0.04    5  0.28 0.07582875
## 5    0.05    5  0.28 0.07582875
## 6    0.06    5  0.29 0.08215838
## 7    0.07    5  0.33 0.10954451
## 8    0.08    5  0.33 0.10954451
## 9    0.09    5  0.33 0.07582875
## 10   0.10    5  0.32 0.04472136
## 11   0.01   10  0.29 0.08215838
## 12   0.02   10  0.28 0.08366600
## 13   0.03   10  0.28 0.08366600
## 14   0.04   10  0.32 0.10368221
## 15   0.05   10  0.34 0.06519202
## 16   0.06   10  0.35 0.05000000
## 17   0.07   10  0.37 0.05700877
## 18   0.08   10  0.37 0.08366600
## 19   0.09   10  0.35 0.06123724
## 20   0.10   10  0.34 0.05477226
## 21   0.01   15  0.27 0.07582875
## 22   0.02   15  0.27 0.07582875
## 23   0.03   15  0.32 0.09082951
## 24   0.04   15  0.34 0.06519202
## 25   0.05   15  0.34 0.04183300
## 26   0.06   15  0.35 0.05000000
## 27   0.07   15  0.36 0.04183300
## 28   0.08   15  0.37 0.08366600
## 29   0.09   15  0.35 0.06123724
## 30   0.10   15  0.34 0.05477226
## 31   0.01   20  0.27 0.07582875
## 32   0.02   20  0.30 0.07905694
## 33   0.03   20  0.37 0.07582875
## 34   0.04   20  0.36 0.06519202
## 35   0.05   20  0.36 0.06519202
## 36   0.06   20  0.35 0.05000000
## 37   0.07   20  0.36 0.04183300
## 38   0.08   20  0.37 0.08366600
## 39   0.09   20  0.35 0.06123724
## 40   0.10   20  0.34 0.05477226
## 41   0.01   25  0.27 0.07582875
## 42   0.02   25  0.32 0.08366600
## 43   0.03   25  0.36 0.06519202
## 44   0.04   25  0.37 0.05700877
## 45   0.05   25  0.36 0.06519202
## 46   0.06   25  0.35 0.05000000
## 47   0.07   25  0.36 0.04183300
## 48   0.08   25  0.37 0.08366600
## 49   0.09   25  0.35 0.06123724
## 50   0.10   25  0.34 0.05477226
## 51   0.01   30  0.28 0.05700877
## 52   0.02   30  0.32 0.10368221
## 53   0.03   30  0.38 0.05700877
## 54   0.04   30  0.37 0.05700877
## 55   0.05   30  0.36 0.06519202
## 56   0.06   30  0.35 0.05000000
## 57   0.07   30  0.36 0.04183300
## 58   0.08   30  0.37 0.08366600
## 59   0.09   30  0.35 0.06123724
## 60   0.10   30  0.34 0.05477226
## 61   0.01   35  0.28 0.05700877
## 62   0.02   35  0.35 0.10000000
## 63   0.03   35  0.37 0.04472136
## 64   0.04   35  0.37 0.05700877
## 65   0.05   35  0.36 0.06519202
## 66   0.06   35  0.35 0.05000000
## 67   0.07   35  0.36 0.04183300
## 68   0.08   35  0.37 0.08366600
## 69   0.09   35  0.35 0.06123724
## 70   0.10   35  0.34 0.05477226
## 71   0.01   40  0.28 0.05700877
## 72   0.02   40  0.37 0.05700877
## 73   0.03   40  0.37 0.04472136
## 74   0.04   40  0.37 0.05700877
## 75   0.05   40  0.36 0.06519202
## 76   0.06   40  0.35 0.05000000
## 77   0.07   40  0.36 0.04183300
## 78   0.08   40  0.37 0.08366600
## 79   0.09   40  0.35 0.06123724
## 80   0.10   40  0.34 0.05477226
## 81   0.01   45  0.28 0.05700877
## 82   0.02   45  0.37 0.05700877
## 83   0.03   45  0.37 0.04472136
## 84   0.04   45  0.37 0.05700877
## 85   0.05   45  0.36 0.06519202
## 86   0.06   45  0.35 0.05000000
## 87   0.07   45  0.36 0.04183300
## 88   0.08   45  0.37 0.08366600
## 89   0.09   45  0.35 0.06123724
## 90   0.10   45  0.34 0.05477226
## 91   0.01   50  0.29 0.06519202
## 92   0.02   50  0.37 0.05700877
## 93   0.03   50  0.37 0.04472136
## 94   0.04   50  0.37 0.05700877
## 95   0.05   50  0.36 0.06519202
## 96   0.06   50  0.35 0.05000000
## 97   0.07   50  0.36 0.04183300
## 98   0.08   50  0.37 0.08366600
## 99   0.09   50  0.35 0.06123724
## 100  0.10   50  0.34 0.05477226
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
##           no |        62 |        10 |        72 | 
##              |     1.695 |     4.584 |           | 
##              |     0.861 |     0.139 |     0.720 | 
##              |     0.849 |     0.370 |           | 
##              |     0.620 |     0.100 |           | 
## -------------|-----------|-----------|-----------|
##          yes |        11 |        17 |        28 | 
##              |     4.360 |    11.788 |           | 
##              |     0.393 |     0.607 |     0.280 | 
##              |     0.151 |     0.630 |           | 
##              |     0.110 |     0.170 |           | 
## -------------|-----------|-----------|-----------|
## Column Total |        73 |        27 |       100 | 
##              |     0.730 |     0.270 |           | 
## -------------|-----------|-----------|-----------|
## 
## 
```
Although the tune method doesn't give the higher accuracy, but it reduce the number of false negatives.
