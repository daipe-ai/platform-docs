# Using Widgets

Widgets are a great tool for parametrizing notebooks in both Databricks and Jupyter Lab.

Standardized **Daipe widgets** support Databricks, Jupyter Lab and CLI.

Import Daipe: 

```python
import daipe as dp```
Create a widget:

```python
@dp.notebook_function()
def create_input_widgets(widgets: dp.Widgets):
    widgets.add_select("base_year", list(map(str, range(2009, 2022))), "2015", "Base year")
```

!!! warning "No more global variables"
    Following the best practices it is **strongly discouraged** to create global variables with widget values. 

Instead of having a global variable, get the value of a widget inside of each decorated function using `get_widget_value`:

```python
@dp.transformation(dp.read_table("silver.tbl_loans"), dp.get_widget_value("base_year"), display=True)
def read_table_bronze_loans_tbl_loans(df: DataFrame, base_year, logger: Logger):
    logger.info(f"Using base year: {base_year}")

    return df.filter(f.col("DefaultDate") >= base_year)
```
