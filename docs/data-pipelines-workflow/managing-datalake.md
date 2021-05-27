# Advance features

### Customizing table names

Table naming can be customized to match your company naming conventions. 

By default, all tables are prefixed with the environment name (dev/test/prod/...):

```yaml
parameters:
  datalakebundle:
    table:
      name_template: '%kernel.environment%_{identifier}'
```

The `{identifier}` is resolved to the table identifier defined in the `datalakebundle.tables` configuration (see above).

By changing the `name_template` option, you may add some prefix or suffix to both the database, or the table names:

```yaml
parameters:
  datalakebundle:
    table:
      name_template: '%kernel.environment%_{db_identifier}.tableprefix_{table_identifier}_tablesufix'
```

## Using table-specific configuration

Besides the [basic configuration options](https://github.com/daipe-ai/databricks-bundle/blob/master/docs/configuration.md), you can also define **configuration for specific datalake tables**:

```yaml
parameters:
  datalakebundle:
    tables:
      customer.my_table:
        params:
          test_data_path: '/foo/bar'
```

Code of the **customer/my_table.py** notebook:

```python
from logging import Logger
from datalakebundle.notebook.decorators import notebook_function, table_params

@notebook_function(table_params('customer.my_table').test_data_path)
def customers_table(test_data_path: str, logger: Logger):
    logger.info(f'Test data path: {test_data_path}')
```

The `table_params('customer.my_table')` function call is a shortcut to using `%datalakebundle.tables."customer.my_table".params%` string parameter.

## Tables management

### Managing datalake tables using TableManager

`TableManager` defines set of commonly used methods to manage datalake tables.
The following example recreates the `my_crm.customers` table (delete old data, create new empty table) every time the notebook runs.

```python
from logging import Logger
from pyspark.sql.dataframe import DataFrame
from datalakebundle.notebook.decorators import data_frame_saver
from datalakebundle.table.TableManager import TableManager

@data_frame_saver()
def customers_table(df: DataFrame, logger: Logger, table_manager: TableManager):
    logger.info('Recreating table my_crm.customers')

    table_manager.recreate('my_crm.customers')

    return df.insertInto(table_manager.get_name('my_crm.customers'))
```

**All TableManager's methods**:

* `get_name('my_crm.customers')` - returns final table name
* `get_config('my_crm.customers')` - returns [TableConfig instance](https://github.com/daipe-ai/datalake-bundle/blob/master/src/datalakebundle/table/config/TableConfig.py)
* `create('my_crm.customers')` - creates table
* `create_if_not_exists('my_crm.customers')` - creates table only if not exist yet
* `recreate('my_crm.customers')` - recreates (deletes Hive table, **deletes data**, create new empty table)
* `exists('my_crm.customers')` - checks if table exists
* `delete('my_crm.customers')` - deletes Hive table, **deletes data**
* `optimize_all()` - runs `OPTIMIZE` command on all defined tables

## Managing datalake tables using the console commands

**Example**: To create the `customer.my_table` table in your datalake, just type `console datalake:table:create customer.my_table --env=dev`
into your terminal within your activated project's virtual environment.
The command connects to your cluster via [Databricks Connect](https://github.com/daipe-ai/databricks-bundle/blob/master/docs/databricks-connect.md) and creates the table as configured.

Datalake management commands: 

* `datalake:table:create [table identifier/path to TableSchema]` - Creates a metastore table based on it's YAML definition (name, schema, data path, ...)

* `datalake:table:recreate [table identifier/path to TableSchema]` - Re-creates a metastore table based on it's YAML definition (name, schema, data path, ...)

* `datalake:table:delete-including-data [table identifier]` - Deletes a metastore table including data on HDFS

* `datalake:table:optimize [table identifier]` - Runs the OPTIMIZE command on a given table

## Parsing fields from table identifier

Sometimes your table names may contain additional flags to explicitly emphasize some meta-information about the data stored in that particular table.

Imagine that you have the following tables:

* `customer_e.my_table`
* `product_p.another_table`

The `e/p` suffixes describe the fact that the table contains *encrypted* or *plain* data. What if we need to use that information in our code?

You may always define the attribute **explictly** in the tables configuration: 

```yaml
parameters:
  datalakebundle:
    table:
      name_template: '{identifier}'
    tables:
      customer_e.my_table:
        encrypted: True
      product_p.another_table:
        encrypted: False
```

If you don't want to duplicate the configuration, try using the `defaults` config option to parse the *encrypted/plain* flag into the new `encrypted` boolean table attribute: 

```yaml
parameters:
  datalakebundle:
    table:
      name_template: '{identifier}'
      defaults:
        encrypted: !expr 'db_identifier[-1:] == "e"'
    tables:
      customer_e.my_table:
      product_p.another_table:
```

For more complex cases, you may also use a custom resolver to create a new table attribute:

```python
from box import Box
from datalakebundle.table.identifier.ValueResolverInterface import ValueResolverInterface

class TargetPathResolver(ValueResolverInterface):

    def __init__(self, base_path: str):
        self.__base_path = base_path

    def resolve(self, raw_table_config: Box):
        encrypted_string = 'encrypted' if raw_table_config.encrypted is True else 'plain'

        return self.__base_path + '/' + raw_table_config.db_identifier + '/' + encrypted_string + '/' + raw_table_config.table_identifier + '.delta'
```

```yaml
parameters:
  datalakebundle:
    table:
      name_template: '{identifier}'
      defaults:
        target_path:
          resolver_class: 'datalakebundle.test.TargetPathResolver'
          resolver_arguments:
            - '%datalake.base_path%'
    tables:
      customer_e.my_table:
      product_p.another_table:
```

## Creating a custom decorator

The Daipe framework is extensible. A user can write their own decorators inside of their project if they follow the interface.

### Example custom write decorator

First let's create a folder `lib` inside the root of our project to contain the custom code.

We are going to create a `@table_stream_append` decorator therefore we need a file for the `table_stream_append` class which is the actual decorator class and `TableStreamAppender` class which handles the logic of writing the data.

The `table_stream_append` class must be decorated with `@DecoratedDecorator`, must inherit from the `OutputDecorator` class 
and implement the `process_result()` method.

The input arguments of the class are completely arbitrary.

```python
@DecoratedDecorator
class table_stream_append(OutputDecorator):
    def __init__(self, identifier: str, table_schema: TableSchema = None, options: dict = None):
```

The `process_result()` function has a fixed interface.

```python
def process_result(self, result: DataFrame, container: ContainerInterface):
```

It recieves the output DataFrame and a container with Daipe services and configuration.

In this function you need to get all the parameters necessary to process the DataFrame e. g. acquire the saving service from the container,....

```python
table_stream_appender: TableStreamAppender = container.get(TableStreamAppender)
```
...get the TableDefinition,...

```python
if self.__table_schema:
    table_definition = table_definition_factory.create_from_table_schema(self.__identifier, self.__table_schema)
else:
    table_definition = table_definition_factory.create_from_dataframe(self.__identifier, result, self.__class__.__name__)
```

or a path from config.

```python
table_parameters_manager: TableParametersManager = container.get(TableParametersManager)
        table_parameters = table_parameters_manager.get_or_parse(self.__identifier)

checkpoint_path = table_parameters.to_dict().get('table_checkpoint_path')
```

Then you call the saving service to save the data.

```python
table_stream_appender.append(result, table_definition, checkpoint_path, self.__options)
```


The complete output decorator class can look like this

```python
from daipecore.decorator.DecoratedDecorator import DecoratedDecorator
from daipecore.decorator.OutputDecorator import OutputDecorator
from datalakebundle.table.parameters.TableParametersManager import TableParametersManager
from injecta.container.ContainerInterface import ContainerInterface
from pyspark.sql import DataFrame

from datalakebundle.table.create.TableDefinitionFactory import TableDefinitionFactory
from datalakebundle.table.schema.TableSchema import TableSchema
from daipedemo.lib.TableStreamAppender import TableStreamAppender


@DecoratedDecorator
class table_stream_append(OutputDecorator):
    def __init__(self, identifier: str, table_schema: TableSchema = None, options: dict = None):
        self.__identifier = identifier
        self.__table_schema = table_schema
        self.__options = options or {}

    def process_result(self, result: DataFrame, container: ContainerInterface):
        table_definition_factory: TableDefinitionFactory = container.get(TableDefinitionFactory)
        table_stream_appender: TableStreamAppender = container.get(TableStreamAppender)

        table_parameters_manager: TableParametersManager = container.get(TableParametersManager)
        table_parameters = table_parameters_manager.get_or_parse(self.__identifier)

        checkpoint_path = table_parameters.to_dict().get('table_checkpoint_path')

        if self.__table_schema:
            table_definition = table_definition_factory.create_from_table_schema(self.__identifier, self.__table_schema)
        else:
            table_definition = table_definition_factory.create_from_dataframe(self.__identifier, result, self.__class__.__name__)

        table_stream_appender.append(result, table_definition, checkpoint_path, self.__options)
```

---

Now we need to implement the saving service class and register it. The saving service can be an arbitrary class which handles your desired saving operation. All we need to do is register it as a service in `_config/config.yaml`

```yaml
services:
  daipedemo.lib.TableStreamAppender:
    arguments:
      - '@datalakebundle.logger'
```

If the class uses a `Logger` it needs to be specified in the arguments. The complete saving service class can for example look like this.

```python
from logging import Logger

from pyspark.sql.dataframe import DataFrame
from datalakebundle.table.create.TableDefinition import TableDefinition
from datalakebundle.table.schema.SchemaChecker import SchemaChecker


class TableStreamAppender:
    def __init__(
        self,
        logger: Logger,
        schema_checker: SchemaChecker,
    ):
        self.__logger = logger
        self.__schema_checker = schema_checker

    def append(self, result: DataFrame, table_definition: TableDefinition, checkpoint_location: str, options: dict):
        output_table_name = table_definition.full_table_name

        self.__schema_checker.check(result, output_table_name, table_definition.schema)

        self.__logger.info(f"Writing data to table: {output_table_name}")
        self.__logger.info(f"Checkpoint location: {checkpoint_location}")

        options['checkpointLocation'] = checkpoint_location

        result.select([field.name for field in table_definition.schema.fields]).writeStream.format("delta").outputMode("append").options(**options).trigger(once=True).table(output_table_name)

        self.__logger.info(f"Data successfully written to: {output_table_name}")
```

