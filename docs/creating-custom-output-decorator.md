# Creating a custom output decorator

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

In our project we simple import the function using

```python
from __myproject__.lib.table_stream_append import table_stream_append
```
