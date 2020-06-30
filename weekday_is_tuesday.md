RMarkDown
================
Xinyu Hu
6/28/2020

  - [About the data](#about-the-data)
  - [Summarization](#summarization)
      - [Simple Statistics](#simple-statistics)
      - [Simple Plots](#simple-plots)
  - [Modeling](#modeling)
      - [Ensemble model](#ensemble-model)
      - [Linear Regression Model](#linear-regression-model)
      - [Model Selection](#model-selection)

\#Introduction

This is a practice project of ST558 course at NC State. The data is
offered by [UCI Machine Learning
Repository](https://archive.ics.uci.edu/ml/datasets/Online+News+Popularity).  
The data set is made of statistics associsated with articles published
by Mashable. It includes 61 attributes, among which are 58 predictive
attributes, 2 non-predictive and 1 goal (reponse) field.  
The goal of this project, from my perspective, is practicing modeling
using R with both linear model and non linear model.The purpose of this
project is to find a relatively better way to predict *Number of Shares*
of those articles, using the provided associated
    statistics.

# About the data

``` r
colnames(News)
```

    ##  [1] "url"                           "timedelta"                     "n_tokens_title"                "n_tokens_content"             
    ##  [5] "n_unique_tokens"               "n_non_stop_words"              "n_non_stop_unique_tokens"      "num_hrefs"                    
    ##  [9] "num_self_hrefs"                "num_imgs"                      "num_videos"                    "average_token_length"         
    ## [13] "num_keywords"                  "data_channel_is_lifestyle"     "data_channel_is_entertainment" "data_channel_is_bus"          
    ## [17] "data_channel_is_socmed"        "data_channel_is_tech"          "data_channel_is_world"         "kw_min_min"                   
    ## [21] "kw_max_min"                    "kw_avg_min"                    "kw_min_max"                    "kw_max_max"                   
    ## [25] "kw_avg_max"                    "kw_min_avg"                    "kw_max_avg"                    "kw_avg_avg"                   
    ## [29] "self_reference_min_shares"     "self_reference_max_shares"     "self_reference_avg_sharess"    "weekday_is_monday"            
    ## [33] "weekday_is_tuesday"            "weekday_is_wednesday"          "weekday_is_thursday"           "weekday_is_friday"            
    ## [37] "weekday_is_saturday"           "weekday_is_sunday"             "is_weekend"                    "LDA_00"                       
    ## [41] "LDA_01"                        "LDA_02"                        "LDA_03"                        "LDA_04"                       
    ## [45] "global_subjectivity"           "global_sentiment_polarity"     "global_rate_positive_words"    "global_rate_negative_words"   
    ## [49] "rate_positive_words"           "rate_negative_words"           "avg_positive_polarity"         "min_positive_polarity"        
    ## [53] "max_positive_polarity"         "avg_negative_polarity"         "min_negative_polarity"         "max_negative_polarity"        
    ## [57] "title_subjectivity"            "title_sentiment_polarity"      "abs_title_subjectivity"        "abs_title_sentiment_polarity" 
    ## [61] "shares"

As we know before, there are 61 fields. According to the [data
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
(\< 1400 and \>=1400).

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

    ##  n_tokens_title  n_tokens_content n_unique_tokens    n_non_stop_words   n_non_stop_unique_tokens
    ##  Min.   : 4.00   Min.   :   0.0   Min.   :  0.0000   Min.   :   0.000   Min.   :  0.0000        
    ##  1st Qu.: 9.00   1st Qu.: 246.0   1st Qu.:  0.4729   1st Qu.:   1.000   1st Qu.:  0.6294        
    ##  Median :10.00   Median : 395.0   Median :  0.5404   Median :   1.000   Median :  0.6911        
    ##  Mean   :10.47   Mean   : 537.7   Mean   :  0.6500   Mean   :   1.148   Mean   :  0.7836        
    ##  3rd Qu.:12.00   3rd Qu.: 690.2   3rd Qu.:  0.6092   3rd Qu.:   1.000   3rd Qu.:  0.7537        
    ##  Max.   :19.00   Max.   :7002.0   Max.   :701.0000   Max.   :1042.000   Max.   :650.0000

2)  Another interesting perspectives are about the NLP metrics. So I
    perform summary function on them as well (also I can check if there
    is any
    missing):

<!-- end list -->

``` r
summary(DataTrain[,47:58])
```

    ##  global_rate_positive_words global_rate_negative_words rate_positive_words rate_negative_words avg_positive_polarity min_positive_polarity
    ##  Min.   :0.00000            Min.   :0.000000           Min.   :0.0000      Min.   :0.0000      Min.   :0.0000        Min.   :0.00000      
    ##  1st Qu.:0.02857            1st Qu.:0.009339           1st Qu.:0.6000      1st Qu.:0.1818      1st Qu.:0.3046        1st Qu.:0.05000      
    ##  Median :0.03902            Median :0.015204           Median :0.7117      Median :0.2778      Median :0.3563        Median :0.10000      
    ##  Mean   :0.03955            Mean   :0.016374           Mean   :0.6872      Mean   :0.2840      Mean   :0.3511        Mean   :0.09469      
    ##  3rd Qu.:0.04975            3rd Qu.:0.021540           3rd Qu.:0.8000      3rd Qu.:0.3786      3rd Qu.:0.4071        3rd Qu.:0.10000      
    ##  Max.   :0.11458            Max.   :0.135294           Max.   :1.0000      Max.   :1.0000      Max.   :0.8333        Max.   :0.70000      
    ##  max_positive_polarity avg_negative_polarity min_negative_polarity max_negative_polarity title_subjectivity title_sentiment_polarity
    ##  Min.   :0.0000        Min.   :-1.0000       Min.   :-1.0000       Min.   :-1.0000       Min.   :0.000      Min.   :-1.00000        
    ##  1st Qu.:0.6000        1st Qu.:-0.3250       1st Qu.:-0.7000       1st Qu.:-0.1250       1st Qu.:0.000      1st Qu.: 0.00000        
    ##  Median :0.8000        Median :-0.2500       Median :-0.5000       Median :-0.1000       Median :0.125      Median : 0.00000        
    ##  Mean   :0.7523        Mean   :-0.2556       Mean   :-0.5119       Mean   :-0.1068       Mean   :0.281      Mean   : 0.07056        
    ##  3rd Qu.:1.0000        3rd Qu.:-0.1823       3rd Qu.:-0.3000       3rd Qu.:-0.0500       3rd Qu.:0.500      3rd Qu.: 0.13636        
    ##  Max.   :1.0000        Max.   : 0.0000       Max.   : 0.0000       Max.   : 0.0000       Max.   :1.000      Max.   : 1.00000

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

![](weekday_is_tuesday_files/figure-gfm/Plots_data1-1.png)<!-- -->

2)  Another interesting point to know before modeling is LDA topics
    distribution. So perform the same exploratory visualization here:

<!-- end list -->

``` r
library(ggpubr)
ggarrange(plot2, plot3, plot4, plot5, plot6, ncol = 2, nrow = 3)
```

![](weekday_is_tuesday_files/figure-gfm/Plots_data3-1.png)<!-- -->

# Modeling

## Ensemble model

Here, I pick bagged tree as preferred approach.

``` r
library(caret)
trCtrl <- trainControl(method = "cv", number = 10)
bagfitTree<- train (NoLessThan1400 ~ ., data = DataTrain[,c(3:60,62)], method = "treebag",
                       trControl = trCtrl, metric = "Accuracy")
bagfitTree
```

    ## Bagged CART 
    ## 
    ## 5912 samples
    ##   58 predictor
    ##    2 classes: '0', '1' 
    ## 
    ## No pre-processing
    ## Resampling: Cross-Validated (10 fold) 
    ## Summary of sample sizes: 5320, 5320, 5321, 5321, 5320, 5321, ... 
    ## Resampling results:
    ## 
    ##   Accuracy   Kappa    
    ##   0.6229656  0.2458772

With the bagged tree model, we can apply to the training dataset to see
how good it
is:

``` r
BaggedTree_TrainingDatePrediction <- predict(bagfitTree, newdata=dplyr::select(DataTrain,-NoLessThan1400))
Result1 <- table(BaggedTree_TrainingDatePrediction,DataTrain$NoLessThan1400)
Result1
```

    ##                                  
    ## BaggedTree_TrainingDatePrediction    0    1
    ##                                 0 2988    1
    ##                                 1    5 2918

The associated miss-classification rate is:

``` r
misClass1 <- 1 - sum(diag(Result1))/sum(Result1)
misClass1
```

    ## [1] 0.001014885

Also, we can apply the model to the test dataset to see how good it
is:

``` r
BaggedTree_TestDatePrediction <- predict(bagfitTree, newdata=dplyr::select(DataTest,-NoLessThan1400))
Result2 <- table(BaggedTree_TestDatePrediction,DataTest$NoLessThan1400)
Result2
```

    ##                              
    ## BaggedTree_TestDatePrediction   0   1
    ##                             0 465 246
    ##                             1 281 486

The associated miss-classification rate is:

``` r
misClass2 <- 1 - sum(diag(Result2))/sum(Result2)
misClass2
```

    ## [1] 0.3565629

## Linear Regression Model

I decide to use Stepwise selection to choose the best regression model,
with AIC as the fit measurement.

``` r
library(MASS)
full_model <- glm(NoLessThan1400 ~., data = DataTrain[,c(3:60,62)], family=binomial)
step_model <- stepAIC(full_model, direction = "both", trace = FALSE)
summary(step_model)
```

    ## 
    ## Call:
    ## glm(formula = NoLessThan1400 ~ n_non_stop_unique_tokens + num_hrefs + 
    ##     num_self_hrefs + average_token_length + data_channel_is_entertainment + 
    ##     data_channel_is_socmed + data_channel_is_tech + data_channel_is_world + 
    ##     kw_min_min + kw_avg_max + kw_min_avg + kw_max_avg + kw_avg_avg + 
    ##     self_reference_min_shares + self_reference_max_shares + LDA_00 + 
    ##     LDA_01 + LDA_02 + LDA_03 + LDA_04 + global_subjectivity + 
    ##     global_rate_positive_words + rate_positive_words + rate_negative_words + 
    ##     title_sentiment_polarity, family = binomial, data = DataTrain[, 
    ##     c(3:60, 62)])
    ## 
    ## Deviance Residuals: 
    ##     Min       1Q   Median       3Q      Max  
    ## -2.7947  -1.0439  -0.6454   1.0728   1.8802  
    ## 
    ## Coefficients:
    ##                                 Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)                    7.841e+02  2.814e+02   2.786 0.005329 ** 
    ## n_non_stop_unique_tokens      -1.188e+00  3.092e-01  -3.843 0.000122 ***
    ## num_hrefs                      1.107e-02  3.192e-03   3.468 0.000525 ***
    ## num_self_hrefs                -1.476e-02  8.993e-03  -1.641 0.100714    
    ## average_token_length          -2.801e-01  1.048e-01  -2.673 0.007523 ** 
    ## data_channel_is_entertainment -2.967e-01  1.096e-01  -2.708 0.006760 ** 
    ## data_channel_is_socmed         9.084e-01  1.376e-01   6.600 4.10e-11 ***
    ## data_channel_is_tech           6.961e-01  1.172e-01   5.942 2.82e-09 ***
    ## data_channel_is_world          2.227e-01  1.383e-01   1.610 0.107470    
    ## kw_min_min                     1.748e-03  5.031e-04   3.474 0.000513 ***
    ## kw_avg_max                    -1.489e-06  3.085e-07  -4.825 1.40e-06 ***
    ## kw_min_avg                    -1.199e-04  3.540e-05  -3.386 0.000709 ***
    ## kw_max_avg                    -1.188e-04  1.359e-05  -8.742  < 2e-16 ***
    ## kw_avg_avg                     8.870e-04  7.450e-05  11.906  < 2e-16 ***
    ## self_reference_min_shares      4.279e-06  2.894e-06   1.479 0.139266    
    ## self_reference_max_shares      1.336e-06  9.429e-07   1.417 0.156500    
    ## LDA_00                        -7.850e+02  2.814e+02  -2.790 0.005277 ** 
    ## LDA_01                        -7.857e+02  2.814e+02  -2.792 0.005235 ** 
    ## LDA_02                        -7.862e+02  2.814e+02  -2.794 0.005210 ** 
    ## LDA_03                        -7.859e+02  2.814e+02  -2.793 0.005226 ** 
    ## LDA_04                        -7.856e+02  2.814e+02  -2.792 0.005244 ** 
    ## global_subjectivity            6.481e-01  3.640e-01   1.780 0.075012 .  
    ## global_rate_positive_words    -3.593e+00  2.216e+00  -1.621 0.104985    
    ## rate_positive_words            1.666e+00  6.049e-01   2.755 0.005870 ** 
    ## rate_negative_words            1.480e+00  5.948e-01   2.487 0.012865 *  
    ## title_sentiment_polarity       3.499e-01  1.073e-01   3.262 0.001106 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for binomial family taken to be 1)
    ## 
    ##     Null deviance: 8194.8  on 5911  degrees of freedom
    ## Residual deviance: 7499.9  on 5886  degrees of freedom
    ## AIC: 7551.9
    ## 
    ## Number of Fisher Scoring iterations: 10

Same as bagged tree model, we can apply the final model coming out from
stepwise select to the training dataset to see how good it
is:

``` r
StepModel_TrainingDatePrediction <- predict(step_model, newdata=dplyr::select(DataTrain,-NoLessThan1400),type="response")
StepModel_TrainingDatePrediction <- as.numeric(StepModel_TrainingDatePrediction >= 0.5)
Result3 <- table(StepModel_TrainingDatePrediction,DataTrain$NoLessThan1400)
Result3 
```

    ##                                 
    ## StepModel_TrainingDatePrediction    0    1
    ##                                0 2003 1112
    ##                                1  990 1807

The associated miss-classification rate is:

``` r
misClass3 <- 1 - sum(diag(Result3))/sum(Result3)
misClass3
```

    ## [1] 0.355548

Also, we can apply the model to the test dataset to see how good it
is:

``` r
StepModel_TestDatePrediction <- predict(step_model, newdata=dplyr::select(DataTest,-NoLessThan1400), type="response")
StepModel_TestDatePrediction <- as.numeric(StepModel_TestDatePrediction >= 0.5)
Result4 <- table(StepModel_TestDatePrediction,DataTest$NoLessThan1400)
Result4
```

    ##                             
    ## StepModel_TestDatePrediction   0   1
    ##                            0 481 248
    ##                            1 265 484

The associated miss-classification rate is:

``` r
misClass4 <- 1 - sum(diag(Result4))/sum(Result4)
misClass4
```

    ## [1] 0.3470907

## Model Selection

Based on my knowledge, The miss-classification rate of the test data set
is a very solid comparison measurement. So I will tag the model with
smallest test data miss-classification rate as the final model.

``` r
 if(misClass2 >= misClass4){
 FinalModel <-bagfitTree
 print("The Better Model is Bagged Tree Model")
 }else{
 FinalModel <- step_model
print("The Better Model is Bagged Tree Model")}
```

    ## [1] "The Better Model is Bagged Tree Model"
