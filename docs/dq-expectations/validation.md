# Validating Your Data
After the *Data Engineer* has added expectations, they need to create validation notebook(s) to validate current data against the defined expectations.

The validation notebook should have the following structure.

Run the [Configuration Notebook](../getting-started/settle-dq-configuration-notebook.md)
```python
%run ./path/to/conf_notebook
```
Run validation on top of your data:
```python
my_table = 'my_db.customers'
results = dq_tool.expectation_store.validate_table(
    table_name=my_table_name
)
results
```
After that, there is a `dq_tool` variable available to work with the DQ Tool.

You'll see the validation result dict in your notebook. The results are written to the Expectation Store database. From there, the *Data User* can see them using the [web application](./view-validation-results.md)