# Databricks Connect setup 

Although Databricks Connect is **NOT required when coding in notebooks**, you may find it useful when working with the [datalake management commands](./managing-datalake.md#7-managing-datalake-tables-using-the-console-commands).

In the *[rootpackage]/_config/bundles/databricksbundle.yaml* project bundle config, add the following configuration:

```yaml
parameters:
  databricksbundle:
    databricksConnect:
      connection:
        address: 'https://dbc-123.cloud.databricks.com'
        token: 'abcd123456'
        clusterId: '0416-084917-doles835'
```

If you work with Azure Databricks, you need to specify the `orgId` as well. This parameter can be found in the Databricks Web UI URL (`?o=[orgId]`).

```yaml
parameters:
  databricksbundle:
    databricksConnect:
      connection:
        address: 'https://westeurope.azuredatabricks.net'
        token: 'abcd123456'
        clusterId: '0416-084917-doles835'
        orgId: 123456789 # Azure specific parameter
```

Storing tokens and other sensitive information in YAML configs is generally not a good idea.
Try moving the token to your environment variables and the *.env* file located in the project root:

```yaml
parameters:
  databricksbundle:
    databricksConnect:
      connection:
        address: 'https://westeurope.azuredatabricks.net'
        token: '%env(DBX_TOKEN)%'
        clusterId: '0416-084917-doles835'
        orgId: 123456789 # Azure specific parameter
```

### How to test Databricks connection?
To test that your local configuration works properly, activate the virtual environment and run:
```
$ console dbx:test-connection --env=dev
```
The environment you want to test can be changed in `--env`.
