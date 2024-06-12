# An Analytic Comparison of Short vs. Long Recipes
Project by Sofia Tkachenko

---

## Abstract
This project was done for the DSC 80: Practice and Application of Data Science class at UCSD. This project categorizes recipes in a dataset provided by Food.com as either "short" or "long." Then, it attempts to explore what distinguishes a short recipe from a long one beyond projected time to completion. Finally, a model will be introduced and improved to predict whether a recipe will be short or long.

---

## Introduction
Cooking&mdash;and food in general&mdash;is integral to our daily lives, whether you like to experiment with new recipes or just need to make a meal to get through the day. Unfortunately, the day can get pretty busy. Balancing our time with cooking can lead us to compromise on what foods we try to make. Sometimes, you only have time to make a quick recipe, as opposed a very elaborate one. This led me to wonder: are recipes of varying lengths made equal? Do people generally share a preference for shorter recipes over long ones? What other factors, such as ratings, recipe descriptions, and ingredient counts  distinguish a short recipe from a long one? ***Is there a systematic way to distinguish short recipes from long ones?***

To answer this question, I used two datasets scraped from [Food.com](https://www.food.com). The first dataset contains 83782 unique recipes posted since 2008, including their nutritional info, associated tags, ingredients, and more. The dataset's 12 columns are summarized below:

| Column         | Description                                                                                                                                                         |
|:---------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name           | Recipe name                                                                                                                                                         |
| id             | Unique recipe ID                                                                                                                                                    |
| minutes        | Length of the recipe, in minutes                                                                                                                                    |
| contributor_id | Unique ID of user submitting the recipe                                                                                                                             |
| submitted      | Date when recipe was submitted                                                                                                                                      |
| tags           | Recipe tags given by Food.com                                                                                                                                       |
| nutrition      | Nutritional information containing, in order: calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), and carbohydrates (PDV) |
| n_steps        | Number of steps                                                                                                                                                     |
| steps          | Description of steps                                                                                                                                                |
| description    | Description of recipe, submitted by user                                                                                                                            |
| ingredients    | Ingredients for the recipe                                                                                                                                          |
| n_ingredients  | Number of ingredients                                                                                                                                               |

The other dataset contained 731927 reviews submitted by users for these recipes. The 5 columns of this dataset are summarized below:

| Column    | Description                                |
|:----------|:-------------------------------------------|
| user_id   | ID of user submitting review               |
| recipe_id | ID of recipe being reviewed                |
| date      | Date of the review                         |
| rating    | Numerical rating (1-5) given to the recipe |
| review    | Text review submitted by user              |

To answer my proposed question, I (mainly) focused on the following columns:
- `minutes` to directly quantify the length of each recipe
    - For the purposes of this project, this was the **only** column used to label recipes as either *short* or *long*. Information on how the threshold was determined will be discussed in the Data Cleaning and Exploratory Data Analysis section of this report.
- `rating` to analyze how user sentiment differed across recipe lengths
- `n_steps` and `n_ingredients` to serve as indirect proxies of recipe length
- `tags` to understand how certain labels could be used to predict the length of a recipe
- `nutrition` to find associations between recipe length and information such as calorie and sugar content

I believe this is a valuable question to explore, because it can be difficult to find time to try new things. Recipe writers can use the insights of this project to make recipes more accessible and enjoyable to people with little time to make them. People trying recipes can better understand the trade-offs between attempting a short vs. long recipe. 

## Data Cleaning and Exploratory Data Analysis

## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis