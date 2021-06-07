# Expectations with Expressions
*When you want to do something slightly custom, but don't want to write custom expectations.*

Expressions in expectations are used when you want to check something custom, but the logic is not far from an expectation that is already there. Chosen expectations have their *expression* versions - these work the same as the ordinary ones, the only difference is that instead of a column name, you provide a `column_expression`. These expressions must be valid spark SQL expressions. Pretty much anything you can write after `SELECT` in spark SQL can be used as an expression, see [the list of functions in SQL spark](https://spark.apache.org/docs/latest/api/sql/index.html).

The following expression groups are currently supported:

* **single-column expressions**: The expression name contains 'expression' behind 'column'. E.g. `expect_column_expression_values_to_be_between` is an expression version of [expect_column_values_to_be_between](https://docs.greatexpectations.io/en/latest/autoapi/great_expectations/dataset/dataset/index.html#great_expectations.dataset.dataset.Dataset.expect_column_values_to_be_between). Instead of the `column` parameter, use `column_expression`.
* **column-pair expressions**: E.g. `expect_column_expression_pair_values_to_be_equal` is an expression version of [expect_column_pair_values_to_be_equal](https://docs.greatexpectations.io/en/latest/autoapi/great_expectations/dataset/dataset/index.html#great_expectations.dataset.dataset.Dataset.expect_column_pair_values_to_be_equal). The parameters are named `column_expression_A` and `column_expression_B`
* **multi-column expressions**: E.g. `expect_multicolumn_expression_values_to_be_unique` is an expression version of [expect_multicolumn_values_to_be_unique](https://docs.greatexpectations.io/en/latest/autoapi/great_expectations/dataset/dataset/index.html#great_expectations.dataset.dataset.Dataset.expect_multicolumn_values_to_be_unique). Instead of the `column_list` parameter use `column_expression_list`.


**Warning: Don't go crazy with expressions.** If it's too complicated it should probably be a spark transformation with results written to your datalake. DQ Tool should then be used to only check the results of these transformations, not a place where these transformations are defined. 

If your data analysts aren't well versed in SQL, or you want to hide the logic behind your expectation and keep it at one place, consider using custom expectations.

## Examples
```python
from dq_tool import DQTool
dq_tool = DQTool(spark=spark)
playground = dq_tool.get_playground(my_df)
```
See what expression expectations are available:
```python
playground.list_available_expectation_with_expersion_types()
```
### A single-column expectation
In this example we define an expectation: Difference between `open` and `close` is smaller or equal `10`, 95% of the time. The definition is as follows. Note you can use all standard parameters of the standard expectation, like `mostly`.
```python
open_close_diff = playground.expect_column_expression_values_to_be_between(
    column_expression='abs(open - close)',
    max_value=10,
    mostly=0.95
)
print(open_close_diff)
```
```
{
  "result": {
    "element_count": 2000,
    "missing_count": 0,
    "missing_percent": 0.0,
    "unexpected_count": 8,
    "unexpected_percent": 0.4,
    "unexpected_percent_nonmissing": 0.4,
    "partial_unexpected_list": [
      16.089981079101477,
      12.25,
      10.209991455078125,
      28.8800048828125,
      11.029998779296875,
      16.3800048828125,
      10.019989013671875,
      10.8599853515625
    ]
  },
  "exception_info": null,
  "meta": {},
  "success": true,
  "expectation_config": {
    "kwargs": {
      "column_expression": "abs(open - close)",
      "max_value": 10,
      "mostly": 0.95
    },
    "expectation_type": "expect_column_expression_values_to_be_between",
    "meta": {}
  }
}
```

### A column-pair expectation
Similarly, you can define a column-pair expectation. Here we check if rounded `open` equals rounded `close`:
```python
open_close_rounded = playground.expect_column_expression_pair_values_to_be_equal(
    column_expression_A='round(open)',
    column_expression_B='round(close)'
)
print(open_close_rounded)
```
```
{
  "result": {
    "element_count": 2000,
    "missing_count": 0,
    "missing_percent": 0.0,
    "unexpected_count": 649,
    "unexpected_percent": 32.45,
    "unexpected_percent_nonmissing": 32.45,
    "partial_unexpected_list": [
      [
        78.0,
        75.0
      ],
      [
        81.0,
        78.0
      ],
...
    ]
  },
  "exception_info": null,
  "meta": {},
  "success": false,
  "expectation_config": {
    "kwargs": {
      "column_expression_A": "round(open, 0)",
      "column_expression_B": "round(close, 0)"
    },
    "expectation_type": "expect_column_pair_expressions_to_be_equal",
    "meta": {}
  }
}
```

### A multi-column expectation
You can also use expressions in multi-column expectations. Here we check that for one `symbol` and `date` there should be only one line. For additional fun, we convert the `date` to a unix timestamp.
```python
symbol_date_unique = playground.expect_multicolumn_expression_values_to_be_unique(
    column_expression_list=['symbol', "unix_timestamp(date, 'yyyy-MM-dd')"]
)
print(symbol_date_unique)
```
```
{
  "result": {
    "element_count": 2000,
    "missing_count": 0,
    "missing_percent": 0.0,
    "unexpected_count": 0,
    "unexpected_percent": 0.0,
    "unexpected_percent_nonmissing": 0.0,
    "partial_unexpected_list": []
  },
  "exception_info": null,
  "meta": {},
  "success": true,
  "expectation_config": {
    "kwargs": {
      "column_expression_list": [
        "symbol",
        "unix_timestamp(date, 'yyyy-MM-dd')"
      ]
    },
    "expectation_type": "expect_multicolumn_expression_values_to_be_unique",
    "meta": {}
  }
}
```

