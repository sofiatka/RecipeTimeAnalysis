# An Analytic Comparison of Short vs. Long Recipes
Project by Sofia Tkachenko

---

## Abstract
This project was done for the DSC 80: Practice and Application of Data Science class at UCSD. This report will categorize recipes in a dataset provided by Food.com as either "short" and "long," and then attempt to explore what distinguishes a short recipe from a long one beyond projected time to completion.

---

## Introduction
Cooking&mdash;and food in general&mdash;are integral to our daily lives, whether you like to experiment with new recipes or just need to make a meal to get through the day. The day can, unfortunately, get pretty busy. Balancing our time with cooking can lead us to compromise on what foods we try to make. Sometimes, you only have time to make a quick recipe, as opposed a very elaborate one. This led me to wonder: are recipes of varying lengths made equal? Do people generally share a preference for shorter recipes over long ones? What other factors, such as ratings, recipe descriptions, and ingredient counts  distinguish a short recipe from a long one? In short,
***<p style="text-align: center;">Is there a systematic way to distinguish short recipes from long ones?</p>***
To answer this question, I used two datasets scraped from <a href=https://www.food.com>food.com</a>. The first dataset contains 83782 unique recipes posted since 2008, including their nutritional info, associated tags, ingredients, and more. The dataset's columns are summarized below:

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


## Data Cleaning and Exploratory Data Analysis

## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis