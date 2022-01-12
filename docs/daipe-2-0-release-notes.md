# Daipe 2.0: Release Notes

## Enhancements

- It is no longer necessary to define tables in a local environment, YAML config is __optional__. Local environment is only necessary for the initial setup of the project
- It is now possible to use Daipe __without__ Databricks on whatever Spark environment or even without Spark just using Pandas
- Functions such as `dp.read_csv()` and `dp.read_table()` can be used as arguments for decorators. This completely replaces the functionality of `@dp.data_frame_loader`, see [docs](data-pipelines-workflow/technical-docs/#functions). Example:
```python
# Imports
import daipe as dp
```

```python
# Old Daipe
@dp.data_frame_loader(display=True)
def my_transformation(spark: SparkSession):
    return spark.read.table("my_database.my_table")
```
```python
# New Daipe
@dp.transformation(dp.read_table("my_database.my_table"), display=True)
def my_transformation(df: DataFrame):
    return df
```
- Support for DBR 8.x
- Decorator `@dp.table_overwrite` which overwrites all data in a table with the data from a DataFrame, see [docs](data-pipelines-workflow/technical-docs/#table_overwrite)
- Decorator `@dp.table_append` which appends the data from a DataFrame to a table, see [docs](data-pipelines-workflow/technical-docs/#table_append)
- Decorator `@dp.table_upsert` which updates existing data based on `primary_key` and inserts new data, see [docs](data-pipelines-workflow/technical-docs/#table_upsert)

- Schema now allows you to define a primary_key (used for `@dp.table_upsert` ), `partition_by` and `tbl_properties` , see [docs](data-pipelines-workflow/technical-docs/#table_schema)
- Schema will be generated for you if you do not provide it to the `@table_*` decorators see example:

![](images/schema_generation_example.png){: style="width: 850px; padding-left: 3%"}

- Schema checking output is greatly improved. Schema diff example:

![](images/schema_diff_example.png){: style="width: 500px; padding-left: 3%"}

## Backwards incompatible changes

- Schema __is no longer loaded__ automatically from the `schema.py` file in the notebook folder. Now the schema can be defined inside the notebook as well as imported from a separate file, see [docs](data-pipelines-workflow/technical-docs/#table_schema) and example:

![](images/schema_definition_example.png){: style="width: 850px; padding-left: 3%"}

- Command `console datalake:table:create-missing`  has been __removed__, because it is no longer possible to rely on the tables being defined in YAML config
- Command `console datalake:table:delete` renamed to `console datalake:table:delete-including-data`

## Deprecations

- Decorator `@dp.data_frame_loader` has been __deprecated__
- Decorator `@dp.data_frame_saver` has been __deprecated__
