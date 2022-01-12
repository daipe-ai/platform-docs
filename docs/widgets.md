# Widgets

Daipe widgets are cross-platform meaning the same interface works on both __Databricks__ and __Jupyter Lab__ as well as when running a `.py` file as a script from __CLI__.

!!! info "Available since"
    * **databricks-bundle=^1.1.2** - for usage in Databricks
    * **jupyter-bundle=^1.1.1** - for usage in Jupyter Lab
    * **daipe-core=^1.1.1** - for usage in CLI

## Imports

```python
import daipe as dp
```

## Usage


__widgets.add_text__(`name: str, default_value: str = "", label: str = None`)

> Creates a text widget

Parameters:

- `name` : str - unique identifier of the widget
- `default_value` : str, default "" - default value of the widget, __optional__
- `label` : str, default "" - label of the widget, __optional__

Example:

```python
@dp.notebook_function()
def create_text_widget(widgets: dp.Widgets):
    widgets.add_text("text_widget", "Hello", "Test widget")
```
___

__widgets.add_select__(self, name: str, choices: list, default_value: str, label: str = None)

> Creates a select/dropdown widget

Parameters:

- `name` : str - unique identifier of the widget
- `choices` : list - the choices to select
- `default_value` : str, default "" - default value of the widget, __optional__
- `label` : str, default "" - label of the widget, __optional__

Example:

```python
@dp.notebook_function()
def create_select_widget(widgets: dp.Widgets):
    widgets.add_select("select_widget", ["option1", "option2", "option3"], "option1", "Test widget")
```

___

__widgets.add_multiselect__(self, name: str, choices: list, default_value: str, label: str = None)

> Creates a multiselect widget

Parameters:

- `name` : str - unique identifier of the widget
- `choices` : list - the choices to select
- `default_value` : str, default "" - default value of the widget, __optional__
- `label` : str, default "" - label of the widget, __optional__

Example:

```python
@dp.notebook_function()
def create_multiselect_widget(widgets: dp.Widgets):
    widgets.add_multiselect("multiselect_widget", ["option1", "option2", "option3"], "option1", "Test widget")
```
___


__widgets.get_value__(self, name: str)

> Returns a value from a widget

Parameters:

- `name` : str - unique identifier of the widget

Example:

```python
@dp.transformation(load_df)
def get_widget_value(df: DataFrame, widgets: dp.Widgets):
    value = widgets.get_value("text_widget")  # value: "Hello"
    return df.filter(f.col("col") == value)
```

---


__widgets.remove__(self, name: str)

> Deletes a widget

Parameters:

- `name` : str - unique identifier of the widget

Example:

```python
@dp.notebook_function()
def remove_widget(widgets: dp.Widgets):
    widgets.remove("text_widget")
```

---


__widgets.remove_all__(self)

> Deletes all widgets

Example:

```python
@dp.notebook_function()
def remove_widget(widgets: dp.Widgets):
    widgets.remove_all()
```

