# Using the @transformation() decorator

There are two main decorators in the Daipe framework - `@notebook_function()` and `@transformation()`.

1. `@notebook_function()` should be used for functions and procedures which don't manipulate with a DataFrame e. g. downloading data..
1. `@transformation()` understands Spark dataframes better and provides you with extra Spark-related functionality like display and duplicate columns checking.

Start with importing the Daipe decorators and functions as usual:

```python
from datalakebundle.imports import *
```

Any decorator can also take functions as parameters:

```python
@transformation(
    read_csv(
        "/data.csv",
        options=dict(header=True, inferSchema=True)
    ),
)
def read_csv(df: DataFrame):
    return df
```

See the [list of all functions](input-decorators.md#transformation) which can be used.

`display=True` option can be used for displaying the DataFrame.

```python
@transformation(
    read_table(
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
