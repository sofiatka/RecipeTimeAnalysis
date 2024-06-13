# An Analytic Comparison of Short vs. Long Recipes
Project by Sofia Tkachenko

---

## Abstract
This project was done for the DSC 80: Practice and Application of Data Science class at UCSD. This project categorizes recipes in a dataset provided by Food.com as either "short" or "long." Then, it attempts to explore what distinguishes a short recipe from a long one beyond projected time to completion. Finally, a model will be introduced and improved to predict whether a recipe will be short or long.

---

## Introduction
Cooking&mdash;and food in general&mdash;is integral to our daily lives, whether you like to experiment with new recipes or just need to make a meal to get through the day. Unfortunately, the day can get pretty busy. Balancing our time with cooking can lead us to compromise on what foods we try to make. Sometimes, you only have time to make a quick recipe, as opposed a very elaborate one. This led me to wonder: are recipes of varying lengths made equal? Do people generally share a preference for shorter recipes over long ones? What other factors, such as ratings, recipe tags, and ingredient counts  distinguish a short recipe from a long one? ***Is there a systematic way to distinguish short recipes from long ones, without using the length of a recipe itself?***

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

I believe this is a valuable question to explore, because it can be difficult to find time to try new things. Recipe writers can use the insights of this project to make recipes more accessible and enjoyable to people with little time to make them. People trying recipes can better understand the trade-offs between attempting a short vs. long recipe. Finally, understanding the fundamental differences between a long and short recipe could help you time manage in the future if you are unsure how long a certain recipe might take.

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
    * I found that several columns, including `tags` and `nutrition`, were strings given in the form of lists. I created a helper function that would take a string formatted as a list and return an array of the same information in its correct type (e.g. float). This result became more interpretable to Python. Since the `nutrition` column had a consistent format for all recipes, I created another helper function that would add individual columns in the DataFrame for each piece of nutritional info, such as calorie count and sugar (PDV).
    * These steps faciliated future analyses in my project, turning nutritional data into floats and allowing me to individually assess nutritional aspects in relation to recipe time.
5. Creating a categorical column called `length` to label each recipe as either *short* or *long*
    * *Note that the details of this step are discussed further under univariate analysis. This section is only meant to summarize the changes I made overall during the cleaning process.*
    * This step allowed me to clearly label (my definition of) short and long recipes without having to repeatedly use the `minutes` column. By making this a separate column, I could save time for future analysis.
    * For the purposes of EDA, I felt it would be better to keep direct, string-type labels (i.e. "short" and "long"). This way, you can immediately understand the meaning from the value itself, not just the name of the column. However, I would later binarize this column (where `short==1` and `long==0`) using a `LabelTransformer` for my prediction problem.

My cleaned DataFrame had 234429 rows and 25 columns after the above changes. The first few rows of the DataFrame are shown below. Be aware that there are repetitions of recipe names! This is because this is a merged dataset of reviews *and* recipes. In the DataFrame below, for example, we have 3 distinct reviews for the same recipe, "broccoli casserole." *This is not an error, and is an expected behavior of this dataset.*

**Note:** Text columns, such as description, steps, and ingredients, were purposefully excluded from this visualization. They were not really used in my analysis either way. This was done to preserve readability. Scroll to see all columns.

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

I observed that a significant portion of recipes took less than 2 hours to complete. The most common recipe length was between 30-40 minutes. There are many decently short recipes, and a few extremely long ones (the longest recipe in the dataset, which isn't captured in the reduced histogram, takes over 1,000,000 minutes to complete!).

Because of this outlier-induced right skew, I decided that my threshold to determine short and long recipes should be robust to outliers. Since the mean is heavily influenced by outliers, I decided to use the **median** instead, which I found to be **35 minutes**. In the context of daily life, I consider this threshold to be reasonable. If you only have an hour for lunch, you will likely not consider a recipe to be "short" if it takes over 35 minutes to complete. You still need time to eat and clean up, after all! For this reason, I decided to use the following threshold:

* Short recipes: 35 or less minutes to complete
* Long recipes: More than 35 minutes to complete

This is the metric I used to create the `length` column described in Step 5 of my Data Cleaning. Performing univariate analysis on the `minutes` column allowed me to determine a reasonable threshold that I could use for the rest of the project.

> An additional benefit of using the median is its ability to evenly split data. This ensures that we can compare short vs. long recipes without worrying too much about drastically differing sample sizes and hence differing variations. Issues with class imbalance (a common problem in classification) can also be reduced using this approach.


### Bivariate Analysis

#### Rating, Conditional on Recipe Length
One question I was curious about concerned the distribution of ratings conditional on recipe length. Do people enjoy short recipes more than long ones? Are they more likely to give a 5-star review to a short recipe or a long one? To examine this relationship, I treated the `rating` column as an ordinal categorical feature and constructed a grouped bar plot, conditional on the `length` column I engineered earlier.
<iframe
  src="assets/rating_cond_bar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The most common rating in either group is overwhelmingly 5 stars, followed (mostly) chronologically by all other ratings. Short recipes (i.e. those taking 35 minutes or less) seem to get 5-star ratings more often than long recipes (78% v.s. 76.6%). Long recipes seem more likely than short recipes to have 4 or less stars. The difference does not appear to be very large, but it is noticeable and has a consistent pattern. It seems reasonable to hypothesize that short recipes *could* receive fundamentally different ratings compared to long ones.

#### Calories, Conditional on Recipe Length
I also examined how the distribution of calories differed across short and long recipes. Since calories are a continuous quantitative variable, I used an overlaid histogram to perform this bivariate analysis.
<iframe
  src="assets/calorie_hist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The shapes of the calorie distributions are similar, but the histogram of long recipes is significantly shorter in height than the histogram for short recipes. Since there is a rough 50-50 split between short and long recipes, we would expect the histograms to overlap more if there were no difference in calories by group. Because the heights are so different, this suggests that long recipes may actually have ***higher*** calorie counts than short recipes. Long recipes may have recipes with higher calorie counts that would skew the mean more than in short recipes.

Indeed, using an aggregation technique, we find that (unique) long recipes have **509 calories on average** versus only **353 calories on average** for short recipes. This is useful to know if we wish to distinguish long recipes from short ones.

#### Relationship between Ingredient and Step Counts
There are other ways to describe recipe length than simply using time to completion. Two such variables are `n_ingredients` (number of ingredients in a recipe) and `n_steps` (number of steps in a recipe). Since these are quantitative variables, we can compare them in a scatterplot to see if there exists an association. Logically, we would expect a positive association: if you have more ingredients, you may need more steps to properly incorporate them into a recipe.
<iframe
  src="assets/ing_step_scat.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

As predicted, there is an overall upward trend. Recipes with more steps tend to use more ingredients. However, the association does not appear to be very strong; there is a lot of variation across ingredient-step count pairs. Note that we are using only unique recipes from the dataset, so each point corresponds to a distinct recipe.

> Ingredient and step counts do not appear to be closely correlated, which can be useful in the future for building a classification model. As features, `n_steps` and `n_ingredients` are semantically related to recipe length, but are not so correlated with each other to produce multicollinearity. This will avoid redundancy in the model.

### Interesting Aggregates
I created a pivot table to compare average step counts (`n_step`) for each rating-length pair. For example, we can observe that there were *8.59 steps* on average in *short* recipes that received *1-star* reviews.

|   rating |    long |   short |
|---------:|--------:|--------:|
|        1 | 12.4263 | 8.58749 |
|        2 | 12.5775 | 8.54529 |
|        3 | 11.8613 | 8.02546 |
|        4 | 11.5324 | 7.80751 |
|        5 | 12.3652 | 7.89932 |

There are some interesting trends in this table. First, we can observe that no matter the rating, long recipes consistently have 3-4 steps more than short recipes, on average. Within both groups, we can observe an (overall) decreasing trend in average step count as rating increases from 1 to 4 stars. This could suggest that people are more likely to review a recipe favorably if it has less steps. This trend reverses for a rating of 5, where there is a sudden increase in the average step count. Earlier, we learned that 5 is the most common rating within both long and short recipe groups. This implies that some recipes with large step counts were reviewed very favorably by users. If we examine the aggregates closer, we can observe that the jump in average step count from 4 to 5 stars is larger in long recipes than short ones. Some particularly long recipes (in terms of step count) may be reviewed quite favorably despite how long they are!

**Takeaway:** While there is a slight decreasing trend in step count as rating increases, this does not necessarily imply that all good recipes must have low step counts.

## Assessment of Missingness
Columns that have substantial amounts of missing data are `description`, `rating` and `review`. I performed a missingness analysis to better understand why this data could be missing.

### NMAR Analysis
By definition, data in a column that is **not missing at random (NMAR)** is missing values in a way that depends on the values of the column itself. The missingness of the `description` column could potentially be NMAR. Since recipe descriptions are user-submitted, it is possible that the author simply opted not to provide a recipe description. Since virtually all recipes have names (and tags, provided by Food.com), some authors may have felt this information was enough and thus did not provide additional information in the description. For this reason, we may not be able to discern missingness using other columns in the dataset, and instead can only reason about why a user would omit the description in its own context. If we wanted to evaluate the data for being missing at random (MAR), we might consider if the user submitting the recipe is the recipe's original author. Perhaps people who upload recipes without a description are more likely to be re-uploading other people's recipes. Since the recipe is not their own, they may care less about providing an adequate description of the recipe. However, this would require additional data collection and testing. For now, we can only speculate about the missingness of this column.

### Missingness Dependency
Next, I analyzed the missingness of a different column: `ratings`. About 6% of all ratings are missing in the merged DataFrame. I performed a missingness dependency analysis, where I examined if ratings were **missing at random (MAR)** based on values in another column.

#### Dependency on `length` Column
First, I considered dependency on the `length` column, which labels recipes as either *short* or *long*. Recall that this column is directly derived from the `minutes` column, where the threshold is the median, 35 minutes. I used a permutation test to evaluate two hypotheses:
* **Null Hypothesis:** The missingness of `rating` **does not** depend on the binary `length` of a recipe.
* **Alternative Hypothesis:** The missingness of `rating` **does** depend on the binary `length` of a recipe.
* **Test Statistic:** Absolute Difference in Proportions of Long Recipes in the distribution of missing ratings and the distribution of non-missing ratings
* **Significance Level:** 0.05

> Note that we can use an absolute difference in proportions (instead of, say, Total Variation Distance) because we only have 2 categories of recipe lengths: *short* and *long*.

<iframe
  src="assets/missing_length.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We reject the null hypothesis because our p-value was 0.0 < 0.05. The observed absolute difference in proportions was about 0.08, or 8%. There is evidence to suggest that `rating` could be missing at random (MAR), dependent on the binary `length` column.

> Recall that the dataset has almost 230,000 rows. Because our sample size is very large, the test is more sensitive to small differences. This rationalizes the extreme p-value. Regardless, it is possible that missingness of `rating` can be predicted (to a degree) by the binary length of a recipe.

#### Dependency on `minutes` Column
Can we use raw `minutes` data to explain missingness in rating as well as we can explain it with recipe `length` data? Since `minutes` is quantitative, I built the test statistic using the average length of a recipe, in minutes. As before, I used a permutation test to evaluate two hypotheses:
* **Null Hypothesis:** The missingness of `rating` **does not** depend on the continuous `minutes` column.
* **Alternative Hypothesis:** The missingness of `rating` **does** depend on the continuous `minutes` column.
* **Test Statistic:** Absolute Difference in Minute Means in the distribution of missing ratings and the distribution of non-missing ratings
* **Significance Level:** 0.05

<iframe
  src="assets/missing_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This time, we did not find a significant result. The p-value of this test was 0.122 > 0.05. We fail to reject our null hypothesis, and we cannot conclude from this test alone whether `rating` is missing at random (MAR) dependent on the continuous `minutes` column.

> This conclusion can only be made in the context of the test statistic used. Recall that the `minutes` data is highly right skewed and continuous, whereas `length` data is categorical and relatively balanced. Even though `length` is a column directly derived from `minutes`, we can come to opposing conclusions based on the test we make. For instance, using the absolute difference in means for `minutes` did not lead to a significant result. Perhaps there is a different test, one that takes into account the skewness of the `minutes` data, that could better communicate MAR dependence. Within the constraints of the above test, however, we were unable to find strong evidence in favor of MAR dependence on the `minutes` column.

## Hypothesis Testing
During my bivariate analysis, I compared the rating distribution of long and short recipes. I observed that a 5-star rating was (slightly) more likely to occur for short recipes than for long ones, and that lower star ratings were more likely to occur for long recipes. To examine if the rating distributions of long and short recipes came from the same distribution, I performed a permutation test.
* **Null Hypothesis:** The distributions of ratings for short and long recipes are the same.
* **Alternative Hypothesis:** The distributions of ratings for short and long recipes are not the same.
* **Test Statistic:** Total Variation Distance (TVD)
* **Significance Level:** 0.05

My justification to use this test is as follows:
1. I decided to use a permutation test because I am comparing two subgroups&mdash;long and short recipes&mdash;to each other in terms of rating distribution. I am not relating either to a general population, which would warrant the use of a standard hypothesis test. Because I do not know the distribution from which either subgroup's data was sourced, I could use a permutation test to assess if their data could be from the same (unknown) distribution.
2. Though ratings can be interpreted as discrete quantitative variables, we can also interpret them to be ordinal categorical (since there are only 5 kinds of them). This warrants the use of proportions to quantify the frequency of various ratings per group.
3. Since I am comparing two categorical distributions (where each rating has a frequency proportion), it makes sense to use Total Variation Distance (TVD) as a test statistic. Furthermore, my test is two-sided and requires a test statistic that captures deviations in either direction. This further reinforces the use of the TVD.

Because the test uses the TVD, we cannot necessarily conclude *which* type of recipe is more likely to receive higher or lower ratings. We can only conclude that short and long recipes could have fundamentally different ratings. The results of the outlined test are shown below:
<iframe
  src="assets/hypothesis.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The observed statistic, 0.014, is significant, with a p-value of 0.0 < 0.05. We reject the null hypothesis and have evidence that the distributions of ratings for short and long recipes may not be the same.

> **Important:** We cannot *definitively* conclude that short and long recipes receive different ratings. Furthermore, the observed TVD is quite small. Because our sample size is so large (at almost 230,000 reviews), the test becomes sensitive to small differences and is more likely to reject a null hypothesis of this sort. Even though our test result was significant, it is unlikely that short recipes are reviewed ***noticeably*** differently from long recipes. Therefore, ratings may not necessarily help us distinguish a short recipe from a long one.

## Framing a Prediction Problem
I decided to make a model to predict whether a recipe will be *short* (35 minutes or less) or *long* (more than 35 minutes), as described in the `length` column of the DataFrame. 
* This is a **binary classification** model that will predict 1 for a recipe it thinks is short, and 0 if it thinks the recipe is long. The classifier type used will be a **RandomForestClassifier** (since individual Decision Trees are prone to higher variance).
* The **response variable** will be the binarized `length` of a recipe, as described above (0 for long, 1 for short).
  * Since we are predicting if a recipe will be short or long, and there already exists a column with this exact information in the dataset, it made sense to use `length` directly as a response variable.
* The **main metric** used will be **precision**, though accuracy will be commented on, as well.
    * Because I used the median, 35 minutes, to split recipes into *short* and *long* groups, I have minimized the risk of class imbalance. In fact, there is about a 48-52 split between long and short recipes, respectively, in the merged dataset. This will discourage artifically high accuracies if the classifier is only guessing the most common recipe length group (short). For this reason, it may be beneficial to consider accuracy in addition to precision, as it can communicate if we're capturing true negatives (truly long recipes) as accurately as we are identifying true positives (truly short recipes). *Why we chose to use precision will be discussed in much further detail in the Context below*.

### Context: Why use Precision as the Main Metric?
When applying this project to real life, it is important to weigh the risks of mislabeling a certain length of recipe. Consider the 1-hour lunch example from the beginning.
* If this model predicts a short (1) recipe will be long (0), it will have produced a *false negative*. This is arguably not a big deal if you are deciding what recipe you will have time to make: you will simply avoid that recipe and find a new one, thus not wasting any of your time.
* If this model predicts a long (0) recipe will be short (1), it will have produced a *false positive*. This is a lot more problematic than a false negative, because if you choose to start making a long recipe with less than 30 minutes to spare, you may not have time to eat, clean up, or even finish cooking in the first place, leaving a mess in the kitchen. This is a pretty bad outcome if you're trying to manage a busy schedule, for example.

Because of this, **precision** is a good metric to optimize our model for. Aiming for high precision will minimize the rate of false positives, which is exactly what we want to avoid in our model. However, if time is not so important, we want to make sure we're labeling short recipes accurately, too. For this reason, accuracy will be used as a sub-metric to assure our model is correctly identifying recipes in general. Recall that class imbalance is not a major issue in this dataset, so a high accuracy will indicate that we're identifying large numbers of true positives (short recipes) *and* true negatives (long recipes).

### What Features Are Known Prior to Prediction?
Evidently, I could not use the `minutes` column directly to label a recipe as short or long. This is because `length` is directly derived from `minutes`, and would lead to artifically high accuracies, precision, and recall. Information that I would know, however, includes the following:
* **Ingredient and step counts** (`n_ingredients` and `n_steps` respectively). If you are told or know a recipe (but not its expected time), you will naturally have access to the number of ingredients you'll need, as well as the steps you'll have to take to complete the recipe. As such, both of these features are information you will likely know prior to knowing how long a recipe will take. A recipe, by construction, must include ingredients and steps.
* **Certain tags in the `tags` column**. Some tags, like '15-minutes-or-less', are of course *unreasonable* to use if you already don't know whether a recipe is short or long. However, it is possible that certain tags, like 'easy' and 'desserts', are information you will know before you might know how long a recipe will take. A dessert is straightforward: if you know the name of the recipe, you will likely know what kind of food it is. It is also likely you will know if a recipe is easy prior to making it, as well. There are a lot of ways to gain this information. If someone tells you a recipe, for example, they may preface it by telling you it's easy or difficult. This will give you the aforementioned information. If you're reading a recipe online, you will likely have access to the tags (which is exactly where we obtained this information), as well as the name of the recipe itself. Though I did not use the name in my final model, typically a recipe may directly have "Easy <Recipe Name>" as its title. There are ways to come to a conclusion about the difficulty of a recipe, whether by reading reviews, having prior experience with cooking, or observing the recipe steps themselves. Consequently, it is not unreasonable to use this is a feature in our prediction model.
* **Nutritional info**. If you have access to ingredients, you will likely be able to make rough estimates for different nutritional information. If there is an exact sugar measurement, you will know the parts-daily-value of sugar in that recipe. Calories are often not so straightforward, but you may be able to make a rough estimate based on the types of ingredients and their quantities from the recipe. While I do not consider ingredient quantities in my model, I believe it is justified to use calorie and sugar information, among other nutritional info, in my model.

> The reason I am using 1 to represent a short recipe is due to the subtext of my project. Suppose you are using this model to decide what recipe to try with a limited amount of time. A "positive" will represent a recipe you *could* finish in a given period of time (which is fixed at 35 minutes in this project). A "negative" will mean that this recipe is not suitable for the limited time you have.

> **Important:** The data was split 75:25 into train and test data, respectively. All models used the same training and test data to assure a consistent analysis of results. Additionally, the response column `length` was transformed via default hyperparameters of the `LabelEncoder` prior to any train-test split. This is where the mapping `long => 0` and `short => 1` appears in the model.

## Baseline Model
As stated earlier, a RandomForestClassifier was used to fit the baseline model. My baseline model had the following characteristics:
* Hyperparameters:
    - Number of estimators: 100
    - Max depth: 20
    - Criterion: Gini (default)
* Features:
    - `n_steps`, a quantitative feature
        - Transformed via `StandardScaler()` to make values comparable to others
    - `n_ingredients`, a quantitative feature
        - Transformed via `StandardScaler()` to make values comparable to others
    - *StandardScaling in a RandomForestClassifier does not necessarily impact model performance.*

**Note:** Because none of the current features were categorical, no encoding was necessary at this juncture.

First, I used 5-fold cross validation on my training data using the stated hyperparameters and features in a Pipeline to the RandomForestClassifier. Then, I assessed my model on the test data. The **precision** metric was **0.67** on average for the cross-validated training data, and **0.67** again for the test data. The accuracy was roughly **0.67** on both validation and test data.

This model was not too bad for a baseline model. At worst, we would expect an accuracy of about 0.52 if the model were predicting the most common recipe length category (short recipes, or 1). However, the accuracy was about 15% higher, which means that, to an extent, `n_steps` and `n_ingredients` provide decent insight into the length of a recipe. Improvements were definitely needed, however, since the the main metric, precision, was still too low to be effective in a real-life setting.

## Final Model
My final model had the following characteristics:
* Hyperparameters:
    - Number of estimators: 140
    - Max depth: 60
    - Criterion: Entropy
* Features:
    - `n_steps`, a discrete quantitative feature (untransformed)
    - `n_ingredients`, a discrete quantitative feature (untransformed)
    - `tags`, a custom-transformed, nominal categorical feature
        - Transformed via the custom-made transformer `HasTagTransformer`. It would transform the entire `tags` column into a nominal categorical feature. If a recipe had an "easy" tag, it would be transformed into the boolean `True` (1) and `False` (0) otherwise. Since boolean values are already numerical, it was not necessary to perform additional encoding.
    - `num_calories`, a quantile-transformed continuous quantitative feature
        - Transformed via the `QuantileTransformer` with the distribution set to "normal"
    - `sugar`, a quantile-transformed continuous quantitative feature
        - Transformed via the `QuantileTransformer` with the distribution set to "normal"

### Justification of Features
#### `n_steps`
From the Interesting Aggregates, I found that long recipes had about 3-4 more steps, on average, than short recipes. This implied that `n_steps` could be a good feature to distinguish a short recipe from a long one. I decided to *remove* the `StandardScaler` from this column in the final model because a RandomForestClassifier does not benefit much from scaling numerical features. Removing this transformation would alleviate the computational workload of my model and not compromise precision or accuracy.

#### `n_ingredients`
From my Bivariate Analysis, I found that there was a slight positive association between `n_steps` and `n_ingredients`. Logically, as number of ingredients increases, we would expect the recipe time to increase as well, since you need more time to prep all ingredients and incorporate them into a recipe. The *slight* association was good to avoid redundancy in the model and add nuance to edge cases. For example, in the event that a short recipe had many steps but few ingredients, the low number of ingredients could be used to reduce the influence of the high step count. In other words, adding this feature could help the model self-regulate. For the same reasons as `n_steps`, I decided to *remove* the `StandardScaler` from this column in the final model because a RandomForestClassifier does not benefit much from scaling numerical features. Removing this transformation would alleviate the computational workload of my model and not compromise precision or accuracy.

#### `tags`
I was curious if recipes labeled as "easy" could be good predictors of a short recipe. I created a grouped bar plot to compare frequencies of easy and non-easy recipes for both long and short recipes. The results are shown below:
<iframe
  src="assets/easygroups.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plots suggested that short recipes were more likely to be easy (about 70% of the time) than non-easy, and that long recipes were more likely to be non-easy (about 53% of the time) than easy. This indicated to me that a recipe being "easy" could be a good predictor of a short recipe. Similarly, a recipe without an "easy" tag could be a good predictor of a long recipe. I created a custom transformer `HasTagTransformer` that checked the `tag` column for the "easy" keyword specifically. The output was a nominal categorical feature in the form of a Boolean. This feature was included in an intermediate model (between my baseline and final models). I found that the cross-validation accuracy and precision did improve (very slightly; only by about 0.5% each) with the inclusion of this feature.

#### `num_calories` and `sugar` 
The justification for these features is similar in nature, so they will be discussed under the same header. From my Bivariate Analysis, I plotted two overlaid histograms of the calorie distributions for short and long recipes. I found that the shape of the distributions was different, with long recipes having a tendency to a more extreme spectrum of calorie counts than short recipes did. This indicated that I could use calorie counts to distinguish a short recipe from a long one. Because the data was right-skewed, I decided to use a `QuantileTransformer` to adjust the distribution of values closer to a normal distribution. Higher quantiles of calories would likely correspond to longer recipes, and lower quantiles of calories would likely correspond to shorter recipes.

The same procedure was done for the `sugar` column of the dataset. I approached the logic behind this from the perspective of baked goods. Baked goods tend to have a lot more preparation involved, and often need time to stay in the oven, which you have little control over in terms of time. Baked goods also have higher sugar content compared to other foods, so a higher sugar content could indicate the recipe needs longer preparation. Indeed, the overlaid histogram I made confirmed a similar trend as with calories:
<iframe
  src="assets/sugar_hist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Due to the difference in heights of the distributions, it appears that long recipes have more severe sugar outliers than short recipes. For the same reasons as `num_calories`, I used a `QuantileTransformer` on the `sugar` column. This would help control the right skew of the data. Ultimately, it seems that `sugar` is another feature you could use to distinguish between short and long recipes, which is why I included it and `num_calories` as features.

### Model Tuning
To optimize my model's **precision** (the main metric), I used `GridSearchCV` to tune a set of hyperparameters with **5 folds** for cross-validation. I tested the following:
* Number of Estimators: 100, 120, 140
* Max Depth: 20, 30, 40, 50, 60
* Criterion: Gini, Entropy

Throughout this project, I have kept a RandomForestClassifier model. A single Decision Tree is prone to bias and variance (a result of overfitting to the data). A RandomForestClassifier would help mitigate effects of certain features and help reduce overall model variance. This was indeed the outcome, as I found that my model's accuracy and precision metrics were *extremely* consistent during cross-validation, and later even for test data. I did not necessarily consider changing the algorithm itself, only the hyperparameters. The hyperparameters chosen by GridSearch are described below.

#### GridSearch Results, in Detail
The ideal parameters were not always consistent, and I found that precision plateaued at about 91% with most sets of hyperparameters. I decided to use the output of the last Grid Search, which found **140 estimators, max depth of 60, and entropy** to be the ideal hyperparameters.
- Increasing the number of estimators would decrease my model's variance and not necessarily affect my model's bias. Because of the precision plateau, 140 estimators is a good compromise to reduce the computational strain of my model while reducing model variance.
- The ideal max depth was found to be anywhere between 30-60 by Grid Search. The maximum tree depth is closely tied to over- (and under-)fitting a model. Going too high would increase risk of overfitting, and unnecessarily large depths would provide little benefit for precision in the model. The various Grid Searches I ran did not consistently decide a max depth. The cross-validation precisions were generally high across all depths I tested. This suggested that, of the depths I tried, neither were deep enough yet to trigger overfitting. Thus, I settled for the results of the last Grid Search, with a **max depth of 60**.
- **Entropy** was consistently chosen as the ideal hyperparameter. This is a more expensive computation than the Gini impurity, but entropy is generally better at determining splits in a tree than Gini. Because Grid Search unanimously decided on entropy, I used it in my final model.

### Model Improvement
The final model had a **precision** of over **0.92** on the test set after the best hyperparameters were applied. The sub-metric, accuracy, increased to 0.91 as well. This was a significant improvement of **well over 20%** in terms of model precision *and* accuracy. This indicates that the new features, especially the quantile-transformed `num_calories` and `sugar` data, were extremely informative in distinguishing long recipes from short ones. The high precision is reassuring as well, since the false positive rate is quite low. The high accuracy indicates that the model is good at predicting recipe length labels *in general*, so short recipes are unlikely to be misclassified as long, either. We would of course need to consider recall specifically to analyze false negative rate, but in the context of our balanced dataset, accuracy is a good holistic measure of both true positives and true negatives in the model.


### Limitations
There is a major limitation to the above Final Model. Recall that at the beginning, two datasets were merged together: one containing unique `recipes` and one containing `interactions` with each recipe. The merge caused some recipes to be duplicated (as can be seen in the Introduction, within the head of the cleaned DataFrame). **This means that nutrional data, tag data, step and ingredient data was all duplicated for several recipes.** Because I did not use unique identifying features in my Final Model (like the rating a user gave the recipe, or the date the user submitted their review of the recipe), my model is trained on (some) perfectly duplicate data points. This *may* artificially raise my model's precision and accuracuy. In the future, it is highly recommended to perform this analysis by either 1) including factors like user input (e.g. ratings), or 2) dropping duplicate recipe IDs to better understand how the data actually performs.

Throughout this project, I've attempted to compare unique recipe statistics against non-unique recipe statistics. Generally, the difference is noticeable but not absurdly large. For this reason, this model should be reevaluated on a different dataset, one that preferably does not repeat data.


## Fairness Analysis
To analyze the fairness of my model, I analyzed subgroups within the `n_steps` column of my data. Specifically, I compared the **precision** of my final model on recipes that had 9 or less steps with recipes that had more than 9 steps. (A justification for why `n_steps` was considered is provided at the end of this section.)

* **Group 1:** Recipes that have over 9 steps
* **Group 2:** Recipes that have 9 or less steps
* **Evaluation Metric:** Precision (the main metric of the entire prediction problem)
* **Null Hypothesis:** Our model is fair. Its precision for recipes with more than 9 steps and recipes with 9 steps or less are roughly the same, and any differences are due to random chance.
* **Alternative Hypothesis:** Our model is unfair. Its precision for recipes with more than 9 steps is lower than its precision for recipes with 9 steps or less.
* **Test Statistic:** Difference in Precision (More than 9 Steps - 9 Steps or Less)
* **Significance Level:** 0.05

<iframe
  src="assets/precision.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

I used a `Binarizer` with a threshold of 9 to create a new column `steps_over_9`. The difference in precision was observed to be -0.019 across groups, which means that we observed 2% less precision in the model for recipes with more than 9 steps compared to recipes with 9 steps or less. The result, as seen above, was significant with a p-value of 0.0 < 0.05. We reject the null hypothesis that our model is fair. There is evidence to suggest that our model's precision is lower for recipes that have more than 9 steps.

> It seems that `n_steps` is not a perfect proxy for the (time-wise) length of a recipe. A high step count may not necessarily indicate a longer recipe. This can be explained by considering the steps themselves. Often, we cannot understand the time of each *individual* step. One step can be extremely simple, like transferring all dry ingredients into a bowl. Another step may be significantly more time consuming, like waiting for dough to rise (a process that often takes several hours). The number of steps is a *limited measure* of how long a recipe may take, which is why precision may suffer as `n_steps` reaches a certain level. At a point, certain step counts become ambiguous: are the 9 steps in my recipe short, easy steps, or are they long, time consuming steps? The above test helps quantify the ambiguity of this feature through the lens of fairness.