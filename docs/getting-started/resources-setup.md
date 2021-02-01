# Spinning up resources

## 1. Create repository for infrastructure and import it's code

- In Azure DevOps click on repositories
- Click on dropdown menu
- Click on New repository

!![](../images/resources_step1.png)

- Name it e.g. `infra`
- Uncheck Add a README
- Click Create

!![](../images/resources_step2.png)

- Click on Import
- In Clone URL fill `https://github.com/DataSentics/adap-infra-template.git`
- Click on Import

!![](../images/resources_step3.png)

## 2. Set infrastructure variables

The file `.cicd/variables/variables.yml` holds variables that you can use to customize your infrastructure.

!![](../images/resources_step4.png)

Replace the placeholders.

- TENANT_ID - from previous steps
- PROJECT_NAME - to your liking
- SERVICE_CONNECTION_NAME_DEV - devops-service-connection-to-{devsubscription}
- SERVICE_CONNECTION_NAME_TEST - devops-service-connection-to-{testsubscription}
- SERVICE_CONNECTION_NAME_PROD - devops-service-connection-to-{prodsubscription}
- GIT_ACCOUNT_NAME - name of your devops organization
- GIT_PROJECT_NAME - name of your devops project

## 3. Create DevOps pipeline for infrastructure build & deployment

- In Azure DevOps click on pipelines
- Click on New pipeline

!![](../images/resources_step5.png)

- Select Azure Repos Git

!![](../images/resources_step6.png)

- Select infra repository

!![](../images/resources_step7.png)

- It will automaticaly locate file `azure-pipelines.yml`
- Click Save

!![](../images/resources_step8.png)

- Go back to Azure pipelines
- Click on All
- Click on infra

!![](../images/resources_step9.png)

- Click on Run pipeline

!![](../images/resources_step10.png)
