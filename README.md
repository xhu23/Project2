Project 2 HomePage
================
Xinyu Hu
6/28/2020

  - [Introduction](#introduction)
  - [About the data](#about-the-data)
  - [Model Selection](#model-selection)
  - [Weekday Models](#weekday-models)

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

# Model Selection

For Ensembled Model, I chose the [Bagged Tree
Model](https://en.wikipedia.org/wiki/Bootstrap_aggregating). Bagged Tree
is used because it can reduce the variance of a regular decision tree.
It create sseveral subsets of data from training sample chosen randomly
with replacement, and each is used to train their decision trees.
Average of all the predictions from different trees are used which is
more robust than a single decision tree. It improves the stability and
accuracy of statistical classification. Inside Bagged Tree, I use Cross
Validation (see more details in each report).

For Regression Model, I used [Logistic
Regression](https://towardsdatascience.com/logistic-regression-detailed-overview-46c4da4303bc).
This is mainly because the response is a binary variable. Using Logistic
Regression, instead of regular regression, can meet the prediction
assumption better. In addtion, instead of handpicking predictors, I use
stepwise selection based on AIC to make decisions on including/excluding
predictors. This will give me a quite good predictive model on the
response.

Lastly, I compare the model from Bagged Tree and Logistic Regression by
their performance on Test dataset. And made decision on which one is
better.

# Weekday Models

Description: Here I perform both Bagged Tree Model and Logistic
regression model on 7 weekdays. You should be able to see each report by
clicking on the links provided below. In each report, Simple statistics
and Plots are also provided.  
The analysis for [Monday is available here](weekday_is_monday.md).  
The analysis for [Tuesday is available here](weekday_is_tuesday.md).  
The analysis for [Wednesday is available
here](weekday_is_wednesday.md).  
The analysis for [Thursday is available here](weekday_is_thursday.md).  
The analysis for [Friday is available here](weekday_is_friday.md).  
The analysis for [Saturday is available here](weekday_is_saturday.md).  
The analysis for [Sunday is available here](weekday_is_sunday.md).
