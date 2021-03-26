# Settle DQ Workflow

![](../images/development-workflow.png){: style="width: 750px; padding-left: 5%"}

Working with Settle DQ is a continuous process. 

* The *Data User* and the *Data Engineer* settle on *DQ Agreements* - a short description of condtions that the data should satisfy. 
* The *Data Engineer* then [develops *DQ Expectations*](develop-store.md) - declarative definitions that check whether data satisfy the given conditions. 
* Then the *Data Engineer* [creates validation notebooks](validation.md) that validate the data against the expectations (this only needs to be done for new checked data sources).
* The *Data User* then [sees the validation results](view-validation-results.md) in a web app every time new data are processed. When conditions change, *Data User* and the *Data Engineer* discuss to update and/or create new *DQ Agreements*. 