ARIMA Model
Autoregressive Integrated Moving Average better known as ARIMA, is probably the most used time series forecasting model and is combination of the individual aforementioned models.

Autoregressive - This is just autoregression, where we forecast future values using a linear combination of the previously observed values.

Integrated - This is the number (order d) of differencing required to make the time series stationary. Stationarity is where the time series has a constant mean and variance, meaning the statistical properties of the series does not change through time. 

It is important to note, that this integrated part only makes the mean constant. We need to apply another transform such as the logarithmic and Box-Cox transform to generate a constant variance.


Moving Average - THe last component, where you forecast using past forecast errors instead of the actual observed values.

y = c + phi_1 * y_(t-1) + phi2 * y_(t-2) + .... + phi_p * y_(t-p) + epsilon_t + theta_1 * epislon_1 + theta_2 * epsilon_2 +  ... + theta_q * epislon_(t-q)

They have good capture of  little seasonality, and trend. However, one disadvantage of this model is that it is lacking awareness of any seasonality. This is where the seasonal Autorefressive Integrated Moving Average, or SARIMA model comes it.


SARIMA Model
It is an extension of ARIMA model that adds a swasonaility component to the model. This allows us to better captire seasonal affect that the regular ARIMA model does not permit.

y = c + Auto regrresive + Moving Average + (previous seasonal values sum _ similariy with other variable ) is the seasonality.
![alt text](Sarima_Image.png)


Harmonic Regression
When we want model seasonality in our time series we often turn to the SARIMA model. THis adds seasonality components to the ARIMA model by findind the autoregressors and moving averages at certain specific lag indeces. For example, monthly data with yearly seasonlity will fit autoregressors and moving averages at multiple of 12.

However, what if we have daily data with a yearly seasonality of 365.25 dys?, or even weekly data with a seasonality of 52.14?

Unfortunately SARIMA can;t handle this as it is non-integer and also struggles computationally due to the memory required to find patterns in 365 data points each season. 

- Fourier series, any periodic function can be decomposed into sum of sine and cosine waves. This is a very simple statement but its implications are very significant. 

- It's mathematical representation
![alt text](fourier-Eqn.png)

Hence, as we are working on time series there would be periodicity in it, so we can actually work with this equation for Time series data.

