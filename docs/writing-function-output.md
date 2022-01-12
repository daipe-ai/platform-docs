# Writing function output

Output decorators are used in conjunction with the `@transformation` input decorator. They take either a string identifier of the table...

```python
import daipe as dp

@dp.transformation(dp.read_csv, display=True)
@dp.table_overwrite("bronze.tbl_customers")
def save(df: DataFrame, logger: Logger):
    logger.info(f"Saving {df.count()} records")
    return df.withColumn("Birthdate", f.to_date(f.col("Birthdate"), "yyyy-MM-dd"))
```

...or you also skip Hive and write directly into delta: 

```python
import daipe as dp

@dp.transformation(dp.read_table("bronze.tbl_customers"), dp.read_table("bronze.tbl_contracts"))
@dp.delta_overwrite("/path/to/dataset.delta")
def join_tables(df1: DataFrame, df2: DataFrame):
    return df1.join(df2, "Customer_ID")
```

For more details see the [Output decorators reference](output-decorators.md).

## Automatic schema

When using the **string** table identifier, the `@dp.table_overwrite` decorator saves the data using the DataFrame schema. This is useful for prototyping. It is **highly** recommended to use explicit schema for production pipelines.
