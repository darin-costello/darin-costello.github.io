---
layout: post
title: Recipe Rating
categories: machineLearning java BYUcs478
date: 2016-04-10
thumbnail: images/recipeRating/recipeRate.jpg
---

This was the final project of my undergraduate machine learning course. I worked with [Emily Moragn][emilylkn] and [Heather Turnbow][heatherlkn], and together we came up with the project then collected, cleaned and analysed the data. Our goal was to create a ML model that would take a recipe, and output a goodness rating. The idea was this model could then be used as a discriminator for a generator which would create recipes.


Data
======

Our data came from [Yummly](https://developer.yummly.com/). We each got an academic api key, and maxed out all our calls. For simplicity and to reduce the number of api calls we had to make, we only used features included in yummly's recipe summary; Time, salt, meaty, piquant, bitter, sour, sweet, number of Ingredients, course, Holiday, Cuisine, calories, fat, protein, sugar, and rating. The taste specifiers are created by yummly based on the ingredients. With this data, our goal was then to predict the rating based on the other features. In the end we had roughly 25,000 recipe summaries to go off of.

#### Cleaning and Feature Selection ####

As with any real world data set, ours was in need of cleaning. The first was to eliminate outliers. For example, there was a recipe which took 91 hours. While this is certainly possible, since it was so beyond the average we set a 24 hour cooking limit. Next, was the caloric count. The data was already normalized by serving, but there where a few instances which claimed tens of thousands of calories per serving. This was probably due to some recipe error, and we eliminated those as well.

Our goal was to classify main courses, though the course feature was pretty homogenous, and it was dropped. Next, the holiday feature was missing in all by 5% of our instances, and we found cutting it produced gains. The final feature to go ws the cuisine type. Only 33% of the instances had this feature, and there where over 30 different values, giving about 140 instances per value. This was not enough to infer anything useful.   


Results
========

Our first pass, before the data cleaning,  looked like this:

|Model | Accuracy |
|:-----|:--------:|
|kNN | 0.617 |
|Naive Bayes | 0.632|
|Random Tree | 0.663|
|Gradient Boosting Machine| 0.688|
|Random Forest | 0.692 |

The ratings ranged from 1 to four, however, 68.5% of them where 4 and 33% of them are 3, with very few of the rest. this means labeling everything as 4 would beat four of the 5 models. We then made the ratings into a binary problem changing (1,2,3) to 0 and (4,5) to 1. Random forest gave an accuracy of 78%, slightly above chance. Balancing the dataset decreased accuracy but allowed for better negative predictions. 

After cleaning, and up sampling  on the original ratings gave

|True Positive Rate| 0.723 |
|False Positive Rate | 0.284|
|Precision | 0.668 |
|Recall | 0.723|

using a random tree.

The conclusion is nothing really got more that a few steps above guessing what would make a good recipes. I think with more features, such as ingredients and cooking instructions, this may be overcome.

[emilylkn]: https://www.linkedin.com/in/emily-morgan-407510b2/
[heatherlkn]: https://www.linkedin.com/in/heather-turnbow-863b78119/