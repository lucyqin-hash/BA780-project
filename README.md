# Beyond Income: How Place Shapes Adult Obesity in the U.S.

**BA780 Introduction to Data Analytics · Boston University Questrom School of Business · Team B09**

> In 2022, adult obesity ranged from **24.3%** (Washington, DC) to **41.0%** (West Virginia). The gap appears in adults as young as 18–24, and in the highest-obesity states even college graduates and high earners are above the national average. This project combines CDC survey data with the USDA Food Environment Atlas to ask what people who live in the same state have in common.

---

## Overview

Roughly one in three U.S. adults now has obesity, but rates are uneven across states. We merged state-level obesity estimates from the CDC's BRFSS survey with food-environment, income and recreation indicators from the USDA Food Environment Atlas. We then compared states, demographic groups and income brackets to find which differences hold up when the usual explanations (age, income) are held constant.

The analysis is written for a **state public-health department** (for example, a chronic disease or nutrition division) that has a limited budget for programs such as SNAP incentives, healthy-store incentives and community recreation.

## Objective

**Guiding question:** How are food environment, socioeconomic status, demographic characteristics and geographic location associated with adult obesity across the U.S., and where should a public-health department focus its prevention programs?

Sub-questions:
1. How much has adult obesity risen from 2011 to 2024, and which states carry the highest burden?
2. Do differences in age or income explain the gap between states?
3. Which parts of the food environment are associated with state obesity rates?
4. How well do food-environment factors alone explain state obesity, and which states do they miss?

## Data Sources

| Dataset | Publisher | Raw size | Level | Used for |
|---|---|---|---|---|
| [Nutrition, Physical Activity, and Obesity – BRFSS](https://catalog.data.gov/dataset/nutrition-physical-activity-and-obesity-behavioral-risk-factor-surveillance-system) | CDC (data.gov) | 110,880 rows × 33 columns, 2011–2024 | State × year × demographic group | Obesity, physical inactivity, fruit/vegetable intake by age, education, income, race and sex |
| [Food Environment Atlas](https://ers.usda.gov/data-products/food-environment-atlas/data-access-and-documentation-downloads) (`StateAndCountyData.csv`, `VariableList.csv`) | USDA Economic Research Service | 957,753 rows (long format), 304 variables, 3,156 counties | County | Stores and restaurants per 1,000 people, food access, recreation facilities, median income, poverty |

The two datasets are joined on the two-letter state code. The final analysis table covers **51 jurisdictions (50 states + DC)**. Territories and the national row in BRFSS have no Atlas match.

## Tools

- **Python** in Google Colab
- **pandas, NumPy**: cleaning, reshaping (long → wide), aggregation, merging
- **Matplotlib, Seaborn**: all 15 charts
- **scikit-learn**: `LinearRegression` and `mean_absolute_error` for the predicted-vs-actual model

## Methodology

**1. BRFSS cleaning (110,880 → 97,666 rows)**
- Removed 13,214 rows with no estimate (suppressed because the sample size was too small). No duplicate rows were found.
- Dropped redundant stratification columns, mapped question IDs to short indicator names (e.g. `Q036 → obesity`, `Q047 → no_activity`), converted sample size to numeric and renamed columns.

**2. Food Environment Atlas cleaning (957,753 rows → 51 × 111 state table)**
- Replaced the sentinel codes `-9999` (86,308 values) and `-8888` (3,951 values) with `NaN`. Left in place, they would have pulled state averages negative.
- Removed rows with missing FIPS codes. No duplicates were found.
- Kept 5 of 9 Atlas categories (Access, Stores, Restaurants, Health, Socioeconomic).
- Fixed variables labeled *Count* in the data dictionary that are actually per-1,000 rates (`FSRPTH16`, `FSRPTH20`).
- Rolled counties up to states: **counts summed**, **rates and percentages averaged**, and non-aggregable fields (% change, classifications) excluded. Then pivoted long → wide.

**3. Integration**
- Many-to-one left merge of BRFSS onto the state-level Atlas table: 97,666 rows, of which 92,590 matched. The 5,076 unmatched rows are the U.S. national row and three territories.

**4. Analysis: from the national picture to the extreme states**
1. **National level.** Trend in adult obesity, 2011–2024 (Chart 1), and differences across demographic groups such as age and education (Charts 3–4).
2. **State level.** Ranking of states by 2022 obesity rate (Chart 2).
3. **Testing common explanations.**
   - Same age group compared across the 10 highest- and 10 lowest-obesity states (Chart 5).
   - State median income vs obesity (Chart 6), then the same income bracket compared across high-obesity (WV, LA, OK) and low-obesity (CO, VT, HI) states (Charts 7–8, 13).
   - Groups usually least likely to have obesity (college graduates, ages 18–24, $75K+) in high- vs low-obesity states (Chart 14).
4. **Food environment.** Pearson correlations between state obesity and nine store, restaurant, access and recreation variables (Chart 9).
5. **Extreme states.** Profiles showing each state's % difference from the all-state average for West Virginia, Vermont and Colorado (Charts 10–12).
6. **Combined model.** Multiple linear regression of 2022 state obesity on the nine food-environment variables, used **descriptively** to see how much of the cross-state variation they jointly account for and which states deviate (Chart 15).

## Visualizations

The notebook contains 15 charts. These five carry the main story:

| | |
|---|---|
| ![State ranking](images/chart02_state_ranking.png) **Highest and lowest obesity states, 2022.** WV 41.0% vs DC 24.3%; 22 states at 35%+. | ![Age gap](images/chart05_age_gap.png) **Same age, different states.** The gap between the 10 highest and 10 lowest states is already ~9 points at age 18–24. |
| ![Income within states](images/chart07_08_income.png) **Same income bracket, different state.** Higher income lowers obesity in low-obesity states, but barely in high-obesity states. | ![Food environment correlations](images/chart09_food_env_corr.png) **Food environment vs obesity (r).** Dollar stores +0.62; full-service restaurants −0.69. |
| ![West Virginia vs Colorado](images/chart10_12_state_profiles.png) **State profiles.** West Virginia differs from the average in the higher-obesity direction on every measure; Colorado in the lower-obesity direction. | |

> Export each chart from the notebook into an `images/` folder using the file names above.

## Key Insights

1. **Obesity is rising and concentrated by place.** U.S. adult obesity rose from **27.4% (2011) to 33.1% (2024)**. In 2022 the state average was **33.8%**, ranging from **24.3% (DC) to 41.0% (WV)**. **22 of 51** jurisdictions were at 35% or higher, mostly in the South and Appalachia.

2. **Demographics matter nationally.** Obesity is **27.3%** among college graduates vs **37.6%** among adults without a high school diploma, and rises from **20.9%** at ages 18–24 to **39.0%** at ages 45–54.

3. **Age does not explain the gap between states.** Comparing the same age group, the 10 highest-obesity states are about **9 points** higher than the 10 lowest at ages 18–24, and nearly **14 points** higher by middle age. The divide starts in early adulthood.

4. **State income matters, but personal income does not close the gap.**
   - State median household income has the strongest single correlation with obesity (**r = −0.71**).
   - Yet in WV, LA and OK, higher-income adults report **19.5 points less physical inactivity** and **11.5 points fewer** "vegetables less than once a day" responses than lower-income adults, but their obesity rate is **0.6 points higher**, not lower.
   - In CO, VT and HI, a similar drop in inactivity (18.4 points) comes with **3.6 points lower** obesity.
   - Among higher-income adults, Louisiana and Colorado differ by **17.4 points**. College graduates, 18–24-year-olds and $75K+ earners in the high-obesity states remain above the national average.

5. **The local food environment tracks obesity, and supermarket access does not** (correlation with state obesity, 2022):

   | Higher obesity | r | Lower obesity | r |
   |---|---|---|---|
   | Dollar stores | +0.62 | Full-service restaurants | −0.69 |
   | Convenience stores | +0.45 | Fast-food restaurants | −0.56 |
   | Supercenters & club stores | +0.44 | Recreation & fitness facilities | −0.49 |
   | SNAP-authorized stores | +0.43 | Grocery stores | −0.20 |
   | | | % low access to a supermarket | **0.00** |

   The negative sign for fast food suggests these density measures partly reflect urbanization and income rather than a direct effect of the outlets themselves.

6. **The extremes have opposite profiles.**
   - West Virginia (41.0%) has **0.26 dollar stores per 1,000 people vs 0.17 on average (+52%)**, fewer full-service restaurants (0.65 vs 0.87) and a median income of $48.4K vs $63.4K.
   - Colorado (25.0%) has **1.71 full-service restaurants per 1,000 (+96%)** and more recreation facilities (+41%), while its median income is only about **6%** above average. Income alone does not account for its position.

7. **Food-environment factors account for most, but not all, of the variation.** A linear model on the nine factors (n = 50 states with complete data) gives an **in-sample R² of 0.69** and a **mean absolute error of 1.7 points**. West Virginia's actual rate is **3.4 points above** the prediction and Vermont's **4.4 points below**, so other factors are also at work.
   - The model is descriptive. With 9 correlated predictors on 50 observations and no hold-out set, the R² is optimistic and individual coefficients are not interpretable. For example, the recreation coefficient turns positive although its simple correlation is −0.49.

## Business Value

For a state public-health department with a limited prevention budget, the analysis supports three decisions:

- **Where to start:** West Virginia, Louisiana and Oklahoma first, then the other states at 35%+. High dollar-store density and low full-service restaurant density are inexpensive public indicators for flagging places that need closer review.
- **Who to reach:** Because even higher earners and college graduates in high-obesity states are above the national average, programs should target **whole communities**, not only low-income residents.
- **What to fund:** Supermarket access shows no state-level relationship with obesity. That argues for improving what is sold in the stores people already use (SNAP produce incentives, healthy-store programs in dollar and convenience stores) and for affordable recreation and safe walking spaces, rather than relying mainly on new supermarkets.

Suggested partners: CDC and USDA (funding), county health offices, city planners and parks departments, retailers, community organizations, schools and employers. We recommend piloting programs in a few counties and measuring results before scaling.

## Conclusion

Adult obesity in the U.S. is strongly tied to where people live. The gap between states appears among young adults, persists within the same income bracket, and reaches groups usually considered low-risk. States with more dollar and convenience stores and fewer full-service restaurants and recreation facilities tend to have higher obesity, while supermarket access shows no relationship. Prevention is therefore likely to work better when it changes community conditions than when it only targets individual behavior.

**Limitations**
- All results are correlational and at the state or group level. Income, urbanization and age are intertwined.
- County values were averaged without population weights. Some Atlas variables are missing for many counties (supercenters ~79%, fitness facilities ~59%).
- Variable years differ (obesity 2022; stores 2020; income 2021; SNAP stores 2023; food access 2019).
- BRFSS uses self-reported height and weight, which tends to understate obesity. Cross-state comparisons remain valid if the bias is similar across states.
- Small subgroups have wide confidence intervals. The two income comparisons use slightly different bracket cutoffs.

**Future work:** add county-level obesity estimates (e.g. CDC PLACES), weight county-to-state aggregation by population, add variables such as food prices, transit and walkability, and evaluate obesity trends before and after specific interventions.

## Repository Structure

```
├── B09-US-Obesity-Food-Environment.ipynb   # full analysis notebook
├── images/                                  # exported charts used in this README
└── README.md
```

**To reproduce:** download `BRFSS_Obesity.csv`, `StateAndCountyData.csv` and `VariableList.csv` from the sources above into a `data/` folder (or set `DATA_DIR` in the setup cell), then run all cells.

## Team

Bhavya Kandhari · Zoya Salman · Danyang (Lucy) Qin, project manager · Sanjay Kumar · Dylan Rubenstein
