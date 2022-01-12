# Using the @dp.transformation() decorator

There are two main decorators in the Daipe framework - `@dp.transformation()` and `@dp.notebook_function()`.

1. `@dp.transformation()` understands Spark dataframes better and provides you with extra Spark-related functionality like display and duplicate columns checking.
1. `@dp.notebook_function()` should be used for functions and procedures which don't manipulate with a DataFrame e. g. downloading data.

First, import everything necessary for a Daipe pipeline workflow:

```python
import daipe as dp
```

Any decorator can also take functions as parameters:

```python
@dp.transformation(
    dp.read_csv(
        "/data.csv",
        options=dict(header=True, inferSchema=True)
    ),
)
def read_csv(df: DataFrame):
    return df
```

See the [list of all functions](decorator-functions.md) which can be used.

`display=True` option can be used for displaying the DataFrame.

```python
@dp.transformation(
    dp.read_table(
        "bronze.tbl_customers",
        options=dict(header=True, inferSchema=True)
    ),
    display=True
)
def read_tbl_customers(df: DataFrame):
    return df
```

For more information see the [technical reference](input-decorators.md#transformation).

## Environments

Each table is prefixed with an environment tag (__dev__, __test__, __prod__) to separate production data from the developement code and vice versa. The Daipe framework automatically inserts the prefix based on your selected environment therefore __the code stays the same__ across all environments.
