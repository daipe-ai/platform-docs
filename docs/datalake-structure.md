# How does a datalake look like?

It is recommended to structure your tables into the following layers:
 
![bronze, silver, gold](images/bronze_silver_gold.png)

!!! info "Detailed description"
    * **bronze** - "staging layer", raw data from source systems
    * **silver** - most business logic, one or multiple tables per use-case
        * **parsed** - data loaded from *bronze* layer to be stored as datalake tables
        * **cleansed** - data cleaning, source systems bug fixing, ...
        * **Smart Data Model (SDM)** - fully prepared data for further analytical and machine-learning use-cases
    * **gold** - additional filtering/aggregations of silver data (using views or materialized tables) to be served to the final customers.
        * **Feature Store** - central place where customer/product/... features are stored, managed and governed within the organization.
        * **reporting marts** - pre-aggregated views of the data suitable for reporting and visualizations.
