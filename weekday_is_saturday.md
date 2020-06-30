WeekDay Report
================
Xinyu Hu
6/30/2020

  - [Introduction](#introduction)
  - [About the data](#about-the-data)
  - [Summarization](#summarization)
      - [Simple Statistics](#simple-statistics)
      - [Simple Plots](#simple-plots)
      - [Plot with Response](#plot-with-response)
  - [Modeling](#modeling)
      - [Ensemble model](#ensemble-model)
      - [Linear Regression Model](#linear-regression-model)
  - [Conclusion](#conclusion)

# Introduction

This is a practice project of ST558 course at NC State. The data is
offered by [UCI Machine Learning
Repository](https://archive.ics.uci.edu/ml/datasets/Online+News+Popularity).  
The data set is made of statistics associsated with articles published
by Mashable. It includes 61 attributes, among which are 58 predictive
attributes, 2 non-predictive and 1 goal (reponse) field.  
The goal of this project, from my perspective, is practicing modeling
using R with both linear model and non linear model.The purpose of this
project is to find a relatively better way to predict *Number of Shares*
of those articles, using the provided associated statistics.

# About the data

**Row-wise, this analysis is about the weekday
    of**

    ## [1] weekday_is_saturday

``` r
colnames(News)
```

    ##  [1] "url"                           "timedelta"                     "n_tokens_title"                "n_tokens_content"              "n_unique_tokens"              
    ##  [6] "n_non_stop_words"              "n_non_stop_unique_tokens"      "num_hrefs"                     "num_self_hrefs"                "num_imgs"                     
    ## [11] "num_videos"                    "average_token_length"          "num_keywords"                  "data_channel_is_lifestyle"     "data_channel_is_entertainment"
    ## [16] "data_channel_is_bus"           "data_channel_is_socmed"        "data_channel_is_tech"          "data_channel_is_world"         "kw_min_min"                   
    ## [21] "kw_max_min"                    "kw_avg_min"                    "kw_min_max"                    "kw_max_max"                    "kw_avg_max"                   
    ## [26] "kw_min_avg"                    "kw_max_avg"                    "kw_avg_avg"                    "self_reference_min_shares"     "self_reference_max_shares"    
    ## [31] "self_reference_avg_sharess"    "weekday_is_monday"             "weekday_is_tuesday"            "weekday_is_wednesday"          "weekday_is_thursday"          
    ## [36] "weekday_is_friday"             "weekday_is_saturday"           "weekday_is_sunday"             "is_weekend"                    "LDA_00"                       
    ## [41] "LDA_01"                        "LDA_02"                        "LDA_03"                        "LDA_04"                        "global_subjectivity"          
    ## [46] "global_sentiment_polarity"     "global_rate_positive_words"    "global_rate_negative_words"    "rate_positive_words"           "rate_negative_words"          
    ## [51] "avg_positive_polarity"         "min_positive_polarity"         "max_positive_polarity"         "avg_negative_polarity"         "min_negative_polarity"        
    ## [56] "max_negative_polarity"         "title_subjectivity"            "title_sentiment_polarity"      "abs_title_subjectivity"        "abs_title_sentiment_polarity" 
    ## [61] "shares"

**Column-wise, this analysis includes 61 fields.** According to the
[data
description](https://archive.ics.uci.edu/ml/datasets/Online+News+Popularity),
from column \#3 to columns \#60 are the predictors. While I have no
knowledge about each of them, I can see there are a couple of
perspecives that we can group them.  
1\) *Column 3 to Column 13* are all numeric predictors. Among them,
Column 3 to Column Column 7, and column 12 are about tokens of the
article; Column 8 to Column Column 11, and column 13 are about the
links, image, video, and keywords of the articles.  
2\) *Column 14 to Column 19* are all about the topic, or channel of the
data. So here it will tag 7 types of channels: Lifestyle, Entertainment,
Business, Social Media, Tech and World.  
3\) *Column 20 to Column 28* are all about key word. For example, worst
keyword, best keyword, average keyword, etc.  
4\) *Column 29 to Column 31* are about self reference. They capture the
min, avg and max shares of referenced atritles in mashable.  
5\) *Colun 32 to Column 29* is aboue the timing of the article. Those
are flags to tag out if the article is published in Monday to Sunday,
and also if it’s on weekend.  
6\) *Column 40 to Column 44* are about the Closeness to LDA topics, with
topic 0 to topic 4 respectively.  
7\) *Colum 45 to Column 60* are all about [Natural Language Processing
(NLP)](https://en.wikipedia.org/wiki/Natural_language_processing) result
of the artcle. They include polarity, subjectivity, sentiment of the
article.

To start the analysis, we first look at Monday data, and use Rmarkdown
*Knit with Parameter* for all the weekdays. Also, a 70-30 split will be
made for modeling training and testing. Last but not least, I choose to
predict a binary response, which is dividing the shares into two groups
(\< 1400 and
\>=1400).

``` r
News_modelscope["NoLessThan1400"] <- ifelse(News_modelscope$shares >= 1400, 1, 0)
News_modelscope$NoLessThan1400 <- as.factor(News_modelscope$NoLessThan1400)
set.seed(3)
train <- sample(1:nrow(News_modelscope), size = nrow(News_modelscope)*0.7)
test <- dplyr::setdiff(1:nrow(News_modelscope), train)
DataTrain <- News_modelscope[train, ]
DataTest <- News_modelscope[test, ]
```

# Summarization

As all the predictors are numeric, now we can start to explore some
feature of them.

## Simple Statistics

1)  Variables describing the tokens are of great interest of me. So I
    want to know the simple statistics before I jump into prediction
    (also I can check if there is any
    missing).

<!-- end list -->

``` r
summary(DataTrain[,3:7])
```

    ##  n_tokens_title  n_tokens_content n_unique_tokens  n_non_stop_words n_non_stop_unique_tokens
    ##  Min.   : 5.00   Min.   :   0.0   Min.   :0.0000   Min.   :0.0000   Min.   :0.0000          
    ##  1st Qu.: 9.00   1st Qu.: 278.0   1st Qu.:0.4577   1st Qu.:1.0000   1st Qu.:0.6116          
    ##  Median :10.00   Median : 514.0   Median :0.5187   Median :1.0000   Median :0.6732          
    ##  Mean   :10.29   Mean   : 607.6   Mean   :0.5140   Mean   :0.9662   Mean   :0.6575          
    ##  3rd Qu.:12.00   3rd Qu.: 803.0   3rd Qu.:0.5924   3rd Qu.:1.0000   3rd Qu.:0.7362          
    ##  Max.   :17.00   Max.   :6505.0   Max.   :0.9574   Max.   :1.0000   Max.   :1.0000

2)  Another interesting perspectives are about the NLP metrics. So I
    perform summary function on them as well (also I can check if there
    is any
    missing):

<!-- end list -->

``` r
summary(DataTrain[,47:58])
```

    ##  global_rate_positive_words global_rate_negative_words rate_positive_words rate_negative_words avg_positive_polarity min_positive_polarity max_positive_polarity
    ##  Min.   :0.00000            Min.   :0.00000            Min.   :0.0000      Min.   :0.0000      Min.   :0.0000        Min.   :0.00000       Min.   :0.0000       
    ##  1st Qu.:0.02885            1st Qu.:0.01022            1st Qu.:0.6000      1st Qu.:0.1875      1st Qu.:0.3134        1st Qu.:0.03333       1st Qu.:0.6000       
    ##  Median :0.04082            Median :0.01582            Median :0.7111      Median :0.2778      Median :0.3681        Median :0.10000       Median :0.8000       
    ##  Mean   :0.04112            Mean   :0.01721            Mean   :0.6791      Mean   :0.2872      Mean   :0.3591        Mean   :0.09033       Mean   :0.7835       
    ##  3rd Qu.:0.05307            3rd Qu.:0.02227            3rd Qu.:0.8000      3rd Qu.:0.3824      3rd Qu.:0.4178        3rd Qu.:0.10000       3rd Qu.:1.0000       
    ##  Max.   :0.12162            Max.   :0.18493            Max.   :1.0000      Max.   :0.9048      Max.   :1.0000        Max.   :1.00000       Max.   :1.0000       
    ##  avg_negative_polarity min_negative_polarity max_negative_polarity title_subjectivity title_sentiment_polarity
    ##  Min.   :-1.0000       Min.   :-1.0000       Min.   :-1.0000       Min.   :0.0000     Min.   :-1.00000        
    ##  1st Qu.:-0.3382       1st Qu.:-0.8000       1st Qu.:-0.1250       1st Qu.:0.0000     1st Qu.: 0.00000        
    ##  Median :-0.2625       Median :-0.5000       Median :-0.1000       Median :0.1667     Median : 0.00000        
    ##  Mean   :-0.2691       Mean   :-0.5597       Mean   :-0.1054       Mean   :0.2833     Mean   : 0.09632        
    ##  3rd Qu.:-0.2000       3rd Qu.:-0.4000       3rd Qu.:-0.0500       3rd Qu.:0.5000     3rd Qu.: 0.20000        
    ##  Max.   : 0.0000       Max.   : 0.0000       Max.   : 0.0000       Max.   :1.0000     Max.   : 1.00000

## Simple Plots

1)  One group of predictors that could be very helpful is the channel
    for each articles. So Understanding how many articles in each
    channel will be a “good to
know”.

<!-- end list -->

``` r
DataTrain["data_channel"] <- ifelse(DataTrain$data_channel_is_lifestyle==1, 'Lifestyle',
                             ifelse(DataTrain$data_channel_is_entertainment==1, 'Entertainment',
                             ifelse(DataTrain$data_channel_is_bus==1,'Business',
                             ifelse(DataTrain$data_channel_is_socmed==1,'Social Media',
                             ifelse(DataTrain$data_channel_is_tech==1, 'Tech',
                             ifelse(DataTrain$data_channel_is_world==1,"World",NA))))))
library(ggplot2)
plot1<-ggplot(data=DataTrain,aes(data_channel))
plot1 + geom_bar(color="black", fill="blue", size=2)+labs(x = "Channels")
```

![](weekday_is_saturday_files/figure-gfm/Plots_data1-1.png)<!-- -->

2)  Another interesting point to know before modeling is LDA topics
    distribution. So perform the same exploratory visualization here:

<!-- end list -->

``` r
library(ggpubr)
ggarrange(plot2, plot3, plot4, plot5, plot6, ncol = 2, nrow = 3)
```

![](weekday_is_saturday_files/figure-gfm/Plots_data3-1.png)<!-- -->

## Plot with Response

1)  One interesting point is to look at the density/distribution of
    average token length by the flag of being shared more than 1,400
    times or not:

<!-- end list -->

``` r
library(wesanderson)
plot7<- ggplot(DataTrain,aes(x=average_token_length, fill=NoLessThan1400)) 
plot7 + geom_histogram(aes(y=..density..))+geom_density(adjust=0.25,alpha=0.5)+ labs(title="Average Token Length By Share Flag", x ="Average Token Length", y = "Count")
```

![](weekday_is_saturday_files/figure-gfm/Plots_data4-1.png)<!-- -->

2)  Another intersting point is to look at number of images, to see if
    the density/distribution are different across the shaing flag (1
    mean “Yes”, 0 means “No”):

<!-- end list -->

``` r
library(wesanderson)
plot8<- ggplot(DataTrain,aes(x=n_tokens_title, fill=NoLessThan1400)) 
plot8 + geom_histogram() + labs(title="Title Token Count By Share Flag",
        x ="Title Token Count", y = "Count")
```

![](weekday_is_saturday_files/figure-gfm/Plots_data5-1.png)<!-- -->

# Modeling

## Ensemble model

Here, I pick bagged tree as preferred approach. The model training and
tuning is based on
[**Cross-validation**](https://en.wikipedia.org/wiki/Cross-validation_\(statistics\)#:~:text=Cross%2Dvalidation%2C%20sometimes%20called%20rotation,to%20an%20independent%20data%20set.).
This method first splits the whole dataset into k folds (here I pick
10). Then, it trains model using 9 folds of data and tuning with 1
remaining fold of data. With inherent cross validation, The CV method
will make very good use of existing data, and have relatively good
result.

``` r
library(caret)
trCtrl <- trainControl(method = "cv", number = 10)
bagfitTree<- train (NoLessThan1400 ~ ., data = DataTrain[,c(3:60,62)], method = "treebag",
                       trControl = trCtrl, metric = "Accuracy")
bagfitTree
```

    ## Bagged CART 
    ## 
    ## 1717 samples
    ##   58 predictor
    ##    2 classes: '0', '1' 
    ## 
    ## No pre-processing
    ## Resampling: Cross-Validated (10 fold) 
    ## Summary of sample sizes: 1545, 1545, 1545, 1545, 1545, 1546, ... 
    ## Resampling results:
    ## 
    ##   Accuracy  Kappa    
    ##   0.747824  0.2288243

With the bagged tree model, we can apply to the **training dataset** to
see how good (*overfitting*) it
is:

``` r
BaggedTree_TrainingDatePrediction <- predict(bagfitTree, newdata=dplyr::select(DataTrain,-NoLessThan1400))
Result1 <- table(BaggedTree_TrainingDatePrediction,DataTrain$NoLessThan1400)
Result1
```

    ##                                  
    ## BaggedTree_TrainingDatePrediction    0    1
    ##                                 0  444    0
    ##                                 1    4 1269

The associated miss-classification rate is:

``` r
misClass1 <- 1 - sum(diag(Result1))/sum(Result1)
misClass1
```

    ## [1] 0.002329645

Also, we can apply the model to the **test dataset** to see how good
(*honest check*) it
is:

``` r
BaggedTree_TestDatePrediction <- predict(bagfitTree, newdata=dplyr::select(DataTest,-NoLessThan1400))
Result2 <- table(BaggedTree_TestDatePrediction,DataTest$NoLessThan1400)
Result2
```

    ##                              
    ## BaggedTree_TestDatePrediction   0   1
    ##                             0  46  49
    ##                             1 131 510

The associated miss-classification rate is:

``` r
misClass2 <- 1 - sum(diag(Result2))/sum(Result2)
misClass2
```

    ## [1] 0.2445652

## Linear Regression Model

I decide to use stepwise selection to choose the best regression model,
with AIC as the fit measurement. [**Akaike Information
Criterion**](https://en.wikipedia.org/wiki/Akaike_information_criterion)
is a very handy measurement to compare the model fit. It starts with a
set of candidate models, and then find the models’ corresponding AIC
values. Then it picks the model that minimizes the information loss
among the candidate models. It penalize number of predictors to avoid
overfitting.

``` r
library(MASS)
full_model <- glm(NoLessThan1400 ~., data = DataTrain[,c(3:60,62)], family=binomial)
step_model <- stepAIC(full_model, direction = "both", trace = FALSE)
summary(step_model)
```

    ## 
    ## Call:
    ## glm(formula = NoLessThan1400 ~ n_tokens_content + n_unique_tokens + 
    ##     n_non_stop_unique_tokens + num_hrefs + num_self_hrefs + average_token_length + 
    ##     num_keywords + data_channel_is_lifestyle + data_channel_is_socmed + 
    ##     data_channel_is_tech + data_channel_is_world + kw_min_min + 
    ##     kw_min_max + kw_max_max + kw_avg_max + kw_max_avg + kw_avg_avg + 
    ##     self_reference_min_shares + self_reference_max_shares + self_reference_avg_sharess + 
    ##     LDA_00 + LDA_02 + global_sentiment_polarity + rate_positive_words + 
    ##     avg_negative_polarity + title_sentiment_polarity + abs_title_sentiment_polarity + 
    ##     avg_positive_polarity, family = binomial, data = DataTrain[, 
    ##     c(3:60, 62)])
    ## 
    ## Deviance Residuals: 
    ##     Min       1Q   Median       3Q      Max  
    ## -2.8118  -0.8967   0.5130   0.7546   1.7228  
    ## 
    ## Coefficients:
    ##                                Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)                  -1.154e+00  8.043e-01  -1.434 0.151481    
    ## n_tokens_content              5.568e-04  2.361e-04   2.358 0.018363 *  
    ## n_unique_tokens               6.829e+00  2.088e+00   3.270 0.001075 ** 
    ## n_non_stop_unique_tokens     -4.118e+00  1.594e+00  -2.584 0.009765 ** 
    ## num_hrefs                     1.650e-02  7.028e-03   2.348 0.018863 *  
    ## num_self_hrefs               -6.192e-02  1.561e-02  -3.966 7.31e-05 ***
    ## average_token_length         -5.522e-01  1.795e-01  -3.075 0.002103 ** 
    ## num_keywords                  1.742e-01  3.776e-02   4.614 3.95e-06 ***
    ## data_channel_is_lifestyle     5.549e-01  2.821e-01   1.967 0.049207 *  
    ## data_channel_is_socmed        9.521e-01  3.261e-01   2.919 0.003508 ** 
    ## data_channel_is_tech          1.101e+00  2.012e-01   5.474 4.40e-08 ***
    ## data_channel_is_world         4.976e-01  2.883e-01   1.726 0.084411 .  
    ## kw_min_min                    3.888e-03  2.405e-03   1.617 0.105930    
    ## kw_min_max                   -1.979e-06  1.126e-06  -1.758 0.078828 .  
    ## kw_max_max                   -1.478e-06  8.409e-07  -1.758 0.078807 .  
    ## kw_avg_max                    1.432e-06  8.239e-07   1.738 0.082140 .  
    ## kw_max_avg                   -4.429e-05  1.650e-05  -2.684 0.007276 ** 
    ## kw_avg_avg                    2.922e-04  1.078e-04   2.711 0.006698 ** 
    ## self_reference_min_shares     3.414e-05  1.479e-05   2.308 0.021014 *  
    ## self_reference_max_shares     6.989e-06  5.854e-06   1.194 0.232525    
    ## self_reference_avg_sharess   -2.410e-05  1.626e-05  -1.482 0.138233    
    ## LDA_00                        2.250e+00  3.574e-01   6.297 3.04e-10 ***
    ## LDA_02                       -7.046e-01  4.185e-01  -1.684 0.092248 .  
    ## global_sentiment_polarity    -5.539e+00  1.670e+00  -3.318 0.000907 ***
    ## rate_positive_words           3.561e+00  8.889e-01   4.007 6.16e-05 ***
    ## avg_negative_polarity         1.771e+00  6.649e-01   2.664 0.007723 ** 
    ## title_sentiment_polarity      7.156e-01  2.647e-01   2.704 0.006855 ** 
    ## abs_title_sentiment_polarity -4.899e-01  3.077e-01  -1.592 0.111298    
    ## avg_positive_polarity         1.569e+00  1.095e+00   1.433 0.151920    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for binomial family taken to be 1)
    ## 
    ##     Null deviance: 1971.2  on 1716  degrees of freedom
    ## Residual deviance: 1687.2  on 1688  degrees of freedom
    ## AIC: 1745.2
    ## 
    ## Number of Fisher Scoring iterations: 5

Same as bagged tree model, we can apply the final model coming out from
stepwise select to the **training dataset** to see how good
(*overfitting*) it
is:

``` r
StepModel_TrainingDatePrediction <- predict(step_model, newdata=dplyr::select(DataTrain,-NoLessThan1400),type="response")
StepModel_TrainingDatePrediction <- as.numeric(StepModel_TrainingDatePrediction >= 0.5)
Result3 <- table(StepModel_TrainingDatePrediction,DataTrain$NoLessThan1400)
Result3 
```

    ##                                 
    ## StepModel_TrainingDatePrediction    0    1
    ##                                0  133   83
    ##                                1  315 1186

The associated miss-classification rate is:

``` r
misClass3 <- 1 - sum(diag(Result3))/sum(Result3)
misClass3
```

    ## [1] 0.2317997

Also, we can apply the model to the **test dataset** to see how good
(*honest check*) it
is:

``` r
StepModel_TestDatePrediction <- predict(step_model, newdata=dplyr::select(DataTest,-NoLessThan1400), type="response")
StepModel_TestDatePrediction <- as.numeric(StepModel_TestDatePrediction >= 0.5)
Result4 <- table(StepModel_TestDatePrediction,DataTest$NoLessThan1400)
Result4
```

    ##                             
    ## StepModel_TestDatePrediction   0   1
    ##                            0  42  53
    ##                            1 135 506

The associated miss-classification rate is:

``` r
misClass4 <- 1 - sum(diag(Result4))/sum(Result4)
misClass4
```

    ## [1] 0.2554348

# Conclusion

Based on my knowledge, The miss-classification rate of the test data set
is a very solid comparison measurement. So I will tag the model with
smallest test data miss-classification rate as the final model.

``` r
 if(misClass2 >= misClass4){
 FinalModel <-bagfitTree
noquote("The Better Model is Bagged Tree Model")
 }else{
 FinalModel <- step_model
noquote("The Better Model is (Step) Logistic Regression Model")}
```

    ## [1] The Better Model is (Step) Logistic Regression Model

Beyond this project, I think a better approach would be replicate the
whole process 100 times, obtain the mis-classification rate of both
model 100 times. Then compare to see how many times that Bagged Tree
beats logistric regression model, and how many times the other way
around. That would be very helpful to determine the stability of both
models performance.
