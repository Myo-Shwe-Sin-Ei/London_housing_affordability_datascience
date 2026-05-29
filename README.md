Data Science Analysis of Housing Affordability Dynamics
in London Boroughs

--------------------------------------------------------------------------------
1. OVERVIEW
--------------------------------------------------------------------------------
This project analyses housing affordability across London boroughs (2004-2024)
using a merged government panel dataset. It examines how social housing
provision, spatial market typologies, and gentrification pressure relate to
affordability outcomes, building on dynamic spatial panel research suggesting
that supply increases have surprisingly weak effects on affordability.

Research questions:
    1. What is the relationship between changes in social housing stock and
       changes in housing affordability ratios across London boroughs?
    2. Can London boroughs be clustered into housing market typologies, and do
       they show different affordability trajectories over time?
    3. Can a composite gentrification index predict affordability outcomes, and
       which factors contribute most to predictive accuracy?

--------------------------------------------------------------------------------
2. DATASET
--------------------------------------------------------------------------------
A panel dataset built from nine authoritative government datasets, merged on ONS
borough codes (E09XXXXXX) and year.

- Observations : 693
- Variables    : 31
- Coverage     : 2004-2024 (pre-crisis, 2008 financial crisis, recovery, and
                 recent market cooling)

Core variables:
    Affordability_ratio    House price to median earnings ratio (TARGET)
    House_price            Average transaction price
    Social_housing_stock   Total social landlord properties
    Social_rent_weekly     Registered social landlord average weekly rent
    La_rent_weekly         Local authority average weekly rent
    Vacant_dwellings       Empty housing count
    Waiting_list           Households on local authority waiting list
    Affordable_completions New completed affordable housing units
    Population             Mid-year population estimates
    Borough_code           Geography code (E09XXXXXX)
    Borough_name           Borough name
    Year                   Calendar year

Data sources (all London Datastore / ONS unless noted):
    - Affordable Housing Supply, by Borough (DCLG)
    - Ratio of House Prices to Residence-Based Earnings (DCLG)
    - Households on Local Authority Waiting List (DCLG)
    - Vacant Dwellings (DCLG)
    - Local Authority Average Weekly Rents (DCLG)
    - Population estimates by single year of age (ONS / NOMIS)
    - Registered Social Landlord Housing Stock (DCLG)
    - Registered Social Landlord Average Weekly Rents (DCLG)
    - UK House Price Index (HM Land Registry)


--------------------------------------------------------------------------------
3. DATA LIMITATIONS
--------------------------------------------------------------------------------
- 104 missing local authority rent observations from boroughs that transferred
  council housing to housing associations (creating discontinuities).
- Affordability ratio imputed from residence-based earnings may not reflect
  local earnings.
- Small residential population (City of London) produces outliers.
- Some datasets contain suppressed values marked as "[x]".
These were handled during cleaning and treated with sensitivity to outliers in
interpretation.


--------------------------------------------------------------------------------
4. DATA PREPARATION
--------------------------------------------------------------------------------
Nine datasets with different structures (financial years like "2004-2005",
integer years, mixed column naming) were standardised with four helper
functions:
    clean_numeric()        Convert values like "[x]" to NaN.
    clean_borough_code()   Filter and validate borough codes.
    wide_to_long()         Reshape year-as-column data into panel format.
    parse_financial_year() Extract calendar years from financial-year strings.

A borough lookup table (built from the affordability dataset) was used as the
reference key for merging across datasets with varied code systems.


--------------------------------------------------------------------------------
5. FEATURE ENGINEERING
--------------------------------------------------------------------------------
- Year-over-year changes via grouped diff() and pct_change() (dynamics, not
  levels).
- Per-capita normalisation: waiting_list_per_100, social_stock_per_100,
  vacancies_per_1000 (to compare boroughs of different population sizes).
- social_rent_annual: weekly social rent x 52.
- rent_gap_ratio: house price / annualised social rent (gentrification pressure
  indicator, following Smith's rent gap theory).
- Composite gentrification index (weighted combination):
      house_price_pct_change          +0.4  (primary displacement mechanism)
      social_housing_stock_pct_change -0.3  (expansion opposes gentrification)
      vacant_dwellings_pct_change     -0.1  (vacancy reduction may signal demand)
      rent_gap_ratio                  +0.2  (rent gap theory)
  Standardised before weighting. A PCA-based alternative
  (gentrification_index_pca) was also computed (29% explained variance).
- inner_london_location: binary indicator from ONS borough classifications.
- Lagged affordability variables (1-3 years) for predictive modelling, built to
  prevent data leakage.


--------------------------------------------------------------------------------
6. METHODS
--------------------------------------------------------------------------------
Q1  Pearson correlation + OLS regression with clustered standard errors
    (clustering by borough, since panel data has repeated observations and OLS
    assumes independence).
Q2  K-means clustering (k=3: affordable, mid-range, unaffordable). Z-score
    standardisation applied before clustering (Euclidean distance). ANOVA used
    to validate cluster separation.
Q3  Composite gentrification index vs affordability; 70/30 train-test split
    comparing Linear Regression, Random Forest, and Gradient Boosting; feature
    importance analysis.


--------------------------------------------------------------------------------
7. KEY RESULTS
--------------------------------------------------------------------------------
Correlations:
    - affordability_ratio vs house_price : r = 0.95
    - rent_gap_ratio vs house_price       : r = 0.92
    - rent_gap_ratio vs affordability     : r = 0.85
    (Multicollinearity prompted using percent changes for the index.)

Q1 - Social housing stock vs affordability:
    - Negative relationship (r = -0.4275): boroughs with larger stock increases
      saw smaller affordability deterioration.
    - Most boroughs GAINED stock over 20 years (exception: Kensington & Chelsea),
      contradicting the "loss of social housing" narrative.

Q2 - Borough clustering:
    - Three distinct typologies confirmed by ANOVA (F = 37.3, p < 0.001).
    - Widening inequality: in 2004 clusters sat at ratios 8-10; by 2017-2022 the
      "Unaffordable" cluster diverged to 21-24 while others stayed at 11-14.

Q3 - Gentrification index prediction:
    - Index explained only 11.8% of affordability variation (R2 = 0.118), weaker
      than the simpler stock-change analysis (R2 = 0.183).
    - All three models performed similarly (R2 = 0.94), validating a linear
      framework.
    - Feature importance: affordability_lag1 dominates (0.953); gentrification
      index minimal (0.046); inner_london_location negligible (0.001).


--------------------------------------------------------------------------------
8. KEY FINDINGS
--------------------------------------------------------------------------------
- Boroughs building more social housing experienced modestly smaller
  affordability deterioration, though the effect is small.
- Three distinct borough typologies show sharply diverging trajectories (the gap
  between Unaffordable and Affordable clusters widened from 2.4 to 10.4).
- The gentrification index failed to predict affordability effectively; complex
  composite features did not beat simpler measures.
- Affordability is highly persistent: lagged affordability drives prediction,
  implying limited scope for short-term policy intervention.


--------------------------------------------------------------------------------
9. LIMITATIONS & FURTHER WORK
--------------------------------------------------------------------------------
Limitations:
    - Observational design prevents causal claims.
    - Treating boroughs as independent units ignores spatial dependencies.
    - No control for fixed borough traits (history, geography).
    - High predictive R2 explains past affordability rather than forecasting.

Further work:
    - Add borough-level income data to separate price effects from earnings.
    - Add transport accessibility to study commuting and demand distribution.
    - Extend coverage pre-2004 to capture earlier cycles and Right to Buy.
    - Explicitly model demand-feedback mechanisms to test whether social housing
      behaves differently from market supply.


--------------------------------------------------------------------------------
10. SELECTED REFERENCES
--------------------------------------------------------------------------------
- Fingleton, B., Fuerst, F. and Szumilo, N. (2018). Housing affordability: Is new
  local supply the key? Environment and Planning A, 51(1), pp.25-50.
  doi:10.1177/0308518x18798372.
- Smith, N. (1979). Toward a Theory of Gentrification: A Back to the City
  Movement by Capital, not People. JAPA, 45(4), pp.538-548.
- Bank of England (2020). How does the housing market affect the economy?
- Office for National Statistics (2025). Housing affordability in England and
  Wales.
- Keep, M. (2023). Housing Market: Key Economic Indicators. House of Commons
  Library.

