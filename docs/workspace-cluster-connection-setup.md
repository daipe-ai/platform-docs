# Workspace & cluster connection setup

Once the environment setup is completed, **finish the project setup by**:

1. Putting your [Databricks workspace address](https://docs.databricks.com/dev-tools/databricks-connect.html#step-2-configure-connection-properties) into the ==src/[ROOT_MODULE]/_config/config_dev.yaml== file.
1. Setting the `DBX_TOKEN` varible in the ==[PROJECT_ROOT]/.env== file with your Databricks personal access token.
1. Activating the Conda environment by running
   ```bash
   conda activate $PWD/.venv # or use the `ca` alias
   ```

1. Deploying your new project to databricks workspace by running
   ```bash
   $ console dbx:deploy --env=dev
   ```

1. Now you should your new project in the configured Databricks workspace: ![](images/pushed_project.png)
