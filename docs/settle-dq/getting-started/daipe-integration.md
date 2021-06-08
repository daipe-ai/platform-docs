# Daipe Integration

If you're already using Daipe and want to start using DQ Tool, you need to set it up, as described below. The DQ Tool configuration and instantiation is handled by Daipe, so you can use a DQ Tool instance in any Daipe-decorated notebook function. 

## Setup
The following steps assume you already have a daipe project and want to start working with DQ Tool in that project. 
### 1. Add Artifact Feed URL to pyproject.toml
The dq-tool-bundle library is distrubuted through a private Datasentics Python package index (artifact feed). Add the following lines to your `pyproject.toml` so that poetry knows where to get private python wheels.
```
[[tool.poetry.source]]
name = "datasentics"
url = "<artifact feed url>"
secondary = true
```
The artifact feed url will be given to you along with your token. 
### 2. Set your artifact feed token in poetry
Run the following command in your terminal with conda activated:
```
poetry config http-basic.datasentics __token__ <the token> --local
```
Now poetry has credentials to access the artifact feed. 
### 3. Add dq-tool-bundle as a dependency
Run the following command in your terminal with conda activated:
```
poetry add dq-tool-bundle
```
Now you have installed the DQ Tool bundle, the DQ Tool and other necessary dependencies. When you deploy to databricks, these will be available there too. 
### 4. Add database connection string to Databricks secrets / Azure Vault
Take credentials to your expectation database and store a connection string to your Databricks secrets, or Azure Vault accessible in Datatabricks secrets. The connection string needs to have the following format:
```
postgresql://username:pasword@host:port/database
```
Note that if you have a `$` in your password, you'll need to escape it `\$` in the connection string. 

Next, store the artifact feed token to your Databricks secrets / Azure Vault.

Remember the scope and secret names where you have stored these secrets. You'll need them in the next step. 

### 5. Set up cluster environment variables  
In your databricks cluster you need to set up the following variables. You can find environment variables under cluster configuration -> Advanced Options -> Spark.
![](../images/databricks-env.png){: style="width: 750px; padding-left: 5%"}
Set the following variables:
```
DQ_TOOL_DB_STORE_CONNECTION_STRING={{secrets/your_scope/postgres_store_connection_string}}
DATABRICKS_HTTP_BASIC_DATASENTICS_PASSWORD={{secrets/your_scope/artifact_feed_token}}
DATABRICKS_HTTP_BASIC_DATASENTICS_USERNAME=__token__
```
You'll need to restart your cluster so that the changes can take effect. 

## Usage
Now your can re-deploy your Daipe project and start using DQ Tool.

### Define expectations
To define a new expectation, run a notebook function like the following. 
```python
from datalakebundle.imports import notebook_function
from datalakebundle.table.parameters.TableParametersManager import TableParametersManager
from dq_tool import DQTool

@notebook_function()
def define_expectations_bronze_covid_tbl(dq_tool: DQTool, table_parameters_manager: TableParametersManager):
    # playground lets you run expectation on top of a table
    params = table_parameters_manager.get_or_parse('my_db.my_table')
    dq_tool.get_playground(table_name=params.table_identifier, database_name=params.db_identifier)
    # the NEVER column values should be between 0 and 1
    never_limits = my_playground.expect_column_values_to_be_between(column="NEVER", min_value=0, max_value=1)
    print(never_limits)
```
For a list of expectations you can define, see the [Great Expectations docs](https://docs.greatexpectations.io/en/0.13.8/reference/glossary_of_expectations.html)

After you have fine-tuned your expectation definition, you can save it within a decorated function like this:
```python
dq_tool.expectation_store.add(
    expectation=never_limits,
    database_name=my_database_name,
    table_name=my_table_name,
    severity='error',
    agreement='The prediction model needs at least 400 rows to predict something meaningful.',
    tags=['Data Science', 'Basic']
)
```

Note that this is a throw-away code that doesn't need to be stored in git. All expectation definitions live in your database. To edit expectations, see the [editing guide](../developing-expectations/edit.md)

### Validate data 
In your pipeline, you'll want to validate data in a table using expectations you saved in the previous step. Use the following snippet as an example. 

```python
from datalakebundle.imports import notebook_function

@notebook_function()
def validate_table(dq_tool: DQTool, table_parameters_manager: TableParametersManager):
    results = dq_tool.expectation_store.validate_table(
        database_name=my_database_name,
        table_name=my_table_name
    )
    print(results)
```
