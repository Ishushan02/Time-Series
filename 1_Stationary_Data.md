Stationarity
A Time Series is stationary if it does not exhibit or include any
long term trends or obvious seasonality. It has a constant variance
through time, a constant mean through time and the statistical 
properties of the time series does not change.

Transforming 

- Differencing Transform: d(t) = y(t) - y(t-1)
- Logarithmic Tranform: log(data point)


There is quantitative techniques to determine whether the data is 
stationary or not stationary or not stationary. One of such method 
is known as Augmented Dickey Fuller (ADF) test. This is a statistical
hypithesis test where the null hypothesis is the series is non-stationary
(also known as unit root test).
[Scipy - adfuller(series)]

All in all Stationary Data has 
- No Long term Trend
- No obvious Seasonality
- Constant Mean
- Constant Variance

