Residuals
In Time series analysis there is very minute difference 
between residuals and Errors.

The Erros is the difference betweenthe actual and 
forecasted values. Residuals are the difference 
between the actual and the fitted(or predicted) value.

We can use the residuals to analyse how well our model has captured the characterstics of the data. 
- They show very little to no autocorrelation or partial autocorrelation.
- The mean of the residuals should be zero, otherwise the forecast will be biased.


Cross Validation.
It is a method to determine the best performing model and 
parameters through training and testing the model on different 
potions of the data. The most common approach is tran-test split, 
but it can be taken one step further by carrying out the 
train-test split numerous times by varying the data we trained 
and tested on,. This process of cross validation as we are using 
every row of data for both training and evaluation to ensure we 
choose the most robust model over all the possible available data.

