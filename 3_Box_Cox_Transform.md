Box Cox Transform
Making the time series stationary is an essential part when 
carrying out any time sries analysis or forecasting. Stationarity 
ensures that our data is not statistically changing through time,
therefore it can more accurately resemble a probability distribution
rendering it easier to model.

One requirement of that time series needs a constant variance. In 
other words, the fluctuations sould be consistently on the same
scale. One way to achieve this is to take natural logarithm of the
series, however this assumes that your original series follows 
an exponential trend.

Box Cox transforms as non-normal data to normal distribution like data.
Well, we would need our time series data to resemble a normal distribution 
because when fitting vertail models, such as ARIMA, they use the maximum
likelihood estimation (MLE) to determine their parameters. MLE by definition
must fit against a certain distribution, which for most packages
is the normal distribution.

y(lambda) = 
    (y_power(lambda) - 1 / lambda) is lambda is not equal to zero
    ln(lambda) when lambda is 0;


The value of lambda is chosen by seeing which is the value that best 
approximates the transformed data to the normal distribution. Luckily,
in computing packages this is easily done for us.

using scipy and matplot lib we can easily find out the best lambda value.

