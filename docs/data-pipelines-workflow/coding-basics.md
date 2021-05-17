# Simple data pipeline workflow

## Introduction

Daipe greatly simplify datalake(house) management: 

* Tools to simplify & automate table creation, updates and migrations.
* Explicit table schema enforcing for Hive tables, CSVs, ...
* Decorators to write well-maintainable and self-documented function-based notebooks
* Rich configuration options to customize naming standards, paths, and basically anything to match your needs

## Recommended notebook structure

It is recommended to divide your tables and notebooks into the following layers:
 
* **bronze** - "staging layer", raw data from source systems
* **silver** - most business logic, one or multiple tables per use-case 
* **gold** - additional filtering/aggregations of silver data (using views or materialized tables) to be served to the final customers

![bronze, silver, gold](../images/bronze_silver_gold.png)

For databases and tables in each of bronze/silver/gold layers it is recommended to follow the **[db_name/table_name]** directory structure.  

```yaml
src
    [PROJECT_NAME]
        bronze_db_batch
            tbl_customers.py
            tbl_products.py
            tbl_contracts # it is possible to place notebooks in folders with the same name if necessary
                tbl_contracts.py
                csv_schema.py
            ...
        silver_db_batch
            tbl_product_profitability.py
            tbl_customer_profitability.py
            tbl_customer_onboarding.py
            ...
        gold_db_batch
            vw_product_profitability.py # view on silver_db_batch.tbl_product_profitability
            tbl_customer_profitability.py # "materialized" view on silver_db_batch.tbl_customer_profitability
            vw_customer_onboarding.py
```

## Benefits of writing function based notebooks

Compared to bare notebooks, the function-based approach brings the **following advantages**: 

1. create and publish auto-generated documentation and lineage of notebooks and pipelines (Daipe Enterprise) 
1. write much cleaner notebooks with properly named code blocks
1. (unit)test specific notebook functions with ease
1. use YAML to configure your notebooks for given environment (dev/test/prod/...)
1. utilize pre-configured objects to automate repetitive tasks

## Import Daipe functions

A simple import obtains everything necessary for a Daipe pipeline workflow.

```python
from datalakebundle.imports import *
```

## Decorators

There are two main decorators in the Daipe framework - `@notebook_function()` and `@transformation()`.

### Notebook function

The `@notebook_function()` decorator is used for functions and procedures which don't manipulate with a DataFrame e. g. downloading data..

Once you run the `download_data` function's cell, it gets is automatically called with the dataframe loaded by the `customers_table` function..

```python
@notebook_function()
def download_data():
    opener = urllib.request.URLopener()
    opener.addheader(
        "User-Agent",
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36",
    )

    opener.retrieve("https://www.bondora.com/marketing/media/LoanData.zip", "/loanData.zip")
    opener.retrieve("https://www.bondora.com/marketing/media/RepaymentsData.zip", "/repaymentsData.zip")
```

... or creating empty databases.

```python
@notebook_function()
def init(spark: SparkSession):
    spark.sql(f"create database if not exists {os.environ['APP_ENV']}_bronze;")  # noqa: F821
    spark.sql(f"create database if not exists {os.environ['APP_ENV']}_silver;")  # noqa: F821
    spark.sql(f"create database if not exists {os.environ['APP_ENV']}_gold;")  # noqa: F821
```

#### Decorators can be injected with objects defined in the app

```python
@notebook_function()
def create_input_widgets(dbutils: DBUtils):
    dbutils.widgets.dropdown("base_year", "2015", list(map(str, range(2009, 2022))), "Base year")
```

The common objects that can be injected are:

* `spark: SparkSession` (`from pyspark.sql.session import SparkSession`)  
The Databricks spark instance itself.

* `dbutils: DBUtils` (`from pyspark.dbutils import DBUtils`)  
[Databricks utilities object](https://docs.databricks.com/dev-tools/databricks-utils.html).

* `logger: Logger` (`from logging import Logger`)  
Logger instance for the given notebook.

### Transformation

The `@transformation()` decorator is used for functions which take a DataFrame as an argument, apply some changes to and return a modified DataFrame. It completely replaces the functionality of `data_frame_loader` and `data_frame_saver`, which are now **obsolete.**

The decorator takes functions as parameters.

- `read_csv()`
- `read_table()`
- a more...


```python
@transformation(
    read_csv("/data.csv",
    options=dict(header=True, inferSchema=True)),
)
def read_csv(df: DataFrame):
    return df
```

`display=True` option can be used for displaying the DataFrame.

```python
@transformation(
    read_table("bronze.tbl_customers",
    options=dict(header=True, inferSchema=True)),
    display=True
)
def read_tbl_customers(df: DataFrame):
    return df
```

## Chaining notebook functions

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
@transformation(active_customers_only)
@table_upsert("silver.tbl_customers")
def save(df: DataFrame):
    return df
```

More compact way:

```python
@transformation(read_table("bronze.tbl_customers"))
@table_upsert("silver.tbl_customers")
def tbl_active_customers(df: DataFrame):
    return df.filter(f.col("active") == 1)
```

Once you run the `active_customers_only` function's cell, it gets is automatically called with the dataframe loaded by the `customers_table` function.

Similarly, once you run the `save_output` function's cell, it gets automatically called with the filtered dataframe returned from the `active_customers_only` function.


## Table {overwrite/append/upsert}

Table manipulation decorators are used in conjunction with the `@transformation` decorator. They take either a string identifier of the table...

```python
@transformation(read_csv, display=True)
@table_overwrite("bronze.tbl_customers")
def save(df: DataFrame, logger: Logger):
    logger.info(f"Saving {df.count()} records")
    return df.withColumn("Birthdate", f.to_date(f.col("Birthdate"), "yyyy-MM-dd"))
```

...or a table class as an argument. More about table classes in the next section.

```python
@transformation(read_table("bronze.tbl_customers"), read_table("bronze.tbl_contracts"))
@table_overwrite(tbl_joined_customers_and_contracts)
def join_tables(df1: DataFrame, df2: DataFrame):
    return df1.join(df2, "Customer_ID")
```

### Automatic schema

When using the **string** table identifier, the `@table_overwrite` decorator saves the data using the DataFrame schema. This is useful for prototyping. It is **highly** recommended to use a predifined schema in a class.

## Table schema

To define a table schema a special class has to created. This class has to meet these conditions

- params = `db`, `fields`, `primary_key`, `partition_by`
- `primary_key`, `partition_by` are optional

```python
def get_schema():
    return TableSchema(
        [
            t.StructField("ReportAsOfEOD", t.DateType(), True),
            t.StructField("LoanID", t.StringType(), True),
            t.StructField("Date", t.DateType(), True),
            t.StructField("PrincipalRepayment", t.DoubleType(), True),
            t.StructField("InterestRepayment", t.DoubleType(), True),
            t.StructField("LateFeesRepayment", t.DoubleType(), True),
        ],
        primary_key=["LoanID", "Date"],
        # partition_by = "Date" #---takes a very long time
    )
```

Using the `fields` property for column selection.

```python
@transformation(read_csv("loans.csv"))
@table_overwrite("bronze.tbl_loans", get_schema())
def save(df: DataFrame, logger: Logger):
    return(
        df.select(table_schema.fields)
    )
```

## (required) Setting datalake storage path

Add the following configuration to `config.yaml` to set the default storage path for all the datalake tables:

```yaml
parameters:
  datalakebundle:
    defaults:
      target_path: '/mybase/data/{db_identifier}/{table_identifier}.delta'
```

When setting `defaults`, you can utilize the following placeholders:

* `{identifier}` - `customer.my_table`
* `{db_identifier}` - `customer`
* `{table_identifier}` - `my_table`
* [parsed custom fields](#8-parsing-fields-from-table-identifier)

To modify storage path of any specific table, add the `target_path` attribute to given table's configuration:

```yaml
parameters:
  datalakebundle:
    tables:
      customer.my_table:
        target_path: '/some_custom_base/{db_identifier}/{table_identifier}.delta'
```
