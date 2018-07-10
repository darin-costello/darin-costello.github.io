---
layout: post
title: Recipe Rating
categories: machineLearning java BYUcs478
date: 2016-04-10
thumbnail: images/recipeRating/recipeRate.jpg
---

This was the final project of my undergraduate machine learning course. I worked with [Emily Moragn][emilylkn] and [Heather Turnbow][heatherlkn], and together we came up with the project then collected, cleaned, and analysed the data. Our goal was to create a ML model that would take a recipe, and output a "goodness" rating. The idea was this model could then be used as a discriminator for a generator which would create new recipes.


Data
======

Our data came from [Yummly](https://developer.yummly.com/). We each got an academic api key, and maxed out all our calls. Instead of using the entire recipe, we chose to use only the summery given by Yummly. This allowed for more recipes per call. The features in the summary include time, salt, meaty, piquant, bitter, sour, sweet, number of ingredients, course, Holiday, cuisine, calories, fat, protein, sugar, and user rating. The taste specifiers,  salt, meaty, piquant, bitter, sour, sweet, are created by Yummly, while the nutrition facts are based on the ingredients. With this data, our goal was then to predict the rating based on the other features. We collected around 25,000 instances.

#### Cleaning and Feature Selection ####

As with any real world data set, ours was in need of cleaning. The first step was to eliminate outliers. For example, there was a recipe which took 91 hours. While this is certainly possible that is the correct, it was so beyond the average adversely affected our models. Thus, we set a 24 hour cooking limit and removed all instance that took longer. The nutrition facts were normalized by number of servings, however there where a few instances which claimed tens of thousands of calories. This was probably due to some recipe error, and we eliminated those as well.

We moved onto feature selection next. Our goal was to classify main courses, thus the course feature was unneeded. Also, the holiday feature was missing in all by 5% of our instances, and we found cutting it produced improvements. The final feature to go ws the cuisine type. Only 33% of the instances had data for this feature, and there where over 30 different values, giving about 140 instances per value. This was not enough to infer anything useful.   


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

The ratings ranged from 1 to four, however, 68.5% of the target ratings where 4 and 33% of them are 3, with very few of the rest. This means labeling everything as 4 would beat four of the 5 models. We then made the ratings into a binary problem changing (1,2,3) to 0 and (4,5) to 1. Random forest gave an accuracy of 78%, slightly above chance. Artificially balancing the dataset decreased the overall accuracy but allowed for more accurate negative predictions. 

Finally, after cleaning, and up sampling on the original the best results we got were with using a random tree. Giving:

|True Positive Rate| 0.723 |
|False Positive Rate | 0.284|
|Precision | 0.668 |
|Recall | 0.723|


The conclusion is nothing really got more that a few steps above guessing what would make a good recipes. The models here weren't powerful enough to predict accurately, so we couldn't really use them in recipe generation. Also, the homogeneity of ratings made it really difficult to tell the good from the bad. I think with more features, such as number of raters, ingredients ,and cooking instructions, and a bit more clever processing, our results could be improved.

[emilylkn]: https://www.linkedin.com/in/emily-morgan-407510b2/
[heatherlkn]: https://www.linkedin.com/in/heather-turnbow-863b78119/