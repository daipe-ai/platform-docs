# Output decorators

Output decorators are used to persist the output of the decorated function in multiple possible formats - table, delta, csv, json and parquet.

## @dp.table_overwrite {#table_overwrite}
__@dp.table_overwrite__(`identifier: str, table_schema: dp.TableSchema = None, recreate_table: bool = False, options: dict = None`)

> Overwrites data in a table with a DataFrame returned by the decorated function 

Parameters:

- `identifier` : str - table name
- `table_schema` : dp.TableSchema, default None - [TableSchema](#table_schema) object which defines fields, primary_key, partition_by and tbl_properties, if `None` the table is saved with the DataFrame schema
- `recreate_table` : bool, default False, if `True` the table is dropped if exists before written to
- `options` : dict, default None - options which are passed to `df.write.options(**options)`


---

## @dp.table_append {#table_append}
__@dp.table_append__(`identifier: str, table_schema: dp.TableSchema = None, options: dict = None`)

> Appends data to a table with a DataFrame returned by the decorated function

Parameters:

- `identifier` : str - table name
- `table_schema` : dp.TableSchema, default None - [TableSchema](#table_schema) object which defines fields, primary_key, partition_by and tbl_properties, if `None` the table is saved with the DataFrame schema
- `options` : dict, default None - options which are passed to `df.write.options(**options)`

---

## @dp.table_upsert {#table_upsert}
__@dp.table_upsert__(`identifier: str, table_schema: dp.TableSchema`)

> Updates data or inserts new data to a table based on primary key with a DataFrame returned by the decorated function

Parameters:

- `identifier` : str - table name
- `table_schema` : dp.TableSchema, default None - [TableSchema](#table_schema) object which defines fields, primary_key, partition_by and tbl_properties, if `None` the table is saved with the DataFrame schema

---

## @dp.csv_append
__@dp.csv_append__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Appends a spark DataFrame to a CSV file

Parameters:

- `path` : str - path to the CSV file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @dp.csv_overwrite
__@dp.csv_overwrite__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Overwrites a CSV file by a spark DataFrame

Parameters:

- `path` : str - path to the CSV file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @dp.csv_write_ignore
__@dp.csv_write_ignore__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a CSV file if it does not exist

Parameters:

- `path` : str - path to the CSV file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @dp.csv_write_errorifexists
__@dp.csv_write_errorifexists__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a CSV file, throws an Exception if it already exists

Parameters:

- `path` : str - path to the CSV file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @dp.delta_append
__@dp.delta_append__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Appends a spark DataFrame to a Delta

Parameters:

- `path` : str - path to the Delta
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @dp.delta_overwrite
__@dp.delta_overwrite__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Overwrites a Delta by a spark DataFrame

Parameters:

- `path` : str - path to the Delta
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @dp.delta_write_ignore
__@dp.delta_write_ignore__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a Delta if it does not exist

Parameters:

- `path` : str - path to the Delta
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @dp.delta_write_errorifexists
__@dp.delta_write_errorifexists__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a Delta, throws an Exception if it already exists

Parameters:

- `path` : str - path to the Delta
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @dp.json_append
__@dp.json_append__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Appends a spark DataFrame to a json file

Parameters:

- `path` : str - path to the json file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @dp.json_overwrite
__@dp.json_overwrite__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Overwrites a json file by a spark DataFrame

Parameters:

- `path` : str - path to the json file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @dp.json_write_ignore
__@dp.json_write_ignore__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a json file if it does not exist

Parameters:

- `path` : str - path to the json file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @dp.json_write_errorifexists
__@dp.json_write_errorifexists__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a json file, throws an Exception if it already exists

Parameters:

- `path` : str - path to the json file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`

---

## @dp.parquet_append
__@dp.parquet_append__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Appends a spark DataFrame to a parquet file

Parameters:

- `path` : str - path to the parquet file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @dp.parquet_overwrite
__@dp.parquet_overwrite__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Overwrites a parquet file by a spark DataFrame

Parameters:

- `path` : str - path to the parquet file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @dp.parquet_write_ignore
__@dp.parquet_write_ignore__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a parquet file if it does not exist

Parameters:

- `path` : str - path to the parquet file
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`


---

## @dp.parquet_write_errorifexists
__@dp.parquet_write_errorifexists__(`path: str, partition_by: Union[str, list] = None, options: dict = None`)

> Saves a spark DataFrame to a parquet, throws an Exception if it already exists

Parameters:

- `path` : str - path to the parquet
- `partition_by` : Union[str, list], default None - Union[str, list], default None - one or multiple fields to partition the data by
- `options` : dict, default None - options passed to `df.write.options(**options)`
---
