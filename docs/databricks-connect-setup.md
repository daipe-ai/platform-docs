# Databricks Connect setup 

Although Databricks Connect is **NOT required when coding in notebooks**, you may find it useful when working with the [datalake management commands](customizing-table-defaults.md#managing-datalake-tables-using-the-console-commands).

In the *[rootpackage]/_config/bundles/databricksbundle.yaml* project bundle config, add the following configuration:

```yaml
parameters:
  databricksbundle:
    databricks_connect:
      connection:
        address: 'https://dbc-123.cloud.databricks.com'
        token: 'abcd123456'
        cluster_id: '0416-084917-doles835'
```

Storing tokens and other sensitive information in YAML configs is generally not a good idea.
Try moving the token to your environment variables and the *.env* file located in the project root:

```yaml
parameters:
  databricksbundle:
    databricks_connect:
      connection:
        address: 'https://westeurope.azuredatabricks.net'
        token: '%env(DBX_TOKEN)%'
        cluster_id: '0416-084917-doles835'
```

### How to test Databricks connection?
To test that your local configuration works properly, activate the virtual environment and run:
```
$ console dbx:test-connection --env=dev
```
The environment you want to test connection against can be changed by using the`--env`option.
