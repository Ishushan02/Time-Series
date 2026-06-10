Forecasting Metrics

Mean Absolute Error (MAE)
    Pros: Intutive, Error in same units as forecasts.
    Cons: Doesn't penalize outliers and is also scale dependent.

Mean Squared Error (MSE)
    Pros: Penalise Outliers
    Cons: Less Interpretable, Scale dependent

Root Mean Squared Error (RMSE)
    Pros: Penalise Outliers, Error in forecast units, best of both MSE and MAE
    Cons: Less interpretable, Scale dependent.

Mean Absolute Percentage Error
    Pros: Easy to interpreate, Scale independent
    Cons: Infinite Error if the actual value is near zero, biased to under-forecast.

Symmetric Mean Absolute Percetange Error
    Pros: No Longer favours under forecasting
    Cons: Infinite erros if the actual value is near zero, Hard to interprest, and not actually symmetrical.

Mean Squared Logarithmic Error.
    Pros: Punsihed under-forecasting
    Cons: Divided by values that are close to zero, and hard to interpret.

