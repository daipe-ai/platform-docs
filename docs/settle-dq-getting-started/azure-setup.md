# Azure Setup

![](../images/settle-dq/infrastructure.png){: style="width: 750px; padding-left: 5%"}

The schema above shows the infrastructure the system operates on. You will need to set up the resources displayed in the *Your Azure Cloud* rectangle.

We recommend to create the resources in the following order:
1. Resource Group
2. Database
3. Databricks

## Prerequisites

![](../images/subscription_resource_providers.png)

You will need a subscription where you have permissions to the following Resource providers. The ones that are **bold** are explicitly needed for the system to be set up. 

- **Microsoft.Databricks**
- Microsoft.DataFactory
- Microsoft.OperationalInsights
- **Microsoft.KeyVault**
- **Microsoft.Storage**
- Microsoft.Network
- **Microsoft.ManagedIdentity**
- Microsoft.Security
- **Microsoft.PolicyInsights**
- Microsoft.GuestConfiguration
- Microsoft.Compute
- **Microsoft.ContainerService**
- Microsoft.Advisor
- microsoft.insights
- Microsoft.MachineLearningServices
- Microsoft.Purview
- **Microsoft.DBforPostgreSQL**
- **Microsoft.Web**

## 1. Resource Group
We recommend [creating a new resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal#create-resource-groups) for all resources that you will create for Settle DQ. 

## 2. Database
The database is the center piece of the DQ Tool. The Python API stores expectation definitions and validations results there. The Web App reads information to display from there. 

If you already have an Azure PostgreSQL Database server, you can just create a new database there. Connect to the server and [create a new database](https://www.postgresql.org/docs/10/sql-createdatabase.html).

If you don't have a database server, you'll need to create one. We recommend using Azure Database for PostgreSQL. Follow the [quickstart guide](https://docs.microsoft.com/en-us/azure/postgresql/quickstart-create-server-database-portal). We recommend the following options:

- Single server
- PosgreSQL 10
- Choose the machine depending on your expected workload, the minimal recommended requirements are 1 vcore, 20GB storage

For later steps you will need the following, so keep it ready:

- host
- port
- database
- username
- password

We recommend storing these in Azure Key Vault, see the [quickstart guide](https://docs.microsoft.com/en-us/azure/key-vault/secrets/quick-create-portal).

## 3. Databricks
If you're not using Azure Databricks yet, you'll need to [create a new Databricks Workspace](https://docs.microsoft.com/en-us/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal?tabs=azure-portal). 

If you're already using Azure Databricks and have a workspace, you can just use that. 

You will need an interactive cluster with Databricks Runtime >= 7.0 to install the wheel to. 

Your Databricks will need access to Azure Key Vault to retrieve the database credentials. Follow [the guide to create an Azure Key Vault-backed secret scope](https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes#akv-ss)

Now we'll install `dq_tool` python wheel to Databricks. The wheel provides a python intefrace that data engineers use to develop and manage data expectations. 
