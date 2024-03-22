# How to make your recipes more popular
by Kening Li
## Introduction
Welcome to our culinary exploration, where data meets deliciousness! Our dataset is a flavorful blend of recipes and user interactions that offer a window into the world of home cooking. It consists of two main parts:

1. **Recipes**: A collection of culinary creations, detailing the ingredients and steps needed to prepare each dish.

2. **Interactions**: A record of user engagements with the recipes, including ratings and reviews that reflect personal experiences and satisfaction

The central question of our project is **"What factors contribute to a recipe's popularity?"** This query is essential because understanding these factors can empower anyone to create recipes that resonate with a wider audience and elevate the cooking experience.🥘

### Dataset Composition and Relevant Columns
Our dataset features a rich tapestry of *83,782* recipes and a whopping *731,927* user interactions. For the purpose of our investigation, we're stirring our focus into the following columns:
- **Name**: The title of the recipe.
- **Minutes**: The time investment required, a crucial factor in our fast-paced world.
- **Nutrition**: Caloric and nutritional information, increasingly relevant in health-conscious times.
- **N_steps**: The number of steps in the recipe, reflecting complexity.
- **Rating**: The average user rating, our primary gauge of a recipe's popularity.


## Data Cleaning and Exploratory Data Analysis

This section delves into the steps we undertook to polish the raw data into a gleaming dataset ready for exploration.

### Steps Undertaken

1. **Merging Datasets**: Our primary step involved unifying two separate datasets: `recipes` and `interactions`. We used a left join on the `id` field, ensuring that only recipes with corresponding user interactions were included. 
    ```python
    #Left merge the recipes and interactions datasets together.
    merged_df = pd.merge(recipes, interactions, on='id', how='left')
    #In the merged dataset, fill all ratings of 0 with np.nan.
    merged_df['rating'] = merged_df['rating'].replace(0, np.nan)
    ```

2. **Find the Average Rating**: Find the average rating per recipe, as a Series. Then add this Series containing the average rating per recipe back to the recipes dataset
    ```python
    average_rating = merged_df.groupby('id')['rating'].mean()
    average_rating
    ```
    ```python
    with_rating=pd.merge(recipes, average_rating, on='id', how='left')
    with_rating.head()
    ```

3. **Cleaning Textual Data**: We scrutinized columns `nutrition`. Remove brackets and split by comma, then concatenate each of the nutritions as a column in the DataFrames.
   ```python
   def turn_float(s):
    if s.startswith('[') and s.endswith(']'):
        # Convert string to list of strings, removing the brackets and splitting by comma
        items = s[1:-1].split(',')
            # Initialize an empty list to store the converted float values
        nutrition_list = []
            # Iterate over each item, trying to convert it to a float
        for item in items:
            try:
                    # Try to convert the item to float and append to the list
                nutrition_list.append(float(item.strip()))
            except ValueError:
                    # If conversion fails, append None
                nutrition_list.append(None)
            # Create a DataFrame with the list
        nutrition_df = pd.DataFrame([{
            'calories': nutrition_list[0],
            'total_fat': nutrition_list[1],
            'sugar': nutrition_list[2],
            'sodium': nutrition_list[3],
            'protein': nutrition_list[4],
            'saturated_fat': nutrition_list[5],
            'carbohydrates': nutrition_list[6]
         }])
        return nutrition_df
   nutrition_df = merged_df['nutrition'].apply(turn_float)
   final_nutrition_df = pd.concat(nutrition_df.tolist(), ignore_index=True)
   print(final_nutrition_df)
   ```
   ```python
   selected_column=['name','minutes','n_steps','n_ingredients','rating']
   merged_df_final = pd.concat([final_nutrition_df, merged_df[selected_column]], axis=1)
   merged_df_final.head()
   ```
4. **Remove Outliers (base on z-score)**
   ```python
   # Define function to remove outliers based on z-score
   def remove_outliers_zscore(data, column, threshold=3):
       z_scores = (data[column] - data[column].mean()) / data[column].std()
       return data[abs(z_scores) < threshold]

    # Remove outliers from the data
    filtered_data = merged_df_final.copy()
    for col in columns:
        filtered_data = remove_outliers_zscore(filtered_data, col)
   ```

*Univariate Analyses*
- Distribution of Ingredients
The histogram for the number of ingredients shows a normal-like distribution, with most recipes requiring a moderate number of ingredients.


*Bivariate Analyses*
- Calories vs Protein
This scatter plot shows the relationship between the calorie content and protein content of recipes. We observe that higher calorie counts do not necessarily correspond to higher protein values, indicating that calorie-dense recipes may not always be protein-rich.


*Pivot Table*

The pivot table aggregates the data to show the average ratings with a more refined categorization based on calories and sugar. It allows for a quick comparison across different nutritional content levels, highlighting potential trends or outliers in how users rate recipes according to their calorie and sugar content.
| calories | sugar  | rating   |
|----------|--------|----------|
| 0.0      | 0.0    | 4.242424 |
| 0.1      | 0.0    | 4.500000 |
| 0.2      | 0.0    | 5.000000 |
| 0.3      | 0.0    | 4.230769 |
| 0.4      | 0.0    | 4.666667 |
| ...      | ...    |   ...    |
| 22371.2  | 884.0  | 5.000000 |
| 26604.4  | 13573.0| 5.000000 |
| 28930.2  | 9245.0 | 5.000000 |
| 36188.8  | 30260.0| 5.000000 |
| 45609.0  | 16901.0| 5.000000 |





## Assessment of Missingness
In our dataset, we hypothesize that the column 'rating' may be NMAR. This hypothesis stems from the observation that a significant number of ratings are missing, and it's plausible to assume that the missingness of ratings is related to the intrinsic quality of the recipe. For example, users may choose not to rate recipes that they did not find appealing, and these unappealing aspects of the recipe may not be captured by the other variables in our dataset.

We conducted permutation tests to examine the missingness mechanism of the 'rating' column. We compared the distribution of the 1. `n_steps` and 2. `protein` columns for recipes with and without a rating, using the **Kolmogorov-Smirnov** statistic as our test statistic.

- `n_steps` Test Result
  1.  Observed KS statistic: `0.048050754819474961`
  2.  p-value from permutation test: `0.0`
     
The p-value suggests that is a statistically significant difference to reject the null hypothesis of the missingness being at random with respect to the number of number of steps in a recipe.

<iframe
  src="assets/Missing Rating and Steps.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


- `protein` Test result
  1.  Observed KS statistic: `0.012859999187318671`
  2.  p-value from permutation test: `0.072`

The p-value suggests that we do not have enough evidence to reject the null hypothesis of the missingness being at random with respect to the number of protein in a recipe.










## Hypothesis Testing












## Framing a Prediction Problem
- **Prediction Problem**:
Given the nutritional information (calories, total fat, sugar, sodium, protein, saturated fat, carbohydrates) and preparation details (minutes, n_steps, n_ingredients), predict the rating of the recipe.

- **Problem Type**:
Regression

- **Response Variable**:
Rating of the recipe
Because we want to improve our recipes to make it more popular!🍾

- **Metric**:
RMSE
RMSE are standard for measuring the performance of regression models as they provide an average measure of error across predictions.


## Baseline Model
## Final Model
## Fairness Analysis
- **Group X**: Recipes with preparation time below or equal to 60 minutes.
- **Group Y**: Recipes with preparation time above 60 minutes.
- **Null Hypothesis (H0)**: My model performs the same for minutes>60 as it does for minutes<60 in Group minutes.
- **Alternative Hypothesis (H1)**: My model performs worse for minutes>60 than it does for minutes<60 in Group minutes
- **Metric**: Root Mean Squared Error (RMSE).
- **Significance Level**: 0.05 for a 95% confidence level
- **Permutation Test Result**:
  1. Observed difference in RMSE: *0.0201943171512441*
  2. P-value: *0.0*

- **Conclusion**: Given the p-value is 0.0, which is less than the typical alpha level of 0.05, we reject the null hypothesis. This result suggests that there is a statistically significant difference in RMSE between recipes with preparation times below or equal to 60 minutes and those with preparation times above 60 minutes. Therefore, preparation time might be an influential factor in predicting recipe ratings.

<iframe
  src="assets/Preparation Times.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
