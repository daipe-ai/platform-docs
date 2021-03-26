# Getting started with Settle DQ

![](../images/architecture-user.png){: style="width: 750px; padding-left: 5%"}

The schema above shows the parts of the system as they are used by Settle DQ users.

*Data Engineer* works with a python interface that is packaged as a `dq_tool` python wheel and installed on a Databricks cluster or notebook.

*Data User* works with a web app in their browser.

The common ground is a PostgreSQL Azure database. 