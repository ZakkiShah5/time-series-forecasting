#### **Code understanding Project 3 (Only imp cells or new things)**





##### ***---Cell#01***



**%matplotlib inline**

**sns.set\_style('whitegrid')**



1. Show plots directly inside the notebook.
2. Gives white bg and gray grid lines for ease and clean plots.







##### ***---Cell#02***



**df\['date'] = pd.to\_datetime(df\['date'])**

**print(df\[date].dtype)**

**print(df\['date'].min(), "-->", df\['date'].max())**



1. Converts date column into datetime format, if not converted pandas read it as string.
2. check the type.
3. Know date range.





##### ***---Cell#03***

##### 

**print("Total days:", (df\['date'].max() - df\['date'].min()).days()+1)**

**print("Unique Stores:", df\['store'].nunique())**

**print("Unique Items:", df\['items'].nunique())**



**print("Total combination:",df\['store'].nunique()\*df\['items'].nunique())**

**print("Rows per store-item", len(df)/(df\['store'].nunique()\*df\['items'].nunique()))**





1. Gives total number of days.
2. Gives number of unique stores : 10
3. Gives number of unique items : 50
4. Total combination: 500
5. This checks number of rows exist for each store item pair.





##### ***---Cell#03***



**daily\_total = df.groupby("date")\['sales'].sum().reset\_index()**



**plt.figure(figsize=(14,5))**

**plt.plot(daily\_total\['date'], daily\_total\['sales'], linewidth=0.6)**





1. We group the data by date and sum the sales for each date.
2. We plot it.



**## Time series aggregation patterns**

***- df.groupby('date')\[value].sum() → total demand per day (collapses store-item dimension)***

***- df.groupby(\['date', 'store'])\[value].sum() → demand per store per day (preserves store dimension)***

***- df.groupby('date')\[value].mean() → average per day (different question)***

***- The aggregation level should match the question you're asking***







##### ***---Cell#04***





**sample\_month = daily\_total\[(daily\_total\['date']>='2017-07-01')\&(daily\_total\['date']<='2017-07-31')]**



**plt.figure(figsize(14,6))**

**plt.plot(sample\_month\['date'], sample\_month\['sales'], markaer='o', linewidth=1.5)**



1. We only take july's month of 2017 to zoom in to see weekly pattern.







##### ***---Cell#05***



**sample\_combos = \[**

&#x09;**(1,1),**

&#x09;**(1,25),**

&#x09;**(5,12),**

&#x09;**(10,45),**

**]**

**fig, axes = plt.subplots(2,2, figsize=(14,8))**

**axes = axes.flatten()**



**for i, (store,item) in enumerate(sample\_combos):**

&#x09;**series= df\[(df\['store']==store) \& (df\['item']==item)]**

&#x09;**axes\[i].plot(series\['date'],series\['sales'], linewidth=0.6)**

&#x09;**axes\[i].set\_title(f'Store {store}, Item {item}')**

&#x09;**axes\[i].set\_ylabel('Daily Sales')**



**plt.tight\_layout()**

**plt.savefig('../sample\_time\_series.png', dpi=100, bbox\_inches='tight')**

**plt.show()**



1. We choose random stores and items combos to inspect them individually if they're identical.
2. .flatten() converts from 2D axes array to 1D, so we can loop easily.





##### ***---Cell#06***



**from statsmodels.ts.seasonal import seasonal\_decompose**



**ts = daily\_total.set\_index('date')\['sales']**



**decomp = seasonal\_decompose(ts, model='additive', period=365)**



**fig, axes = plt.subplots(4,1, figsize=(14, 10))**



**axes\[0].plot(decomp.observed, linewidth=0.7)**

**axes\[0].set\_title('Original (Observed)')**



**axes\[1].plot(decomp.trend, linewidth=0.7)**

**axes\[1].set\_title('Trend')**



**axes\[2].plot(decomp.seasonal, linewidth=0.7)**

**axes\[2].set\_title('Seasonal (Annual)')**



**axes\[3].plot(decomp.resid, linewidth=0.7)**

**axes\[3].set\_title('Residual (Noise)')**



**plt.tight\_layout()**

**plt.savefig('../decomposition.png', dpi=100, bbox\_inches='tight')**

**plt.show()**





1. We convert our data to a time series, because decomposition needs a date-indexed series.
2. We use additive model where sale = trend+seasonality+Noise.
3. then we use the seasonal\_decompose function to plot the original, trend, seasonality, and residuals.



***## Additive vs multiplicative decomposition***

***- Additive: value = trend + seasonal + residual (constant-magnitude seasonality)***

***- Multiplicative: value = trend × seasonal × residual (proportional-magnitude seasonality)***

***- Choose by visual inspection: if peaks grow with trend, use multiplicative***

***- Convert to additive by log-transforming the series, then decomposing***





##### ***---Cell#07***



**from statsmodels.graphics.tsaplots import plot\_acf, plot\_pacf**



**fig, axes = plt.subplots(2,1, figsize=(14,8))**



**plot\_acf(ts, lag=60, ax=axes\[0])**

**axes\[0].set\_title('Autocorrelation (ACF)')**



**plot\_acf(ts, lag=60, ax=axes\[1], method='ywm')**

**axes\[1].set\_title('Partial Autocorrelation (PACF)')**



**plt.tight\_layout()**

**plt.savefig('../acf\_pacf.png', dpi=100, bbox\_inches='tight')**

**plt.show()**





1. We draw acf and pacf graphs with lags=60 on certain axes, with PACF method = 'ywm', Yule-Walker with a modification for better numerical stability.
2. ACF: How correlated is today's sales with sales from previous days?
3. PACF: What is the direct effect of a past day on today, after removing indirect effects?







##### ***---Cell#08***



**from statsmodels.tsa.stattools import adfuller, kpss**



**adf\_result = adfuller(ts.dropna())**

**print("=== ADF Test (Augmented Dickey-Fuller) ===")**

**print(f"ADF Statistic: {adf\_result\[0]:.4f}")**

**print(f"p-value:       {adf\_result\[1]:.4f}")**

**print(f"Verdict:       {'STATIONARY' if adf\_result\[1] < 0.05 else 'NON-STATIONARY'}")**

**print()**



**kpss\_result = kpss(ts.dropna(), regression='c', nlags='auto')**

**print("=== KPSS Test ===")**

**print(f"KPSS Statistic: {kpss\_result\[0]:.4f}")**

**print(f"p-value:        {kpss\_result\[1]:.4f}")**

**print(f"Verdict:        {'STATIONARY' if kpss\_result\[1] > 0.05 else 'NON-STATIONARY'}")**







1. Firstly, we import both of the tests from statsmodel...
2. we apply adfuller first on our time series and dropna() is just dripping the NANs, bcz both tests can't handle the NANs.
3. Then we print adf\_result\[0]-->ADF stats and adf\_result\[1]-->p-value
4. If p-value < 0.05 meaning that we reject Null hypothesis H0 and accept alternative H1, simple words: TS is stationary. ADF checks if there exist unit root(trend/instability) in our series?
5. Similary we apply kpss test and use dropna() to drop NANs.
6. we print KPSS stats \[0] and p-value \[1]
7. If p-value > 0.05 means that we accept NULL hypothesis H0 (series is stationary), and reject Alt H1.
8. KPSS is opposite to ADF.
9. In kpss() regression='c' tests the null hypothesis that the series is stationary around a constant (level stationary).
10. nlags='auto' automatically selects the number of lags using the Hobijn et al. (1998) method.





##### ***---Cell#09***



**train = ts\[ts.index < '2017-01-01']**

**test = ts\[ts.index >= '2017-01-01']**



**print(f"Train: {len(train)} days, ranges from {train.index.min()} to {train.index.max()}")**

**print(f"Test: {len(test)} days, ranges from {test.index.min()} to {test.index.max()}")**





1. We split our time series into train and test.
2. We print the no. of the days in each dataset and date range.





##### ***---Cell#10***



**naive\_value = train.iloc\[-1]**

**print(f"Last training day value: {naive\_value}")**



**naive\_forecast = pd.Series(**

&#x09;**\[naive\_value]\*len(test),**

&#x09;**index = test.index**

**)**





**plt.figure(figsize=(14,5))**

**plt.plot(train.index\[-100:], train.value\[-100:], label='Train (last 100 days)', linewidth=0.8)**

**plt.plot(test.index, test.value, label='Actual (Test)', linewidth=0.8)**

**plt.plot(naive\_forecast.index, naive\_forecast.value, label='Naive Forecast(Dumb)', linewidth=0.8)**

**plt.xlabel("Date")**

**plt.ylabel("Total Sales")**

**plt.tight\_layout()**

**plt.savefig('../naive\_forecast.png', dpi=100, bbox\_inches='tight')**

**plt.show()**



1. We build a naive model, where every value is the last value of the train dataset. iloc is to access rows by position, -1 means the last row of the dataset.
2. We create pandas series, and multiply naive value with the length of the test so that we get the same prediction value for each future day and we attach date of test dataset as index.
3. In first train plot we take just last 100 values so that the graph doesn't seem crowded.
4. We plot test value and then the naive forecast which is meant to be a horizontal line.







##### ***---Cell#11***





**seasonal\_naive\_forecast = pd.Series(index=test.index, dtype=float)**



**for i, date in enumerate(test.index):**

&#x09;**lookup\_date = date - pd.Timedelta(days=7)**

&#x09;**if lookup\_date in train.index:**

&#x09;	**seasonal\_naive\_forecast.iloc\[i] = train.loc\[lookup\_date]**

&#x09;**elif lookup\_date in test.index:**

&#x09;	**seasonal\_naive\_forecast.iloc\[i] = test.loc\[lookup\_date]**



**plt.figure(figsize=(14,5))**

**plt.plot(train.index\[-100:], train.values\[-100:], label='Train (last 100 days)', linewidth=0.8)**



**plt.plot(test.index, test.values, label='Actual (Test)', linewidth=0.8)**

**plt.plot(seasonal\_naive\_forecast.index, seasonal\_naive\_forecast.values, label='Seasonal Naive Forecast (Better Naive)', linewidth=0.8)**

**plt.title('Seasonal Naive Forecast: today = 7 days ago')**

**plt.xlabel('Date')**

**plt.ylabel('Total Sales')**

**plt.legend()**

**plt.tight\_layout()**

**plt.savefig('../seasonal\_naive\_forecast.png', dpi=100, bbox\_inches='tight')**

**plt.show()**





1. We build a seasonal naive series initially NaN but just the test dates as index with type plot.
2. we run a loop through test.index.
3. we make a variable named lookup date so that date substracts from 7 days before where Timedelta(days=7) means a time difference of 7 days. So 2017-03-15 - 7 days = 2017-03-08.
4. For the first week the previous week is in train data. so we run a check.
5. Then we find sales on that earlier day and copy it as a prediction and .loc accesses by label/date.
6. In The elif statement, it happens after the first week of the test so we have the value of that day from last week.
7. So we keep on copying the values from the last week e.g: This monday = last monday, and so on.
8. We plot everything same way.





##### ***---Cell#11***



**from sklearn.metrics import mean\_squared\_error, mean\_absolute\_error**



**def evaluate\_forecast(actual, predicted, model\_name):**

&#x09;**rmse = np.sqrt(mean\_squared\_error(actual, predicted))**

&#x09;**mae = mean\_absolute\_error(actual,predicted)**

&#x09;**mape = np.mean(np.abs((actual-predicted)/actual))\*100**



&#x09;**print(f"=== {model\_name} ===")**

&#x09;**print(f"RMSE: {rmse:.2f}")**

&#x20;   	**print(f"MAE:  {mae:.2f}")**

&#x20;   	**print(f"MAPE: {mape:.2f}%")**

&#x20;   	**print()**

&#x20;

&#x20;   	**return {'model': model\_name, 'RMSE': rmse, 'MAE': mae, 'MAPE': 	mape}**



**results = \[]**

**results.append(evaluate\_forecast(test, naive\_forecast, "Naive Forecast"))**

**results.append(evaluate\_forecast(test, Seasonal\_naive\_forecast, "Seasonal Naive Forecast"))**











1. We make a function to evaluate the results from the models which takes actual = test, predicted by our model, and its name.
2. RMSE is Root Mean squared Error, RMSA =sqrt(1/n​∑i=1n​(yi​−y^​i​)2), it substracts actual with predicted and squares it and divides by total no of rows. And take a square root. It punishes big mistakes more.
3. MAE means Mean Absolute Error MAE= 1/n ​∑i=1n​∣yi​−y^​i​∣, it treats every error equally, actual-predicted, takes its absolute value then takes average.
4. MAPE is Mean Absolute Percentage Error MAPE= 100​/n ∑i=1n​ |​(yi​−y^​i)/yi​​|. instead of error =10, it tells 10 is what percentage of actual value?





##### ***---Cell#12***







**ts\_diff = ts.diff().dropna()**



**print("Original series — first 5 values:")**

**print(ts.head())**

**print("\\nDifferenced series — first 5 values:")**

**print(ts\_diff.head())**



**plt.figure(figsize=(14, 4))**

**plt.plot(ts\_diff.index, ts\_diff.values, linewidth=0.6)**

**plt.axhline(y=0, color='red', linestyle='--', linewidth=0.5)**

**plt.title('First-Differenced Total Daily Sales')**

**plt.xlabel('Date')**

**plt.ylabel('Day-over-day change in sales')**

**plt.tight\_layout()**

**plt.savefig('../differenced\_series.png', dpi=100, bbox\_inches='tight')**

**plt.show()**





1. To make our series stationary to utilize ARIMA and SARIMA, we differentiate our series. which is basically subtracting today's value from yesterday's value: y`t=yt-yt-1. Subtract each value from previous value. Instead of modeling total sales, we model change in sales, this often removes trend, non-stationarity, changing mean.
2. we do ts.diff() which is exactly yt-yt-1 and we use dropna() to drop the NaN value, as first value turns NaN as no previous observation exists.
3. plt.axhline(y=0, color='red', linestyle='--', linewidth=0.5) simply draws a horizontal line at 0. so that the values above that line means positive growth and below means negative growth.





##### ***---Cell#13***





We did the ADF and KPSS test again and serie got stationary.





##### ***---Cell#14***



**from statsmodels.graphics.tsaplots import plot\_acf, plot\_pacf**



**fig, axes = plt.subplots(2,1, figsize=(14, 8))**





**plot\_acf(ts\_diff, lags=40, ax=axes\[0])**

**axes\[0].set\_title("ACF-- Differenced Series")**



**plot\_pacf(ts\_diff, lags=40, method='ywm',ax=axes\[1])**

**axes\[1].set\_title("PACF-- Differenced Series")**



**plt.tight\_layout()**

**plt.savefig('../acf\_pacf\_differenced.png', dpi=100, bbox\_inches='tight')**

**plt.show()**







1. We plot the ACF and PACF graphs of our differenced time series and notice the peaks at 7,14,21,... in ACF while PACF has decaying peaks.
2. Tells us that we need a seasonal model because of weekly seasonality.





##### ***---Cell#15***



**from statsmodels.tsa.arima.model import ARIMA**





**arima\_model = ARIMA(train, order(1,1,1))**

**arima\_result = arima\_model.fit()**

**print(arima\_result.summary())**



**arima\_forecast = arima\_result.get\_forecast(steps=(len(test))**

**arima\_forecast.index = test.index**



1. We apply arima model first to see the results with the order(p=1,I=1,q=1).
2. We fit arima model.
3. We tune the p and q value according to AIC and BIC.
4. We get forecast of over model, and steps we choose are equal to the length or no. of rows in the test.
5. And we attach index of test to our model.





##### ***---Cell#16***



**plt.figure(figsize=(14,8))**

**plt.plot(train.index\[-100:], train.values\[-100:], label='Train (last 100 days)', linewidth=1)**

**plt.plot(test.index, train.values, label='Test (Actual)', linewidth=1)**

**plt.plot(arima\_forecast.index, arima\_forecast.value, color='orange',label='ARIMA(1,1,1) Forecast', linewidth=1)**



**plt.title('ARIMA(1,1,1) Single shot Forecast')**

**plt.xlabel('Date')**

**plt.ylabel('Total Sales')**

**plt.legend()**

**plt.tight\_layout()**

**plt.savefig('../arima\_forecast.png', dpi=100, bbox\_inches='tight')**

**plt.show()**









1. We plot the Forecast against actual test same way as we did with seasonal naive.



##### ***---Cell#17***





**from statsmodels.tsa.statespace.sarimax import SARIMAX**



**sarima\_model = SARIMAX(**

&#x09;**train,**

&#x09;**order(1,1,1),**

&#x09;**seasonal\_order(1,1,1,7)**

**)**



**sarima\_results = sarima\_model.fit()**

**print(sarima\_results.summary())**

**sarima\_forecast = sarima\_results.get\_forecast(steps=len(test))**

**sarima\_forecast.index = test.index**





1. We apply Sarima model on our tain set. Difference between order=(1,1,1) is that they p=1,d=1,q=1, means today's sale depends on yesterday, yt = yt-yt-1, and today's error depends on yesterday respectively. While seasonal order **(1,1,1,7)** P=1 means todays sale depends on 7 days back, D=1 means yt= yt-yt-7, and Q=1 means todays error depends on 7 days back error, and S=7 is weekly cycle, tells SARIMA that season repeats every 7 observations.
2. disp=False means dont display, optimization/training messages while fitting.













##### ***---Cell#18***





#### 

#### **Walk-forward SARIMA forecasting**





**import time**



**history=list(train.values)**

**walk\_forward\_predictions=\[]**

**step\_size=7**



**start\_time = time.time()**

**n\_steps = len(test)//step\_size**



**for i in range(n\_steps):**

&#x09;**model=SARIMAX(**

&#x09;	**history,**

&#x09;	**order=(1,1,1),**

&#x09;	**seasonal\_order=(1,1,1,7)**

&#x09;**)**

&#x09;**model\_fit = model.fit(disp=False)**

&#x09;

&#x09;**forecast = model\_fit.forecast(steps=step\_size)**

&#x09;**walk\_forward\_predictions.extend(forecast)**



&#x09;**actual\_next = test.values\[i\*step\_size:(i+1)\*step\_size]**

&#x09;**history.extend\[actual\_next]**



&#x09;**if(i+1)%10 == 0:**

&#x09;	**eplapsed = time.time() - start\_time**

&#x09;	**print(f"Completed {i+1}/{n\_steps} steps, elapsed time: 			{elapsed:.2f} seconds.")**



&#x09;**elapsed\_total = time.time() - start\_time**

&#x09;**print(f"\\nDone. Total time:{elapsed\_total:.2f} seconds")**

&#x09;

&#x09;**wf\_forecast = pd.Series(**

&#x09;	**walk\_forward\_predictions,**

&#x09;	**index=test.index\[:len(walk\_forward\_predictions)]**

&#x09;**)**



&#x09;**test\_aligned = test.iloc\[:len(wf\_forecast)]**

&#x09;**wf\_metrics = evaluate\_forecast(test\_aligned, wf\_forecast, "Walk  	Forward SARIMA")**

&#x09;**results.append(wf\_metrics)**





1. Used time to measure execution time.
2. Converts training data into a python list. Called history because it stores all observations known so far, initially history is the training data, later actual test observations get added too.
3. We make an empty list to store predictions or to collect forecasts. As forecasting proceeds new predictions get appended here.
4. step\_size=7 means forecasts 7 days at a time. bcz of weekly seasonality in the data.
5. Stores current timestamp, used late for calculating run time.
6. No. of forecasting iterations, our length of test is 365 (1 year) and step\_size is 7 then n\_steps = 52, meaning 52 weekly forecasting rounds.
7. Looping over each forecasting window. meaning each iteration is one week. iteration 1 -> week 1, 2 --> week 2, and so on until 52nd week.
8. At each iteration, we use all known data and rebuild SARIMA model.And the model keeps learning from newly observed data.
9. SARIMA estimates coefficients using current history. And disp= False hides optimization logs.
10. Forecast according to step\_size, in our case it's 7. SO predict next 7 days.
11. Adds forecasted values into prediction list. append(): adds entire object as one item, while extend(): adds each value individually.
12. actual\_next extracts the real observed values for that week. Suppose: step\_size = 7 and i=2 then test.values\[14:21], gets week 3 actual observations. First the model predicts the future week 3 then it collects the test values for that week and modify itself to predict next week and repeats.
13. We keep updating the history list by adding newly observed real data into training history. The key idea of the walk-forward, forecast-> observe reality -> retrain.
14. Progress printing: Every 10 iterations: print progress. i+1 bcz loops start at i=0.
15. Calculate the elapsed time. computes the runtime.
16. We simply calculate the total time taken by the model and print it.
17. Converts the prediction list into a proper time series.
18. Ensures actual length == prediction length, its important bcz of integer division //, it had left some leftover days.
19. Then we put the values in the function to evaluate the metrics, and append to results list.















##### ***---Cell#19***



**df\_ml = pd.DataFrame({'sales': ts})**



**df\_ml\['lag\_1'] = df\_ml\['sales'].shift(1)**

**df\_ml\['lag\_7'] = df\_ml\['sales'].shift(7)**

**df\_ml\['lag\_14'] = df\_ml\['sales'].shift(14)**

**df\_ml\['lag\_30'] = df\_ml\['sales'].shift(30)**



**df\_ml\['rolling\_mean\_7'] = df\_ml\['sales'].shift(1).rolling(window=7).mean()**

**df\_ml\['rolling\_mean\_30'] = df\_ml\['sales'].shift(1).rolling(window=30).mean()**



**df\_ml\['day\_of\_week'] = df\_ml.index.dayofweek**

**df\_ml\['day\_of\_month'] = df.ml.index.day**

**df\_ml\['month'] = df\_ml.index.month**



**df\_ml = df\_ml.dropna()**





**print(df\_ml.shape)**

**print(df\_ml.head())**



1. We convert our time series to a pandas table/df for our ml models.
2. We build various lags, like lag\_1, 7, 14, 30. meaning lag\_1 is that yesterday's sale, sales from 7 days ago, 14 days ago, and 30 days ago respectively.
3. rolling mean 7 with shift(1) means we take yesterdays lag or lag\_1 and averages all of 7 last days with lag\_1 to use only past infos. and similarly for 30.
4. we create more features including dow,dom, and month.
5. We drop the NAN values.
6. we print the shape and first 5 rows to see the df.





##### ***---Cell#20***





**X = df\_ml.drop('sales', axis=1)**

**y = df\_ml\['sales']**



**train\_cutoff = '2017-01-01'**

**X\_train = X\[X.index < train\_cutoff]**

**X\_test = X\[X.index >= train\_cutoff]**

**y\_train = y\[y.index < train\_cutoff]**

**y\_test = y\[y.index >= train\_cutoff]**



**print(f"X\_train: {X\_train.shape}")**

**print(f"X\_test:  {X\_test.shape}")**

**print(f"y\_train: {y\_train.shape}")**

**print(f"y\_test:  {y\_test.shape}")**





**## Time Series Split Rule**

NEVER use train\_test\_split(shuffle=True) on time series data.

ALWAYS split chronologically by date:

&#x20;   X\_train = X\[X.index < cutoff\_date]

&#x20;   X\_test  = X\[X.index >= cutoff\_date]

Why: random shuffle leaks future data into training, breaks evaluation.

Same rule applies to cross-validation: use TimeSeriesSplit instead of KFold.





##### ***---Cell#21***



**rf\_model = RandomForestRegressor(**

&#x09;**n\_estimators=200,**

&#x09;**max\_depth=10,**

&#x09;**random\_state=42,**

&#x09;**n\_jobs=-1**

**)**



**rf\_model.fit(X\_train,y\_train)**



**rf\_predictions = rf\_model.predict(X\_test)**

**rf\_forecast = pd.Series(**

&#x09;**rf\_predictions,**

&#x09;**index=test.index**

**)**





**print(f"Random Forest trained on {len(X\_train)} days, predicting {len(X\_test)} days")**



**rf\_metrics = evaluate\_forecast(y\_test, rf\_forecast, "Random Forest (Lag features)")**

**results.append(rf\_metrics)**





1. We import and create RF model to work on. RF is many decision trees combined together, each tree learns different pattern and final prediction is avg of all trees.
2. n\_estimators=200 means build 200 trees, more trees mean better accuracy, more stable prediction. but slow training. Usually 100-500 works well.
3. **max\_depth** = 10 limits how deep each tree can grow, without limit trees memorize training data which causes overfitting. small depth means more generalized patterns, large depth means complex trees, can memorize noise.
4. random\_state=42 makes results reproducible.
5. n\_jobs=-1 means use all cores.
6. We fit model by feeding it the training features set and the target set so that the model can learn the patterns.
7. Then the model generates the forecast when we provide the X\_test values and **rf\_predictions** is just a NumPy array .
8. Then we convert it to panda Series and attach dates to make predictions become proper time series.
9. At last we put the values in the function of evaluate\_forecast and append the metrics to the array.





##### ***---Cell#22***





**xg\_model = XGBRegressor(**

&#x09;**n\_estimators=200,**

&#x09;**random\_state=42,**

&#x09;**n\_jobs=-1,**

&#x09;**max\_depth=6,**

&#x09;**learning\_rate=0.5**

**)**



**xg\_model.fit(X\_train,y\_train)**



**xg\_predictions = xg\_model.predict(X\_test)**

**xg\_forecast = pd.Series(**

&#x09;**xg\_predictions,**

&#x09;**index=X\_test.index**

**)**



**xgb\_metrics = evaluate\_forecast(y\_test, xgb\_forecast, "XGBoost (lag features)")**

**results.append(xgb\_metrics)**



1. XG-Boost means Extreme Gradient Boosting, boosting is very different from RF, RF builds many trees independently and then averages them while XGBoost builds them sequentially, each new tree tries to fix errors made by previous trees.
2. learning\_rate=0.05 controls how aggressively trees correct errors. 0.3-> large learning rate means large corrections, faster learning but can overshoot, while small -->0.05 means small careful corrections usually more stable and accurate



##### ***---Cell#23***





**results\_df = pd.DataFrame(results)**

**results\_df = results\_df.sort\_values("MAPE").reset\_index(drop=True)**

**results\_df\['Rank'] = results\_df.index + 1**



**rint("="\*60)**

**print("FINAL MODEL COMPARISON")**

**print("="\*60)**

**print(results\_df.to\_string(index=False))**



**results\_df.to\_csv('../results/model\_comparison.csv', index=False)**

**print("\\nSaved to ../results/model\_comparison.csv")**



1. Converts a list to panda dataframe.
2. sort values by MAPE ascending.
3. adds a rank column to df.
4. results\_df.to\_string(index=False) means we make table print cleanly in terminal/notebook. and index=False to not to get numbering.









##### ***---Cell#24***





**feature\_importance = pd.DataFrame({**

&#x20;   **'feature': X\_train.columns,**

&#x20;   **'importance': xgb\_model.feature\_importances\_**

**}).sort\_values('importance', ascending=False)**





**print("=== XGBoost Feature Importance ===")**

**print(feature\_importance.to\_string(index=False))**



**plt.figure(figsize=(10, 5))**

**plt.barh(feature\_importance\['feature'], feature\_importance\['importance'])**

**plt.gca().invert\_yaxis()**

**plt.xlabel('Feature Importance')**

**plt.title('XGBoost — Which Features Drove The Forecast?')**

**plt.tight\_layout()**

**plt.savefig('../results/feature\_importance.png', dpi=100, bbox\_inches='tight')**

**plt.show()**



1. THe code analyzes which features were most important for XGBoost predictions.
2. We make a data frame, with feature names, and their importance.
3. xgb\_model.feature\_importances gives importance score for each feature.
4. and we print it and draw a graph.

