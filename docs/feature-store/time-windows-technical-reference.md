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
__time_windowed__(`self, agg_columns_function: Callable[[str], List[WindowedColumn]] = lambda x: list(), non_agg_columns_function: Callable[[str], List[Column]] = lambda x: list(), extra_group_keys: List[str] = [], unnest_structs: bool = False`) -> `WindowedDataFrame`

> Returns a new WindowedDataFrame with calculated aggregated and non aggregated columns

- `agg_columns_function: Callable[[str], List[WindowedColumn]] = lambda x: list()`: Function which takes `time_window: str` and returns `List[WindowedColumn]`
- `non_agg_columns_function: Callable[[str], List[Column]] = lambda x: list()`: Function which takes `time_window: str` and returns `List[Column]`
- `extra_group_keys: List[str] = []`: By default it groups by `entity.primary_key`, use `extra_group_keys` to add more columns to group by
- `unnest_structs: bool = False`: if `True`, all structs will be expanded into regular columns

Example:

```python
@dp.transformation(card_transactions)
def card_channel_country_features(wdf: WindowedDataFrame):
    def country_agg_features(time_window: str) -> List[tw.WindowedColumn]:
        return [
            tw.sum_windowed(
                f.col("cardtr_country").isin("CZ", "CZE").cast("integer"),
                f"card_tr_location_czech_count_{time_window}",
            ),
            tw.sum_windowed(
                (~f.col("cardtr_country").isin("CZ", "CZE")).cast("integer"),
                f"card_tr_location_abroad_count_{time_window}",
            ),
            tw.sum_windowed(
                f.when(
                    f.col("cardtr_country").isin("CZ", "CZE"),
                    f.col("cardtr_amount_czk"),
                ).otherwise(0),
                f"card_tr_location_czech_volume_{time_window}",
            ),
            tw.sum_windowed(
                f.when(
                    ~f.col("cardtr_country").isin("CZ", "CZE"),
                    f.col("cardtr_amount_czk"),
                ).otherwise(0),
                f"card_tr_location_abroad_volume_{time_window}",
            ),
        ]

    def flag_features(time_window: str) -> List[Column]:
        return [
            tw.column(
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
                tw.column(
                    f.col(f"transaction_city_count_{time_window}"),
                    f"card_tr_location_city_most_common_count_{time_window}",
                ),
                tw.column(
                    f.col(f"transaction_city_volume_{time_window}"),
                    f"card_tr_location_city_most_common_volume_{time_window}",
                )
            ),
        )

    return wdf.apply_per_time_window(make_structs)

```

---

## is_time_window {#is_time_window}
__is_time_window__(`self, time_window: str`) -> `Column`

> Returns a boolean `Column` to indicate if a row in the desired `time_window`

- `time_window: str`: time window as a `[0-9]+[dhw]`, suffixes `d` = days, `h` = hours, `w` = weeks 

---

## get_windowed_column_list {#get_windowed_column_list}
__get_windowed_column_list__(`self, column_names: List[str]`) -> `List[str]`

> Get windowed column names

- `column_names: List[str]`: List of column names with a `{time_window}` placeholder, such as `["feature1_{time_window}", "feature2_{time_window}", "feature3_{time_window}"]`

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

## column {#column}
__column__(`name: str, col: Column`) -> `Column`
> Alias for `col.alias(name)`

- `name` : name of the column
- `col` : PySpark `Column`

!!! warning
    * **No time window functionality**: `column` is intended to only be used in non aggregated columns function, it does __not__ handle time windows on its own.

---

## most_common {#most_common}
__most_common__(name: str, *columns: Column): -> `Column`
> Performs a most common element calculation based on the input `columns`

- `name` : name of the column
- `columns` : PySpark `Column`

Example:

This example code calculates __most common__ cities based on the number of transactions conducted in the city.

The order of the argument `columns` determines the outcome. The __most common__ columns is the maximum struct.
The maximum struct si determined the same way the `max` function works in Python.
```python
a = (10, -5, "Praha")
b = (11, -1000, "Zlín")
c = (10, -5, "Brno")

max(a, b, c)  # Result: (11, -1000, 'Zlín')
max(a, c)     # Result: (10, -5, "Praha")
```
When the number of transactions is 0 for __all__ cities, the result is `NULL`.

```python
@dp.transformation(city_amount, display=False)
def most_common_city(wdf: WindowedDataFrame):
    def most_common_features(time_window: str):
        return [
            tw.most_common(
                f"most_common_city_{time_window}",
                tw.column(
                    f"card_tr_location_city_most_common_count_{time_window}",
                    f.col(f"transaction_city_count_{time_window}")
                ),
                tw.column(
                    f"random_number_{time_window}",
                    f.hash(*wdf.primary_key)
                ),
                tw.column(
                    f"card_tr_location_city_most_common_{time_window}",
                    f.when(f.col(f"transaction_city_count_{time_window}") > 0, f.col("cardtr_transaction_city"))
                ),
            )
        ]

    return wdf.time_windowed(most_common_features, unnest_structs=True)
```

---

## sum_windowed {#sum_windowed}
__sum_windowed__(`name: str, col: Column`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.sum`

- `name` : name of the column with a `{time_window}`
- `col` : PySpark `Column`


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

---

## count_distinct_windowed {#count_distinct_windowed}
__count_distinct_windowed__(`name: str, cols: List[Column]`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.countDistinct`

- `name` : name of the column
- `cols` : PySpark `Column`

---

## min_windowed {#min_windowed}
__min_windowed__(`name: str, col: Column`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.min`

- `name` : name of the column
- `col` : PySpark `Column`

---

## max_windowed {#max_windowed}
__max_windowed__(`name: str, col: Column`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.max`

- `name` : name of the column
- `col` : PySpark `Column`

---

## avg_windowed {#avg_windowed}
__avg_windowed__(`name: str, col: Column`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.avg`

- `name` : name of the column
- `col` : PySpark `Column`
- 
---

## mean_windowed {#mean_windowed}
__mean_windowed__(`name: str, col: Column`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.mean`

- `name` : name of the column
- `col` : PySpark `Column`

---

## first_windowed {#first_windowed}
__first_windowed__(`name: str, col: Column`) -> `WindowedColumn`
> Returns and aggregated WindowedColumn for the PySpark function `f.first`

- `name` : name of the column
- `col` : PySpark `Column`

---
