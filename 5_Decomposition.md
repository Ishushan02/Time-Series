Decomposition
Time Series basically consists of 3 things, 
Trend(T) - This is the overall motion of the serires, either increasing
or decreasing overtime or the combination of the both.
Seasonality(S) - Any regular seasonal pattern in the series, like the 
sale of mangoes peaks durin the summer.
Residual/Remainder(R) - This is that bit that is left over after we
take in to account of the Trend and the Seasonality, We can also think
it as some statistical noise.


Additive vs Multiplicative Model.
Additive Model : Y = T + S + R
Multiplicative Model : Y = T * S * R
    We can apply log for multiplicative Model.
     ln(Y) = ln(T) + ln(S) + ln(R)
Y is the series.

The additive Model is appropriate when the size of the series variations
are on a consistent numerical scale. On the other hand, the 
multiplicative model is when the series fluctutations are on a relative scale.

Foe examples, if the ice cream sales are higher in summer by 1000 every year,
then the model is additive. If the sales are higher by a consistent 20% every
summer, but the absolute number of sales are changing then the model is multiplicative.

THere are also several other methods available for deomposition such as STL, X11,
and SEARS. This are advances methods and ass to the basic approach from the classical 
method and imrpove upon its shortcomings.

in python we can use
from statsmodels.tsa.seasonal import seasonal_decompose