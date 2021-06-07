# Developing and Storing DQ Expectations
After the *Data Engineer* settles with the *Data User* on DQ Agreements, it's time for them to develop expectations. 

You can think of this process as if you're using a command-line - you write some code that defines expectations and save them to the store. You only run this code once and then you can throw it away. When you want to update your expectations, you don't change your code, but write a different piece of code that updates the expectations in the store. 

First, you need to run the [Configuration Notebook](../getting-started/configuration-notebook.md)
```python
%run ./path/to/conf_notebook
```
After that, there is a `dq_tool` variable available to work with the DQ Tool.

To develop expectations, you need a *playground* - an object that is used to define and try out expectations on top of a given metastore table.
```python
my_table = 'my_db.customers'
playground = dq_tool.get_playground(table_name=my_table)
```
By default the playground executes expectations on a limited number of rows. If you want to increase the limit, pass the `row_count_limit` parameter to `get_playground`. If you want to use all rows, pass `None` as a value.

An expectation describes a condition that needs to be met by data in a Hive Metastore table. There's a [long list of expectation types](https://docs.greatexpectations.io/en/0.12.1/reference/glossary_of_expectations.html) available. You can also see all available expectations by running `playground.list_available_expectation_types()` Each expectation type is a method on the `playground` object.

To define an expectation meaning *There are at least 400 rows in the table*, you execute the following function:
```python
line_count = my_playground.expect_table_row_count_to_be_between(min_value=9500, max_value=11000)
line_count
```
The moment you execute the function, the expectation is run on the given metastore table. You can see the result and iterate until you reach the definition you're satisfied with. 

Expectation store is a database containing your expectation definitions. When you're happy with your expectation, add it to the store:
```python
dq_tool.expectation_store.add(
    expectation=line_count,
    table_name=my_table,
    severity='error',
    agreement='The customer count in a dataset exported to the marketing tool should be between 9500 and 11000.'
)
```
The `severity` parameter defines how serious damage there will be, should the expectation fail. Use `'error'` and `'warning'` values.  

The `agreement` parameter is a text of the agreement you settled upon with the *Data User*. It should be clear and concise and it should contain explicit values. There's no automatic sync between the agreement and the rest of the definition, so you need to make sure these are consistent.

Continue adding expectations by repeating the steps above. 

The scope of what you can check using expectations is intentionally limited. If you want do some heavy lifting, like computing aggregations, joining different tables, you should do these in a data pipeline, store the results in a table and add an expectation on top of this table. 
