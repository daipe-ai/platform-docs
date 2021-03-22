# Writing function-based notebooks 

## 1. Introduction

Compared to bare notebooks, the function-based approach brings the **following advantages**: 

1. create and publish auto-generated documentation and lineage of notebooks and pipelines (Daipe Enterprise) 
1. write much cleaner notebooks with properly named code blocks
1. (unit)test specific notebook functions with ease
1. use YAML to configure your notebooks for given environment (dev/test/prod/...)
1. utilize pre-configured objects to automate repetitive tasks

Function-based notebooks have been designed to provide the same user-experience as bare notebooks.
Just write the function, annotate it with the `@notebook_function` decorator and run the cell.

![alt text](../images/notebook-functions.png)

Once you run the `active_customers_only` function's cell, it gets is automatically called with the dataframe loaded by the `customers_table` function.

Similarly, once you run the `save_output` function's cell, it gets automatically called with the filtered dataframe returned from the `active_customers_only` function.

## 2. Using pre-configured objects

Notebook functions can be injected with objects defined in the app:

```python
from databricksbundle.notebook.decorator.notebook_function import notebook_function
from logging import Logger
from pyspark.sql.session import SparkSession

@notebook_function()
def customers_table(spark: SparkSession, logger: Logger):
    logger.info('Reading my_crm.customers')

    return spark.read.table('my_crm.customers')
```

The common objects that can be injected are:

* `spark: SparkSession` (`from pyspark.sql.session import SparkSession`)  
The Databricks spark instance itself.

* `dbutils: DBUtils` (`from pyspark.dbutils import DBUtils`)  
[Databricks utilities object](https://docs.databricks.com/dev-tools/databricks-utils.html).

* `logger: Logger` (`from logging import Logger`)  
Logger instance for the given notebook.

### (Expert) Passing explicitly defined services into notebook functions

Services, which cannot be autowired (= classes with multiple instances), can be injected into the notebook functions explicitly using the `@service_name` notation:

```python
from databricksbundle.notebook.decorator.notebook_function import notebook_function

@notebook_function('@my.service')
def customers_table(my_service: MyClass):
    my_service.do_something()
```

See [Injecta](https://github.com/pyfony/injecta)'s documentation for more details on the syntax.

## 3. Configuring notebook functions

Configuration values defined in your app configuration can be simply passed into the notebook functions using the `%path.to.config%` notation: 

Example `config.yaml` configuration:

```yaml
parameters:
  csvdata:
    path: '/data/csv'
```

Example notebook code:

```python
from logging import Logger
from databricksbundle.notebook.decorator.notebook_function import notebook_function

@notebook_function('%csvdata.path%')
def customers_table(csv_data_path: str, logger: Logger):
    logger.info(f'CSV data path: {csv_data_path}')
```
