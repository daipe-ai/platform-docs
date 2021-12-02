# Using templates for metadata extraction

To get metadata from features, it is necessary to use __templates__ instead of exact feature names.

A __template__ is a feature name containing a section in curly braces e. g. `feature_example_{time_window}`.
The part of the actual column name is then _matched_ by the `time_window` placeholder e. .g if `column_name` is `feature_example_90d`
then `time_window` metadata will be `90d`.

The matched metadata is saved into the __Extra__ `key` `value` column of the metadata table and is also propagated into the description using a description template.

!!! warning "Template rules"
    
    - Placeholders will match any string which __DOES NOT__ inlude `_`.
    - One template __CAN__ match multiple columns.
    - All columns __MUST__ be matched by __EXACLY ONE__ template.
    - All templates __MUST__ match __AT LEAST__ one column.
    - `"{feature_name}"` __IS__ a valid template and it __WILL__ match __EVERY__ column which doesn't include `_` and its __ONLY__ piece of metadata will be `feature_name`.
    - `"there_is_no_template_here"` __IS__ also a valid template and it __WILL__ match __ONLY__ a column with that exact name and it will __NOT__ have any metadata.
    
    
    - Templates matching priority is from top to bottom –> always put more specific templates on top and more general template on the bottom. 


```python
@transformation(card_transactions)
@client_feature_writer(
   # Feature template
  ("card_tr_location_{location}_{channel}_{agg_fun}_{time_window}",
   # Description template
   'Total {agg_fun} of {location} {channel} withdrawals in the last {time_window}.'),
  category = 'card_transaction_country_channel'
)
def card_channle_country_features(card_transactions: DataFrame):
  columns_for_agg = []
  for time_window in time_windows:
      columns_for_agg.extend([
        f.sum(
          windowed(f.when((f.col("cardtr_country").isin('CZ', 'CZE')) & (f.col('cardtr_channel_name') == 'ATM'),
                    f.col('cardtr_amount_czk')).otherwise(f.lit(0)), time_window)
        ).alias(f"{col_name}czech_atm_volume_" + f"{time_window}",),
        f.sum(
          windowed(f.when((~f.col("cardtr_country").isin('CZ', 'CZE')) & (f.col('cardtr_channel_name') == 'ATM'),
                    f.col('cardtr_amount_czk')).otherwise(f.lit(0)), time_window)
        ).alias(f"{col_name}abroad_atm_volume_" + f"{time_window}",),
        f.sum(
          windowed(f.when((f.col("cardtr_country").isin('CZ', 'CZE')) & (f.col('cardtr_channel_name') == 'POS'),
                    f.col('cardtr_amount_czk')).otherwise(f.lit(0)), time_window)
        ).alias(f"{col_name}czech_pos_volume_" + f"{time_window}",),
        f.sum(
          windowed(f.when((~f.col("cardtr_country").isin('CZ', 'CZE')) & (f.col('cardtr_channel_name') == 'POS'),
                    f.col('cardtr_amount_czk')).otherwise(f.lit(0)), time_window)
        ).alias(f"{col_name}abroad_pos_volume_" + f"{time_window}",),
        f.sum(
          windowed(f.when((f.col("cardtr_country").isin('CZ', 'CZE')) & (f.col('cardtr_channel_name') == 'ATM'),
                    f.lit(1)).otherwise(f.lit(0)), time_window)
        ).alias(f"{col_name}czech_atm_count_" + f"{time_window}",),
        f.sum(
          windowed(f.when((~f.col("cardtr_country").isin('CZ', 'CZE')) & (f.col('cardtr_channel_name') == 'ATM'),
                    f.lit(1)).otherwise(f.lit(0)), time_window)
        ).alias(f"{col_name}abroad_atm_count_" + f"{time_window}",),
        f.sum(
          windowed(f.when((f.col("cardtr_country").isin('CZ', 'CZE')) & (f.col('cardtr_channel_name') == 'POS'),
                    f.lit(1)).otherwise(f.lit(0)), time_window)
        ).alias(f"{col_name}czech_pos_count_" + f"{time_window}",),
        f.sum(
          windowed(f.when((~f.col("cardtr_country").isin('CZ', 'CZE')) & (f.col('cardtr_channel_name') == 'POS'),
                    f.lit(1)).otherwise(f.lit(0)), time_window)
        ).alias(f"{col_name}abroad_pos_count_" + f"{time_window}",),
      ])
  
  df_channle_country = (
    card_transactions.groupby(["customer_id"])
           .agg(
             *columns_for_agg
           )
  )
  
  ids = spark.read.table("work.table_id").select('customer_id_new', 'client_id')
  return (
    df_channle_country
    .join(ids, df_channle_country.customer_id==ids.customer_id_new, 'inner')
    .drop('customer_id', 'customer_id_new')
    .withColumn('run_date', f.lit(run_date))
  )
```

### Matched metadata

All matched placeholders are shown in the __Extra__ column as `key` `value` pairs.

__Notice__ that the descriptions correspond with the metadata.

![time windowed features result](images/feature_store_templates.png)
