Holt's Winter Model
Holt's Winter Method finds a way to include seasonality 
in the the exponential smoothing model. This can also be 
known as triple Exponential Smoothing.

Holt's Winter model further extends Holt's Linear trend 
model method by adding seasonality to the forecast. 
The addition of seasonality gives rise to 2 different 
Holt's Winter Model the first one being Additive, and the 
next one being Multiplicative.

(Remember the difference of Multiplicative and 
Additive. from Decomposition page); for an additive 
model the seasonality fluctuations are mostly constant. 
However, for multiplicative model the fluctuations are 
propotional to the value of time series at the given time.

Additive Model:

Overall Estimation: y_(t+h) = l_t + h * b_t + S_(t + h - m)
Level Equation: l_t = alpha * (y_t - S_(t-m)) + (1 - alpha) * (l_(t-1) + b_(t-1))
Trend Equation: b_t = beta * (l_t - l_(t-1)) + (1 - beta) * b_(t-1)
Seasonality Equation: S_t = gamma * (y_t - l_(t-1) - b_(t-1)) + (1 - gamma) * S_(t -m)

where m is seasonality of the time series and  S_(t -m) is the forecast for the previous season

gamma = [0, 1 - alpha)


Multiplicative Model: 

Overall Estimation: y_(t+h) = (l_t + h * b_t) * S_(t + h - m)
Level Equation: l_t = alpha * (y_t / S_(t-m)) + (1 - alpha) * (l_(t-1) + b_(t-1))
Trend Equation: b_t = beta * (l_t - l_(t-1)) + (1 - beta) * b_(t-1)
Seasonality Equation: S_t = gamma * (y_t / l_(t-1) - b_(t-1)) + (1 - gamma) * S_(t -m)

What both of these equations are trying to do is to calculate 
the trend line for the times series and weight the values on 
the trend line by the seasonal variations.

We can also add damping into it, but it will be too complex.. but we can doo it.. as the previous Linear Holt's Model.

we can use statsmodels to use it in python.