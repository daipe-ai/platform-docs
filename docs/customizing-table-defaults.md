# Customizing table defaults

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
```

