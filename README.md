# Exploration of Recipes on [Food.com](https://www.food.com/) 🍱

&nbsp;  

## PART I: Introduction and Question Identification
Our dataset generally includes recipe information, food descriptions and user feedback from [Food.com](https://www.food.com/). The particular information we're going to focus on is the ratings of recipes. To be specific, the question that we are about to explore is:
- Is the distribution of recipe submission years that get a score of 5 randomly selected from the distribution of all recipe submission years? 

We asked this question because we wanted to see if food.com users' ideas of a good recipe change from year to year, and we’ve chosen the 5-star rating group because the users who rated five must be satisfied enough with the recipe to give it the highest rating. 

To answer our question, we need to extract information from the `submitted` and `rating` in the dataset. `submitted` is the time when the user uploads the recipe. `rating` represents the average rating of the recipe. We are going to use all the rows in the dataset.

&nbsp;  

## PART II: Data Cleaning
We first calculated the average `rating` of each recipe, and combined them with other information about the recipes. Then we replaced all the `rating` of 0 with np.nan. Such replacement is reasonable due to the fact that there is no rating of 0 on the website, and the 0s in the `rating` column is actually representing missing data.

Here are a few things we've done to clean up the data:
1. Obtain time information from the `submitted` column, and assigned a new column `year` for the submitted year. 
2. Change the string format of the list to lists of strings in `tags`, `ingredients` and `steps`, which makes it easier to access each string. 
3. Separate `nutritions` into 7 columns, each representing a different nutrient. This makes it easier for us to access individual nutrients afterwards. 
4. Get rid of unreasonable data. (We are not replacing nan in the dataframe with an ideal element at this stage. Since we have no way to know the specific generating process of the data by now, we can't fill in the content casually.)
- Replace the entries in `calories (#)` with more than 2000 calories with np.nan since there is hardly any recipe in life with such high calories. 
- Replace the data in `minutes` that are greater than 24*60 with np.nan, because it is unreasonable to prepare a recipe for over a day. 

Due to limited space, here are the first few rows of the data frame `cleaned` with the columns that we are about to use a lot. We’ll refer to the data inside `cleaned` from now on. 

|id|minutes|n_steps|n_ingredients|rating|year|calories (#)|total fat (PDV)|
|---|---|---|---|---|---|---|---|
|333281|40.0|10|9|4.0|2008|138.4|0.10|
|453467|45.0|12|11|5.0|2011|595.1|0.46|
|306168|40.0|6|9|5.0|2008|194.8|0.20|
|286009|120.0|7|7|5.0|2008|878.3|0.63|
|475785|90.0|17|13|5.0|2012|267.0|0.30|	

&nbsp;  

### Univariate Analysis 
#### Histogram of Number of Tags (`n_tags`)
We plot the distribution of the number of tags using histogram. As you can see from the figure, the recipes with 10-19 tags are the most.

<iframe src="assets/n_tag_hist.html" width=570 height=400 frameBorder=0></iframe>

&nbsp;  

#### Histogram of Number of steps ( `n_steps`)
We plot the distribution of n_steps using histogram. As you can see from the figure, most recipes take between 0 and 20 steps to make.

<iframe src="assets/Hist_n_step.html" width=570 height=400 frameBorder=0></iframe>

&nbsp;  

### Bivariate Analysis 
#### Possible correlation between `n_steps` and `minutes` 
By common sense, we know that more steps usually take up more time. In order to explore the accuracy of this statement, we plot a scatter plot and the OLS linear regression line. This image confirms our guess.

<iframe src="assets/Scat_step_minute.html" width=570 height=400 frameBorder=0></iframe>

&nbsp;  

#### Possible correlation between `calories (#)` and `total fat (PDV)`
In order to explore if more calories is associated with more total fat, we plot a scatter plot and the OLS linear regression line. As is shown in the graph, calories and total fat are positively correlated.

<iframe src="assets/Scat_cal_fat.html" width=570 height=400 frameBorder=0></iframe>

&nbsp;  

#### Histogram of number of `rating` of 5 in each year
We've plotted a histogram of the number of ratings of 5 for each year. From the figure, we can see that the rating of 5 in 2008 is the highest. Though the trend of the plot seems decreasing over years, we cannot conclude that people are giving fewer and fewer fives to the recipe, since the number of total recipes varies in each year. More investigation of the proportion of recipes that got 5 each year is needed for an accurate conclusion.

<iframe src="assets/Scat_year_rating.html" width=570 height=400 frameBorder=0></iframe>

&nbsp;  

### Interesting Aggregates
We want to know whether the distribution of `rating` is similar for each `calories (#)` group. Therefore, we created a pivot table with rating group as index, and calories group as columns. This table displays the proportions of each average rating group for each calory group. 

|rating/calories|   (-0.001, 169.1] |   (169.1, 300.1] |   (300.1, 483.7] |   (483.7, 1999.8] |
|:--------------|------------------:|-----------------:|-----------------:|------------------:|
| (0, 1]        |        0.00835114 |       0.00865128 |       0.00622024 |        0.00738509 |
| (1, 2]        |        0.0138862  |       0.0148238  |       0.0154534  |        0.0154018  |
| (2, 3]        |        0.0537483  |       0.0538518  |       0.0559335  |        0.0602954  |
| (3, 4]        |        0.18188    |       0.189064   |       0.195451   |        0.196968   |
| (4, 5]        |        0.742134   |       0.733609   |       0.726941   |        0.719949   |

&nbsp;  

## PART III: Assessment of Missingness 
After cleaning the data and analyzing the distribution of data by making univariate and bivariate plots, we have a vague idea of what kind of data we have in our `cleaned` data frame. Now, it’s time to dive deeper into analyzing the missingness of data in particular columns of our data frame.

To begin with, there are two columns with non-trivial missingness within the raw data frame (`description`, `rating`) . 

We cannot say for sure that any of the data in either column is **NMAR**, because both of them could be related to data in other columns. For example, the missingness in `description` could be related to the complexity of the recipe. If the recipe is pretty easy and self explanatory, the user who submitted the recipe probably doesn’t put in a description to explain the recipe repetitively. Thus, the missingness in `description` may be dependent on the `tags` or `n_steps` columns which indicate the complexity of the recipe. Similarly, the missingness for `rating` could potentially be related to the preparation time, the number of ingredients or estimated calories of the recipe. 

Nevertheless, **hypothesis testing** is needed to determine if the data in these columns are **MAR** (Missing At Random), which means they are missing depending on other columns. 

We are interested in if the data in `rating` is **MAR** dependent on the number of ingredients. Therefore, we decide to investigate whether the distribution of the number of ingredients used in each recipe when `rating` is missing comes from the same population distribution as the one when `rating` is not missing .  Below are our null and alternative hypotheses:
- H0: The missingness in `rating` **has nothing to do** with the number of ingredients in each recipe.
- Ha: The missingness in `rating` is **dependent** on the number of ingredients in each recipe.

By plotting the distributions of the number of ingredients in the group in which `rating` is and is not missing, we find that the mean and median of the two distributions are roughly the same. Therefore, instead of using differences in means or medians, we decide to use **K-S statistic** (The Kolmogorov–Smirnov statistic) as our test statistic. 

<iframe src="assets/KDE_ing_rating.html" width=570 height=400 frameBorder=0></iframe>

The p-value is larger than 0.05, so we fail to reject the null hypothesis and conclude that the missingness of rating seems to be independent on the distribution of numbers of ingredients of each recipe to some extent, which means `rating` is not MAR dependent on `num_ing`.

&nbsp;  

## PART IV: Hypothesis Testing
Aside from analyzing the missing mechanism, we still have another question that was proposed at the top of this page – is the distribution of submitted years of recipes that got a rating of 5 randomly drawn from the distribution of submitted years of all recipes? To answer this question, we need to conduct a hypothesis test. Below are our null and alternative hypotheses. 
- H0: The distribution of the submitted years of recipes that got a rating of 5 was the same as the distribution of the submitted years of all recipes.
- Ha: The distribution of the submitted years of recipes that got a rating of 5 was not the same as the distribution of the submitted years of all recipes.

Since we are dealing with two categorical distributions, we’ll use **TVD (Total Variation Distance)** as our test statistic.

The p-value is much smaller than 0.05, so we reject the null hypothesis and conclude that the distribution of the submitted `year` of recipes that got a rating of 5 was not the same as the distribution of the submitted `year` of all recipes.
