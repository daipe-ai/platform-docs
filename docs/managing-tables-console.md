# Managing datalake tables using console commands

**Example**: To create the `customer.my_table` table in your datalake, just type `console datalake:table:create customer.my_table --env=dev`
into your terminal within your activated project's virtual environment.
The command connects to your cluster via [Databricks Connect](databricks-connect-setup.md) and creates the table as configured.

Datalake management commands: 

* `datalake:table:create [table identifier] [path to TableSchema module]` - Creates a metastore table based on it's YAML definition (name, schema, data path, ...)

* `datalake:table:recreate [table identifier] [path to TableSchema module]` - Re-creates a metastore table based on it's YAML definition (name, schema, data path, ...)

* `datalake:table:delete-including-data [table identifier]` - Deletes a metastore table including data on HDFS

* `datalake:table:optimize [table identifier]` - Runs the OPTIMIZE command on a given table
