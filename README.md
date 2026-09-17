# Can Google search interest anticipate which foods are about to grow in popularity?

This project uses six years of Google Trends data (2019–2026) across approximately 200 food and beverage keywords to investigate whether recent search behavior can help predict future changes in food trends.

I initially approached the problem as a regression task, predicting future search interest six months ahead. Because search interest is highly persistent over time, I then reframed the problem as a classification task: will a food's average search interest be higher six months from now than it is today?

The classification approach produced a stronger and more consistent signal.

## Key Results
Task	Best Model	Performance
Predict future search level	XGBoost	R² = 0.735
Predict trend direction	Random Forest (Ranger)	AUC = 0.861
Predict trend direction	Random Forest (Ranger)	Accuracy = 0.757
Naive direction baseline	Majority class	AUC = 0.500

The results suggest that the direction of future search interest is more predictable than its exact magnitude.

## Project Overview
### Data Collection

Google Trends data was collected using the R package gtrendsR.

Time period: January 2019 – August 2026
Geography: United States
Keywords: ~200 foods and beverages
Frequency: Monthly search interest
Search values below 1 were represented as 0.5

Examples of keywords include:

matcha, boba, birria, hot honey, chili crisp, ube, mochi, sourdough, ramen, tacos, pizza, and many others.

To reduce repeated API requests, downloaded trend data was stored locally as .rds files and combined into a single dataset.

### Feature Engineering

I created time-series features designed to capture recent growth, momentum, volatility, and changes in search behavior.

#### Growth:
- growth_3m
- growth_6m
- growth_12m

#### Moving Averages
- mean_3m
- mean_6m
- mean_12m
- mean_ratio_3m_12m

#### Momentum
- slope_3m
- slope_6m
- slope_12m
- acceleration_3m
- acceleration_6m
- acceleration_12m

#### Volatility
- volatility_3m
- volatility_6m
- volatility_12m

#### Trend Behavior
- breakout_12m
- drawdown
- monthly_change
- trend_strength

The primary regression target was the average search interest during the following six months.

For classification, the target was:

#### 1: Future six-month average search interest is higher than the current six-month average
#### 0: Future six-month average search interest is not higher

### Modeling Approach

To avoid look-ahead bias, I used a time-based train/test split:

Training: before January 2025
Testing: January 2025 – May 2026
Regression

I compared:

- Linear Regression
- K-Nearest Neighbors
- Decision Tree
- Random Forest (ranger)
- XGBoost
- Naive no-change baseline


| Model | RMSE | MAE | R² |
|---|---:|---:|---:|
| XGBoost | 9.257 | 7.122 | **0.735** |
| Ranger | 10.217 | 7.960 | 0.677 |
| Naive No Change | 10.448 | 7.653 | 0.662 |
| KNN | 12.188 | 9.483 | 0.540 |
| Decision Tree | 13.971 | 11.359 | 0.396 |
| Linear Regression | 15.414 | 12.451 | 0.264 |

Only XGBoost substantially improved over the naive persistence baseline.

### Classification

I then predicted whether search interest would increase over the following six months.

Models included:

- Majority-class baseline
- Logistic Regression
- KNN
- Decision Tree
- Random Forest (ranger)
- XGBoost

| Model | Accuracy | Balanced Accuracy | AUC |
|---|---:|---:|---:|
| Majority Baseline | 0.631 | 0.500 | 0.500 |
| Logistic Regression | 0.762 | 0.758 | 0.846 |
| KNN | 0.735 | 0.745 | 0.824 |
| Decision Tree | 0.756 | 0.745 | 0.789 |
| Random Forest | **0.759** | **0.770** | **0.863** |
| XGBoost | 0.742 | 0.756 | 0.836 |

Random Forest produced the strongest classification performance, with an AUC of approximately 0.86.

### Most Important Features

Feature importance varied between models and tasks, but several variables consistently appeared among the more influential predictors.

For the regression XGBoost model, the most important features included:

- volatility_12m
- drawdown
- mean_ratio_3m_12m
- acceleration_12m

For the classification models, important predictors included:

- acceleration_6m
- mean_ratio_3m_12m
- growth_12m
- trend_strength
- volatility_6m
- acceleration_12m

These features capture concepts similar to momentum, acceleration, volatility, and drawdown, rather than simply relying on the current search level.

## Why the Problem Was Reframed

The original goal was to predict the exact future level of search interest.

However, the naive model performed surprisingly well because Google search interest is persistent over time. Simply assuming that recent search interest would remain similar already explained a large amount of the future variation.

This led to a more useful question:

Instead of asking exactly how popular a food will become, can we predict whether its search interest will increase?

The direction-based formulation produced a substantially stronger signal, with Random Forest reaching an AUC of approximately 0.86 compared with 0.50 for the majority-class baseline.

## Tech Stack

Language

R

## Data & Analysis

- gtrendsR
- dplyr
- tidyr
- lubridate
- zoo

## Machine Learning

- mlr3
- mlr3learners
- mlr3benchmark
- mlr3extralearners
- ranger
- xgboost

## Visualization

- ggplot2
- mlr3viz
- Project Structure
- PredictingFoodTrends/

## Limitations

There are several important limitations to this analysis:

- Google search interest is a proxy for food popularity, not a direct measure of consumption.
- The analysis uses only U.S. Google search data, limiting generalizability.
- Ranger and XGBoost were run with default hyperparameters rather than extensive tuning.
- Model performance is based on one time-based train/test split.
- A rolling-origin validation strategy would provide a more reliable estimate of how stable the classification performance is across different time periods.

## Future Work

Potential improvements include:

- Incorporating seasonality explicitly
- Using rolling-origin validation across multiple historical cutoff dates
- Hyperparameter tuning for tree-based models
- Adding external predictors beyond Google search data
- Investigating whether search spikes represent temporary events or sustained trends
- Developing a ranking system for emerging foods based on predicted probability of growth
  
## Takeaway

The main finding is that predicting the exact magnitude of future search interest is difficult because search behavior is highly persistent, but predicting the direction of change appears more feasible.

This project demonstrates an end-to-end workflow involving:

Data collection → Time-series feature engineering → Time-based validation → Machine learning → Model comparison → Baseline evaluation → Interpretation
