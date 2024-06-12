# An Analytic Comparison of Short vs. Long Recipes
Project by Sofia Tkachenko

---

## Abstract
This project was done for the DSC 80: Practice and Application of Data Science class at UCSD. This project categorizes recipes in a dataset provided by Food.com as either "short" or "long." Then, it attempts to explore what distinguishes a short recipe from a long one beyond projected time to completion. Finally, a model will be introduced and improved to predict whether a recipe will be short or long.

---

## Introduction
Cooking&mdash;and food in general&mdash;is integral to our daily lives, whether you like to experiment with new recipes or just need to make a meal to get through the day. Unfortunately, the day can get pretty busy. Balancing our time with cooking can lead us to compromise on what foods we try to make. Sometimes, you only have time to make a quick recipe, as opposed a very elaborate one. This led me to wonder: are recipes of varying lengths made equal? Do people generally share a preference for shorter recipes over long ones? What other factors, such as ratings, recipe descriptions, and ingredient counts  distinguish a short recipe from a long one? ***Is there a systematic way to distinguish short recipes from long ones?***

To answer this question, I used two datasets scraped from [Food.com](https://www.food.com). The first dataset, `recipes`, contains 83782 unique recipes posted since 2008, including their nutritional info, associated tags, ingredients, and more. The dataset's 12 columns are summarized below:

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

The other dataset, `interactions`, contained 731927 reviews submitted by users for these recipes. The 5 columns of this dataset are summarized below:

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

### Data Cleaning
Before beginning my analysis, I cleaned the data described earlier. The steps I took are listed below, in order:
1. Merging the two datasets together
    * A left merge with `recipes` and `interactions` allowed me to match reviews to each recipe under one DataFrame.
2. Filling all ratings of 0 with `NaN` values
    * After analyzing the `rating` column, I found that some ratings were stored as 0. Since ratings range from 1-5, this meant that ratings of 0 corresponded to missing values. If I were to average the ratings as-is, I would artificially skew the rating downwards. To avoid creating bias in the `rating` column, I opted to replace all 0 ratings with `NaN` values first.
3. Finding the average rating of each recipe
    * Since I only had individual ratings per recipe, finding average rating per recipe could provide insights into how people perceived a recipe overall.
    * After finding the average rating per recipe, I merged this information with the DataFrame formed after steps 1 and 2.
4. Making new columns for nutritional content
    * I found that several columns, included `tags` and `nutrition`, were strings given in the form of lists. I created a helper function that would take a string formatted as a list and return an array of the same information in its correct type (e.g. float). This result became more interpretable to Python. Since the `nutrition` column had a consistent format for all recipes, I created another helper function that would add individual columns in the DataFrame for each piece of nutritional info, such as calorie count and sugar (PDV).
    * These steps faciliated future analyses in my project, turning nutritional data into floats and allowing me to individually assess nutritional aspects in relation to recipe time.
5. Creating a categorical column called `length` to label each recipe as either *short* or *long*
    * This step allowed me to clearly label (my definition of) short and long recipes without having to repeatedly use the `minutes` column. By making this a separate column, I could save time for future analysis.
    * For the purposes of EDA, I felt it would be better to keep direct, string-type labels (i.e. "short" and "long"). This way, you can immediately understand the meaning from the value itself, not just the name of the column. However, I would later binarize this column (where `short==1` and `long==0`) using a `LabelTransformer` for my prediction problem.

My cleaned DataFrame had 234429 rows and 25 columns after the above changes. The first few rows of the DataFrame are shown below. **Note:** Text columns, such as description, steps, and ingredients, were purposefully excluded from this visualization. This was done to preserve readability.

| name                                 |   recipe_id |   minutes |   contributor_id | submitted   | tags                                                                                                                                                                                                                        |   n_steps |   n_ingredients |          user_id | date       |   rating |   avg_rating |   num_calories |   total_fat |   sugar |   sodium |   protein |   saturated_fat |   carbohydrates | length   |
|:-------------------------------------|------------:|----------:|-----------------:|:------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------:|----------------:|-----------------:|:-----------|---------:|-------------:|---------------:|------------:|--------:|---------:|----------:|----------------:|----------------:|:---------|
| 1 brownies in the world    best ever |      333281 |        40 |           985201 | 2008-10-27  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts', 'lunch', 'snacks', 'cookies-and-brownies', 'chocolate', 'bar-cookies', 'brownies', 'number-of-servings'] |        10 |               9 | 386585           | 2008-11-19 |        4 |            4 |          138.4 |          10 |      50 |        3 |         3 |              19 |               6 | long     |
| 1 in canada chocolate chip cookies   |      453467 |        45 |          1848091 | 2011-04-11  | ['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', 'north-american', 'for-large-groups', 'canadian', 'british-columbian', 'number-of-servings']                                                               |        12 |              11 | 424680           | 2012-01-26 |        5 |            5 |          595.1 |          46 |     211 |       22 |        13 |              51 |              26 | long     |
| 412 broccoli casserole               |      306168 |        40 |            50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                        |         6 |               9 |  29782           | 2008-12-31 |        5 |            5 |          194.8 |          20 |       6 |       32 |        22 |              36 |               3 | long     |
| 412 broccoli casserole               |      306168 |        40 |            50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                        |         6 |               9 |      1.19628e+06 | 2009-04-13 |        5 |            5 |          194.8 |          20 |       6 |       32 |        22 |              36 |               3 | long     |
| 412 broccoli casserole               |      306168 |        40 |            50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                        |         6 |               9 | 768828           | 2013-08-02 |        5 |            5 |          194.8 |          20 |       6 |       32 |        22 |              36 |               3 | long     |

### Univariate Analysis

I explored the distribution of various columns in my cleaned DataFrame. Most importantly, I analyzed the distribution of the `minutes` column to decide the threshold I should use to distinguish short recipes from long ones. To do this, I examined the distribution of `minutes`. First, I removed duplicate recipes that came as a result of using merge. This would allow me to see the distribution of `minutes` in reference to unique recipes. Then, I used the `describe()` function on the column to get a summary of statistics. The summary is shown below:

| Statistic   |        Minutes |
|:------------|---------------:|
| count       | 83782          |
| mean        |   115.031      |
| std         |  3990.87       |
| min         |     0          |
| 25%         |    20          |
| 50%         |    35          |
| 75%         |    65          |
| max         |     1.0512e+06 |

The standard deviation and maximum value are extremely large. This indicated a heavy right skew in my data, which I confirmed after plotting my first histogram of the `minutes` column. The vast majority of recipes appeared to fall within 2 hours, so I restricted my plot to only include recipes that took 5 or less hours (i.e. 300 minutes). This allowed me to see clearer trends in the data, since so few recipes fell under the extreme spectrum of the right tail. The modified histogram of the `minutes` column is shown below.
<iframe
  src="assets/minutes_hist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis