Holts Linear Trend Model

If you see the previous Exponential Smoothening process, 
it's just simple and there is no accountability of Seasonality 
and trend in it. This would leads to the particular model to 
oftern deliver inadequate forecasts for most time series. 

Whereas in Holt's linear trend method (it is also know as 
double exponential smoothing), which adds a linear trend 
component to the simple exponential smootjing model.

overall equation: y_(t + h) = l_t + h * b_t
level equation: l_t = alpha * y_t + (1-alpha) * (l_(t-1) + b_(t-1))
trend equation: b_t = beta * (l_t - l_(t-1)) + (1 - beta) * b_(t-1)

beta = [0, 1]
b_t = is the dorecasted trend component, and l_t the level equation

The trend equation is computed from the step per step change in the level component. Additionally, from the overall equation, the trend component is now being multiplied by the time step, h, therefore the forecasts are no longer flat but are a linear function of h. Hence, the models name is Holt's linear trend method.

b_0 = (y_t - y_0)/t

One issue in the above formulation is that the forecasts will increase or decrease arbitrarily into the future. In reality, nothing grows nor decays indefinetily. Therefor there is oftern a dampening term phi, added to curtail the forecasts at long horizons:

overall equation: y_(t + h) = l_t + (phi + phi^2 + phi^3.... phi^h) * b_t
level equation: l_t = alpha * y_t + (1-alpha) * (l_(t-1) + phi * b_(t-1))
trend equation: b_t = beta * (l_t - l_(t-1)) + (1 - beta) * phi * b_(t-1)

phi = (0, 1)

Keytakeaway
- Holt's Linear Trend model adds trend as well as level exponential smothing. (but not seasonality)

