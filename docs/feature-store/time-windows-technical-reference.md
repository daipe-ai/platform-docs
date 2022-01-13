# Time windows helper classes and functions

## WindowedDataFrame {#WindowedDataFrame}
__WindowedDataFrame__(`self, df: DataFrame, entity: Entity, time_column: str, time_windows: List[str]`) -> `WindowedDataFrame`

> DataFrame which allows for time_windowed calculations

- `df` : DataFrame, input DataFrame
- `entity` : Entity, the feature store `Entity` instance, such as `client`
- `time_column` : str, name of `Date` or `Timestamp` Column in `df`, which is subtracted __from__ the `run_date` to create the time window interval
- `time_windows`: `List[str]`, list of time windows as a `[0-9]+[dhw]`, suffixes `d` = days, `h` = hours, `w` = weeks 

### Methods:

## time_windowed {#time_windowed}
__time_windowed__(`self, agg_columns_function: Callable[[str], List[WindowedColumn]] = lambda x: list(), non_agg_columns_function: Callable[[str], List[Column]] = lambda x: list(), extra_group_keys: List[str] = []`) -> `WindowedDataFrame`

> Returns a new WindowedDataFrame with calculated aggregated and non aggregated columns

- `agg_columns_function: Callable[[str], List[WindowedColumn]] = lambda x: list()`: Function which takes `time_window: str` and returns `List[WindowedColumn]`
- `non_agg_columns_function: Callable[[str], List[Column]] = lambda x: list()`: Function which takes `time_window: str` and returns `List[Column]`
- `extra_group_keys: List[str] = []`: By default it groups by `entity.primary_key`, use `extra_group_keys` to add more columns to group by

Example:

```python
@dp.transformation(card_transactions)
def card_channle_country_features(card_transactions: WindowedDataFrame):
    def country_agg_features(_) -> List[tw.WindowedColumn]:
        return [
            tw.sum_windowed(
                f.col("cardtr_country").isin("CZ", "CZE").cast("integer"),
                "card_tr_location_czech_count_{time_window}",
            ),
            tw.sum_windowed(
                (~f.col("cardtr_country").isin("CZ", "CZE")).cast("integer"),
                "card_tr_location_abroad_count_{time_window}",
            ),
            tw.sum_windowed(
                f.when(
                    f.col("cardtr_country").isin("CZ", "CZE"),
                    f.col("cardtr_amount_czk"),
                ).otherwise(0),
                "card_tr_location_czech_volume_{time_window}",
            ),
            tw.sum_windowed(
                f.when(
                    ~f.col("cardtr_country").isin("CZ", "CZE"),
                    f.col("cardtr_amount_czk"),
                ).otherwise(0),
                "card_tr_location_abroad_volume_{time_window}",
            ),
        ]

    def flag_features(time_window: str) -> List[Column]:
        return [
            tw.column_windowed(
                (f.col(f"card_tr_location_abroad_count_{time_window}") > 0).cast(
                    "integer"
                ),
                f"card_tr_location_abroad_flag_{time_window}",
            )
        ]
  
    return wdf.time_windowed(country_agg_features, flag_features)
```


## apply_per_time_windowed {#apply_per_time_windowed}
__apply_per_time_window__(`self, function: Callable[[WindowedDataFrame, str], DataFrame]`) -> `WindowedDataFrame`

> Apply user defined function per time_windows

- `function: Callable[[WindowedDataFrame, str], WindowedDataFrame]`: Function which takes `WindowedDataFrame` and `time_window: str` and returns `WindowedDataFrame`

Example:

```python
@dp.transformation(frequencies, display=False)
def frequencies_structs(wdf: WindowedDataFrame):
    def make_structs(wdf: WindowedDataFrame, time_window: str):
        return wdf.withColumn(
            f"values_{time_window}",
            f.struct(
                tw.column_windowed(
                    f.col(f"transaction_city_count_{time_window}"),
                    f"card_tr_location_city_most_common_count_{time_window}",
                ),
                tw.column_windowed(
                    f.col(f"transaction_city_volume_{time_window}"),
                    f"card_tr_location_city_most_common_volume_{time_window}",
                )
            ),
        )

    return wdf.apply_per_time_window(make_structs)
```

## get_windowed_column_list {#get_windowed_column_list}
__get_windowed_column_list__(`self, column_names: List[str]`) -> `List[str]`

> Get windowed column names

- `column_names: List[str]`: List of column names with placeholder `{time_window}`, such as `["feature1_{time_window}", "feature2_{time_window}", "feature3_{time_window}"]`

---

## make_windowed {#with_time_windows}
__make_windowed__(`df: DataFrame, entity: Entity, time_column: str`) -> `WindowedDataFrame`

> Used for creating a WindowedDataFrame instance.


- `df` : DataFrame, input DataFrame
- `entity` : Entity, the feature store `Entity` instance, such as `client`
- `time_column` : str, name of `Date` or `Timestamp` Column in `df`, which is subtracted __from__ the `run_date` to create the time window interval

Note:
- `time_windows` are read from a widget called `time_windows`

Example:

```python
@dp.transformation(
    make_windowed(
        card_transactions,
        client_entity,
        "cardtr_process_date",
    ),
    display=False,
)
def card_transactions_with_time_windows(windowed_card_transactions: WindowedDataFrame):
    return windowed_card_transactions
```

---

## WindowedColumn {#WindowedColumn}
__WindowedColumn__

> Alias for type `Callable[[str], Column]`

---

## sum_windowed {#sum_windowed}
__sum_windowed__(`name: str, col: Column`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.sum`

- `name` : name of the column with a `{time_window}`
- `col` : PySpark `Column`

!!! info "`name` is an f-string"
    * **Do**: ```tw.sum_windowed(f"card_tr_location_czech_count_{time_window}", f.col("cardtr_country").isin("CZ", "CZE").cast("integer"))```
    * **Don't**: ```tw.sum_windowed("card_tr_location_czech_count_{time_window}", f.col("cardtr_country").isin("CZ", "CZE").cast("integer"))```

Example:

```python
tw.sum_windowed(
    "card_tr_location_czech_count_14d",
    f.col("cardtr_country").isin("CZ", "CZE").cast("integer"),
)
```

---

## count_windowed {#count_windowed}
__count_windowed__(`name: str, col: Column`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.count`

- `name` : name of the column
- `col` : PySpark `Column`

!!! info "`name` is an f-string"
    * **Do**: ```tw.count_windowed(f"card_tr_location_czech_count_{time_window}", f.col("cardtr_country").isin("CZ", "CZE").cast("integer"))```
    * **Don't**: ```tw.count_windowed("card_tr_location_czech_count_{time_window}", f.col("cardtr_country").isin("CZ", "CZE").cast("integer"))```

---

## count_distinct_windowed {#count_distinct_windowed}
__count_distinct_windowed__(`name: str, col: Column`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.countDistinct`

- `name` : name of the column
- `col` : PySpark `Column`

!!! info "`name` is an f-string"
    * **Do**: ```tw.count_distinct_windowed(f"card_tr_location_czech_count_{time_window}", f.col("cardtr_country").isin("CZ", "CZE").cast("integer"))```
    * **Don't**: ```tw.count_distinct_windowed("card_tr_location_czech_count_{time_window}", f.col("cardtr_country").isin("CZ", "CZE").cast("integer"))```

---

## min_windowed {#min_windowed}
__min_windowed__(`name: str, col: Column`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.min`

- `name` : name of the column
- `col` : PySpark `Column`

!!! info "`name` is an f-string"
    * **Do**: ```tw.min_windowed(f"card_tr_location_czech_min_{time_window}", f.col("cardtr_country").isin("CZ", "CZE").cast("integer"))```
    * **Don't**: ```tw.min_windowed("card_tr_location_czech_min_{time_window}", f.col("cardtr_country").isin("CZ", "CZE").cast("integer"))```

---

## max_windowed {#max_windowed}
__max_windowed__(`name: str, col: Column`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.max`

- `name` : name of the column
- `col` : PySpark `Column`

!!! info "`name` is an f-string"
    * **Do**: ```tw.max_windowed(f"card_tr_location_czech_max_{time_window}", f.col("cardtr_country").isin("CZ", "CZE").cast("integer"))```
    * **Don't**: ```tw.max_windowed("card_tr_location_czech_max_{time_window}", f.col("cardtr_country").isin("CZ", "CZE").cast("integer"))```

---

## mean_windowed {#mean_windowed}
__mean_windowed__(`name: str, col: Column`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.mean`

- `name` : name of the column
- `col` : PySpark `Column`

!!! info "`name` is an f-string"
    * **Do**: ```tw.mean_windowed(f"card_tr_location_czech_mean_{time_window}", f.col("cardtr_country").isin("CZ", "CZE").cast("integer"))```
    * **Don't**: ```tw.mean_windowed("card_tr_location_czech_mean_{time_window}", f.col("cardtr_country").isin("CZ", "CZE").cast("integer"))```

---

## column_windowed {#column_windowed}
__column_windowed__(`name: str, col: Column`) -> `Column`
> Alias for `col.alias(name)`

- `name` : name of the column
- `col` : PySpark `Column`

!!! info "`name` is an f-string"
    * **Do**: ```tw.column_windowed(f"card_tr_location_flag_{time_window}", f.col("card_tr_location_abroad_count_{time_window}") > 0)```
    * **Don't**: ```tw.column_windowed("card_tr_location_flag_{time_window}", f.col("card_tr_location_abroad_count_{time_window}") > 0)```

---

