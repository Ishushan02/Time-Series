Autocorrelation
In the time series analysis we oftern make inferences about
the past to produce forecasts about the future. Autocorrelation 
helps us detect certan features in out series to enable us 
to choose the most optimal forecasting model for our data.

Autocorrelation is just the correlation of the data with 
itself. So, instad of measuring the correlation between 
two random variables, we are measuring the correlation 
betweem random variable against itself. Hence, why 
it is called auto correlation.

If 
+1 the varianles are perfectly prositvelu correlated, 
-1 then they are perfectly egativelu correlated
and 0 then there is no correlation.

We use autocorrelation to measure the correlation of a 
time series with a lagged version of itself. This 
computation allows us to gain some interesting insight 
into the characterstivs of our series.

Seasonality and Trend we can find it using Autocorrelation.
Autocorrelation is denoted by r(X, Y) correlation coefficient.

statsmodel to use Autocorrelation in Python.


Partial Autocorrelation(PACF)
Partial autocorrelation is simply just the partial correlation 
of a time series at two different states in time. Taking it one 
step further, it is the correlation between the time series at 
two different lags not considering the effect of any intermeiate 
lags. For example, the partial autocorrelation for a lag of 2 is 
only the correlation tha laf 1 didn;t explain.


Unlike autocorrelation, partial autocorrelation hasn't got as many 
uses for a time series analysis. However, its main and very important 
impact comes in when building forecating models. The PACF is used to 
estimate the number/order of autoregressive components when fitting 
Autoregressive, AEMS or ARIMA models as defined by the BOX Jenkins 
procedure. These models are probably the most used and often provide 
the best results when training a forecasting model.
