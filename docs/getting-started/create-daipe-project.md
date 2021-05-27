# Create your first Daipe-powered project

## Prerequisites

The following software needs to be installed first:

  - [Miniconda package manager](https://docs.conda.io/en/latest/miniconda.html)
  - [Git for Windows](https://git-scm.com/download/win) or standard Git in Linux (_apt-get install git_)
  
We recommend using the following IDEs:

  - [PyCharm Community or Pro](https://www.jetbrains.com/pycharm/download/) with the [EnvFile plugin](https://plugins.jetbrains.com/plugin/7861-envfile) installed
  - [Visual Studio Code](https://code.visualstudio.com/download) with the [PYTHONPATH setter extension](https://marketplace.visualstudio.com/items?itemName=datasentics.pythonpath-setter) installed

## Create your first Daipe-powered project

* On **Windows**, use Git Bash
* On **Linux/Mac**, the use standard terminal

Create a **new Daipe project** by using the following command:

```
curl -s https://raw.githubusercontent.com/daipe-ai/project-creator/master/create_project.sh | bash -s skeleton-databricks
```

**What is does:**

1. Asks for project & directory name of your new project 
2. Download the [Daipe project skeleton template](https://github.com/daipe-ai/skeleton-databricks)
3. Create the new project skeleton based on the template
4. Runs the Daipe [development environment initialization script](https://github.com/daipe-ai/benvy)

**Configure the local project:** 

When the environment setup is completed, [configure your Databricks cluster connection details](https://docs.databricks.com/dev-tools/databricks-connect.html#step-2-configure-connection-properties):

Update *src/[ROOT_MODULE]/_config/config_dev.yaml* with your Databricks `address` and optionally `cluster_id`.

Add your Databricks token to the `[PROJECT_ROOT]/.env` file

**Activate your project environment:**

Now activate the Conda environment for your new project:

```bash
$ conda activate $PWD/.venv
```

or use a shortcut

```bash
$ ca
```

Now you can push your project to Databricks Workspace using following command

```bash
$ console dbx:deploy --env=dev
```

And you should see the pushed project in Databricks workspace

![](../images/pushed_project.png)


## (required) Setting datalake storage path

Add the following configuration to `config.yaml` to set the default storage path for all the datalake tables:

```yaml
parameters:
  datalakebundle:
    defaults:
      target_path: '/mybase/data/{db_identifier}/{table_identifier}.delta'
```

When setting `defaults`, you can utilize the following placeholders:

* `{identifier}` - `customer.my_table`
* `{db_identifier}` - `customer`
* `{table_identifier}` - `my_table`
* [parsed custom fields](#8-parsing-fields-from-table-identifier)

To modify storage path of any specific table, add the `target_path` attribute to given table's configuration:

```yaml
parameters:
  datalakebundle:
    tables:
      customer.my_table:
        target_path: '/some_custom_base/{db_identifier}/{table_identifier}.delta'
```

**Important scripts:**

1. ```poe flake8``` - checks coding standards
1. ```poe container-check``` - check app container consistency (if configured properly)
