Simple Exponential Smoothing
The idea is basically a way of stating that we will put 
more weight on the recent oversevations, whereas older 
observations will receive less weight at an exponentially 
decaying rate. Hence it is called expomential smoothing.

y_(t+1) = alpha * y_(t) + alpha * (1-alpha)* y_(t-1) + alpha * (1-alpha) * (1-alpha) * y_(t-2) + ...

alpha is the smoothing parameter takes values [0, 1].
the higher the value of alpha more is the weightage 
that is put on the recent obervations.. ex: if alpha is 0.99
recent gets o.99, prev to that get (1 - 0.99) ... the 
next prev gets (1 - 0.99) * (1 - 0.99) ... and so on 

Keypoints to note:
- Simple as doesn't take into account seasonality or trend
- Weights recent observations more than older ones
- Not very good because its so simple
- Good baseline model

