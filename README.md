# Recipes-and-Ratings
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
   def parse_nutrition_df(nutrition_str):
    try:
        # Remove brackets and split by comma
        nutrition_list = nutrition_str.strip('[]').split(',')
        # Extract specific nutrient values
        calories = float(nutrition_list[0])
        total_fat = float(nutrition_list[1])
        sugar = float(nutrition_list[2])
        sodium = float(nutrition_list[3])
        protein = float(nutrition_list[4])
        saturated_fat = float(nutrition_list[5])
        carbohydrates = float(nutrition_list[6])
        # Create DataFrame
        nutrition_df = pd.DataFrame({
            'calories': [calories],
            'total_fat': [total_fat],
            'sugar': [sugar],
            'sodium': [sodium],
            'protein': [protein],
            'saturated_fat': [saturated_fat],
            'carbohydrates': [carbohydrates]
        })
        return nutrition_df
    except (IndexError, ValueError) as e:
        # If there's an error in parsing, return None
        return None
   ```

# Apply the function to each row of the 'nutrition' column
nutrition_df = merged_df['nutrition'].apply(lambda x: parse_nutrition_df(x))

# Concatenate the resulting DataFrames
final_nutrition_df = pd.concat(nutrition_df.tolist(), ignore_index=True)

print(final_nutrition_df)

5. **Converting Time Formats**: The `date` column was standardized to `YYYY-MM-DD` format, facilitating temporal analyses and eliminating potential confusion arising from inconsistent date formats.

6. **Deduplication**: Duplicate entries were identified and removed, ensuring that each recipe and interaction was unique. This step is critical to avoid skewed analysis due to overrepresentation of certain data points.

    ```python
    merged_df.drop_duplicates(inplace=True)
    ```

7. **Outlier Detection**: Using Z-scores, we identified and removed outliers in the numerical columns such as `calories` and `n_steps`, which could distort statistical models.

    ```python
    filtered_data = remove_outliers_zscore(merged_df, 'calories', threshold=3)
    ```

8. **Final Touches**: The last step was to reassess the dataset for any remaining inconsistencies and to confirm the data types were appropriate for each column. Categorical data were encoded suitably, and continuous variables were checked for any remaining outliers or anomalies.

### Impact on Analysis
The cleaning process had a profound impact on our analyses. It allowed us to work with a dataset that more accurately reflects the genuine interactions and characteristics of the recipes. By removing noise and inconsistencies, we can trust that any patterns or insights derived are closer to the underlying truths of the data.

### Cleaned DataFrame Preview
Below is a preview of the cleaned dataset, which will be used for subsequent analysis.



## Assessment of Missingness
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
