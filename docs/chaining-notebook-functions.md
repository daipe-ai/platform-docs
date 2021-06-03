# Chaining notebook functions

Calls of the notebook functions can be chained by passing function names as decorator arguments:

```python
@transformation(read_table("bronze.tbl_customers"))
def tbl_customers(df: DataFrame):
    return df
```
```python
@transformation(tbl_customers)
def active_customers_only(df: DataFrame):
    return df.filter(f.col("active") == 1)
```
```python
@transformation(active_customers_only, display=True)
def save(df: DataFrame):
    return df
```

More compact way:

```python
@transformation(read_table("bronze.tbl_customers"), display=True)
def tbl_active_customers(df: DataFrame):
    return df.filter(f.col("active") == 1)
```

Once you run the `active_customers_only` function's cell, it gets is automatically called with the dataframe loaded by the `customers_table` function.

Similarly, once you run the `save_output` function's cell, it gets automatically called with the filtered dataframe returned from the `active_customers_only` function.
