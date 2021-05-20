# Technical documentation

## Imports

All Daipe components are now loaded using a single import.

```python
from datalakebundle.imports import *
```

## Decorators

### @transformation
__@transformation__(`*objects, display = False, check_duplicate_columns = True`)

> Used for decorating a function which manipulates with a DataFrame. Runs the decorated function upon declaration.

- `*objects` : an arbitrary number of objects passed to the decorated function
- `display` : bool, default False - if `True` the output DataFrame is displayed
- `check_duplicate_columns` : bool, default True - if `True` raises an Exception if there are duplicate columns in the DataFrame

Example:

```python
@transformation(read_table("silver.tbl_loans"), read_table("silver.tbl_repayments"), display=True)
@table_overwrite("silver.tbl_joined_loans_and_repayments", get_joined_schema())
def join_loans_and_repayments(df1: DataFrame, df2: DataFrame):
    return df1.join(df2, "LoanID")
```

---

### @notebook_function
__@notebook_function__(`*objects`)

> Used for decorating any other function which is not decorated with the `@transformation` decorator. Runs the decorated function upon declaration.

Parameters:

- `*objects` : an arbitrary number of objects passed to the decorated function
- `display`: bool, default False - if `True` the output DataFrame is displayed
- `check_duplicate_columns` : bool, default True - if `True` raises an Exception if there are duplicate columns in the DataFrame

Example:

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

---

Objects available in __@transformation__ and __@notebook_function__

- spark: SparkSession

- dbutils: DBUtils

- logger: Logger


```python
from logging import Logger
from pyspark.sql.session import SparkSession

@notebook_function()
def customers_table(spark: SparkSession, logger: Logger):
    logger.info('Reading my_crm.customers')

    return spark.read.table('my_crm.customers')
```

---

### @table_overwrite
__@table_overwrite__(`identifier: str, table_schema: TableSchema = None, recreate_table: bool = False, options: dict = None`)

> Overwrites data in a table with a DataFrame returned by the decorated function 

Parameters:

- `identifier` : str - table name
- `table_schema` : TableSchema, default None - [TableSchema](#table_schema) object which defines fields, primary_key, partition_by and tbl_properties, if `None` the table is saved with the DataFrame schema
- `recreate_table` : bool, default False, if `True` the table is dropped if exists before written to
- `options` : dict, default None - options which are passed to `df.write.options(**options)`


---

### @table_append
__@table_append__(`identifier: str, table_schema: TableSchema = None, options: dict = None`)

> Appends data to a table with a DataFrame returned by the decorated function

Parameters:

- `identifier` : str - table name
- `table_schema` : TableSchema, default None - [TableSchema](#table_schema) object which defines fields, primary_key, partition_by and tbl_properties, if `None` the table is saved with the DataFrame schema
- `options` : dict, default None - options which are passed to `df.write.options(**options)`

---

### @table_upsert
__@table_upsert__(`identifier: str, table_schema: TableSchema`)

> Updates data or inserts new data to a table based on primary key with a DataFrame returned by the decorated function

Parameters:

- `identifier` : str - table name
- `table_schema` : TableSchema, default None - [TableSchema](#table_schema) object which defines fields, primary_key, partition_by and tbl_properties, if `None` the table is saved with the DataFrame schema

---

## Functions

### read_csv
__read_csv__(`path: str, schema: StructType = None, options: dict = None`)

> Reads a CSV file into a spark DataFrame

Parameters:

- `path` : str - path to the CSV file
- `schema` : StructType, default None - schema of the CSV file
- `options` : dict, default None - options passed to `spark.read.options(**options)`

Example:

```python
@transformation(read_csv("/LoanData.csv", options=dict(header=True, inferSchema=True)), display=True)
@table_overwrite("bronze.tbl_loans")
def save(df: DataFrame):
    return df.orderBy("LoanDate")
```

---

### read_table
__read_table__(`identifier: str`)

> Reads a table into a spark DataFrame

Parameters:

- `identifier` : str - table name

Example:

```python
@transformation(read_table("silver.tbl_loans"))
def read_table_bronze_loans_tbl_loans(df: DataFrame, dbutils: DBUtils):
    base_year = dbutils.widgets.get("base_year")

    return df.filter(f.col("DefaultDate") >= base_year)
```

---

## Classes

### TableSchema {#table_schema}

__TableSchema__(`fields: list, primary_key: Union[str, list] = None, partition_by: Union[str, list] = None, tbl_properties: dict = None`)

> Defines a table schema

Parameters:

- `fields` : list - list of StructField defining columns of the table
- `primary_key` : Union[str, list], default  None - primary key or a list of keys used for `@table_upsert`
- `partition_by` : Union[str, list], default None - one or multiple fields to partition the data by, __optional__
- `tbl_properties` : dict, default None - key value pairs to be added to `TBLPROPERTIES`, __optional__

Example:

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
        partition_by = "Date",
        tbl_properties = {
            "delta.enableChangeDataFeed" = "true",
        }
    )
```

---

