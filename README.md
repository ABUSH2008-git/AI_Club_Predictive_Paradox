# Predictive Paradox - Power Demand Forecasting

Predicting next hour electricity demand for Bangladesh's power grid using historical data, weather and economic indicators.

## Overall flow ->

1. Loading the Data
2.EDA
3.Data Preprocessing
4.Feature Engineering
5.Model training
6.Evaluation
7.Feature Importance 

##Description of each step :

### Loading the data :-
Nothing out of the ordinary here , just loading the data , along with slight modifications in weather for future comfort.

###EDA :-
This is one of the most important steps in the entire project.
    PGCB::
    1.First we plot demand_mw against datetime to get overall idea of its distribution.
    2.See total missing values is each file -> necessary for preprocessing.
    3.Boxplot and histogram of frequency of demand_mw -> for outlier detection. We can see data is skewed through histogram and the fact that IQR is best for outlier removal through boxplot too.
    4. Managing the 30 min timestamps -> choosing 50/50 -> comparitively better MAPE
    5. Missing timestamps : The PGCB dataset has gaps in its hourly timeline. The full hourly date range was reconstructed using pd.date_range, and missing demand_mw entries were left as NaN intentionally — they are dropped later thru dropna() so fabricated demand values never enter the target column. Only the derived lag features are interpolated (linear, max 3 consecutive hours).
    WEATHER::
    4.The chosen columns in weather have no null values so we can skip imputation.
    5.Moreover the max and min values of weather data are logically possible so no outlier detection needed -> they are real values.
    GENERAL::
    6.Graphing the hourly , monthly and yearly variations of demand_mw for our analysis, use of each is given in comments in the code itself.

###Data Preprocessing:
Using our EDA now we will move on to preparing the data for our model.
     PGCB::
     1.IQR on demand_mw
     WEATHER::
     2.Converting the required things in weather data to numbers.
     ECONOMIC::
     3.Selecting the relevant fields from economic data.
     4.Filter and reshape -> convert years to rows and selected factors to columns.
     5.Renaming for convinience.
     6.Using ffill for filling nans.
    MERGING::
    7.Merge the three dataframes into df on basis of datatime/year
    8.Fill the missing values using ffill(time series)

###Feature Engineering:
Creating features that help in prediction from data.

    1.We chose time featues : hour {daily cycle}
                            :weekday/weekend {demand might differ on both}
                            :month {montly cycle}
                            :year {yearly cycle}
    2.Lag features: day lag , 2 days lag , 1 week lag
    
    3.Rolling mean : mean of last 24 hour values
    4.We create target variable -> next row/hour
    5.Drop nans
    6.A correlation heatmap was used to analyze relationships between features and electricity demand, helping identify important predictors and detect redundancy among variables.

###Model Training:

Split at 2024 -> train using data before 2024 to test for 2024 values

##Why use LightBGM::

The basic problem is on regression.
Here we have a tabular data with a lot of variables and data is in tabular format. Moreover the patterns are non linear and complex. The lag features create complex dependencies. Due to all these features we lean more towards models which use more number of decision trees like RandomForest , xg boost or LightBGM.

Amongst these we can use any one -> but LightBGM was chosen because the dataset was large{Better than XG boost} and timeseries{better than RandomForest} was involved. LightBGM infact gave the best MAPE amongst the three of them.

###Evaluation:

1.Simple Mape and Mae calculation 
2.Plot of actual vs predicted

###Feature Importance:
To understand how the model makes predictions, feature importance was analyzed using the trained LightGBM model. This shows which features have the most influence on predicting electricity demand.

From the results:
  1. Hour is the most important feature, indicating that electricity demand strongly follows daily patterns (e.g., peak hours in evening).
  2. Lag features (lag_24h, lag_48h) are also highly important, showing that past demand plays a major role in predicting future demand.
  3. Temperature has a moderate impact, reflecting the effect of weather on electricity usage.
  4.  Other features contribute less but still help improve overall prediction.


