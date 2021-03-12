# Data Pipelines Workflow Overview

Our Workflow contains three environments: DEV, TEST, PROD.

Environments correspond to the separate resource groups.

In the current setup, the DEV and TEST environments are located under one Subscription and the PROD environment is placed in another one.


![](../images/dev_workflow_diagram.png){: style="width: 500px; padding-left: 5%"}

As you can see on the diagram above, the workflow contains protected master branch, which is auto-deployed to the DEV and (with approval) to PROD environment after every merge.

The feature branches can be merged to the master branch once Pull Request is approved:

- When the Pull Request is made, the feature branch is automatically deployed to the TEST environment and the tests are run
- As soon as the tests are completed successfully, the release manager can approve the Pull Request to merge the new branch to master
- Merging is done using the "squash" strategy (all changes are squashed into a single commit)

