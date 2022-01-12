# TableSchema

__dp.TableSchema__(`fields: list, primary_key: Union[str, list] = None, partition_by: Union[str, list] = None, tbl_properties: dict = None`)

> Defines a table schema

Parameters:

- `fields` : list - list of StructField defining columns of the table
- `primary_key` : Union[str, list], default  None - primary key or a list of keys used for `@dp.table_upsert`
- `partition_by` : Union[str, list], default None - one or multiple fields to partition the data by, __optional__
- `tbl_properties` : dict, default None - key value pairs to be added to `TBLPROPERTIES`, __optional__

Example:

```python
import daipe as dp

def get_schema():
    return dp.TableSchema(
        [
            t.StructField("ReportAsOfEOD", t.DateType(), True),
            t.StructField("LoanID", t.StringType(), True),
            t.StructField("Date", t.DateType(), True),
            t.StructField("PrincipalRepayment", t.DoubleType(), True),
            t.StructField("InterestRepayment", t.DoubleType(), True),
            t.StructField("LateFeesRepayment", t.DoubleType(), True),
        ],
        primary_key=["LoanID", "Date"],
        partition_by = "Date",
        tbl_properties = {
            "delta.enableChangeDataFeed" = "true",
        }
    )
```
