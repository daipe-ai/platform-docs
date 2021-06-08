# Setting datalake storage path

Add the following configuration to `config.yaml` to set the default storage path for all the datalake tables:

```yaml
parameters:
  datalakebundle:
    defaults:
      target_path: '/mybase/data/{db_identifier}/{table_identifier}.delta'
```

When setting `defaults`, you can utilize any of the following placeholders:

* `{identifier}` - `customer.my_table`
* `{db_identifier}` - `customer`
* `{table_identifier}` - `my_table`
* [parsed custom fields](customizing-table-defaults.md#8-parsing-fields-from-table-identifier)

!!! tip "How to set storage path for a specific table?"
    Storage path of any specific table can be easily changed by adding the `target_path` attribute to given table's configuration:
    
    ```yaml
    parameters:
      datalakebundle:
        tables:
          customer.my_table:
            target_path: '/some_custom_base/{db_identifier}/{table_identifier}.delta'
    ```
