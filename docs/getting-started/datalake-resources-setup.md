# Spinning up Data Lake resources

## 1. Create repository for infrastructure and import it's code

- In Azure DevOps click on repositories
- Click on dropdown menu
- Click on New repository

![](../images/resources_step1.png)

- Name it e.g. `infra`
- Uncheck Add a README
- Click Create

![](../images/resources_step2.png)

- Click on Import
- In Clone URL fill `https://github.com/DataSentics/adap-infra-template.git`
- In Username fill your Datasentics email
- In Password fill your Github password
- Click on Import

![](../images/resources_step3.png)

## 2. Set infrastructure variables

The file `.cicd/variables/variables.yml` holds variables that you can use to customize your infrastructure.

![](../images/resources_step4.png)

Replace the placeholders.

- TENANT_ID - from [Azure setup](azure-setup.md) section 5
- PROJECT_NAME - <span style="color: red">!! should be simple lowercase name (max 5 characters) !!</span>
- SERVICE_CONNECTION_NAME_DEV - devops-service-connection-to-{devsubscription}
- SERVICE_CONNECTION_NAME_TEST - devops-service-connection-to-{testsubscription}
- SERVICE_CONNECTION_NAME_PROD - devops-service-connection-to-{prodsubscription}
- GIT_ACCOUNT_NAME - name of your devops organization
- GIT_PROJECT_NAME - name of your devops project

In the files `.cicd/variables/variables-{dev/test/prod}.yml` change ADMIN_OBJECT_ID to object id of user of your choice. This user will have admin access to created keyvault.

You can find user object id in Active Directory.

![](../images/user_object_id.png)

## 3. Create DevOps pipeline for infrastructure build & deployment

- In Azure DevOps click on pipelines
- Click on New pipeline

![](../images/resources_step5.png)

- Select Azure Repos Git

![](../images/resources_step6.png)

- Select infra repository

![](../images/resources_step7.png)

- It will automaticaly locate file `azure-pipelines.yml`
- Click Save

![](../images/resources_step8.png)

- Go back to Azure pipelines
- Click on All
- Click on infra

![](../images/resources_step9.png)

- Click on Run pipeline

![](../images/resources_step10.png)

- Make sure that you run all stages

![](../images/resources_step11.png)
![](../images/resources_step12.png)

- Click Run

![](../images/resources_step13.png)

## 4. Create Key Vault Secret Scope in Databricks

When the pipeline is finished you need to create secret scope for Databricks.

<span style="color: red">!! This needs to be done for all environments dev/test/prod !!</span>

- Go to Databricks workspace

![](../images/resources_step14.png)

- Look in the URL
- There should be something like `https://adb-3076017168624144.4.azuredatabricks.net/?o=3076017168624144`
- Add `#secrets/createScope` at the end of URL
- URL now should look like `https://adb-3076017168624144.4.azuredatabricks.net/?o=3076017168624144#secrets/createScope`
- Hit enter and you should be redirected to the page below

![](../images/resources_step15.png)

- Fill in information
- Scope Name - `unit-kv`
- DNS Name and Resource ID can be found in key vault properties

![](../images/resources_step16.png)

## 5. Resources overview

After the infrastructure is deployed you can check the resources under resource group `adap-cz-PROJECT_NAME-rg-dev`

![](../images/resources_rg_overview.png)

**Main components**

- Databricks workspace - this is place where you develop your spark notebooks
- Storage accoount - this is place where your data lives
- Key vault - this is place where secrets are stored
- Data factory - main orchestration engine for your Databricks notebooks
- Virtual network - Key vault and Databricks clusters are deployed in this virtual network for better isolation

![](../images/resources_overview.png)