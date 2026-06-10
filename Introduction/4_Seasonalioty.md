Seasionality
It is cucial aspect of Time Series Analysis. Seasionalty, think
it in this way the the Sale of Mango in India rises in Summer 
as the total number of production increases in it. This is 
seasonality.

Seasonality can come in different time intervals such as days, weeks
months etc. Models such as SARIMA that model the seasonal affects for you.

Removing Seasionalty
We can remove seasonality in the data using seasonal Differencing. This 
calculates the difference between the current value and its value in the 
previous season. The reason this is done is to make the time series statiobary 
rendering its statistical properties constant throug time. Seasonality 
causes the mean of the time series to be different when we are in a particular 
season. Hence, its statistical properties are not constant.

d(t) = y(t) - y(t-m) where m is the length of 1 season.

We can obviously also apply Box Cox