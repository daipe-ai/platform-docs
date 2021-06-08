# Databricks clusters don't have internet access

This section is maninly for those who are working in some strict environment where you may not have full internet access.

However there are couple of assumptions

- Git for Windows installed
- Python 3.7 or Conda installed
- Ability to clone or download Github repository
- Ability to call Databricks API
- Databricks Personal Access Token

This is one of the most common cases. Daipe framework works on your local machine as usual but Databricks clusters can't reach internet, hence cell `%run install_master_package` is failing.

Daipe framework has built in feature to solve this situation.

First you need to build dependencies (python wheel packages) needed by master package. You can check what dependencies are needed by running `poetry export --without-hashes`. Then you need to add those dependencies (.whl files) to `dependencies/` directory at the project root.

You can do this process by your self by downloading packages manually or building them on some linux based os/docker.

Be aware that if you are downloading the packages manually you need to make sure that

- You are downloading exact version that is needed
- You are downloading linux compatible package according to Databricks runtime 
- You are downloading python compatible package according to Databricks runtime python version

Or you can use automated way which daipe framework offers.

In ==src/__myproject__/_config/bundles/dbxdeploy.yaml== you can configure on which Databricks workspace and runtime you want to build packages.

```yaml
parameters:
  dbxdeploy:
    target:
      package:
        build:
          databricks:
            host: '%dbxdeploy.databricks.host%'
            token: '%dbxdeploy.databricks.token%'
            job_cluster_definition:
              spark_version: '7.3.x-scala2.12'
              node_type_id: 'Standard_DS3_v2'
              num_workers: 1
```

Note that to build the dependencies the cluster must have internet access. We assume that if you want to build dependecies with ease you will have some non-productional Databricks workspace with internet access.

When your config is setup you can just use command `console dbx:build-dependencies` and daipe will build dependencies on that cluster for you and automatically downloads them in `dependencies/` directory.

Now to deploy project in the way that `%run install_master_package` cell doesn't need to reach internet you need to set `offline_install: True` in dbxdeploy config.

```yaml
parameters:
  dbxdeploy:
    target:
      package:
        offline_install: True
```

Finally you can use `console dbx:deploy` to deploy your project.
