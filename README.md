# How to make your recipes more popular
by **Kening Li** *(kel016@ucsd.edu)*
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

|   calories |   total_fat |   sugar |   sodium |   protein |   saturated_fat |   carbohydrates |
|-----------:|------------:|--------:|---------:|----------:|----------------:|----------------:|
|      138.4 |          10 |      50 |        3 |         3 |              19 |               6 |
|      595.1 |          46 |     211 |       22 |        13 |              51 |              26 |
|      194.8 |          20 |       6 |       32 |        22 |              36 |               3 |
|      194.8 |          20 |       6 |       32 |        22 |              36 |               3 |
|      194.8 |          20 |       6 |       32 |        22 |              36 |               3 |

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

### Univariate Analyses*
- Distribution of Ingredients
The histogram for the number of ingredients shows a normal-like distribution, with most recipes requiring a moderate number of ingredients.
<iframe
  src="assets/Distribution of Ingredients.html"
  width="600"
  height="375"
  frameborder="0"
></iframe>



### Bivariate Analyses*
- Calories vs Protein
This scatter plot shows the relationship between the calorie content and protein content of recipes. We observe that higher calorie counts do not necessarily correspond to higher protein values, indicating that calorie-dense recipes may not always be protein-rich.

<iframe
  src="assets/Calories vs Protein.html"
  width="600"
  height="375"
  frameborder="0"
></iframe>


### Group Table/Pivot Table*

The grouped table below presents the mean rating for recipes, categorized by the number of calories and amount of sugar they contain. This table helps us understand how the nutritional content of recipes correlates with user ratings. For instance, we can observe that recipes with fewer calories and sugar do not necessarily receive higher ratings, challenging the assumption that healthier recipes are rated more favorably.

|   calories |   sugar |   rating |
|-----------:|--------:|---------:|
|        0   |       0 |  4.24242 |
|        0.1 |       0 |  4.5     |
|        0.2 |       0 |  5       |
|        0.3 |       0 |  4.23077 |
|        0.4 |       0 |  4.66667 |






## Assessment of Missingness
In our dataset, we hypothesize that the column 'rating' may be NMAR. This hypothesis stems from the observation that a significant number of ratings are missing, and it's plausible to assume that the missingness of ratings is related to the intrinsic quality of the recipe. For example, users may choose not to rate recipes that they did not find appealing, and these unappealing aspects of the recipe may not be captured by the other variables in our dataset.

We conducted permutation tests to examine the missingness mechanism of the 'rating' column. We compared the distribution of the 1. `n_steps` and 2. `protein` columns for recipes with and without a rating, using the **Kolmogorov-Smirnov** statistic as our test statistic.

- `n_steps` Test Result
  1.  Observed KS statistic: `0.048050754819474961`
  2.  p-value from permutation test: `0.0`
     
The p-value suggests that is a statistically significant difference to reject the null hypothesis of the missingness being at random with respect to the number of number of steps in a recipe.

<iframe
  src="assets/Missing Rating and Steps.html"
  width="600"
  height="375"
  frameborder="0"
></iframe>


- `protein` Test result
  1.  Observed KS statistic: `0.012859999187318671`
  2.  p-value from permutation test: `0.052`

The p-value suggests that we do not have enough evidence to reject the null hypothesis of the missingness being at random with respect to the number of protein in a recipe.

<iframe
  src="assets/Missing Rating and Protein.html"
  width="600"
  height="375"
  frameborder="0"
></iframe>










## Hypothesis Testing

- Null Hypothesis (H0): The sugar content of food name that contains brownies is the same as the sugar content of food name that contains chocolate.
- Alternative Hypothesis (H1): The sugar content of food name that contains brownies is different from the sugar content of food name that contains chocolate.


- Test statistic used: Two-sample t-test
- Significance level (α): 0.05

### Results and Conclusion
- Observed p-value: *0.09675951132243926*
- Result: With a p-value of 0.09675951132243926(greater than 0.05), we fail to reject the null hypothesis that there is a difference in the sugar content between recipes that contain brownies and recipes that contains chocolate.

*Interpretation*: Given the p-value is greater than our α level of 0.05, we have insufficient evidence to suggest that there is a difference in the sugar content between recipes that contain brownies and recipes that contains chocolate.




## Framing a Prediction Problem
- **Prediction Problem**:
Given the nutritional information (calories, total fat, sugar, sodium, protein, saturated fat, carbohydrates) and preparation details (minutes, n_steps, n_ingredients), predict the rating of the recipe.

- **Problem Type**:
Regression

- **Response Variable**:
Rating of the recipe
Because we want to improve our recipes to make them more popular!🍾

- **Metric**:
RMSE
RMSE is standard for measuring the performance of regression models as it provides an average measure of error across predictions.


## Baseline Model

**features**: `calories`, `sugar` (Both of them are quantitative)

Three models was used to train data:
- Random Forest Regressor:
  - Mean Squared Error (MSE): 1.0674807515332745
  - Root Mean Squared Error (RMSE): 1.0331896090960673
- Gradient Boosting Regressor:
  - Mean Squared Error (MSE): 0.7608854079700182
  - Root Mean Squared Error (RMSE): 0.8722874571894395
- Linear Regression:
  - Mean Squared Error (MSE): 0.7599658743963008
  - Root Mean Squared Error (RMSE): 0.8717602611123784

I used ColumnTransformer with StandardScaler in the pipeline to perform standardization on numerical features.

Within the three models, the Random Forest Regressor had the highest MSE and RMSE, while the Linear Regression and Gradient Boosting Regressor had significantly lower errors, indicating better performance.

Considering the performance metrics, the current model seems to have a good predictive ability, particularly the Linear Regression and Gradient Boosting Regressor models, which have similar MSE and RMSE values. 

However, there's still room for improvement.



## Final Model

- Best hyperparameters: {'gb__learning_rate': 0.01, 'gb__max_depth': 3, 'gb__n_estimators': 200}
- Mean Squared Error on test set: 0.7586199258826435
- MSE scores for each fold are: [0.7775671  0.75371931 0.76424836 0.75754774 0.76898762]
- Mean MSE score across all folds is: 0.7644140249353026
- Standard deviation of MSE scores is: 0.008431741545314954

**Features Added**: Add all of the nutritions: not only `calories` and `sugar`, but also `total_fat`, `sodium`, `protein`, `saturated_fat`, `carbohydrates` and one of the categorical data `n_steps` (step >10 considered as 1, step <10 as 0) for the final model.

**Why?** As we examine MAR and NMAR in the previous section, we conclude that the missing rating value may be influenced by the column `n_steps`. Also, we try to figure out if nutritions other than `calories` and `sugar` also play parts in affecting the rating.

**Modeling Algorithm**: The chosen algorithm for the final model is the **Gradient Boosting Regressor**, which showed a good predvtive ability on our data based on the baseline model.


**Hyperparameters**
- `n_estimators`: 200, indicating that the ensemble should consist of 200 trees. 
- `learning_rate`: 0.01, which means each tree added to the ensemble helps to improve the model's performance incrementally.
- `max_depth`: 3. The maximum depth of 3 for each tree suggests that only simple decision trees are needed, which helps to keep the model simple and prevent overfitting. 

**Final Model’s performance**
- The mean MSE across all folds was 0.7644140249353026, which performs almost the same as our best performed baseline model.
- The standard deviation of MSE is 0.008431741545314954, indicating relatively low variability.


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

- **Conclusion**: Given the p-value is `0.0`, which is less than the typical alpha level of **0.05**, we reject the null hypothesis. This result suggests that there is a statistically significant difference in RMSE between recipes with preparation times below or equal to 60 minutes and those with preparation times above 60 minutes. Therefore, preparation time might be an influential factor in predicting recipe ratings.

<iframe
  src="assets/Preparation Times.html"
  width="600"
  height="375"
  frameborder="0"
></iframe>
