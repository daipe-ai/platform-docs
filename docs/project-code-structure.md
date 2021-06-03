# Recommended project code structure

For databases and tables in each of bronze/silver/gold layers it is recommended to follow the **[db_name/table_name]** directory structure:  

```yaml
[PROJECT_ROOT]/src
    [ROOT_MODULE_NAME] # most commonly name of your company or product
        bronze_db_batch
            tbl_customers.py
            tbl_products.py
            tbl_contracts # it is possible to place notebooks in folders with the same name if necessary
                tbl_contracts.py
                csv_schema.py
            ...
        silver_db_batch
            tbl_product_profitability.py
            tbl_customer_profitability.py
            tbl_customer_onboarding.py
            ...
        gold_db_batch
            vw_product_profitability.py # view on silver_db_batch.tbl_product_profitability
            tbl_customer_profitability.py # "materialized" view on silver_db_batch.tbl_customer_profitability
            vw_customer_onboarding.py
```