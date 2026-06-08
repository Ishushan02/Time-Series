Auto Regression
It is just Linear Regression, and nothing else.

Y = c + m1 * X1 + m2 * X2 + .... mP * XP + epislon_t

To build autoregressive Model, it is recommended to 
have a stationary time series. Stationarity means 
the time series doesn't exhibit any long term trend 
or obvious seasonality. The reason we need stationarity 
it to ensure statistical properties of the time series 
is consistent through time, rendering it easier to model.

Stationaroty can be achieved by stabilising the trend 
through differencing, or applying logarithm, or Box-Cox.

The need for stationarity becomes clearer when we are 
training the mode. stationary data has constant statistical 
properties such as mean and variance. Therefore, all the 
data points belong to the same statistical probability 
distribution that we can base our model on. Furthermore, 
the forecasts are treated as random variables and will 
belong to the same distribution as the training data. 
It basically guarantees the data in the future will be somewhat 
like the past.

As the stationary data belong to some distribution typically the 
normal distribution, we often estimate the coefficients and 
parameters of the autoregressive model using Maximum Likelihood 
Estimation. MLE deduces the optimal values of the parameters and 
coefficients that produce the highest probability of obtaining 
our time series data. The MLE for normally distributed data, 
is the same result as carrying ordinary least squares. 
We should also see how many parameters to use, as it should not 
overfit the model.