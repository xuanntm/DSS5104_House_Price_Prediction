```
 #   Column         Non-Null Count  Dtype  
---  ------         --------------  -----  
 0   date           9200 non-null   object 
 1   price          9200 non-null   float64
 2   bedrooms       9200 non-null   float64
 3   bathrooms      9200 non-null   float64
 4   sqft_living    9200 non-null   int64  
 5   sqft_lot       9200 non-null   int64  
 6   floors         9200 non-null   float64
 7   waterfront     9200 non-null   int64  
 8   view           9200 non-null   int64  
 9   condition      9200 non-null   int64  
 10  sqft_above     9200 non-null   int64  
 11  sqft_basement  9200 non-null   int64  
 12  yr_built       9200 non-null   int64  
 13  yr_renovated   9200 non-null   int64  
 14  street         9200 non-null   object 
 15  city           9200 non-null   object 
 16  statezip       9200 non-null   object 
 17  country        9200 non-null   object 
```
### Data cleasing:
```
    Remove the duplicate Data (entire column is duplicated)
    we wont remove outliner (~1%) and use clip data to handle outliner (~1%)
    Country Column: remove 
    Date : convert Date time to Date
    
```
### Feature Engineering
```
Including:  

1. Log10 for Price Column : because the price distribution is right tail

2. Numberic data exploration

=== TOP PRICE CORRELATIONS ===
sqft_living      0.694668
sqft_above       0.594288
bathrooms        0.529689
view             0.383297
bedrooms         0.337051
sqft_basement    0.333939
floors           0.261500
waterfront       0.193942
sqft_lot         0.082877
condition        0.056373
yr_renovated     0.047015
yr_built         0.031033
Name: price, dtype: float64

1. High Priority (Keep, but Check Redundancy)
sqft_living (0.69) and sqft_above (0.59): These are your strongest predictors. However, they likely suffer from multicollinearity because sqft_living is often just the sum of sqft_above and sqft_basement.
Recommendation: Use a Variance Inflation Factor (VIF) check. If VIF is > 5-10, consider dropping sqft_above and keeping sqft_living. 

2. Moderate Priority (Keep)
bathrooms (0.53), view (0.38), bedrooms (0.34), and sqft_basement (0.33): These show a solid moderate relationship with price. They represent different functional aspects of a home and should remain in the model. 
3. Low Correlation (Keep for Now)
sqft_lot (0.08), condition (0.06), yr_renovated (0.05), and yr_built (0.03): These appear weak, but don't drop them yet.
Reasoning: yr_built might not have a linear relationship (old and new houses can both be expensive), but it is a critical factor for price. Similarly, condition is often a categorical or ordinal variable where simple correlation doesn't tell the full story.

```
```
run VIF code and get below correlation score:

  Feature          VIF
1     sqft_living  3895.027861
2      sqft_above  3130.267794
6   sqft_basement   918.525401
3       bathrooms     3.266422
12       yr_built     1.903429
7          floors     1.877827
5        bedrooms     1.684190
10      condition     1.434266
4            view     1.333296
11   yr_renovated     1.318363
8      waterfront     1.141971
9        sqft_lot     1.076333

Those massive VIF scores (~3900!) confirm a perfect linear dependency. Mathematically, your dataset likely follows the rule: sqft_living = sqft_above + sqft_basement. Keeping all three will cause your predictive model (especially Linear Regression) to "explode" with unreliable coefficients.
The Best Option: Keep sqft_above and sqft_basement, Drop sqft_living
While sqft_living has the highest individual correlation (0.69), splitting it into its two components provides more granular intelligence for your model.
Reasoning:
Market Value: In real estate, 1,000 sqft of "above-ground" space is usually valued higher than 1,000 sqft of "basement" space. By keeping them separate, your model can learn these two different price rates.
Redundancy: Since keeping all three provides zero new information to the model, but creates massive mathematical noise.
Interpretability: You can tell a user exactly how much a basement renovation adds to their home value versus an attic extension.

House_age = date - Yr_built
yr_renovated -> transform to binary data: either 1 with year or 0
Renovated_age = date - yr_renovated. (if yr_renovated = 1 or put 0)

view -> 
```
```
Categorical Feature

Feature	Type	Recommendation	Reason
| Feature    | Type                | Recommendation                      | Reason                                                                                                                                                                                               |
| ---------- | ------------------- | ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| condition  | Ordinal             | Label Encoding (keep numeric order) | Condition represents a ranked quality scale (e.g., poor → excellent). Maintaining numeric order allows the regression model to capture increasing value with better house condition.                 |
| waterfront | Binary              | Keep as 0/1                         | Already encoded as binary (0 = No, 1 = Yes). Waterfront homes typically command a strong price premium and the variable is already suitable for regression models.                                   |
| view       | Ordinal             | Keep numeric ranking                | View quality is typically ranked from poor to excellent. Maintaining the numeric order allows the model to capture increasing value for better scenic views.                                         |
| city       | Nominal             | One-Hot Encoding or Target Encoding | Cities have no natural order. One-hot encoding avoids introducing artificial ranking. If many cities exist, target encoding helps reduce dimensionality while capturing neighborhood price levels.   |
| statezip   | Nominal / Location  | Target Encoding                     | ZIP codes represent neighborhood-level housing markets. Target encoding summarizes each ZIP code using average house prices, avoiding hundreds of dummy variables while preserving location effects. |
| renovated  | Binary (engineered) | Convert yr_renovated to binary      | Indicates whether a house has been renovated. Renovated houses often have higher market value due to modernization and improved condition.                                                           |

Temporal Features
| Feature        | Type       | Recommendation                       | Reason                                                                                                                                           |
| -------------- | ---------- | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| yr_built       | Temporal   | Transform into house_age             | The construction year itself is less meaningful than the house age at the time of sale.                                                          |
| house_age      | Numerical  | Use directly                         | Captures depreciation effects of aging houses. Older houses often have lower prices due to wear and outdated design.                             |
| house_age_sq   | Polynomial | Include squared term                 | Allows the regression model to capture nonlinear aging effects. Very old homes may regain value due to historical or architectural significance. |
| yr_renovated   | Temporal   | Convert to binary renovated variable | The renovation year itself may not be informative, but whether renovation occurred significantly affects house price.                            |
| renovation_age | Numerical  | sale_year − yr_renovated             | Measures how recent the renovation occurred. Recent renovations often increase house value.                                                      |

Size & Structure Features
| Feature           | Type       | Recommendation                               | Reason                                                                                                                                     |
| ----------------- | ---------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| sqft_above        | Numerical  | Keep numeric                                 | Represents above-ground living space, which typically has higher market value than basement space.                                         |
| sqft_basement     | Numerical  | Keep numeric                                 | Represents basement area, which contributes to total house value but is usually priced lower than above-ground space.                      |
| log_sqft_above    | Engineered | Apply log transformation                     | Living area influences price but with diminishing returns. Log transformation helps linear regression model the nonlinear relationship.    |
| log_sqft_basement | Engineered | Apply log transformation                     | Basement area may have diminishing marginal value as size increases. Log transformation stabilizes variance.                               |
| basement_ratio    | Engineered | sqft_basement / (sqft_above + sqft_basement) | Captures the proportion of basement space relative to total house area. Houses with large basements may have different valuation patterns. |


Layout & Efficiency Features
| Feature           | Type             | Recommendation                          | Reason                                                                                                |
| ----------------- | ---------------- | --------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| bedrooms          | Numerical        | Keep numeric                            | Represents the number of sleeping spaces in the house and affects housing capacity.                   |
| bathrooms         | Numerical        | Keep numeric                            | Bathrooms improve functionality and convenience, increasing property value.                           |
| floors            | Numerical        | Keep numeric                            | Multi-floor houses may provide better space utilization and different price levels.                   |
| sqft_per_bedroom  | Engineered Ratio | (sqft_above + sqft_basement) / bedrooms | Measures spaciousness per bedroom. Larger rooms are typically associated with higher quality housing. |
| bedrooms_per_sqft | Engineered Ratio | bedrooms / (sqft_above + sqft_basement) | Captures housing density. Too many bedrooms relative to house size may indicate cramped layouts.      |


Interaction Features
| Feature            | Type        | Recommendation              | Reason                                                                                                        |
| ------------------ | ----------- | --------------------------- | ------------------------------------------------------------------------------------------------------------- |
| bath_x_above       | Interaction | bathrooms × log_sqft_above  | The value of additional bathrooms depends on house size. Larger homes benefit more from additional bathrooms. |
| view_x_above       | Interaction | view × log_sqft_above       | Scenic views often add more value to larger houses than to smaller ones.                                      |
| waterfront_x_above | Interaction | waterfront × log_sqft_above | Waterfront premium increases with house size, particularly for luxury properties.                             |


```

```
