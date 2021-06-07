# Using the @notebook_function() decorator

Then, define the `download_data` function and run it.

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

After you run the function, it gets is automatically called.

## Passing objects into decorated functions

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
  
!!! info "Technical Reference"
    Check the [Technical reference](input-decorators.md#notebook_function) for more details about the @notebook_function and other decorators.
