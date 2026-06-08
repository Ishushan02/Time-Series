Basic Forecasting Methods for Time Seires Analysis

Average Model
 - average value, of the prev values and this is used to forecast future values.
 y_T + h = y_mean

Naive Model
 - Just the previous value is used to forecast the future values.
 y_T + h = y_T

Seasonal Naive Model
 - The sorecase is equal to the mosr recent ovserve value in the same season. Hence it is know as the seasonal naive model.
 y_T + h = y_T+h - m, where m is the seasonality of the data.
 
Drift Model
 - This is also and extension of the naive forecast where we let the prediction either linealy increase or descrease through time as a function of time step, h scaled by the average historical trend:
 y_T + h = y_T + h * (y_T - y1)/(T-1)

