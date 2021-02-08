# Creating first Data Pipelines project

## Introduction and Prerequisites

[`bricksflow-project-creator.yml`](https://github.com/DataSentics/adap-infra-template/blob/master/.cicd/pipelines/bricksflow-project-creator.yml) is Azure DevOps pipeline that initiates a new [Bricksflow](https://github.com/bricksflow/bricksflow) project from the up-to-date Bricksflow template.

The pipeline also contains the script for automatic protection of the master branch in newly created project.

**Bricksflow project consists of the following components:**

- A git repo with DataFactory pipelines, Bricksflow project structure and sample notebooks to give you sense about the usage and workflow.

- CI/CD pipelines:
  
    - for testing/deployment of your Databricks notebooks / pipelines to the Databricks workspace based on the environment
    - for testing/deployment of DataFactory linked to your project

**Prerequisites**:

For initial setup of Bricksflow based project / repository:

- Successfully deployed **Infrastructure** from [Spinning up resources](datalake-resources-setup.md) page
- Permission to create Personal Access token in your DevOps organization project

## How to set up Bricksflow based project?

###1. Create Personal access token for the first run of the pipeline
- The Personal Access token is needed just for the first run of the pipeline to create a new Bricksflow based project DevOps repository and initial set up of the CICD pipeline
- **It can be deleted** after the successful run of the `bricksflow-project-creator.yml` 

**Go to**:

![](../images/pat_step1.png){: style="width: 300px; padding-left: 5%"}

Then create the new Personal Token with these permissions.

![](../images/pat_step2.png)

###2. Register the Bricksflow project creator pipeline and run it
**Create** a new DevOps pipeline based on `.cicd/pipelines/bricksflow-project-creator` located in infrastructure repo

![](../images/resources_step5.png)
![](../images/resources_step6.png)
![](../images/resources_step7.png)

![](../images/bricks_create1.png)

Set a **PA_TOKEN** variable under the Variables by providing Personal Access Token and **save** the pipeline. 

![](../images/bricks_create2.png)
![](../images/bricks_create3.png)

You can now rename the pipeline, if you'd like to.

**Run** the newly created pipeline and provide the **name of the project** you want to create.

![](../images/bricks_run.png)

Pipeline will need the permission to access the service principal for getting the Databricks workspace variables.

Open the run of the pipeline and under the **View** permit the usage of service principal. 

![](../images/bricks_permissions.png)

## Created resources 

After successful execution, you will find the following **resources**: 

- A new **git repo**
  
  ![](../images/bricks_repo.png){: style="width: 300px; padding-left: 5%"}

  
- **Protected master branch** 
  
    - the initial setup will create the master branch and deploy it to the DEV environment
    - for merging feature branch:
        - the Pull Request needs to be done
        - the notebooks will be deployed to the TEST environment for testing successful run of the pipelines
        - there needs to be in minimum ONE approval for the PR
        - Merging is done by the Squash commit

* **Main CI/CD pipeline** created for the repo. The pipeline is called `master-$yourprojectname-deployment`. 
  
  ![](../images/bricks_created_pipeline.png)


**Note**: the pipeline will be executed automatically after creation of the Bricksflow project repository for deploying sample notebooks to Databricks and will ask for permission to the service connection, as it is shown on the picture below.

![](../images/bricks_permission_cp.png)

## Next steps 

After an initial run of the pipelines, you will have **codes** available in the **Databricks workspace** corresponding to the Databricks environment.

![](../images/bricks_dbx_ws.png){: style="width: 350px; padding-left: 5%"}

You can find the links to the corresponding Databricks workspace directly in the CICD pipeline - Deploy Bricksflow section.
Note that if you enter the workspace for the first time, you have to launch it from the corresponding Azure resource page using Azure portal. 

![](../images/bricks_dbx_link.png)

Also the **DataFactory** in the specific environment **will be linked** to the codes in the Databricks workspace.

You can find the links to the corresponding DataFactory instance directly in the CICD pipeline - Deploy Data Factory section. 

![](../images/bricks_adf_link.png)

For workflow details see [Developers workflow](../data-pipelines-workflow/index.md) page.
