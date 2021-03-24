# Editing Expectations
After you have [developed and stored](./develop-store.md) your expectations, you need to make sure these stay up to date. Business requirements change all the time, Data Quality is a continuous process, rather than a one-time project.

Think of this process as if you're using a command-line - you write some code that edits expectations and save them to the store. You only run this code once and then you can throw it away.

First, you need to run the [Configuration Notebook](../getting-started/settle-dq-configuration-notebook.md)
```python
%run ./path/to/conf_notebook
```
After that, there is a `dq_tool` variable available to work with the DQ Tool.

First, list expectations that are defined for a given table.
```python
my_table = 'my_db.customers'
dq_tool.expectation_store.print(table_name=my_table)
```
```
database_name: my_db
table_name: customers
suite_key: default
table_expectations:
- 1: expect_table_row_count_to_be_between(min_value=9500, max_value=11000)
column_expectations:
  symbol:
  - 2: expect_column_values_to_not_be_null(column='symbol', mostly=0.99)
  - 3: expect_column_value_lengths_to_be_between(column='symbol', max_value=3, min_value=1)
```
Each expectation has its `expectation_id` a number that is unique for a given table (and suite key). You can get an expectation by this number:
```python
line_count = dq_tool.expectation_store.get(table_name=my_table_name, expectation_id=1)
print(line_count)
```
```
expect_table_row_count_to_be_between(min_value=9500, max_value=11000)
```
Let's say you acquired some more customers and need to update the expectation.
```python
line_count.kwargs['min_value'] = 10500
line_count.kwargs['max_value'] = 12000
```
Then you can run this expectation using a playground object:
```python
playground = dq_tool.get_playground(table_name=my_table)
playground.run_expectation(line_count)
```
When you're satisfied with the new definition, you can save it to the store.
```python
line_count.agreement = 'The customer count in a dataset exported to the marketing tool should be between 10500 and 12000.'
dq_tool.expectation_store.update(line_count)
```
Now your updated expectations are used in data validations. You don't need to make any changes to the validation notebook(s)

You can also delete an expectation from the store by its id:
```python
dq_tool.expectation_store.remove(table_name=table_name, expectation_id=3)
```