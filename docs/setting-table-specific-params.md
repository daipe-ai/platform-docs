# Setting table-specific configuration

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
