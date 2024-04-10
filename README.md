
# **1. Introduction**

The U.S. Presidential elections, occurring every four years, mark critical junctures in American politics and global geopolitics due to the U.S.'s significant role in the world economy. The 2024 Presidential Election is poised to be the next pivotal moment in American politics, shaping the nation’s trajectory. Furthermore, with the U.S. currently boasting the largest economy in the world in terms of nominal GDP, the U.S. Presidential Election holds significant implications for global geopolitics, economic policies, and diplomatic relations. As candidates vie for public attention and support, it becomes imperative to gauge the sentiments and interests of the electorate. 

In the context of increasingly digital campaign strategies, utilisation of Wikipedia’s webpage API provides a rich source of real-time data & insights into the trends and patterns of the public interests. By examining web traffic, we aim to uncover how the candidates gain public interest along with their campaign strategies & political dynamics. Our focus on the U.S. presidential candidates allows us to hone in on a high-stakes political arena, where public perception and electoral outcomes is of utmost importance to the international economy. 

From the web traffic data from the 2016 elections, it is shown that the two candidates, Donald Trump and Hillary Clinton, had wikipedia pageviews of **6,137,438** and **1,233,122 **respectively. From this statistic we have hypothesized that the politician with the most Wikipedia pageviews would garner the most votes and hence win the election. 

Building a robust time-series predictive model allows us to provide valuable insights to political analysts, campaign strategists, and researchers interested in understanding the evolving landscape of American politics as well as identifying key factors that correlate with winning the elections.


# **2. Methodology**


## 2.1. Dataset

Data is collected from [Wikitech’s Pageview API](https://wikitech.wikimedia.org/wiki/Analytics/AQS/Pageviews), a public API developed and maintained by the Wikimedia Foundation. As an organisation that aids in data analysis about article pageviews of Wikipedia and sister projects, we are able to view the articles of a certain project and timespan. We will be using RStudio as the primary domain to extract from the English Wikipedia domain (_en.wikipedia.org_). The “_pageviews_” package in RStudio is an API client for the Wikipedia Traffic Data, providing pageview data from “Wikimedia” sites.

To build a reliable model that is applicable to the current candidates of 2024, we first based our analysis on a random past candidate, and fetched monthly pageview data for Barack Obama’s Wikipedia article from January 2016 to December 2023. The obtained data was converted into a time series object (_‘obama_ts_’) with a monthly frequency, which was divided by the monthly pageviews to obtain the average daily views for each month (“_obama_ts_DA_”). This procedure was repeated using daily pageview data, instead of monthly, covering the same time period, to obtain the time series object with a daily frequency (“_obama_ts_daily_”). The daily frequency time series data collected in Figure 2 shows a higher volatility due to outliers compared to the monthly frequency. Refer to Section A of the Reference section for the exploratory visualisations of each extracted data. 


## 2.2. Data Pre-processing

The dataset was partitioned into training (2016-2021) and testing sets (2022-2023) for both monthly and daily frequencies. The resulting sets are “_obama_train_month_”, “_obama_test_month_”, “_obama_train_daily_”, and “_obama_test_daily_”.

The decomposition process helps us identify underlying patterns and trends in the data, aiding in model selection and forecasting. Initial decomposition on the 2 training sets allowed us to infer potential variance or skewness. However, both the data exhibited relatively constant variance, making a Box-Cox transformation unnecessary.  Refer to Section B, Figures 3 and 4, of the Reference section for the initial decomposition plots.

So far, we applied Seasonal Decomposition using Loess (STLF) to decompose the time series into their trend, season, and remainder components. Due to its applicability to the current data, the chosen two benchmark models were used - “Naive”, “Random Walk with Drift” to help evaluate other models we will be using. Along with the benchmark models, we also created an ARIMA model that is applied directly to the decomposed time series as an extra layer of guestimate due to its suitability for the data that we are using . These models were then auto-layered to visualise and guide the selection of appropriate models for further analysis. Refer to Section B, Figures 5-6 and Table 1, of the Reference section to view the STLF decomposition visualisation along with their forecasts. 

Along with the respective forecasts, the prior splitting of train and test sets, allowed us to display accuracy computations using Mean Error (ME), Root Mean Squared Error (RMSE), Mean Absolute Error (MAE), and Mean Absolute Percentage Error (MAPE). 


# **3. Model Selection**

From our initial models, the Naive and ARIMA models showed the smallest RMSE values for “Average Daily Views” data and “Average Daily Views by Month” data, which served as complementary insights. Basic benchmark models have limitations in capturing non-linear or multiple seasonal patterns that may potentially be involved. We delve further into “Splines”, “Auto ARIMA”, and “ARIMA-X” for our next steps in our iterative modelling approach. 


## 3.1. Splines

Splines is a flexible non-linear regression technique, advantageous in capturing the underlying trend component in the time series data. Given the computational demands of daily series, our analysis predominantly targets monthly aggregates, trying to strike a balance between granularity and computational efficiency. 

We utilised the ‘_splinef_” function to fit splines to the monthly data, with a forecast horizon of 24 months, mirroring the critical pre-election period where public interest intensifies.. This captures the underlying trend component in the time series data, providing a smooth representation of the trend pattern over time. We then created additional fitted models by combining the spline-fitted values (_“obama_month_fitted_s”_) with the predictions generated by the earlier models (Naive, Random Walk with Drift, and ARIMA). This step allows us to enhance the benchmark models with the spline component, capturing both short- and long-term fluctuations. We plotted the observed monthly data against the spline-fitted values and combined predictions from each benchmark model with the spline component overlaid for comparison. Finally, we evaluated the performance of each model using the standard accuracy metrics against the test dataset.  Refer to Figure 7 in Section C for the autoplot of fitted models with Splines.

To improve the predictive capabilities of spline models in our time series analysis and fine-tune knot selection, we can implement a series of strategic enhancements. Initially, leveraging tools such as the Akaike Information Criterion (AIC) or cross-validation facilitates the automatic determination of the optimal number and location of knots, striking a balance between model adaptability and precision. 


## 3.2. MSTL (Multiple Seasonal Decomposition of Time Series by Loess)

MSTL is a decomposition method used to extract multiple seasonal components from time-series data. This technique is particularly useful when the data exhibit complex seasonal patterns with multiple frequencies. Unlike Splines, building the MSTL model on the “Average Daily Views” data is beneficial as this data exhibits complex seasonal patterns, including daily, weekly, monthly and yearly fluctuations. These components interact in non-linear ways, making it challenging to model the data accurately using traditional methods.

We applied MSTL to decompose the daily time series data (“_obama_train_daily_”) into its various seasonal components using the ‘_mstl_’ function. The resulting decomposition provided insights into the weekly, monthly, and yearly seasonal patterns present in the data, as seen in Figure 8 from Section C of References. We then split the decomposition into its seasonal and seasonally adjusted components to facilitate forecasting. The seasonally adjusted component (“_obama_s_adj_”) was obtained by summing the trend and residual components. To split the individual seasonal components, we extracted the weekly, monthly and yearly frequency from the MSTL decomposition results to “_obama_seasonal_weekly_”, “_obama_seasonal_monthly_” and “_obama_seasonal_yearly_”. Each seasonal component was forecasted using simple exponential smoothing ‘_snaive_’. Finally, we employed “_auto.arima_” to fit an ARIMA model to the seasonally adjusted component (“_obama_s_adj_”) and generated forecasts, which were then combined to obtain the overall ARIMA model. Refer to Figure 9-10 of Section C of the References for the separate forecasts of each seasonal component, and the combined forecasts from all components.


## 3.3. ARIMA (Auto Regressive Integrated Moving Average)

The ARIMA model is a popular time series forecasting method that combines autoregressive, differencing and moving average components to capture temporal dependencies and patterns present in the data. Aligning to the initial finding where ARIMA displayed better performance for the “Average Daily Views by Month” data, and to simplify the complexity of multiple seasonal components in MSTL, this approach is suitable for time series data exhibiting stationary behaviour and linear trends.

First, we applied the “_auto.arima_” function to automatically select the best-fitting ARIMA model for the seasonally adjusted component (“_obama_s_adj_”) obtained from the MSTL decomposition. The “_auto.arima_” function employs a heuristic algorithm to identify the optimal ARIMA parameters based on minimising information criteria such as AIC and BIC. The resulting ARIMA model used was ARIMA(5, 1, 1) with an AIC of 52430.75 and BIC of 52470.59. The ARIMA(5, 1, 1) model was then used to generate forecasts for the seasonally adjusted component over a forecasting horizon of 730.5 time points (approximately 2 years). The forecast can be seen in Figure 11 of Section C in the References. The final forecast for the original time series, we combined the ARIMA forecasts for the seasonally adjusted component with forecasts generated using simple exponential smoothing (“_snaive_”) for the yearly seasonal component (“_obama_seasonal_yearly_”). This combined forecasts represents the predicted values for the entire time series, incorporating both the trend captured by the ARIMA model and the seasonal patterns captured by the “_snaive_” method. Refer to Figure 12 of Section C in the References for the forecast plot.


## 3.4. ARIMA-X (ARIMA with Exogenous Variables)

The ARIMA-X model extends the traditional ARIMA framework by incorporating additional external variables, known as exogenous variables, into the model. These variables provide additional information or predictors that enhance the forecasting accuracy of the ARIMA model.

Similar to the ARIMA model, we applied the “_auto.arima_” function to select the best-fitting ARIMA model for the seasonally adjusted component (“_obama_s_adj_”). In this model, we included an exogenous variable represented by the daily pageviews data for the Democratic Party (“_dem_ts_daily_”). The specification of ‘_x-reg_’ in the “_auto.arima_” function allows us to include external regressors in the ARIMA model. The selected ARIMA-X model, ARIMA(5, 1, 1), was then used to generate forecasts over the same forecasting horizon of 730.5 time points. The final forecast for the original time series was a combination of ARIMA-X forecasts and “_snaive_” smoothing for both the seasonally adjusted and yearly seasonal component. The combined forecast can be seen in Figure 13 of Section C in the Reference.


# **4. Implementation of Model**


## 4.1 Forecasting with ARIMA-X

Upon our previous findings, ARIMA-X model demonstrated effectiveness in forecasting pageviews for Obama, particularly leveraging the influence of party-related pageviews. We now turn our focus to predict the online engagement of key presidential candidates for the 2024 U.S. elections. We will be incorporating the pageviews of their respective political parties, namely the Democrats and Republicans, to provide insightful forecasts for the following prominent candidates. The Wikipedia page for Jason Palmer was created on 6 March 2024, hence we will not be able to forecast or provide any page view analysis for Palmer.


<table>
  <tr>
   <td colspan="2" ><strong>2024 U.S. Presidential Candidates</strong>
   </td>
  </tr>
  <tr>
   <td><strong><em>Democrats</em></strong>
   </td>
   <td><strong><em>Republicans</em></strong>
   </td>
  </tr>
  <tr>
   <td>Marianne Williamson
   </td>
   <td rowspan="3" >Donald Trump
   </td>
  </tr>
  <tr>
   <td>Jason Palmer
   </td>
  </tr>
  <tr>
   <td>Joe Biden
   </td>
  </tr>
</table>


For each candidate, we will be using the above line of process taken to view any preliminary information that may be useful in finding a better performing model. These steps will allow us to see if there is any information we can use to help campaigning parties plan their campaigns when using page views as a proxy for their political traction. Our last step will be to observe the accuracy metrics from the selected model

More importantly, we will focus on the **MAPE** values of each model due to the large differences in pageviews for each politician in order to establish a comparable metric between all models of each candidate. 


## 4.2 Findings


<table>
  <tr>
   <td><strong>Candidate </strong>
   </td>
   <td><strong>Selected Model </strong>
   </td>
   <td><strong>MAPE </strong>
   </td>
  </tr>
  <tr>
   <td>Williamson 
<p>
<em>(Democratic)</em>
   </td>
   <td>ARIMA-X
   </td>
   <td>126.520
   </td>
  </tr>
  <tr>
   <td>Biden 
<p>
<em>(Democratic)</em>
   </td>
   <td>ARIMA-X
   </td>
   <td>89.128
   </td>
  </tr>
  <tr>
   <td>Trump 
<p>
<em>(Republican)</em>
   </td>
   <td>ARIMA-X
   </td>
   <td>183.572
   </td>
  </tr>
</table>


Our findings have shown that generally benchmark models had the smallest MAPE, however we decided not to proceed with the benchmark models as this is a special case of forecasting, where people do not have much interest in the politicians except for the period right before the presidential election. The next best model with the smallest MAPE is ARIMA-X with only yearly seasonality for all 3 candidates, outperforming ARIMA and MSTL with ARIMA with all seasonalities significantly. It was therefore chosen as the best model for forecasting pageviews for all candidates in the upcoming election.

Refer to Section D of References for detailed MAPE values of each model and each candidate in Table 2-4. Forecasted Graphs with ARIMA-X for each Candidate can be found in Table 5 of Section D in the References.


# 5. Conclusion 

In conclusion, the findings of this report revealed that ARIMA-X models exhibited good performances for the Obama dataset, which was applicable to the other politicians and their datasets as well. While the test errors provide a good indication of how the models perform with in-sample data, the errors should only be used as a guideline and ultimately require some contextual application. In the case of forecasting pageviews, despite benchmark models performing well, we chose not to use them as they fail at capturing the spikes occurring around the election season. We believe ARIMA-X is a better model despite a higher MAPE as each election there are different candidates. We decided to go with the candidates’ respective parties as the exogenous variables as the U.S. generally exhibits partisan views on presidential candidates, thus we believe that the party serves as a good influence for the respective pageviews.

Moving forward, extending ARIMA, ARIMAX models to include other predictors such as social media sentiment indices, economic indicators, or significant political events. This could provide a more nuanced understanding of the factors driving pageviews and improving forecasting accuracy.
For time series forecasting tasks like predicting website pageviews, neural network architectures that can capture sequential dependencies and patterns over time are particularly effective. Long Short-Term Memory (LSTM) Networks are a good type as they overcome the vanishing gradient problem, allowing them to learn long-term dependencies. LSTMs can capture the seasonal and trend components in pageview data, learning from past traffic patterns to predict future pageviews, which may be influenced by factors like seasonal trends, promotions, or events.


On a fun sidenote, here is our predicted probability for each candidate winning the next presidential election, based purely on candidates’ pageviews: 


<table>
  <tr>
   <td>Candidate
   </td>
   <td>Peak Pageview
   </td>
   <td>Probability of Winning
   </td>
  </tr>
  <tr>
   <td>Williamson (D)
   </td>
   <td>72,066
   </td>
   <td>2%
   </td>
  </tr>
  <tr>
   <td>Biden (D)
   </td>
   <td>798,411
   </td>
   <td>23%
   </td>
  </tr>
  <tr>
   <td>Trump (R)
   </td>
   <td>2,589,166
   </td>
   <td>70%
   </td>
  </tr>
  <tr>
   <td>Others
   </td>
   <td>N/A
   </td>
   <td>5%
   </td>
  </tr>
</table>


All candidates have a **non-zero** probability of winning, as there is always a chance of winning, which makes our predictions always correct to a certain extent regardless.



# **6. References**


## Section A - Datasets




![Figure 1](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%201_Datasets.png)
 \
_(Figure 1 - “obama_ts_DA”)_


![Figure 2](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%202_Datasets.png)
 \
_(Figure 2 - “obama_ts_daily”)_


## Section B - Data - Preprocessing


![Figure 3](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%203_DataPreprocessing.png)


_(Figure 3 - Decomposition of Average Daily Views by Month)_



![Figure 4](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%204_DataPreprocessing.png)


_(Figure 4 - Decomposition of Average Daily Views)_




![Figure 5](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%205_DataPreprocessing.png)


_(Figure 5 - STL Decomposition with Forecasts for Average Daily Views per Month)_




![Figure 6](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%206_DataPreprocessing.png)


_(Figure 6 - STL Decomposition with Forecasts for Daily Views)_


<table>
  <tr>
   <td><strong>Data</strong>
   </td>
   <td><strong>Benchmark</strong>
   </td>
   <td><strong>ME</strong>
   </td>
   <td><strong>RMSE</strong>
   </td>
   <td><strong>MAE</strong>
   </td>
   <td><strong>MAPE</strong>
   </td>
  </tr>
  <tr>
   <td rowspan="3" ><strong>Average Daily Views per Month</strong>
   </td>
   <td><strong>Naive</strong>
   </td>
   <td>6836.542
   </td>
   <td>11827.813
   </td>
   <td>7274.521
   </td>
   <td>23.164
   </td>
  </tr>
  <tr>
   <td><strong>Random Walk with Drift</strong>
   </td>
   <td>8505.438
   </td>
   <td>13313.557
   </td>
   <td>8630.990
   </td>
   <td>28.216
   </td>
  </tr>
  <tr>
   <td><strong>ARIMA</strong>
   </td>
   <td>-4328.455
   </td>
   <td>10112.845
   </td>
   <td>7949.292
   </td>
   <td>32.864
   </td>
  </tr>
  <tr>
   <td rowspan="3" ><strong>Daily Views</strong>
   </td>
   <td><strong>Naive</strong>
   </td>
   <td>6880.324
   </td>
   <td>27520.051
   </td>
   <td>7453.441
   </td>
   <td>20.412
   </td>
  </tr>
  <tr>
   <td><strong>Random Walk with Drift</strong>
   </td>
   <td>8249.794
   </td>
   <td>28064.768
   </td>
   <td>8557.363
   </td>
   <td>24.658
   </td>
  </tr>
  <tr>
   <td><strong>ARIMA</strong>
   </td>
   <td>8486.012
   </td>
   <td>28134.812
   </td>
   <td>8737.597
   </td>
   <td>25.407
   </td>
  </tr>
</table>


_(Table 1 - Benchmark Model Error Values, rounded to 3 d.p.)_


## Section C - Models




![Figure 7](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%207_Models.png)


_(Figure 7 - Splines Autoplot of Fitted Models)_




![Figure 8](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%208_Models.png)


_(Figure 8 - MSTL Decomposition of Daily Time Series Data)_




![Figure 9](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%209_Models.png)


_(Figure 9 - MSTL Forecasts of Each Seasonal Component)_




![Figure 10](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%2010_Models.png)


_(Figure 10 - MSTL Combined Forecasts from All Components)_




![Figure 11](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%2011_Models.png)


_(Figure 11 - Forecasts from ARIMA(5,1,1))_




![Figure 12](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%2012_Models.png)


_(Figure 12 - ARIMA final forecast with “snaive”)_




![Figure 13](https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%2013_Models.png)


_(Figure 13 - ARIMA-X final forecast with “snaive”)_


## 


## Section D - Findings


<table>
  <tr>
   <td><strong>Data</strong>
   </td>
   <td><strong>Model</strong>
   </td>
   <td><strong>MAPE</strong>
   </td>
  </tr>
  <tr>
   <td rowspan="5" ><strong>Daily Views</strong>
<p>
<em>(Williamson)</em>
   </td>
   <td><em>Naive</em>
   </td>
   <td>53.152
   </td>
  </tr>
  <tr>
   <td><em>Random Walk with Drift</em>
   </td>
   <td>51.241
   </td>
  </tr>
  <tr>
   <td><em>ARIMA</em>
   </td>
   <td>53.152
   </td>
  </tr>
  <tr>
   <td><em>Auto Arima with MSTL</em>
   </td>
   <td>161.498
   </td>
  </tr>
  <tr>
   <td><em>ARIMA-X</em>
   </td>
   <td>126.521
   </td>
  </tr>
</table>


_(Table 2 - Model MAPE Values of Williamson, rounded to 3 d.p.)_


<table>
  <tr>
   <td><strong>Data</strong>
   </td>
   <td><strong>Model</strong>
   </td>
   <td><strong>MAPE</strong>
   </td>
  </tr>
  <tr>
   <td rowspan="5" ><strong>Daily Views</strong>
<p>
<em>(Biden)</em>
   </td>
   <td><em>Naive</em>
   </td>
   <td>30.494
   </td>
  </tr>
  <tr>
   <td><em>Random Walk with Drift</em>
   </td>
   <td>32.968
   </td>
  </tr>
  <tr>
   <td><em>ARIMA</em>
   </td>
   <td>29.428
   </td>
  </tr>
  <tr>
   <td><em>Auto Arima with MSTL</em>
   </td>
   <td>155.438
   </td>
  </tr>
  <tr>
   <td><em>ARIMA-X</em>
   </td>
   <td>89.128
   </td>
  </tr>
</table>


_(Table 3 - Model MAPE Values of Biden, rounded to 3 d.p.)_


<table>
  <tr>
   <td><strong>Data</strong>
   </td>
   <td><strong>Model</strong>
   </td>
   <td><strong>MAPE</strong>
   </td>
  </tr>
  <tr>
   <td rowspan="5" ><strong>Daily Views</strong>
<p>
<em>(Trump)</em>
   </td>
   <td><em>Naive</em>
   </td>
   <td>209.525
   </td>
  </tr>
  <tr>
   <td><em>Random Walk with Drift</em>
   </td>
   <td>287.740
   </td>
  </tr>
  <tr>
   <td><em>ARIMA</em>
   </td>
   <td>194.618
   </td>
  </tr>
  <tr>
   <td><em>Auto Arima with MSTL</em>
   </td>
   <td>204.559
   </td>
  </tr>
  <tr>
   <td><em>ARIMA-X</em>
   </td>
   <td>183.572
   </td>
  </tr>
</table>


_(Table 4 - Model MAPE Values of Trump, rounded to 3 d.p.)_


<table>
  <tr>
   <td><strong>Candidate </strong>
   </td>
   <td><strong>Forecast Graph</strong>
   </td>
  </tr>
  <tr>
   <td>Williamson (D)
   </td>
   <td>


<img src="https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%2014_Findings.png" width="" alt="alt_text" title="Williamson">

   </td>
  </tr>
  <tr>
   <td>Biden (D)
   </td>
   <td>


<img src="https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%2015_Findings.png" width="" alt="alt_text" title="Biden">

   </td>
  </tr>
  <tr>
   <td>Trump (R)
   </td>
   <td>


<img src="https://github.com/potafrey/DSA301-Time-Series-Project/blob/main/Submissions/img/Figure%2016_Findings.png" width="" alt="alt_text" title="Trump">

   </td>
  </tr>
</table>


_(Table 5 - ARIMA-X Forecast Graph for 2024 Candidates, rounded to 3 d.p.)_
