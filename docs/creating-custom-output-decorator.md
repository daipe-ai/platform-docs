# Creating a custom output decorator

We are going to create a `@table_append_and_send_to_api` decorator which appends DataFrame to a table while simultaneously sends the data to an API.

Inside our project's `src/__project_name__/lib` we create a file called `table_append_and_send_to_api.py`

We must adhere to this interface. 

```python
@DecoratedDecorator
class table_append_and_send_to_api(OutputDecorator):
    def __init__(self, identifier: str, url: str, table_schema: TableSchema = None, options: dict = None):
        # init
        
        def process_result(self, result: DataFrame, container: ContainerInterface):
            # the decorator logic
```

The input arguments of the class are completely arbitrary. We are using an `url` variable to specify where to send the data.

The `process_result()` function has a fixed interface.

It recieves the output DataFrame and a container with Daipe services and configuration.

In this function you need to get all the objects necessary to process the DataFrame from the container e. g. `TableDefinitionFactory` and `SchemaChecker` for checking the dataframe metadata.

```python
table_definition_factory: TableDefinitionFactory = container.get(TableDefinitionFactory)
schema_checker: SchemaChecker = container.get(SchemaChecker)
logger: Logger = container.get("datalakebundle.logger")
```
These are common instances which could be useful in your custom decorators as well.

Lastly we create a custom function for sending data to an API.

```python
def __send_to_api(self, df):
    df_json = df.toPandas().head().to_json()
    conn = http.client.HTTPSConnection(self.__url)
    conn.request("POST", "/", df_json, {'Content-Type': 'application/json'})
```

The complete output decorator class can look like this

```python
from logging import Logger

import http.client
from daipecore.decorator.DecoratedDecorator import DecoratedDecorator
from daipecore.decorator.OutputDecorator import OutputDecorator
from injecta.container.ContainerInterface import ContainerInterface
from pyspark.sql import DataFrame

from datalakebundle.table.create.TableDefinitionFactory import TableDefinitionFactory
from datalakebundle.table.schema.TableSchema import TableSchema
from datalakebundle.table.schema.SchemaChecker import SchemaChecker


@DecoratedDecorator
class table_append_and_send_to_api(OutputDecorator):
    def __init__(self, identifier: str, url: str, table_schema: TableSchema = None, options: dict = None):
        self.__identifier = identifier
        self.__table_schema = table_schema
        self.__options = options or {}
        self.__url = url

    def process_result(self, result: DataFrame, container: ContainerInterface):
        # Get instances from container
        table_definition_factory: TableDefinitionFactory = container.get(TableDefinitionFactory)
        schema_checker: SchemaChecker = container.get(SchemaChecker)
        logger: Logger = container.get("datalakebundle.logger")

        # Check table_schema, recommended but potencially not necessary
        if self.__table_schema:
            table_definition = table_definition_factory.create_from_table_schema(self.__identifier, self.__table_schema)

            schema_checker.check(result, table_definition)
        else:
            table_definition = table_definition_factory.create_from_dataframe(self.__identifier, result,
                                                                              self.__class__.__name__)
        output_table_name = table_definition.full_table_name

        logger.info(f"Appending {result.count()} records to {output_table_name}")

        # Using our custom api function
        self.__send_to_api(result)

        logger.info(f"Sending {result.count()} records to API")

        result.select([field.name for field in table_definition.schema.fields]).write.format("delta").mode(
            "append").options(**self.__options).saveAsTable(output_table_name)

    def __send_to_api(self, df):
        df_json = df.toPandas().head().to_json()
        conn = http.client.HTTPSConnection(self.__url)
        conn.request("POST", "/", df_json, {'Content-Type': 'application/json'})
```

In our project we simple import the decorator using

```python
from __myproject__.lib.table_append_and_send_to_api import table_append_and_send_to_api
```
