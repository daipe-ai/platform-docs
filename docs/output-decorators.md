# Output decorators

Output decorators are used to persist the output of the decorated function in multiple possible formats - table, delta, csv, json and parquet.

## @table_overwrite {#table_overwrite}
__@table_overwrite__(`identifier: str, table_schema: TableSchema = None, recreate_table: bool = False, options: dict = None`)

> Overwrites data in a table with a DataFrame returned by the decorated function 

Parameters:

- `identifier` : str - table name
- `table_schema` : TableSchema, default None - [TableSchema](#table_schema) object which defines fields, primary_key, partition_by and tbl_properties, if `None` the table is saved with the DataFrame schema
- `recreate_table` : bool, default False, if `True` the table is dropped if exists before written to
- `options` : dict, default None - options which are passed to `df.write.options(**options)`


---

## @table_append {#table_append}
__@table_append__(`identifier: str, table_schema: TableSchema = None, options: dict = None`)

> Appends data to a table with a DataFrame returned by the decorated function

Parameters:

- `identifier` : str - table name
- `table_schema` : TableSchema, default None - [TableSchema](#table_schema) object which defines fields, primary_key, partition_by and tbl_properties, if `None` the table is saved with the DataFrame schema
- `options` : dict, default None - options which are passed to `df.write.options(**options)`

---

## @table_upsert {#table_upsert}
__@table_upsert__(`identifier: str, table_schema: TableSchema`)

> Updates data or inserts new data to a table based on primary key with a DataFrame returned by the decorated function

Parameters:

- `identifier` : str - table name
- `table_schema` : TableSchema, default None - [TableSchema](#table_schema) object which defines fields, primary_key, partition_by and tbl_properties, if `None` the table is saved with the DataFrame schema

---

## @csv_append
__@csv_append__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Appends a spark DataFrame to a CSV file

Parameters:

- `path` : str - path to the CSV file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @csv_overwrite
__@csv_overwrite__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Overwrites a CSV file by a spark DataFrame

Parameters:

- `path` : str - path to the CSV file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @csv_write_ignore
__@csv_write_ignore__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a CSV file if it does not exist

Parameters:

- `path` : str - path to the CSV file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @csv_write_errorifexists
__@csv_write_errorifexists__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a CSV file, throws an Exception if it already exists

Parameters:

- `path` : str - path to the CSV file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @delta_append
__@delta_append__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Appends a spark DataFrame to a Delta

Parameters:

- `path` : str - path to the Delta
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @delta_overwrite
__@delta_overwrite__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Overwrites a Delta by a spark DataFrame

Parameters:

- `path` : str - path to the Delta
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @delta_write_ignore
__@delta_write_ignore__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a Delta if it does not exist

Parameters:

- `path` : str - path to the Delta
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @delta_write_errorifexists
__@delta_write_errorifexists__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a Delta, throws an Exception if it already exists

Parameters:

- `path` : str - path to the Delta
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @json_append
__@json_append__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Appends a spark DataFrame to a json file

Parameters:

- `path` : str - path to the json file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @json_overwrite
__@json_overwrite__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Overwrites a json file by a spark DataFrame

Parameters:

- `path` : str - path to the json file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @json_write_ignore
__@json_write_ignore__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a json file if it does not exist

Parameters:

- `path` : str - path to the json file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @json_write_errorifexists
__@json_write_errorifexists__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a json file, throws an Exception if it already exists

Parameters:

- `path` : str - path to the json file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @parquet_append
__@parquet_append__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Appends a spark DataFrame to a parquet file

Parameters:

- `path` : str - path to the parquet file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @parquet_overwrite
__@parquet_overwrite__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Overwrites a parquet file by a spark DataFrame

Parameters:

- `path` : str - path to the parquet file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @parquet_write_ignore
__@parquet_write_ignore__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a parquet file if it does not exist

Parameters:

- `path` : str - path to the parquet file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @parquet_write_errorifexists
__@parquet_write_errorifexists__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a parquet, throws an Exception if it already exists

Parameters:

- `path` : str - path to the parquet
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## Functions

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
