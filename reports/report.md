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

```
shall we remove outliner (~1%) or we use clip data to handle outliner (~1%) ?
```
```
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

```
```
Categorical Feature

Feature	Type	Recommendation	Reason
| Feature    | Type     | Recommendation              | Reason                                                                                                |  
|------------|----------|-----------------------------|-------------------------------------------------------------------------------------------------------|
| condition  | Ordinal  | Label Encoding (1, 2, 3...) | Houses are "ranked" (e.g., 1 is poor, 5 is excellent). Maintaining this order helps the model.        | 
| waterfront | Binary   | Keep as 0/1                 | It’s already numeric (0=No, 1=Yes), which is perfect for models.                                      | 
| city       | Nominal  | Target Encoding or One-Hot  | Cities have no "order." If you have many cities, use Target Encoding to avoid creating 100+ columns.  |
| yr_built   | Temporal | Binning or Age              | Don't treat this as a category. Transform it into house_age = current_year - yr_built                 |  

```

```
