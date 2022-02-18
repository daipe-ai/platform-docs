# Features Tracking

Feature store can track what features were computed and when.

Let's suppose that we have the following Feature Store.

![](../images/features_tracking_1.png)

And we want to add following dataframe which contains new feature `f3`.

![](../images/features_tracking_2.png)

The Feature Store can keep track that feature `f3` was not computed for
previous dates.

![](../images/features_tracking_3.png)

You can think of that `RED NULLS` as uncomputed values.
