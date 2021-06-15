# Input decorators

These decorators are used to wrap the entire content of a cell. 

## @transformation {#transformation}
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

## @notebook_function {#notebook_function}
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

## Objects available in __@transformation__ and __@notebook_function__ {#objects_available_in_decorators}

- spark: SparkSession

- dbutils: DBUtils

- logger: Logger

Using `Spark` and `Logger`

```python
from logging import Logger
from pyspark.sql.session import SparkSession

@notebook_function()
def customers_table(spark: SparkSession, logger: Logger):
    logger.info('Reading my_crm.customers')

    return spark.read.table('my_crm.customers')
```

Using `DBUtils`

```python
@notebook_function()
def create_input_widgets(dbutils: DBUtils):
    dbutils.widgets.dropdown("base_year", "2015", list(map(str, range(2009, 2022))), "Base year")
```

---

