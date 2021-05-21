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