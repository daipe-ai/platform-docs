# DevOps CI/CD setup

## Update project variables

**Set** SERVICE_CONNECTION_NAME variables for your newly created project in `.cicd/variables/variables.yml`.

![](images/bricks_update_variables.png)

**Commit** the changes:

- set the name of the branch, eq. `update-variables`
- check Create pull request

![](images/bricks_commit.png){: style="width: 500px; padding-left: 15%"}

It will automatically create Pull request to the master branch, get it approved and merge the changes.

The deployment pipeline will be executed automatically, after the merge of updated variables.  
You can find it running under the Pipelines tab.

![](images/bricks_created_pipeline.png)

The project will be deployed to the DEV Databricks and pipeline will ask for permission to the service connection, as it is shown on the picture below.

![](images/bricks_permission_cp.png)
