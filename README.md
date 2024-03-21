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

2. **Handling Missing Values**: Post-merging, we addressed missing ratings by imputing them with `np.nan`, acknowledging that not all interactions include an explicit rating. 

    ```python
    merged_df['rating'] = merged_df['rating'].replace(0, np.nan)
    ```

3. **Cleaning Textual Data**: We scrutinized columns such as `description` and `review` for textual inconsistencies. Steps included stripping unnecessary whitespace, correcting common misspellings, and removing non-alphanumeric characters, which could skew text analysis.

4. **Converting Time Formats**: The `date` column was standardized to `YYYY-MM-DD` format, facilitating temporal analyses and eliminating potential confusion arising from inconsistent date formats.

5. **Deduplication**: Duplicate entries were identified and removed, ensuring that each recipe and interaction was unique. This step is critical to avoid skewed analysis due to overrepresentation of certain data points.

    ```python
    merged_df.drop_duplicates(inplace=True)
    ```

6. **Outlier Detection**: Using Z-scores, we identified and removed outliers in the numerical columns such as `calories` and `n_steps`, which could distort statistical models.

    ```python
    filtered_data = remove_outliers_zscore(merged_df, 'calories', threshold=3)
    ```

7. **Final Touches**: The last step was to reassess the dataset for any remaining inconsistencies and to confirm the data types were appropriate for each column. Categorical data were encoded suitably, and continuous variables were checked for any remaining outliers or anomalies.

### Impact on Analysis
The cleaning process had a profound impact on our analyses. It allowed us to work with a dataset that more accurately reflects the genuine interactions and characteristics of the recipes. By removing noise and inconsistencies, we can trust that any patterns or insights derived are closer to the underlying truths of the data.

### Cleaned DataFrame Preview
Below is a preview of the cleaned dataset, which will be used for subsequent analysis.

```python
# Show the head of the cleaned DataFrame
merged_df.head()

## Assessment of Missingness
## Hypothesis Testing
## Framing a Prediction Problem
## Baseline Model
## Final Model
## Fairness Analysis
