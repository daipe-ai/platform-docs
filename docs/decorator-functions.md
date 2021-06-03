# Decorator functions

## read_csv
__read_csv__(`path: str, schema: StructType = None, options: dict = None`)

> Reads a CSV file into a spark DataFrame

Parameters:

- `path` : str - path to the CSV file
- `schema` : StructType, default None - schema of the CSV file
- `options` : dict, default None - options passed to `spark.read.options(**options)`

Example:

```python
@transformation(read_csv("/LoanData.csv", options=dict(header=True, inferSchema=True)), display=True)
@table_overwrite("bronze.tbl_loans")
def save(df: DataFrame):
    return df.orderBy("LoanDate")
```

---

## read_delta
__read_delta__(`path: str, schema: StructType = None, options: dict = None`)

> Reads a Delta from a path

Parameters:

- `path` : str - path to the Delta
- `schema` : StructType, default None - Union[str, list], default None - schema of the Delta
- `options` : dict, default None - options passed to `spark.read.options(**options)`

---

## read_json
__read_json__(`path: str, schema: StructType = None, options: dict = None`)

> Reads a json file from a path

Parameters:

- `path` : str - path to the json file
- `schema` : StructType, default None - Union[str, list], default None - schema of the json file
- `options` : dict, default None - options passed to `spark.read.options(**options)`

---

## read_parquet
__read_parquet__(`path: str, schema: StructType = None, options: dict = None`)

> Reads a parquet from a path

Parameters:

- `path` : str - path to the parquet
- `schema` : StructType, default None - Union[str, list], default None - schema of the parquet
- `options` : dict, default None - options passed to `spark.read.options(**options)`

---

## read_table
__read_table__(`identifier: str`)

> Reads a table into a spark DataFrame

Parameters:

- `identifier` : str - full table name, format `db.table_name`

Example:

```python
@transformation(read_table("silver.tbl_loans"))
def read_table_bronze_loans_tbl_loans(df: DataFrame, dbutils: DBUtils):
    base_year = dbutils.widgets.get("base_year")

    return df.filter(f.col("DefaultDate") >= base_year)
```

---

## table_params
__table_params__(`identifier: str, param_path_parts: list = None`)

> Reads parameters from _datalakebundle.tables.[`identifier`]_

Parameters:

- `identifier` : str - full table name, format `db.table_name`
- `param_path_parts` : list, default None - Union[str, list], default None - list of parameter levels leading to result
---
w