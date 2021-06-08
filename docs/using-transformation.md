# Using the @transformation() decorator

First, import everything necessary for a Daipe pipeline workflow:

```python
from datalakebundle.imports import *
```

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

See the [list of all functions](decorator-functions.md) which can be used.

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
