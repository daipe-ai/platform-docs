# Developing Data Pipelines

## Introduction 

Our Workflow contains three environments: DEV, TEST, PROD.

Environments correspond to the separate resource groups.

In the current setup the DEV and TEST are located under one Subscription and PROD lies on different one.


!![](../images/dev_workflow_diagram.png)

As you can see on the diagram above, the workflow contains protected master branch, which is auto-deployed to the DEV and (with approval) to PROD environment after every merge.

The feature branches can be merged to the master by approved Pull Requests:

- When the Pull Request is made, the feature branch is automatically deployed to the TEST environment and the tests are run
- After successfully run tests, the dedicated person can approve the Pull Request and it can be merged
- Merging is done by Squash commit
