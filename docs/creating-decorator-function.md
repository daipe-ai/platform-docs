# Creating custom decorator function

First let's create a folder `lib` inside the root of our project to contain the custom code.

We are going to create a `table_stream_read` decorator to read a table as stream. Let's create a file called `table_stream_read`.

Now we just need to ashere to the interface. The function must be decorated using `@input_decorator_function` and it must contain a definition of a `wrapper` function which it returns. Example code:

```python
from daipecore.function.input_decorator_function import input_decorator_function
from injecta.container.ContainerInterface import ContainerInterface
from pyspark.sql import SparkSession
from datalakebundle.table.parameters.TableParametersManager import TableParametersManager


@input_decorator_function
def read_stream_table(identifier: str):
    def wrapper(container: ContainerInterface):
        table_parameters_manager: TableParametersManager = container.get(TableParametersManager)
        table_parameters = table_parameters_manager.get_or_parse(identifier)

        spark: SparkSession = container.get(SparkSession)

        return spark.readStream.format("delta").table(table_parameters.full_table_name)

    return wrapper
```

## Usage

```python
from __myproject__.lib.read_stream_table import read_stream_table

@transformation(read_stream_table("bronze.steaming_events"))
@table_overwrite("silver.tbl_loans")
def save(df: DataFrame):
    return df.filter(f.col("type") == "new_loan").orderBy("LoanDate")
```
