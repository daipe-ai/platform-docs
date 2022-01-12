# Bad practices when writing features

PySpark offers many functions and methods for developing complex features out of any data.

It does not mean that all of them should be used.

## Short list of forbidden functions

- [pyspark.sql.DataFrame.collect()](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrame.collect.html)
- [pyspark.sql.DataFrame.toPandas()](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrame.toPandas.html)
- [pyspark.sql.DataFrame.dropDuplicates()](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrame.dropDuplicates.html)
- [pyspark.sql.DataFrame.union()](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrame.union.html)
- [pyspark.sql.DataFrame.count()](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrame.count.html)
- [pyspark.sql.functions.rank()](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.functions.rank.html) in certain situations

## Explanations and alternatives

### DataFrame.collect()

#### Explanation

Spark is built upon the idea of [lazy evaluation](https://en.wikipedia.org/wiki/Lazy_evaluation#:~:text=In%20programming%20language%20theory%2C%20lazy,avoids%20repeated%20evaluations%20(sharing).) meaning it doesn't calculate until it's necessary.

`collect()` is an action which means it triggers the calculation and thus breaks the lazy evaluation sequence.
It also brings the whole `DataFrame` onto the driver which might fill its memory and crash it.

#### Alternative

If your code contains `collect()` it is most definitely not optimal, try to come up with a way, how to get the data which needs to be collected
into the `DataFrame` lazily such as by joining it in or by calculating it on the fly by using [arrays](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.functions.array.html)
or [structs](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.functions.struct.html).

### DataFrame.toPandas()

#### Explanation

`toPandas()` is very similar to `collect()`. It forces Spark to calculate and it brings the data to the driver.

#### Alternative

If your code uses `toPandas()` then try to rewriting the Pandas logic into PySpark.

### DataFrame.dropDuplicates() {#dropDuplicates}

#### Explanation

`dropDuplicates()` always keeps the first occurrence of "unique" row and drops all subsequent duplicates of it.
Therefore its outcome is dependent on the order of rows in the `DataFrame`. Order of rows depends on partitioning and other
frequent operations such as [join](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrame.join.html) and [cache](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrame.cache.html).

#### Alternative

To produce replicable and testable code it is necessary NOT to use `dropDuplicates()` making the code deterministic.
A good alternative can be using [structs](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.functions.struct.html)
and [groupBy](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrame.groupBy.html) allowing you to control how the remaining rows are selected.

### DataFrame.union()

#### Explanation

`union()` doesn't check if the columns are in the same order.
It will just glue two DataFrames of the column size together.

#### Alternative

Therefore always use [unionByName](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrame.unionByName.html) instead of `union`.

### DataFrame.count()

#### Explanation

`count()` is an action which triggers calculation.

#### Alternative

To preserve the laziness of Spark, if you need to use the number of rows, just use `f.count(f.lit(1))`.

### f.rank()

#### Explanation

`f.rank()` is a useful Window function. Nevertheless it can lead to some unexpected results.

`rank` assigns the same number to rows with equal values.

So if you use it in a combination with `filter` - `df.filter(f.rank().over(window) == 1)` it gives you multiple rows per `rank == 1`.

This leads to frequent usage of `dropDuplicates` to solve this issue.

#### Alternative

This algorithm can be better solved by using the solution of [dropDuplicates](#dropDuplicates).
