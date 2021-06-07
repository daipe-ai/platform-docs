# Settle DQ Configuration Notebook

The configuration notebook isn't needed when you use Daipe. If you're using Daipe, continue to the [Daipe Integration Guide](./daipe-integration.md). If not, continue reading. 

The python interface to Settle DQ is called DQ Tool. It is distributed as a python wheel that you install to your Databricks cluster. 

To use DQ Tool, you need to configure it so that it knows how to connect to the database. We highly recommend storing this configuration in a single notebook and run the notebook whenever you need to work with DQ Tool. 

We higly recommend storing connection paremeters in a Databricks secrets service, especially the password. If you already have these in Azure Key Vault, [create a Azure Key Vault-backed secret scope](https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes#akv-ss) in Databricks and read the params from there.
If you want to keep the parameters just in Databricks, [create a Databricks-based secret scope](https://docs.databricks.com/security/secrets/secret-scopes.html) and store the parameters there.

The notebook should contain code like shown in this example:
```python
from dq_tool import DQTool
dq_tool = DQTool(
    spark=spark,
    db_store_connection={
        'drivername': 'postgresql',
        'host': dbutils.secrets.get(scope='dbx_scope', key='host'),
        'port': dbutils.secrets.get(scope='dbx_scope', key='port'),
        'database': dbutils.secrets.get(scope='dbx_scope', key='database'),
        'username': dbutils.secrets.get(scope='dbx_scope', key='username'),
        'password': dbutils.secrets.get(scope='dbx_scope', key='password')
    }
)
```